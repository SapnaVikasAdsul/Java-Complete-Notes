# JPA — Cascading, Lazy/Eager Loading & N+1 Problem — Last-Minute Revision Notes

> Continuation of JPA Relationships notes. Covers: Cascading, Lazy vs Eager loading, N+1 problem, Entity Graph.

---

## 1. Cascading — "operate on parent → auto-operate on children"

**Problem without cascading:** to save a `Department` with its `Student`s, you must save each one manually:

```java
departmentRepository.save(dept);
studentRepository.save(s1);
studentRepository.save(s2);
studentRepository.save(s3);
studentRepository.save(s4);
```

Tedious and error-prone — forget one `save()` and data goes out of sync.

### The fix: `cascade` on the relationship

```java
@Entity
public class Department {
    @Id @GeneratedValue private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.PERSIST)
    private List<Student> students = new ArrayList<>();
}
```

Now:

```java
department.getStudents().addAll(List.of(s1, s2, s3, s4));
departmentRepository.save(department);   // ONE call saves department + all 4 students
```

Hibernate automatically runs: `INSERT department` → `INSERT s1` → `INSERT s2` → `INSERT s3` → `INSERT s4`.

### CascadeType options

| Type | Effect |
|---|---|
| `PERSIST` | saving parent → saves children |
| `REMOVE` | deleting parent → deletes children |
| `MERGE` | updating parent → updates children |
| `ALL` | applies every cascade type (persist + remove + merge + refresh + detach) |

**Rule of thumb:** use cascade when the child **cannot logically exist without the parent** (e.g. `OrderItem` without `Order`, `Student` without `Department` in a college-modeling sense). Don't cascade `REMOVE` for relationships where the child is independent (e.g. deleting a `User` shouldn't delete unrelated `Orders` that must be kept for records).

### Without `CascadeType.REMOVE` — the DB error you'll hit

Deleting a parent that still has children throws:

```
Cannot delete or update a parent row: a foreign key constraint fails
```

Because the child rows still reference the parent's FK. Fix: either cascade the remove, or manually delete children first.

```java
@OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
private List<Student> students = new ArrayList<>();
```

```java
@Transactional
public void removeDepartment(Long id) {
    Department dept = departmentRepository.findById(id);
    departmentRepository.removeDepartment(dept);   // cascades REMOVE to all students automatically
}
```

Generated delete sequence (visible in logs):
```sql
SELECT department...
SELECT student...           -- fetch children first
DELETE FROM student WHERE id = ...   -- once per child
DELETE FROM student WHERE id = ...
DELETE FROM department WHERE id = ...
```

---

## 2. Lazy vs Eager Loading ⭐ (very frequently asked)

### The problem: complete reachable object graph

A `Student` can hold a `Department`, which holds `Teachers`; a `Student` can hold `Course`s, each with an `Instructor`; a `Student` can hold a `Profile`, etc.

```
Student ──> Department ──> Teachers
   │
   ├──> Profile
   │
   └──> Course ──> Instructor
```

If Hibernate fetched **everything** reachable from one `Student` the moment you load it, one `find()` call could trigger dozens of joins/queries — very slow.

### The solution: FetchType

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "dept_id")
private Department department;

@OneToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "profile_id")
private Profile profile;
```

| FetchType | Behaviour |
|---|---|
| `EAGER` | related entity loaded **immediately**, in the same query (via JOIN) |
| `LAZY` | related entity loaded **only when accessed** (`getDepartment()` triggers a separate query at that point); until then Hibernate holds a **proxy** |

> Default fetch type: `@ManyToOne` / `@OneToOne` → `EAGER`; `@OneToMany` / `@ManyToMany` → `LAZY`. Always set it explicitly for clarity.

### Demonstration

```java
Student s1 = entityManager.find(Student.class, 1001L);
System.out.println("Fetched student");        // only id/name fetched so far

Department d1 = s1.getDepartment();            // query fired HERE (lazy)
System.out.println("Fetched department");

Profile p1 = s1.getProfile();                  // query fired HERE (lazy)
System.out.println("Fetched profile");
```

Log order with `LAZY`:
```
Fetched student
(SELECT department...)
Fetched department
(SELECT profile...)
Fetched profile
```

With `EAGER`, all three come back in **one query** (via LEFT JOIN) immediately when `find()` is called.

### Gotcha: serializing lazy entities to JSON (Jackson)

Returning an entity straight from a `@RestController` forces Jackson to touch every field — including lazy ones — which then get fetched anyway. Also watch for **infinite recursion** in bidirectional mappings:

```
Student → Department → List<Student> → Department → ... (StackOverflow)
```

**Fix:** break the cycle — don't map the back-reference in the response (e.g. remove `List<Student>` from `Department`, or use DTOs / `@JsonIgnore`).

### When to use which

- **LAZY** → default choice, especially for deep/complex graphs. Avoids unnecessary joins/queries.
- **EAGER** → only when you *know* you'll always need the related data immediately (small, always-used relations).

---

## 3. The N+1 Problem ⭐⭐ (classic interview topic)

### What happens

```java
List<Student> students = studentRepository.findAll();   // Query 1: SELECT * FROM student  (returns 100 rows)

for (Student s : students) {
    System.out.println(s.getDepartment().getName());    // lazy-loads department — 1 query PER student
}
```

- Query 1 → fetch 100 students (**1 query**)
- Loop → 100 separate `SELECT * FROM department WHERE id = ?` calls (**N queries**)
- **Total = 1 + N = 1 + 100 = 101 queries** for what should've been 1–2 queries.

```
1 query  → SELECT * FROM student
100 query → SELECT * FROM department WHERE id = ? (once per student)
------------------------------------------------
101 queries total  = the "N + 1 problem"
```

### Why it's dangerous
- Massive unnecessary load on the DB.
- Easy to write by accident — looks like normal, correct-looking code.
- Gets worse as data grows (100 students today, 100,000 tomorrow).

### First-level cache reduces it — but only if data is repetitive
The `PersistenceContext` (first-level cache) means: if many students **share the same department**, only the *first* lookup per unique department id hits the DB — subsequent students with that same department are served from cache.

```
100 students, but only 3 unique departments
→ becomes 1 + 3 = 4 queries instead of 1 + 100 = 101
```

But if every student belongs to a **different** department (worst case), the cache gives no benefit and you're back to `1 + N`.

### How to detect it
- App feels slow for no obvious reason.
- Check logs / SQL trace: one method call fires **way more queries than expected** (e.g. 1 call → 101 SQL statements).

### The fix: `@EntityGraph`

Instead of blindly making the relation `EAGER` everywhere (which slows down *every* use of the entity), use `@EntityGraph` to force eager fetching **only for specific query methods** that need it:

```java
public interface StudentRepository {

    @EntityGraph(attributePaths = "department")
    Student findById(Long id);          // this call: department fetched eagerly (JOIN)

    @EntityGraph(attributePaths = {"department", "profile"})
    List<Student> findAll();            // this call: both fetched eagerly, avoiding N+1
}
```

- The field stays declared as `LAZY` on the entity (good default everywhere else).
- `@EntityGraph` overrides it **per query method**, generating a single JOIN query instead of N+1 separate ones — used exactly where you know you'll loop through and access the relation.

---

## 4. Quick Interview Answers

**Q: What's the difference between `CascadeType.PERSIST` and `CascadeType.ALL`?**
`PERSIST` only propagates the save operation to children. `ALL` propagates every operation (persist, remove, merge, refresh, detach) — use it when the child's lifecycle is fully bound to the parent.

**Q: Why do you get "foreign key constraint fails" when deleting a parent?**
Child rows still reference the parent via FK, and there's no `CascadeType.REMOVE` (or `ALL`) to delete them first. Hibernate won't violate referential integrity.

**Q: What is lazy loading and why use it?**
Related entities are represented by a proxy and only queried when actually accessed, avoiding loading the entire deep object graph up front — saves computation and DB load, especially for complex/nested entities.

**Q: What causes the N+1 problem?**
Fetching a list (1 query), then lazily accessing a related entity inside a loop, causing one extra query **per item** in the list (N queries) → total N+1.

**Q: How do you fix N+1 without making everything EAGER?**
Use `@EntityGraph(attributePaths = "...")` on the specific repository method that needs the relation eagerly loaded — keeps the entity's default fetch type `LAZY` everywhere else, avoiding unnecessary joins in other code paths.

**Q: Does the first-level cache always solve N+1?**
No — it only helps when many rows share the same related entity (repeated FK values). If every row references a distinct related entity, the cache provides no benefit and N+1 still occurs in full.

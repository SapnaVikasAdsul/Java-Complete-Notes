# JPA / Hibernate Relationships — Last-Minute Revision Notes

---

## 1. The Core Problem

In **Java**, an object can hold another object:

```java
class Student {
    Long id;
    String name;
    Department department;   // a whole object
}
```

In **SQL**, a column cannot hold an object. So the DB stores only the **foreign key**:

```
STUDENT                              DEPARTMENT
+----+--------+-----------+          +---------+--------+
| id | name   | dept_id   |          | id (PK) | name   |
+----+--------+-----------+          +---------+--------+
| 1  | Aditya | 1001      |-------->| 1001    | CSC    |
| 2  | Rohit  | 1002      |-------->| 1002    | Physics|
+----+--------+-----------+          +---------+--------+
```

Without an ORM you'd write:

```sql
SELECT d.name FROM student s
JOIN department d ON s.dept_id = d.id
WHERE s.id = 1;
```

…and in code you'd need **two repository calls** (fetch student → read deptId → fetch department).

With JPA you just write:

```java
student.getDepartment().getName();
```

Hibernate figures out the join for you. **That is the whole point of relationship mapping.**

---

## 2. Lombok (boilerplate remover)

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

| Annotation | Generates |
|---|---|
| `@Getter` / `@Setter` | all getters / setters |
| `@NoArgsConstructor` | empty constructor (**JPA requires this**) |
| `@AllArgsConstructor` | constructor with all fields |
| `@Data` | getters + setters + `toString` + `equals`/`hashCode` |
| `@Builder` | builder pattern |

> For entities, prefer `@Getter @Setter @NoArgsConstructor @AllArgsConstructor` over `@Data`.

---

## 3. Two Things to Decide for Every Relationship

### A. Cardinality — *how many on each side?*

| Type | Example | Meaning |
|---|---|---|
| `@ManyToOne` | Student → Department | many students, one department |
| `@OneToMany` | Department → Students | one department, many students |
| `@OneToOne` | User ↔ Profile | one each side |
| `@ManyToMany` | Student ↔ Course | many each side |

> `@ManyToOne` and `@OneToMany` are the **same relationship viewed from opposite sides**.

### B. Direction — *unidirectional or bidirectional?*

```
Unidirectional:   Student ──────> Department      (only Student has the field)
Bidirectional:    Student <─────> Department      (both classes hold each other)
```

---

## 4. Owning Side vs Inverse Side  ⭐ (most asked)

| | Owning side | Inverse side |
|---|---|---|
| Holds | the **foreign key column** | nothing in DB |
| Annotation | `@JoinColumn` / `@JoinTable` | `mappedBy = "fieldNameOnOwningSide"` |
| Writes to DB | **yes** | ignored by Hibernate |

**Rule:** the side with `@ManyToOne` is *always* the owning side (the "many" side holds the FK).

**`mappedBy` value = the name of the field on the owning side**, not the table/column name.

---

## 5. @ManyToOne (Student → Department)

```java
@Entity
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne(optional = false)                 // Java-level: must not be null
    @JoinColumn(name = "dept_id", nullable = false)  // DB-level: NOT NULL column
    private Department department;
}
```

```java
@Entity
@Getter @Setter @NoArgsConstructor @AllArgsConstructor
public class Department {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

**`optional = false` vs `nullable = false`**

- `optional = false` → entity-level contract: every Student **must** have a Department.
- `nullable = false` → tells Hibernate to generate the column as `NOT NULL` in DDL.

Generated tables:

```
student(id PK, name, dept_id FK) ──many-to-one──> department(id PK, name)
```

---

## 6. @OneToMany (Department → Students) — bidirectional

```java
@Entity
public class Department {
    @Id @GeneratedValue private Long id;
    private String name;

    @OneToMany(mappedBy = "department")   // "department" = field name in Student
    private List<Student> students = new ArrayList<>();
}
```

> **Always initialise the collection** (`= new ArrayList<>()`), otherwise `department.getStudents().add(...)` throws `NullPointerException`.

**Key insight:** the table structure is **identical** whether you map many-to-one, one-to-many, or both. Only the `dept_id` FK exists. `@OneToMany` adds no column.

**Why can't Department hold the student id?**

```
department(id PK, name, student_id)
1001  CSC  Aditya
1001  CSC  Rohit     ❌ PK 1001 repeated — impossible
```

So the FK must live on the **many** side.

---

## 7. Service Layer — saving with a relationship

### Using an existing department

```java
@Service
public class StudentService {

    private final StudentRepository studentRepository;
    private final DepartmentRepository departmentRepository;

    @Transactional
    public void createStudent(Student student, Long deptId) {
        Department dept = departmentRepository.getDepartmentById(deptId);
        student.setDepartment(dept);            // owning side  ← REQUIRED
        dept.getStudents().add(student);        // inverse side ← for object consistency
        studentRepository.save(student);
    }
}
```

### Creating department + student together

```java
@Transactional
public void createStudent(Student student, String deptName) {
    Department dept = new Department();
    dept.setName(deptName);
    departmentRepository.save(dept);

    student.setDepartment(dept);
    dept.getStudents().add(student);
    studentRepository.save(student);
}
```

Hibernate runs: `INSERT department` → `INSERT student` automatically.

### Repository (plain EntityManager)

```java
@Repository
public class StudentRepository {
    @PersistenceContext
    private EntityManager entityManager;

    public void save(Student s)      { entityManager.persist(s); }
    public Student findById(Long id) { return entityManager.find(Student.class, id); }
}
```

> `@Transactional` sits on the **service**, not the repository.

---

## 8. Data Integrity Trap in Bidirectional Mapping ⭐

If you only set one side:

```java
student.setDepartment(cse);     // set
// forgot: cse.getStudents().add(student);
```

→ DB is fine (FK written), but **the in-memory object graph is wrong**. `cse.getStudents()` won't contain the student within the same transaction/session.

### Fix: helper methods on the entity

```java
public void addDepartment(Department department) {
    this.department = department;
    department.getStudents().add(this);
}

public void removeDepartment() {
    if (this.department != null) {
        this.department.getStudents().remove(this);
        this.department = null;
    }
}
```

Now call `student.addDepartment(dept)` instead of `setDepartment()` — both sides stay in sync automatically.

---

## 9. @OneToOne (User ↔ Profile)

Either side can own the FK. Here **Profile** owns it.

```java
@Entity
public class Profile {
    @Id @GeneratedValue private Long id;
    private String bio;

    @OneToOne
    @JoinColumn(name = "user_id", unique = true)   // unique ← critical
    private User user;
}
```

```java
@Entity
public class User {
    @Id @GeneratedValue private Long id;
    private String name;

    @OneToOne(mappedBy = "user")    // inverse side, optional
    private Profile profile;
}
```

```
user(id PK, name)  <──── profile(id PK, bio, user_id FK + UNIQUE)
```

**Why `unique = true` matters:**

```
profile
id    | bio    | user_id
1001  | ...    | 1        ✔
1002  | ...    | 1        ❌ same user has 2 profiles → becomes OneToMany
```

Two constraints are needed on `user_id`: **foreign key** + **unique key**. Without the unique constraint your "one-to-one" silently degrades into one-to-many.

---

## 10. @ManyToMany (Student ↔ Course)

Neither table can hold the other's FK — the PK would repeat on both sides. **A third (join) table is mandatory.**

```
student(id PK, name)        student_course              course(id PK, name)
                          +------------+-----------+
                          | student_id | course_id |   ← composite PK
                          +------------+-----------+
                          | 1          | 1001      |
                          | 1          | 1002      |
                          | 2          | 1001      |
                          | 2          | 1002      |
```

```java
@Entity
public class Student2 {
    @Id @GeneratedValue private Long id;
    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns        = @JoinColumn(name = "student_id"),  // this side
        inverseJoinColumns = @JoinColumn(name = "course_id")    // other side
    )
    private List<Course> courses = new ArrayList<>();
}
```

```java
@Entity
public class Course {
    @Id @GeneratedValue private Long id;
    private String name;

    @ManyToMany(mappedBy = "courses")   // "courses" = field name in Student2
    private List<Student2> students = new ArrayList<>();
}
```

Neither `student_id` nor `course_id` is a PK alone — **the combination is a composite PK**.

---

## 11. Summary Table

| Mapping | Extra column | New table | Owning side | Inverse side |
|---|---|---|---|---|
| `@ManyToOne` / `@OneToMany` | FK on **many** side | ❌ | the "many" side | `mappedBy` on "one" side |
| `@OneToOne` | FK on **either** side (+ UNIQUE) | ❌ | side with `@JoinColumn` | `mappedBy` on the other |
| `@ManyToMany` | none | ✅ join table | side with `@JoinTable` | `mappedBy` on the other |

---

## 12. Useful `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/studentdb
spring.datasource.username=root
spring.datasource.password=root

spring.jpa.hibernate.ddl-auto=update   # auto-create/alter tables
spring.jpa.show-sql=true               # print SQL
spring.jpa.properties.hibernate.format_sql=true
```

---

## 13. Quick Interview Answers

**Q: Difference between `@ManyToOne` and `@OneToMany`?**
Same relationship, opposite viewpoints. `@ManyToOne` is the owning side and holds the FK; `@OneToMany` with `mappedBy` is the inverse and creates no column.

**Q: What does `mappedBy` do?**
Marks the side as inverse (non-owning). Its value is the **field name on the owning entity**. Without it, Hibernate treats both sides as owning and creates an extra join table.

**Q: Why does `@ManyToMany` need a join table?**
Adding the other side's id to either table would duplicate the primary key. Only a separate table with a composite key `(student_id, course_id)` can represent it.

**Q: Unidirectional vs bidirectional — which is better?**
Bidirectional isn't automatically better. It lets you navigate both ways but forces you to keep both sides in sync (helper methods) and risks inconsistency. Use unidirectional if you only ever navigate one way.

**Q: Why initialise collections to `new ArrayList<>()`?**
Hibernate only populates them on load; on a newly created entity they are `null`, so `getStudents().add(...)` would NPE.

# Java Multithreading — Last-Minute Interview Revision Notes

---

## 1. Process vs Thread

| | Process | Thread |
|---|---|---|
| Definition | Independent program in execution | Smallest unit of execution *within* a process |
| Memory | Own **isolated** memory space | **Shares** memory of its parent process |
| Creation cost | Heavy | Lightweight |
| Communication | Inter-Process Communication (IPC) needed | Direct, via shared memory |

> A thread is often called a **"lightweight sub-process."**

```
Process
 ├── Thread 1  ┐
 ├── Thread 2  ├── all share the same heap / memory space
 └── Thread 3  ┘
```

---

## 2. Creating Threads in Java

### Way 1 — extend `Thread`

```java
class MyThread extends Thread {
    public void run() {
        // task
    }
}

MyThread t = new MyThread();
t.start();     // start() → creates a new call stack & invokes run() on it
```

### Way 2 — implement `Runnable`

```java
class MyRunnable implements Runnable {
    public void run() {
        // task
    }
}

MyRunnable r = new MyRunnable();
Thread t = new Thread(r);   // Runnable has no start(); wrap it in a Thread
t.start();
```

**Why wrap `Runnable` in a `Thread`?** `Runnable` only defines `run()` — plain business logic, no threading capability. `Thread` provides `start()`, which asks the JVM/OS to create a new call stack and schedule the thread; calling `run()` directly just runs it as a normal method on the current thread (no new thread!).

> **Prefer `Runnable` over extending `Thread`** — Java has single inheritance, so extending `Thread` uses up your one `extends`. Implementing `Runnable` keeps that slot free and also decouples the *task* from the *execution mechanism*.

---

## 3. Thread Lifecycle (States)

```
                 start()
      NEW ───────────────────> RUNNABLE <───────────────┐
                                   │  ▲                   │
                        scheduler  │  │ sleep done /       │
                          picks it │  │ I/O complete /      │
                                   ▼  │ lock acquired /      │
                                RUNNING│ notify()/notifyAll()│
                                   │   │                      │
     sleep() / wait() /            │   └──────────────────────┘
     blocked on I/O /              ▼
     waiting for lock  ──────>  NON-RUNNABLE (BLOCKED / WAITING)
                                   │
                     run() completes
                                   ▼
                              TERMINATED
```

| State | Meaning |
|---|---|
| **New** | Thread object created, `start()` not yet called |
| **Runnable** | `start()` called; eligible to run, waiting for CPU/scheduler |
| **Running** | Scheduler picked it; actually executing `run()` |
| **Non-Runnable (Blocked/Waiting)** | Paused due to `sleep()`, `wait()`, waiting on I/O, or waiting for a lock |
| **Terminated (Dead)** | `run()` method has finished executing |

**Moves Running → Non-Runnable:** `sleep()`, blocked on I/O, waiting for a lock, `wait()`
**Moves Non-Runnable → Runnable:** sleep time elapses, I/O completes, lock becomes available, `notify()` / `notifyAll()`

> ⚠️ `suspend()` / `resume()` are **deprecated** (unsafe, can cause deadlocks) — don't use them in real code, but they may still come up in interview trivia.

---

## 4. Common Thread Methods

```java
Thread t = Thread.currentThread();
```

| Method | Purpose |
|---|---|
| `getName()` / `setName(String)` | get/set thread name |
| `getId()` | unique thread id |
| `getPriority()` / `setPriority(int)` | thread priority (1–10) |
| `run()` | the task logic |
| `start()` | begins a new thread of execution |
| `sleep(ms)` | pause current thread (static method) |
| `join()` | wait for another thread to finish before continuing |
| `interrupt()` | request the thread to stop/interrupt its wait/sleep |

---

## 5. `synchronized` — Mutual Exclusion

Ensures only **one thread at a time** can execute a method/block on a given lock (monitor) — other threads must wait their turn.

```java
public synchronized void increment() {
    count++;
}

// or block-level, locking on a specific object
public void increment() {
    synchronized (this) {
        count++;
    }
}
```

Used to protect **critical sections** — shared mutable state accessed by multiple threads.

---

## 6. Inter-Thread Communication — `wait()` / `notify()` / `notifyAll()`

Used for **producer-consumer** style coordination. All three **must be called from inside a `synchronized` block**, on the object whose lock is held — otherwise `IllegalMonitorStateException`.

| Method | Behaviour |
|---|---|
| `wait()` | Called on an object; current thread **releases the lock** on that object and waits until another thread calls `notify()`/`notifyAll()` on the same object, or a timeout elapses |
| `notify()` | Wakes up **one** waiting thread on that object (unspecified which one); it then competes for the lock |
| `notifyAll()` | Wakes up **all** threads waiting on that object; useful when several threads are waiting and all should get a chance to re-check their condition |

```
Producer thread                 Consumer thread
      │                               │
      │  produces item                │  waits (wait())
      │  notify()/notifyAll() ───────>│  wakes up, re-acquires lock
      │                               │  consumes item
```

---

## 7. Returning a Result — `Callable` vs `Runnable`

`Runnable.run()` returns `void` — you **cannot** get a result back or throw a checked exception.

**`Callable<V>`** solves this: it can **return a value** and **throw a checked exception**.

```java
class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        Thread.sleep(1000);
        return "result from " + Thread.currentThread().getName();
    }
}
```

---

## 8. Executor Framework

### Why not create threads manually?
- Creating many raw `Thread` objects consumes lots of memory.
- OS-level thread scheduling & context switching overhead.
- Thread creation itself is expensive/time-consuming.

➡ Java introduced the **Executor Framework** — manages a **pool of reusable worker threads** instead of creating a new thread per task.

```
                      ┌───────────── Executor (Thread Pool) ─────────────┐
   task1 ──┐          │   Worker-1  Worker-2  Worker-3   ...             │
   task2 ──┤ submit   │      ▲          ▲          ▲                     │
   task3 ──┼────────> │      │          │          │                     │
   task4 ──┤          │  ┌───┴──────────┴──────────┴────┐                │
   task5 ──┘          │  │     Task Queue (waiting)      │                │
                       │  └───────────────────────────────┘                │
                       └────────────────────────────────────────────────────┘
```

- Tasks are handed to available worker threads.
- If all workers are busy, new tasks sit in the **queue**.
- If the queue is also full, new tasks are **rejected**.

### Hierarchy

```
Executor (interface)  → execute(Runnable)
     │
ExecutorService (interface) → execute(), submit(), shutdown(), shutdownNow()
     │
     ├── ThreadPoolExecutor
     ├── ScheduledThreadPoolExecutor
     └── ForkJoinPool
```

### Creating an executor (via `Executors` utility class)

```java
ExecutorService executor = Executors.newFixedThreadPool(2);
```

### `execute()` vs `submit()`

| | `execute(Runnable)` | `submit(...)` |
|---|---|---|
| Accepts | `Runnable` only | `Runnable` **or** `Callable` |
| Returns | `void` | `Future<?>` (Runnable) / `Future<V>` (Callable) |
| Exceptions | Lost — thread just dies silently, cannot be caught by caller | Captured inside the `Future`; surfaces when you call `.get()` (wrapped in `ExecutionException`) |

```java
executor.execute(() -> doWork());          // fire-and-forget, exceptions swallowed

Future<Integer> f1 = executor.submit(() -> 10);   // Callable, returns Future<Integer>
```

### `Future<V>` — handling asynchronous results

Since a submitted task might not finish immediately (it may wait in the pool's queue, run in a background thread), we can't just assign its result directly like `int i = 4;`. Java gives us `Future<V>` — a placeholder for a result that will be available *later*.

```java
Future<Integer> f1 = executor.submit(() -> 10);

Integer result = f1.get();   // BLOCKS the calling (main) thread until the task completes
```

- Task executes in a background worker thread.
- Result gets stored inside the `Future` object once done.
- `f1.get()` blocks the caller until that result is available.

### Shutting down an executor

| Method | Behaviour |
|---|---|
| `shutdown()` | Stops accepting **new** tasks; lets already-submitted/running tasks finish |
| `shutdownNow()` | Stops accepting new tasks **and** attempts to interrupt/stop currently running tasks |

### Types of Thread Pools (`Executors` factory methods)

| Pool | Config | Risk |
|---|---|---|
| `newFixedThreadPool(n)` | Fixed number of threads, **unbounded** queue | Queue can grow indefinitely → **memory issues** under heavy load |
| `newCachedThreadPool()` | No fixed limit, creates threads on demand, no queue | Can spawn too many threads → **system slowdown** under load |
| `newSingleThreadExecutor()` | 1 thread + queue | Tasks run **serially**, one at a time |
| `newScheduledThreadPool(n)` | Runs tasks after a delay / periodically | Use `.schedule()`, `.scheduleAtFixedRate()`, etc. |

---

## 9. Race Condition ⭐

Happens when multiple threads access/modify shared data **concurrently without synchronization**, and the final result depends on unpredictable thread interleaving (timing/scheduling).

### Example: `count++` is NOT atomic

`count++` is actually **3 steps**:
1. Read `count` from memory
2. Increment the value by 1
3. Write the updated value back to memory

If a context switch happens between these steps, two threads can interleave and **lose an update**:

```
Time   Thread-1              Thread-2
 1     read count = 0
 2                            read count = 0     <-- both read stale 0
 3     count = 0 + 1 = 1
 4                            count = 0 + 1 = 1
 5     write count = 1
 6                            write count = 1    <-- overwrites, increment LOST

Expected result: count = 2
Actual result:   count = 1   ❌ (race condition)
```

### Atomic vs Non-atomic

- **Atomic operation** — happens completely, as a single indivisible step; cannot be interrupted midway.
- **Non-atomic operation** — made up of multiple sub-steps (like `count++`), so it can be interrupted midway by a context switch, causing race conditions.

**Fixes:** `synchronized`, `Lock`/`ReentrantLock`, or `Atomic*` classes (`AtomicInteger`, etc.).

---

## 10. `volatile` Keyword

Solves the **visibility problem** (not the atomicity problem).

- Each thread may cache a variable's value in its own CPU cache/register for performance.
- Without `volatile`, one thread's update to a shared variable might not be **immediately visible** to other threads.
- Declaring a variable `volatile` ensures every read/write goes **directly to main memory**, so all threads always see the most up-to-date value.

```java
private volatile boolean running = true;
```

> ⚠️ **`volatile` does NOT make compound actions atomic.** `volatile int count; count++;` can still race — `volatile` only guarantees visibility of reads/writes, not exclusive access. For atomic increments use `synchronized` or `AtomicInteger`.

### `volatile` vs `synchronized`

| | `volatile` | `synchronized` |
|---|---|---|
| Solves | Visibility | Visibility **+** Atomicity (mutual exclusion) |
| Cost | Cheap | More expensive — locking/unlocking can flush cache, block threads |
| Use case | Simple flags (e.g. `running`, `isStopped`) | Compound operations / critical sections |

---

## 11. Deadlock

A **deadlock** occurs when two (or more) threads are each waiting for a lock the other holds — neither can proceed, and both wait forever.

```
Thread-1                         Thread-2
   holds Lock A                     holds Lock B
   wants Lock B  ───wait───>        wants Lock A  ───wait───>
        ▲                                  │
        └──────────────  circular wait  ───┘
```

**Classic example:**

```java
// Thread 1
synchronized (lockA) {
    synchronized (lockB) { ... }
}

// Thread 2
synchronized (lockB) {
    synchronized (lockA) { ... }   // deadlock if Thread 1 holds lockA and Thread 2 holds lockB simultaneously
}
```

### 4 necessary conditions for deadlock
1. **Mutual exclusion** — resource can't be shared
2. **Hold and wait** — thread holds one lock while waiting for another
3. **No preemption** — a lock can't be forcibly taken away
4. **Circular wait** — a cycle of threads each waiting on the next

### Prevention
- **Lock ordering** — always acquire multiple locks in the **same global order** across all threads.
- Use a **timeout** when trying to acquire a lock (`tryLock(timeout)` from `ReentrantLock`) instead of waiting forever.
- Avoid nested locks where possible; minimize the scope of `synchronized` blocks.
- Use higher-level concurrency utilities (`java.util.concurrent`) instead of manual lock juggling.

---

## 12. `wait()` vs `sleep()` vs `yield()` ⭐

| | `wait()` | `sleep()` | `yield()` |
|---|---|---|---|
| Belongs to | `Object` class | `Thread` class (static) | `Thread` class (static) |
| Lock behaviour | **Releases** the lock it holds | **Does NOT release** the lock | N/A — doesn't involve locks |
| Needs `synchronized`? | Yes — must be called with the lock held | No | No |
| Wakes up via | `notify()` / `notifyAll()` (or timeout) | Time elapses (or `interrupt()`) | Rescheduled by scheduler (no fixed time) |
| Purpose | Inter-thread communication | Pause execution for a fixed time | Hint to scheduler: "let other threads of same priority run" |

---

## 13. Quick Interview Answers

**Q: Why implement `Runnable` instead of extending `Thread`?**
Keeps your single inheritance slot free, decouples the task from the threading mechanism, and lets the same task be reused with `ExecutorService` — better practice than extending `Thread` directly.

**Q: Difference between `execute()` and `submit()`?**
`execute()` takes only a `Runnable` and returns nothing — exceptions are lost. `submit()` accepts `Runnable`/`Callable`, returns a `Future`, and lets you retrieve results or catch exceptions via `future.get()`.

**Q: What problem does `volatile` solve, and what doesn't it solve?**
It solves the **visibility** problem (ensures all threads see the latest value from main memory). It does **not** make compound operations like `count++` atomic — use `synchronized` or `AtomicInteger` for that.

**Q: What causes a race condition?**
Multiple threads reading/modifying shared, non-atomic state concurrently without synchronization, so the outcome depends on unpredictable thread interleaving/scheduling.

**Q: How do you prevent deadlocks?**
Consistent lock ordering across threads, using lock timeouts (`tryLock`), minimizing nested/held locks, and preferring `java.util.concurrent` utilities over manual synchronization.

**Q: `wait()` vs `sleep()`?**
`wait()` (from `Object`) releases the lock and requires being called inside a `synchronized` block; used for inter-thread communication. `sleep()` (from `Thread`) pauses the thread for a fixed time **without releasing any lock**.

# JAVA CORE — Chapter 10: Multithreading, Synchronization & Executor Framework

## 1. Why Multithreading? (Core idea, not memorization)
A **process** is a running program with its own memory. A **thread** is a lightweight sub-unit of a process — multiple threads share the same memory (heap) but each gets its own **Stack, PC Register**.

**Why it matters:** Doing two things "at once" (downloading a file while updating UI) needs threads. CPU with multiple cores can truly run threads in parallel; single core just switches fast (context switching) giving the *illusion* of parallelism.

## 2. Creating Threads — 2 Ways (Very frequently asked)

| Method | How | Limitation |
|---|---|---|
| **Extend `Thread` class** | Override `run()`, call `.start()` | Java has single inheritance — if your class extends Thread, it can't extend anything else |
| **Implement `Runnable`** | Implement `run()`, pass object to `new Thread(runnableObj)`, call `.start()` | Preferred — allows extending another class too, better OOP practice |

```java
class MyThread extends Thread {
    public void run() { System.out.println("Running via Thread"); }
}
class MyTask implements Runnable {
    public void run() { System.out.println("Running via Runnable"); }
}
// Usage:
new MyThread().start();
new Thread(new MyTask()).start();
```

**Capgemini trap #1:** Calling `run()` directly (`t.run()`) does **NOT** start a new thread — it just calls the method like a normal method call on the current thread. Only `.start()` creates a new thread of execution.

**Capgemini trap #2:** Calling `.start()` twice on the same Thread object throws `IllegalThreadStateException`. A thread, once started (and finished), cannot be restarted.

## 3. Thread Lifecycle (States) — frequently asked as diagram/sequence

```
NEW → RUNNABLE → RUNNING → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
```

| State | Meaning |
|---|---|
| **NEW** | Thread object created, `.start()` not yet called |
| **RUNNABLE** | Eligible to run; may be running or waiting for CPU turn |
| **BLOCKED** | Waiting to acquire a lock (`synchronized`) held by another thread |
| **WAITING** | Waiting indefinitely for another thread's signal (`wait()`, `join()` with no timeout) |
| **TIMED_WAITING** | Waiting for a specified time (`sleep(ms)`, `wait(ms)`, `join(ms)`) |
| **TERMINATED** | `run()` completed or thread died |

**Trap:** Java does NOT have a separate "RUNNING" enum constant in `Thread.State` — RUNNING is a sub-state of RUNNABLE from the OS's perspective. The actual `Thread.State` enum values are: `NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED`.

## 4. Key Thread Methods

| Method | Purpose |
|---|---|
| `start()` | Begins new thread, JVM calls `run()` internally |
| `run()` | Contains the task logic |
| `sleep(ms)` | Pauses current thread, does NOT release lock it holds |
| `join()` | Current thread waits until the called-on thread finishes |
| `yield()` | Hint to scheduler to let other threads of same priority run (not guaranteed) |
| `setPriority(int)` | 1 (MIN) to 10 (MAX), default 5 (NORM) — priority is only a *hint* to the scheduler |
| `interrupt()` | Signals a thread to stop waiting/sleeping (throws `InterruptedException` if sleeping) |

**Trap:** `sleep()` is a `static` method of `Thread` — it always pauses the **currently executing thread**, not the thread object it's called on. `t.sleep(1000)` still pauses whichever thread executes that line, not `t`.

## 5. Synchronization — WHY it exists

**Race condition:** When 2+ threads access/modify shared data concurrently without coordination, causing unpredictable results. Classic example: two threads incrementing a shared counter — final value can be wrong because `count++` is actually 3 steps (read, increment, write) and threads can interleave.

**Fix: `synchronized` keyword** — ensures only ONE thread can execute a synchronized block/method on a given object at a time (mutual exclusion using an intrinsic lock / "monitor").

```java
class Counter {
    private int count = 0;
    public synchronized void increment() { count++; }   // method-level lock
    public void increment2() {
        synchronized(this) { count++; }                 // block-level lock (same effect)
    }
}
```

| Type | Locks on |
|---|---|
| `synchronized` instance method | `this` object |
| `synchronized` static method | the `.class` object (Class-level lock, shared by ALL instances) |
| `synchronized(obj)` block | whatever object you specify |

**Capgemini trap:** A `static synchronized` method locks on the **Class object**, not each instance — so two threads calling the same static synchronized method on *different objects* still block each other. A non-static synchronized method locks per-**instance** — two threads calling it on *different objects* do NOT block each other.

## 6. wait() / notify() / notifyAll() — Inter-thread communication

- Must be called **inside a synchronized block**, else `IllegalMonitorStateException`.
- `wait()` — releases the lock and pauses until notified (unlike `sleep()`, which keeps the lock).
- `notify()` — wakes ONE waiting thread (arbitrary choice).
- `notifyAll()` — wakes ALL waiting threads; they then compete for the lock.

**Trap:** `sleep()` does NOT release the lock; `wait()` DOES release the lock. This is the #1 confusion point in interviews.

## 7. Deadlock

Occurs when two+ threads each hold a lock the other needs, and neither releases — both wait forever.
```
Thread A: locks Resource1, then wants Resource2
Thread B: locks Resource2, then wants Resource1  → DEADLOCK
```
**Fix approach (conceptual, for MCQs):** always acquire locks in the same fixed global order across all threads.

## 8. Executor Framework — WHY it replaces manual Thread management

Creating a new `Thread` per task is expensive (OS resources) and hard to manage at scale. **`ExecutorService`** (java.util.concurrent) manages a **pool of reusable threads**.

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> System.out.println("Task running"));
executor.shutdown();  // must call, else JVM may not exit
```

| Factory Method | Behavior |
|---|---|
| `newFixedThreadPool(n)` | Fixed number of reusable threads |
| `newCachedThreadPool()` | Creates threads as needed, reuses idle ones, unbounded — good for many short tasks |
| `newSingleThreadExecutor()` | Exactly 1 thread — tasks execute sequentially, one at a time |
| `newScheduledThreadPool(n)` | Runs tasks after a delay or periodically |

**`submit()` vs `execute()`:**
| | Accepts | Returns |
|---|---|---|
| `execute(Runnable)` | Runnable only | `void` — no way to get result/exception back |
| `submit(...)` | Runnable OR Callable | `Future<T>` — can call `.get()` to retrieve result (blocks until done) or catch exceptions |

**`Callable` vs `Runnable`:**
| | `run()` returns | Can throw checked exception? |
|---|---|---|
| `Runnable` | `void` | No |
| `Callable<V>` | `V` (a value) | Yes |

**Trap:** `Future.get()` **blocks** the calling thread until the task completes — if you call it right after `submit()` with no other work in between, you lose the benefit of async execution.

**Shutdown methods:**
- `shutdown()` — stops accepting new tasks, lets existing/queued tasks finish.
- `shutdownNow()` — attempts to stop all actively executing tasks immediately (best-effort, via interrupt), returns list of tasks that never started.

## One-Page Revision — Chapter 10
- Thread = lightweight unit of a process; shares heap, has own Stack + PC Register.
- Create via `extends Thread` (override `run()`) or `implements Runnable` (preferred, avoids single-inheritance limit).
- `start()` → new thread executes `run()` in parallel. Calling `run()` directly = normal method call, NO new thread.
- Thread states: NEW → RUNNABLE → (BLOCKED/WAITING/TIMED_WAITING) → TERMINATED.
- `sleep()` keeps lock, doesn't release it. `wait()` releases lock. `wait/notify/notifyAll` must be inside `synchronized`.
- `synchronized` instance method locks on `this`; `static synchronized` locks on the Class object (shared globally).
- Race condition = unsynchronized shared mutable state accessed by multiple threads → unpredictable result.
- Deadlock = circular lock-wait between 2+ threads. Fix: consistent lock ordering.
- ExecutorService = thread pool manager. `execute()` = fire-and-forget (Runnable, void). `submit()` = returns `Future` (works with Callable, can get result/exception).
- `Callable<V>` returns a value & can throw checked exceptions; `Runnable` cannot.
- `shutdown()` = graceful; `shutdownNow()` = forceful interrupt of running tasks.

---

# MCQs — Chapter 10 (22 Questions)

**Q1.** [Easy | Thread Basics] What is the correct method to start a new thread of execution?
A) `run()` B) `start()` C) `execute()` D) `begin()`
**Answer: B**
*Explanation:* `start()` creates a new call stack and invokes `run()` on a separate thread internally.
*Why others wrong:* A) Calling `run()` directly executes on the current thread, no new thread created. C) `execute()` is an ExecutorService method, not a Thread method. D) Not a real method.

**Q2.** [Easy | Thread Creation] Which interface is preferred over extending `Thread` for creating a thread task?
A) `Callable` B) `Runnable` C) `Executor` D) `Future`
**Answer: B**
*Explanation:* `Runnable` avoids Java's single-inheritance limitation and separates the task from the thread mechanism.
*Why others wrong:* A) Callable returns a value and is used with ExecutorService, not the basic Thread creation comparison. C) Executor is a framework interface, not a task definition. D) Future represents a pending result, not a task.

**Q3.** [Medium | Trap] What happens if you call `t.run()` instead of `t.start()`?
A) A new thread starts as usual B) It executes on the current thread like a normal method call, no new thread C) Compilation error D) `IllegalThreadStateException`
**Answer: B**
*Explanation:* `run()` is just a regular method; only `start()` triggers JVM's thread-creation machinery.
*Why others wrong:* A) That's what `start()` does, not `run()`. C) It compiles fine. D) That exception occurs when calling `start()` twice, not when calling `run()`.

**Q4.** [Medium | Lifecycle] Which of these is NOT a valid `Thread.State` enum constant?
A) NEW B) RUNNING C) RUNNABLE D) TERMINATED
**Answer: B**
*Explanation:* Java's `Thread.State` enum has NEW, RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, TERMINATED — "RUNNING" is not a separate enum value.
*Why others wrong:* A, C, D are all real `Thread.State` values.

**Q5.** [Medium | sleep vs wait] Which statement is TRUE?
A) `sleep()` releases the object's lock B) `wait()` releases the object's lock C) Both release the lock D) Neither releases the lock
**Answer: B**
*Explanation:* `wait()` releases the monitor lock so other threads can acquire it; `sleep()` merely pauses execution while retaining any lock held.
*Why others wrong:* A) Reverses the fact. C) sleep does not release. D) wait does release — so "neither" is false.

**Q6.** [Medium | Synchronization] A `static synchronized` method locks on:
A) The specific object instance calling it B) The Class object (shared across all instances) C) No lock is taken D) Each thread gets its own separate lock
**Answer: B**
*Explanation:* Static methods belong to the class, so the intrinsic lock used is the `Class` object itself, shared by every instance and thread.
*Why others wrong:* A) That's for instance synchronized methods. C) A lock is always taken for synchronized methods. D) Locks are shared, not per-thread.

**Q7.** [Hard | Scenario] Thread A calls a non-static `synchronized` method on object `obj1`. Thread B calls the SAME synchronized method on a DIFFERENT object `obj2`. Will B block waiting for A?
A) Yes, always B) No — different objects have different locks C) Only if priorities match D) Only if run on the same CPU core
**Answer: B**
*Explanation:* Instance-level synchronized methods lock on `this`; since `obj1` and `obj2` are different objects, they have independent locks, so no blocking occurs.
*Why others wrong:* A) Wrong — this only happens if it's the same object or a static method. C) Priority is irrelevant to locking. D) Core assignment is irrelevant to synchronized logic.

**Q8.** [Hard | wait/notify] Calling `wait()` outside a synchronized block/method causes:
A) Compilation error B) `IllegalMonitorStateException` at runtime C) Silent no-op D) Deadlock
**Answer: B**
*Explanation:* `wait()`, `notify()`, `notifyAll()` require the calling thread to hold the object's monitor (i.e., be inside a `synchronized` block on that object); otherwise the JVM throws `IllegalMonitorStateException`.
*Why others wrong:* A) It's a valid statement syntactically, so it compiles. C) It throws, doesn't silently pass. D) It's an immediate exception, not a hang.

**Q9.** [Medium | Executor] Which ExecutorService factory method creates exactly one thread that processes tasks sequentially?
A) `newFixedThreadPool(1)` B) `newSingleThreadExecutor()` C) `newCachedThreadPool()` D) Both A and B
**Answer: D**
*Explanation:* `newFixedThreadPool(1)` and `newSingleThreadExecutor()` both effectively run tasks one at a time on a single reusable thread (though `newSingleThreadExecutor` additionally guarantees the pool size can't be reconfigured later).
*Why others wrong:* A) Correct behavior but incomplete alone since B also qualifies. B) Correct but incomplete alone. C) Cached pool creates threads on demand and can run many tasks in parallel.

**Q10.** [Medium | Runnable vs Callable] Which interface allows a task to return a value and throw checked exceptions?
A) `Runnable` B) `Thread` C) `Callable` D) `Executor`
**Answer: C**
*Explanation:* `Callable<V>`'s `call()` method returns a value of type `V` and is declared to throw `Exception`.
*Why others wrong:* A) `run()` returns void and can't throw checked exceptions. B) Thread is a class managing execution, not a functional task type. D) Executor is the interface that just runs Runnables, no return value.

**Q11.** [Medium | Executor] What does `Future.get()` do if the task hasn't completed yet?
A) Returns `null` immediately B) Throws an exception immediately C) Blocks the calling thread until the task completes D) Cancels the task
**Answer: C**
*Explanation:* `get()` is a blocking call — the calling thread waits until the associated task finishes and the result is available (or a timeout overload is used).
*Why others wrong:* A) It doesn't return null, it waits. B) No exception unless task itself threw one or was cancelled. D) get() doesn't cancel anything; `cancel()` does.

**Q12.** [Hard | Deadlock] Deadlock occurs when:
A) A single thread waits for itself B) Two or more threads each hold a lock the other needs, and neither releases C) A thread finishes execution normally D) `sleep()` is called too many times
**Answer: B**
*Explanation:* Classic circular-wait: Thread A holds Lock1 and wants Lock2, Thread B holds Lock2 and wants Lock1 — both wait forever.
*Why others wrong:* A) Not the standard definition. C) Normal completion isn't deadlock. D) sleep() alone doesn't cause locking issues.

**Q13.** [Easy | Priority] Default thread priority in Java is:
A) 1 B) 5 C) 10 D) 0
**Answer: B**
*Explanation:* `Thread.NORM_PRIORITY` = 5 is the default; range is 1 (MIN) to 10 (MAX).
*Why others wrong:* A) That's MIN_PRIORITY. C) That's MAX_PRIORITY. D) 0 is not a valid priority value.

**Q14.** [Medium | execute vs submit] What's the key advantage of `submit()` over `execute()`?
A) `submit()` runs tasks faster B) `submit()` returns a `Future` so you can retrieve results/exceptions C) `execute()` cannot accept Runnable D) There is no difference
**Answer: B**
*Explanation:* `submit()` wraps the task and returns a `Future<T>`, letting you call `.get()` for the result or catch exceptions thrown inside the task; `execute()` returns void and swallows/propagates exceptions differently.
*Why others wrong:* A) No inherent speed difference. C) execute() accepts Runnable fine — that's its main use. D) There is a real difference in return type and exception handling.

**Q15.** [Hard | Trap] What is the output risk in this code without synchronization?
```java
class Counter {
    int count = 0;
    void increment() { count++; }
}
// Two threads call increment() 1000 times each on the same Counter object
```
A) Always exactly 2000 B) Possibly less than 2000 due to race condition C) Compilation error D) Always exactly 1000
**Answer: B**
*Explanation:* `count++` is read-modify-write (3 steps); without synchronization, threads can interleave and overwrite each other's updates, losing increments.
*Why others wrong:* A) Not guaranteed due to race conditions. C) Compiles fine, it's a runtime issue. D) 1000 assumes only one thread ran, which is false.

**Q16.** [Medium | join()] What does `t.join()` do when called from the main thread?
A) Terminates thread `t` immediately B) Main thread waits until thread `t` finishes execution C) Thread `t` waits for main thread D) Starts thread `t`
**Answer: B**
*Explanation:* `join()` blocks the calling thread (here, main) until the target thread `t` completes, enabling ordered completion.
*Why others wrong:* A) join() doesn't kill threads. C) It's the reverse — the caller waits, not `t`. D) `start()` starts a thread, not `join()`.

**Q17.** [Hard | Scenario] Which shutdown method attempts to interrupt actively running tasks and returns the list of tasks that never started?
A) `shutdown()` B) `shutdownNow()` C) `terminate()` D) `close()`
**Answer: B**
*Explanation:* `shutdownNow()` makes a best-effort attempt to stop actively executing tasks via interrupt and returns a list of the pending (never-started) tasks.
*Why others wrong:* A) `shutdown()` is graceful — lets running and queued tasks finish, just stops accepting new ones. C, D) Not real ExecutorService shutdown methods.

**Q18.** [Medium | Trap] Calling `.start()` twice on the same Thread object results in:
A) The thread runs twice B) `IllegalThreadStateException` C) `IllegalMonitorStateException` D) It silently does nothing
**Answer: B**
*Explanation:* A Thread object can only be started once in its lifetime; a second `start()` call on an already-started (or terminated) thread throws `IllegalThreadStateException`.
*Why others wrong:* A) It won't run twice, it throws instead. C) That's a wait/notify related exception, not relevant here. D) It doesn't fail silently — an exception is thrown.

**Q19.** [Hard | notify vs notifyAll] What's the difference between `notify()` and `notifyAll()`?
A) No difference B) `notify()` wakes one arbitrary waiting thread; `notifyAll()` wakes all waiting threads C) `notifyAll()` wakes only the highest priority thread D) `notify()` releases the lock immediately, `notifyAll()` does not
**Answer: B**
*Explanation:* `notify()` picks one (JVM-arbitrary) thread from the wait set to wake up; `notifyAll()` wakes every thread waiting on that object's monitor, and they then compete for the lock.
*Why others wrong:* A) There is a clear difference. C) Priority doesn't determine which thread `notify()` picks. D) Neither directly releases the lock — the lock is released only when the `synchronized` block/method exits.

**Q20.** [Easy | Cached Pool] `Executors.newCachedThreadPool()` is best suited for:
A) A single long-running task B) Many short-lived, bursty tasks C) Scheduled periodic tasks D) Tasks that must run sequentially only
**Answer: B**
*Explanation:* Cached pools create new threads as needed and reuse idle ones after a task finishes (idle threads are terminated after 60s by default) — ideal for many short-lived concurrent tasks.
*Why others wrong:* A) Single long-running task doesn't benefit from pooling/reuse. C) That's `newScheduledThreadPool`'s job. D) Sequential-only maps to `newSingleThreadExecutor`.

**Q21.** [Medium | Terminology] A "race condition" is best defined as:
A) Two threads racing to finish first, with no downside B) Unpredictable program behavior due to unsynchronized concurrent access to shared mutable state C) A CPU scheduling algorithm D) A type of deadlock
**Answer: B**
*Explanation:* When timing/interleaving of thread execution affects the correctness of the result, that's a race condition — it's a data-correctness bug, not just "who finishes first."
*Why others wrong:* A) Understates the real risk (data corruption). C) Not a scheduling algorithm. D) Deadlock is a distinct concept (mutual blocking), not a race condition.

**Q22.** [Hard | Code Trace] What is guaranteed about this code's output?
```java
Thread t1 = new Thread(() -> System.out.println("A"));
Thread t2 = new Thread(() -> System.out.println("B"));
t1.start();
t2.start();
```
A) "A" always prints before "B" B) "B" always prints before "A" C) Order between A and B is NOT guaranteed D) Compilation error
**Answer: C**
*Explanation:* Once both threads are started, the OS scheduler decides execution order; without `join()` or synchronization forcing an order, either "A" or "B" could print first (or interleave with other output).
*Why others wrong:* A, B) Neither order is guaranteed by the JVM/language spec. D) The code is valid and compiles.

---

## Chapter 10 Complete ✅
Score yourself: if you missed Q6, Q7, Q8, Q19, or Q22 — reread Synchronization (§5) and wait/notify (§6), since Capgemini heavily tests lock-object confusion.

**Next up: Chapter 11 — Garbage Collection & Memory Management.**

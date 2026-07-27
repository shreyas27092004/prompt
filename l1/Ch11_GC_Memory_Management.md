# JAVA CORE — Chapter 11: Garbage Collection & Memory Management

## 1. Why GC exists (Core idea)
In C/C++, the programmer manually frees memory (`free()`/`delete`) — forget it, and you get a **memory leak**; do it twice, and you get a crash. Java's **Garbage Collector (GC)** automatically finds objects that are no longer reachable/used and reclaims their heap memory — the programmer never manually deletes objects.

**Key idea:** GC only manages **Heap** memory (objects). It never touches **Stack** memory (local variables/frames) — those are automatically popped when a method returns, no GC involved.

## 2. What makes an object "eligible for GC"?

An object becomes eligible for garbage collection when it has **NO reachable references** pointing to it from any live thread's stack, static fields, or other reachable objects.

| Scenario | Eligible for GC? |
|---|---|
| `obj = null;` (only reference nulled) | Yes |
| Reference reassigned to a new object | Yes (old object, if nothing else points to it) |
| Object created inside a method, method returns | Yes (local reference goes out of scope) |
| Object referenced by a `static` field | **No** — static fields live for the class's lifetime |
| Two objects reference only each other (island of isolation), nothing else points to either | Yes — modern GC (reachability-based, not just reference-counting) detects and collects both |

**Capgemini trap:** "If two objects only reference each other, but nothing else references them, are they eligible for GC?" → **YES**. Java's GC uses **reachability from GC roots** (not simple reference counting like some older systems), so a mutually-referencing "island" with no external reference is still collected.

## 3. GC Roots (starting points of reachability)
- Local variables on any thread's active stack
- Active thread objects themselves
- Static variables (class-level)
- JNI references

Anything traceable/reachable starting from a GC root is "alive"; everything else is garbage.

## 4. `finalize()` method (legacy, but still asked)
- Called by GC (maybe) just before reclaiming an object's memory — **not guaranteed to run**, and **not guaranteed WHEN** it runs.
- Deprecated since Java 9 in favor of `try-with-resources` / `Cleaner` API — but Capgemini MCQs still test the old behavior.

**Trap:** `finalize()` does NOT guarantee immediate cleanup, and calling `System.gc()` is only a **request/hint** to the JVM to run GC — it is NOT guaranteed to actually run GC at that moment.

## 5. Heap Structure — Generational GC (very frequently asked)

Java Heap is divided based on the **"most objects die young"** observation:

```
HEAP
├── Young Generation
│   ├── Eden Space        ← new objects created here
│   ├── Survivor Space S0
│   └── Survivor Space S1
└── Old Generation (Tenured)  ← long-lived objects promoted here
```

- **Minor GC** — cleans the Young Generation (fast, frequent). Surviving objects move Eden → S0/S1, and after surviving several minor GC cycles, get **promoted** to Old Generation.
- **Major/Full GC** — cleans the Old Generation (and often the whole heap) — slower, less frequent, causes more noticeable pause ("stop-the-world").

| | Young Gen | Old Gen |
|---|---|---|
| Contains | New, short-lived objects | Long-lived, promoted objects |
| GC type | Minor GC (fast) | Major/Full GC (slow) |
| Frequency | High | Low |

**Metaspace (Java 8+):** Stores class metadata (replaced "PermGen" from Java 7 and earlier). Grows in **native memory**, not the JVM heap, and by default is NOT a fixed size (unlike old PermGen which often threw `OutOfMemoryError: PermGen space`).

**Trap:** PermGen was removed starting Java 8 and replaced by Metaspace. "PermGen space" `OutOfMemoryError` is a Java 7-and-earlier concept for MCQ purposes; Java 8+ uses Metaspace instead.

## 6. GC Algorithms (conceptual level — enough for MCQs)

| Algorithm | Idea |
|---|---|
| **Mark and Sweep** | Mark: traverse from GC roots, marking reachable objects. Sweep: reclaim memory of unmarked (unreachable) objects. |
| **Mark-Sweep-Compact** | Same as above + compacts (moves) surviving objects together to remove fragmentation |
| **Copying Collector** | Used in Young Gen — copies live objects from Eden to a Survivor space, then clears Eden entirely |
| **G1 GC (Garbage First)** | Default since Java 9 — divides heap into regions, collects regions with most garbage first, aims for predictable pause times |

## 7. Stack vs Heap — Quick Comparison (recap, heavily tested combined with Ch1)

| | Stack | Heap |
|---|---|---|
| Stores | Method frames, local variables, references | Objects, instance variables |
| Scope | Per-thread | Shared across all threads |
| Managed by | Automatic (pop on method return) | Garbage Collector |
| Speed | Faster | Slower (GC overhead) |
| Error on exhaustion | `StackOverflowError` | `OutOfMemoryError` (Heap space) |

**Trap:** `StackOverflowError` (e.g., from infinite recursion) is a completely DIFFERENT error from `OutOfMemoryError: Java heap space` — Stack exhaustion vs Heap exhaustion. Both extend `Error`, not `Exception` — they are NOT meant to be caught/handled in normal application logic (though technically catchable since `Error extends Throwable`).

## 8. Memory Leaks in Java — Can they still happen?
Yes! Even with GC, memory leaks happen when references are unintentionally kept alive:
- Objects stored in a `static` collection (e.g., `static List`) that's never cleared.
- Unclosed resources (streams, connections) holding references.
- Listener/callback objects registered but never deregistered.

**Trap:** "Java is garbage collected, so memory leaks are impossible" → **FALSE**. GC only reclaims *unreachable* objects; if a reference is unintentionally kept (e.g., in a static list), the object stays reachable and "leaks" logically even though GC is running fine.

## One-Page Revision — Chapter 11
- GC automatically reclaims heap memory of unreachable objects; never touches Stack (auto-freed on method return).
- Eligibility = no reachable reference from any **GC Root** (stack locals, static fields, active threads, JNI refs).
- Mutually-referencing objects with no external reference ARE still eligible (reachability-based, not reference-counting).
- `finalize()` — not guaranteed to run or run promptly; deprecated since Java 9.
- `System.gc()` is only a *request/hint*, not a guarantee GC runs immediately.
- Heap = Young Gen (Eden + S0 + S1, Minor GC, frequent) + Old Gen (Tenured, Major/Full GC, less frequent, slower).
- Metaspace (Java 8+) replaced PermGen; stores class metadata in native memory.
- GC algorithms: Mark-Sweep, Mark-Sweep-Compact, Copying (Young Gen), G1 (default since Java 9, region-based).
- `StackOverflowError` (stack exhaustion, e.g. infinite recursion) ≠ `OutOfMemoryError` (heap exhaustion) — different errors, different causes.
- Memory leaks ARE possible in Java despite GC — via lingering references (static collections, unclosed resources, un-deregistered listeners).

---

# MCQs — Chapter 11 (20 Questions)

**Q1.** [Easy | GC Basics] Garbage Collection in Java primarily reclaims memory from which area?
A) Stack B) Heap C) Method Area only D) PC Register
**Answer: B**
*Explanation:* Objects live on the Heap; GC identifies and reclaims heap memory of objects with no reachable references.
*Why others wrong:* A) Stack memory is freed automatically when a method returns, no GC involved. C) Method Area/Metaspace is collected far less often and isn't the primary GC target. D) PC Register isn't heap-managed memory.

**Q2.** [Easy | Eligibility] Setting an object reference to `null` makes the object:
A) Immediately destroyed B) Eligible for garbage collection (if no other references exist) C) Moved to Old Generation instantly D) Converted to a static object
**Answer: B**
*Explanation:* Nulling the only reference removes reachability; GC will collect it at some future, unspecified time — not instantly.
*Why others wrong:* A) Destruction isn't immediate or guaranteed at that exact moment. C) Nulling doesn't promote objects. D) Nulling has nothing to do with static status.

**Q3.** [Medium | Trap] Two objects A and B reference only each other, and nothing else in the program references either. Are they eligible for GC?
A) No — they still have references pointing to them B) Yes — reachability-based GC detects and collects such "islands" C) Only one of them is eligible D) They cause a guaranteed memory leak
**Answer: B**
*Explanation:* Java's GC works via reachability from GC Roots, not simple reference counting, so a mutually-referencing pair unreachable from any root is collected.
*Why others wrong:* A) Reference counting logic doesn't apply to Java's GC design. C) Both are equally unreachable, so both qualify. D) This is exactly the case GC is designed to handle, not a leak.

**Q4.** [Medium | finalize()] Which statement about `finalize()` is TRUE?
A) It is guaranteed to run exactly once before GC B) It is guaranteed to run immediately when `System.gc()` is called C) It is not guaranteed to run at all, and its timing is unpredictable D) It replaces the need for constructors
**Answer: C**
*Explanation:* `finalize()` execution (if it happens) is entirely at the JVM's discretion — no guarantee of running or of timing; it's deprecated since Java 9.
*Why others wrong:* A) Not guaranteed at all, contradicting "guaranteed... exactly once". B) `System.gc()` is only a request/hint. D) Unrelated to constructors' purpose.

**Q5.** [Medium | System.gc()] Calling `System.gc()` in code:
A) Forces immediate garbage collection B) Is merely a request/suggestion to the JVM; GC may or may not run immediately C) Throws a compile-time error D) Deletes all objects in the Heap
**Answer: B**
*Explanation:* `System.gc()` is a hint; the JVM's GC scheduler decides whether and when to actually run a collection cycle.
*Why others wrong:* A) Not guaranteed/forced. C) Valid, compilable code. D) It doesn't wipe the entire heap, only reclaims unreachable objects if/when it runs.

**Q6.** [Medium | Heap Structure] New objects are initially created in which part of the Heap?
A) Old Generation B) Eden Space (Young Generation) C) Metaspace D) Survivor Space S1 directly
**Answer: B**
*Explanation:* All new objects are allocated in Eden Space first; only after surviving Minor GC cycles do they move to Survivor spaces and eventually Old Generation.
*Why others wrong:* A) Old Gen holds only long-lived, promoted objects. C) Metaspace holds class metadata, not regular objects. D) Objects go to Eden first, not directly to a Survivor space.

**Q7.** [Medium | Minor vs Major GC] Minor GC operates on:
A) Old Generation only B) Young Generation (Eden + Survivor spaces) C) Metaspace only D) The entire heap always
**Answer: B**
*Explanation:* Minor GC specifically cleans the Young Generation, which is fast and frequent since most objects die young.
*Why others wrong:* A) That's the domain of Major/Full GC. C) Metaspace is a separate memory area, not cleaned by Minor GC. D) Minor GC targets Young Gen specifically, not the whole heap (that's Full GC).

**Q8.** [Hard | Metaspace] Since Java 8, class metadata is stored in:
A) PermGen (a fixed part of the Heap) B) Metaspace (native memory, grows dynamically by default) C) Young Generation D) Stack
**Answer: B**
*Explanation:* Java 8 replaced PermGen with Metaspace, which lives in native (off-heap) memory and by default isn't capped at a small fixed size, reducing the classic `OutOfMemoryError: PermGen space` issue.
*Why others wrong:* A) PermGen is the pre-Java-8 mechanism, now removed. C) Young Gen holds regular objects, not class metadata. D) Stack holds method frames/locals, unrelated to class metadata.

**Q9.** [Hard | Trap] Which JVM error/exception is thrown due to Heap exhaustion specifically (not stack)?
A) `StackOverflowError` B) `OutOfMemoryError: Java heap space` C) `IllegalStateException` D) `ClassNotFoundException`
**Answer: B**
*Explanation:* When the Heap cannot allocate a new object because it's full and GC can't free enough space, the JVM throws `OutOfMemoryError: Java heap space`.
*Why others wrong:* A) That's a Stack-specific error (e.g. from infinite recursion), unrelated to Heap. C, D) Unrelated runtime exceptions, not memory-exhaustion errors.

**Q10.** [Medium | Stack vs Heap] Infinite recursion (a method calling itself with no base case) typically causes:
A) `OutOfMemoryError: Java heap space` B) `StackOverflowError` C) Infinite loop with no error D) Compilation error
**Answer: B**
*Explanation:* Each recursive call adds a new frame to the thread's Stack; unbounded recursion exhausts the Stack, throwing `StackOverflowError`.
*Why others wrong:* A) That's heap-related, but recursion frames live on the stack, not heap. C) It does terminate — with an error, not run forever. D) It's a runtime error, not caught at compile time.

**Q11.** [Easy | Errors] `StackOverflowError` and `OutOfMemoryError` both directly extend:
A) `Exception` B) `RuntimeException` C) `Error` D) `Throwable` directly, bypassing Error
**Answer: C**
*Explanation:* Both are subclasses of `java.lang.Error`, representing serious problems that applications generally shouldn't try to catch/recover from.
*Why others wrong:* A, B) They are NOT Exceptions — Error and Exception are sibling classes under Throwable. D) They extend Error, which itself extends Throwable — they don't skip Error.

**Q12.** [Medium | Trap] "Java is garbage collected, therefore memory leaks are impossible." True or False?
A) True B) False
**Answer: B**
*Explanation:* Memory leaks can still occur if references are unintentionally kept alive (e.g., in a never-cleared static collection), keeping objects reachable and thus never collected by GC.
*Why others wrong:* A is the Capgemini trap — people assume GC = zero leaks, but "logical" leaks via lingering references are a real, common issue.

**Q13.** [Hard | GC Algorithm] Which GC algorithm is the DEFAULT collector since Java 9?
A) Serial GC B) Parallel GC C) G1 GC (Garbage First) D) CMS (Concurrent Mark Sweep)
**Answer: C**
*Explanation:* G1 GC became the default garbage collector starting Java 9, designed for large heaps with predictable, low pause times using region-based collection.
*Why others wrong:* A, B) Older/alternative collectors, not the modern default. D) CMS was deprecated and later removed; it was never the Java 9+ default.

**Q14.** [Medium | Mark and Sweep] The "Mark" phase of Mark-and-Sweep GC does what?
A) Deletes unreachable objects B) Traverses from GC roots and marks all reachable (live) objects C) Compacts memory to remove fragmentation D) Allocates new objects in Eden
**Answer: B**
*Explanation:* Mark phase identifies which objects are still reachable/alive by traversing references starting from GC roots; Sweep phase then reclaims memory of everything left unmarked.
*Why others wrong:* A) Deletion happens in the Sweep phase, not Mark. C) Compaction is a separate optional step (Mark-Sweep-Compact). D) Allocation is unrelated to the GC marking process.

**Q15.** [Hard | GC Roots] Which of these is NOT typically considered a GC Root?
A) Local variables on an active thread's stack B) Static variables C) A regular instance field inside a heap object with no external reference D) JNI references
**Answer: C**
*Explanation:* GC Roots are the fixed starting points (stack locals, statics, active threads, JNI refs) from which reachability is traced; a plain instance field on an otherwise unreachable heap object is not itself a root — it's just part of the object graph being traced.
*Why others wrong:* A, B, D) All are standard, textbook GC Root categories.

**Q16.** [Medium | Object Promotion] An object survives multiple Minor GC cycles in the Young Generation. What eventually happens to it?
A) It is deleted automatically after a fixed number of cycles regardless of use B) It gets promoted ("tenured") to the Old Generation C) It moves to Metaspace D) It stays in Eden forever
**Answer: B**
*Explanation:* Objects that repeatedly survive Minor GC (bouncing between Survivor spaces) are eventually promoted to the Old Generation, since they're likely long-lived.
*Why others wrong:* A) Survival doesn't mean automatic deletion — quite the opposite, it means it's being kept. C) Metaspace is for class metadata only, not regular long-lived objects. D) Eden is only for brand-new objects; survivors move out of Eden after the first GC.

**Q17.** [Medium | Comparison] Which of these correctly matches memory area to what it stores?
A) Stack → Objects, Heap → Method frames B) Stack → Method frames/locals, Heap → Objects C) Heap → Method frames, Metaspace → Objects D) Stack → Class metadata, Heap → Method frames
**Answer: B**
*Explanation:* Stack holds per-thread method call frames and local variables/references; Heap holds actual objects and their instance data — the standard, frequently tested pairing.
*Why others wrong:* A, C, D all swap or scramble the correct area-to-content mapping.

**Q18.** [Hard | Scenario] A static field `static List<Object> cache = new ArrayList<>();` keeps growing and objects are never removed from it, even though they're no longer needed. What is this an example of?
A) Normal GC behavior, no issue B) A logical memory leak, since the static reference keeps objects reachable indefinitely C) `StackOverflowError` waiting to happen D) A compilation warning that must be fixed before running
**Answer: B**
*Explanation:* Because `cache` is `static`, it lives for the class's entire lifetime and stays reachable from a GC Root, so every object added to it remains reachable — GC can never collect them even if logically "done," causing a memory leak.
*Why others wrong:* A) This is a real problem, not expected/normal usage. C) Stack overflow relates to recursion/call depth, unrelated to a growing list. D) It's a runtime design issue, not something the compiler flags.

**Q19.** [Medium | Terminology] "Stop-the-world" in the context of GC refers to:
A) A permanent JVM shutdown B) A pause where all application threads are suspended while GC performs its work C) A network timeout D) A compiler optimization phase
**Answer: B**
*Explanation:* During certain GC phases (especially Major/Full GC), the JVM pauses all application threads ("stop-the-world") so it can safely traverse/reclaim the heap without interference.
*Why others wrong:* A) It's a temporary pause, not a shutdown. C) Unrelated to networking. D) It's a runtime GC behavior, not a compile-time step.

**Q20.** [Hard | Comparison] Which correctly ranks GC pause impact and frequency from most frequent/lightest to least frequent/heaviest?
A) Full GC → Minor GC B) Minor GC → Full GC C) They occur with identical frequency and pause time D) Metaspace GC → Minor GC → no Full GC ever needed
**Answer: B**
*Explanation:* Minor GC (Young Gen) happens frequently and is fast/lightweight since most objects die young; Full/Major GC (Old Gen, often whole heap) happens less often but causes longer pauses.
*Why others wrong:* A) Reverses the correct order. C) They differ significantly in both frequency and pause duration. D) Full GC is still needed periodically for Old Generation cleanup; it doesn't disappear.

---

## Chapter 11 Complete ✅
Score yourself: if you missed Q3, Q5, Q8, Q12, or Q15 — reread "Eligibility for GC" (§2) and "Heap Structure" (§5), the two highest-trap-density sections here.

**Next up: Chapter 12 — File Handling, Serialization, Annotations & Reflection Basics.**

# DSA — Chapter 15: Linked List, Stack, Queue, Hashing

## 1. Linked List

A sequence of **nodes**, each holding data + a reference (pointer) to the next node. Unlike arrays, elements are NOT stored in contiguous memory.

```java
class Node {
    int data;
    Node next;
}
```

### Types
| Type | Description |
|---|---|
| Singly Linked List | Each node points to next only; last node → null |
| Doubly Linked List | Each node has `next` AND `prev` pointers |
| Circular Linked List | Last node points back to the head (no null end) |

### Array vs Linked List

| Feature | Array | Linked List |
|---|---|---|
| Memory | Contiguous | Non-contiguous (scattered) |
| Access by index | O(1) | O(n) |
| Insertion/Deletion at beginning | O(n) (shifting) | O(1) |
| Insertion/Deletion at end | O(1) amortized | O(n) singly / O(1) if tail pointer kept |
| Size | Fixed (or resized with cost) | Dynamic |
| Extra memory | None | Pointer per node |

**Capgemini trap:** "Linked List insertion is always O(1)" → **False**. Insertion at a *known* position (e.g., head, or given a node reference) is O(1); inserting at an arbitrary position requires O(n) traversal first.

## 2. Stack (LIFO — Last In, First Out)

Operations: `push` (add to top), `pop` (remove from top), `peek`/`top` (view top).
- All operations: **O(1)**
- Used for: function call stack, undo features, expression evaluation (infix→postfix), balanced parentheses checking, DFS.

```java
Stack<Integer> stack = new Stack<>();
stack.push(10);
stack.pop();
stack.peek();
```

**Java note:** `java.util.Stack` extends `Vector` (legacy, synchronized). Modern preferred choice: `Deque<Integer> stack = new ArrayDeque<>();` used via `push()`/`pop()`.

## 3. Queue (FIFO — First In, First Out)

Operations: `enqueue` (add to rear), `dequeue` (remove from front).
- All operations: **O(1)**
- Used for: BFS, task scheduling, print queues.

### Variants
| Type | Description |
|---|---|
| Simple Queue | FIFO, insert rear, remove front |
| Circular Queue | Rear wraps around to reuse freed front space (avoids wasted array slots) |
| Deque (Double-Ended Queue) | Insert/remove from BOTH ends |
| Priority Queue | Elements served by priority, not insertion order (usually backed by a Heap) |

**Capgemini trap:** Java's `Queue` interface — `poll()` returns null if empty, `remove()` throws exception if empty. Similarly `offer()` returns false on failure, `add()` throws exception.

## 4. Hashing

**HashMap** stores key-value pairs using a **hash function** to compute an index (bucket) for storage.

- Average time for get/put: **O(1)**
- Worst case (many collisions in same bucket): O(n) — in Java 8+, buckets convert to a balanced tree (Red-Black Tree) once a threshold (8 entries) is hit, making worst-case **O(log n)** in modern Java.

### Collision Handling
| Method | How it works |
|---|---|
| Chaining | Each bucket holds a linked list (or tree) of entries with the same hash |
| Open Addressing | On collision, probe for the next empty slot (linear/quadratic probing) |

Java's `HashMap` uses **chaining** (with treeification for large buckets).

### HashMap vs HashSet vs HashTable

| | Stores | Null keys/values | Thread-safe |
|---|---|---|---|
| HashMap | Key-value pairs | 1 null key, multiple null values | No |
| HashSet | Unique values only (backed by HashMap internally) | 1 null element | No |
| Hashtable | Key-value pairs | No nulls allowed | Yes (synchronized, legacy) |

**Capgemini trap:** `HashMap` allows one null key; `Hashtable` throws `NullPointerException` on null key/value.

### equals() and hashCode() Contract
- If two objects are equal (`equals()` returns true), they **must** have the same `hashCode()`.
- The reverse is NOT required — different objects **can** share the same hashCode (hash collision) but must still be equal only if `equals()` confirms it.
- **Trap:** Overriding `equals()` without overriding `hashCode()` breaks HashMap/HashSet behavior (duplicate-looking objects may both get inserted).

## 5. Frequently Asked Capgemini Questions
- "Time complexity to insert at the head of a Singly Linked List?" → O(1)
- "Time complexity to reverse a Linked List?" → O(n) time, O(1) space (iterative)
- "Which data structure is used for undo/redo functionality?" → Stack
- "Which data structure is used in BFS traversal?" → Queue
- "What happens when hashCode() is overridden but not equals()?" → HashMap behaves inconsistently / duplicates may occur

## 6. Interviewer Traps
1. Detecting a cycle in a Linked List → **Floyd's Cycle Detection (slow/fast pointer)**, O(n) time, O(1) space. Not "use a HashSet" (that's O(n) space — valid but not optimal).
2. `Stack` (`java.util.Stack`) is legacy/synchronized — prefer `ArrayDeque` for non-thread-safe stack usage in modern Java.
3. Circular Queue "wastes" one slot traditionally to distinguish full vs empty state — common trick question.
4. HashMap iteration order is **not guaranteed**; use `LinkedHashMap` for insertion order or `TreeMap` for sorted order.
5. `ConcurrentModificationException` — modifying a collection while iterating with a normal iterator (not using `iterator.remove()`).

## One-Page Revision
- Linked List: non-contiguous, O(1) insert/delete at known position, O(n) search/index access.
- Stack: LIFO, O(1) push/pop/peek; used for recursion, undo, DFS, expression parsing.
- Queue: FIFO, O(1) enqueue/dequeue; used for BFS, scheduling. Deque = both ends; PriorityQueue = priority order.
- HashMap: O(1) avg get/put, O(log n) worst (Java 8+ treeification), uses chaining for collisions.
- equals() true ⟹ hashCode() must match. Overriding one without the other breaks collections.
- Cycle detection in Linked List → Floyd's slow/fast pointer, O(n) time O(1) space.

---

# MCQs — Chapter 15 (17 Questions)

**Q1.** [Easy | Linked List] Time complexity to access the k-th element in a Singly Linked List?
A) O(1) B) O(log n) C) O(n) D) O(k)
**Answer: C**
*Explanation:* No direct indexing exists; must traverse from head → O(n).
*Why others wrong:* O(1) applies to arrays; D is a distractor mimicking O(n) but framed differently.

**Q2.** [Easy | Stack] Which operation removes and returns the top element of a stack?
A) peek() B) push() C) pop() D) top()
**Answer: C**
*Explanation:* `pop()` removes and returns the top element; `peek()`/`top()` only view without removing.
*Why others wrong:* `push()` adds an element instead.

**Q3.** [Easy | Queue] Which principle does a Queue follow?
A) LIFO B) FIFO C) Random access D) Priority-based only
**Answer: B**
*Explanation:* First element inserted is the first one removed.
*Why others wrong:* LIFO describes Stack; random access describes arrays; priority is specific to PriorityQueue only.

**Q4.** [Medium | Linked List Trap] Is "Linked List insertion is always O(1)" a true statement?
A) True B) False
**Answer: B**
*Explanation:* O(1) only applies when inserting at a known position (like head); inserting at an arbitrary position requires O(n) traversal first.
*Why others wrong:* A ignores the need to locate the insertion point.

**Q5.** [Medium | HashMap] Average time complexity of HashMap's get() and put()?
A) O(n) B) O(log n) C) O(1) D) O(n log n)
**Answer: C**
*Explanation:* Hashing computes the bucket index directly, giving average constant time.
*Why others wrong:* O(n)/O(log n) apply only to worst-case scenarios with heavy collisions.

**Q6.** [Medium | Collision Handling] Which collision-resolution method does Java's HashMap use?
A) Open addressing (linear probing) B) Chaining (linked list/tree per bucket) C) Double hashing D) Cuckoo hashing
**Answer: B**
*Explanation:* Java's HashMap stores colliding entries as a linked list per bucket, converting to a tree if the bucket grows large (Java 8+).
*Why others wrong:* Open addressing, double hashing, and cuckoo hashing are alternate strategies not used by Java's standard HashMap.

**Q7.** [Medium | equals/hashCode] What happens if you override `equals()` but NOT `hashCode()`?
A) Compilation error B) HashMap/HashSet may behave inconsistently, allowing logical duplicates C) Nothing changes D) The object becomes immutable
**Answer: B**
*Explanation:* Breaking the equals-hashCode contract can cause equal objects to land in different buckets, breaking lookups/duplicate prevention.
*Why others wrong:* This is a runtime logic issue, not a compile-time error.

**Q8.** [Hard | Cycle Detection] Best approach to detect a cycle in a Linked List with O(1) space?
A) Use a HashSet to track visited nodes B) Floyd's Cycle Detection (slow/fast pointer) C) Reverse the list and compare D) Convert to an array and check duplicates
**Answer: B**
*Explanation:* Two pointers moving at different speeds will meet inside a cycle, using no extra data structures.
*Why others wrong:* HashSet approach works but uses O(n) space, not optimal.

**Q9.** [Medium | HashSet] What is a HashSet internally backed by in Java?
A) An array B) A HashMap (values used as a dummy) C) A LinkedList D) A Tree
**Answer: B**
*Explanation:* HashSet internally delegates to a HashMap, storing elements as keys with a constant dummy value.
*Why others wrong:* Arrays/LinkedList/Tree are not the internal backing structure for HashSet.

**Q10.** [Easy | Deque] What does a Deque allow that a regular Queue does not?
A) Sorting elements B) Insertion and removal from both ends C) Duplicate elimination D) Thread safety
**Answer: B**
*Explanation:* Deque (Double-Ended Queue) supports operations at both front and rear.
*Why others wrong:* Sorting, duplicate elimination, and thread safety are unrelated to the Deque's defining feature.

**Q11.** [Medium | Java Collections] Which is generally preferred over `java.util.Stack` for stack operations in modern Java?
A) ArrayList B) ArrayDeque C) LinkedHashMap D) TreeSet
**Answer: B**
*Explanation:* `java.util.Stack` is legacy and synchronized (slower); `ArrayDeque` is faster and recommended for non-thread-safe stack use.
*Why others wrong:* ArrayList, LinkedHashMap, TreeSet aren't designed as stack replacements.

**Q12.** [Medium | Queue Methods] What is the difference between `poll()` and `remove()` on an empty Queue?
A) No difference B) `poll()` returns null; `remove()` throws an exception C) `poll()` throws an exception; `remove()` returns null D) Both throw exceptions
**Answer: B**
*Explanation:* `poll()` is the "safe" method returning null on failure; `remove()` throws `NoSuchElementException`.
*Why others wrong:* This is a commonly tested Java Collections API distinction.

**Q13.** [Hard | Priority Queue] A PriorityQueue in Java is typically implemented internally using which data structure?
A) Sorted Array B) Binary Heap C) Linked List D) Hash Table
**Answer: B**
*Explanation:* Java's `PriorityQueue` uses a binary heap internally, giving O(log n) insertion/removal while maintaining priority order.
*Why others wrong:* Sorted arrays would need O(n) insertion; linked lists/hash tables don't naturally maintain priority ordering efficiently.

**Q14.** [Medium | HashMap Nulls] How many null keys does Java's HashMap allow?
A) Zero B) One C) Unlimited D) Depends on capacity
**Answer: B**
*Explanation:* HashMap permits exactly one null key (multiple null values are allowed though).
*Why others wrong:* Hashtable (legacy) allows zero null keys — a common point of confusion.

**Q15.** [Hard | Iteration Order] Which Map implementation maintains insertion order during iteration?
A) HashMap B) TreeMap C) LinkedHashMap D) Hashtable
**Answer: C**
*Explanation:* LinkedHashMap maintains a doubly-linked list internally to preserve insertion order.
*Why others wrong:* HashMap's order is unspecified; TreeMap sorts by key; Hashtable's order is also unspecified.

**Q16.** [Medium | Circular Queue] Why do circular queues traditionally "waste" one array slot?
A) To improve cache performance B) To distinguish between the "full" and "empty" states using front/rear pointers C) Java requires it D) It doesn't waste any slot
**Answer: B**
*Explanation:* Without a dedicated slot or a counter, front==rear is ambiguous (could mean empty or full).
*Why others wrong:* This is purely a logical design choice, not a performance or language requirement.

**Q17.** [Hard | ConcurrentModificationException] What causes a `ConcurrentModificationException` when working with a List?
A) Using an enhanced for-loop while modifying the list directly (not via iterator.remove()) B) Adding elements before iterating C) Declaring the list as final D) Using a for-loop with an index
**Answer: A**
*Explanation:* Modifying a collection structurally during iteration (except via the iterator's own remove method) invalidates the iterator's internal state tracking.
*Why others wrong:* Adding before iteration starts, using final, or index-based loops (on ArrayList specifically) don't trigger this exception the same way.

---

## Chapter 15 Complete ✅
If you missed Q4, Q7, Q8, or Q17 — reread sections 1 and 4; these are the highest-trap-density concepts (linked list insertion cost, equals/hashCode contract, and iterator safety).

**Next up: Chapter 16 — Trees, Heap, Graphs (BFS/DFS).**

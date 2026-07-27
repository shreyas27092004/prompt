# JAVA CORE — Chapter 8: Collections Framework

## 1. Collection Hierarchy Overview

```
Iterable
└── Collection
    ├── List (ordered, duplicates allowed)
    │   ├── ArrayList
    │   ├── LinkedList
    │   └── Vector (legacy, synchronized)
    ├── Set (no duplicates)
    │   ├── HashSet (no order)
    │   ├── LinkedHashSet (insertion order)
    │   └── TreeSet (sorted order)
    └── Queue (FIFO typically)
        ├── LinkedList (also implements Queue)
        └── PriorityQueue

Map (separate hierarchy — NOT a Collection!)
├── HashMap (no order)
├── LinkedHashMap (insertion order)
└── TreeMap (sorted order)
```

**Trap #1:** `Map` does NOT extend `Collection`. This is one of the most commonly tested facts — Map is a completely separate interface hierarchy (it deals with key-value pairs, not single elements).

## 2. List Implementations — Comparison

| Feature | ArrayList | LinkedList |
|---|---|---|
| Internal structure | Dynamic array | Doubly linked list |
| Random access (`get(i)`) | **Fast — O(1)** | Slow — O(n) |
| Insertion/deletion (middle) | Slow — O(n), shifts elements | **Fast — O(1)** once position found |
| Insertion/deletion (at ends) | O(1) amortized at end, O(n) at start | O(1) at both ends |
| Memory overhead | Lower | Higher (node pointers) |
| Implements Deque? | No | Yes |

**Trap:** "Which is faster for random access?" → ArrayList. "Which is faster for frequent insertions/deletions in the middle?" → LinkedList.

## 3. Set Implementations — Comparison

| Feature | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Order | No guaranteed order | Insertion order preserved | Sorted (natural or Comparator) |
| Underlying structure | HashMap internally | HashMap + LinkedList internally | Red-Black Tree (TreeMap internally) |
| Null allowed? | One null allowed | One null allowed | **No null allowed** (throws NPE if elements need comparison) |
| Performance | O(1) average for add/remove/contains | O(1) average | O(log n) for add/remove/contains |

**Trap:** TreeSet throws `NullPointerException` when you try to add null, because it needs to compare elements to maintain sorted order, and comparing with null fails.

## 4. Map Implementations — Comparison

| Feature | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| Order | No guaranteed order | Insertion order | Sorted by key |
| Null key | **One null key allowed** | One null key allowed | **No null key** (NPE) |
| Null values | Multiple null values allowed | Multiple null values allowed | Multiple null values allowed |
| Thread-safe | No | No | No |
| Performance | O(1) average | O(1) average | O(log n) |

**Trap:** HashMap allows exactly ONE null key (and multiple null values); TreeMap does NOT allow a null key (throws NullPointerException) since it needs to compare keys for sorting.

**HashMap internal working (frequently asked):**
1. `key.hashCode()` computed → determines bucket index.
2. If two keys land in the same bucket ("collision"), they're stored as a linked list (or a Red-Black Tree if the bucket gets large, since Java 8) within that bucket.
3. `key.equals()` is used to check actual key equality when retrieving/comparing within a bucket.
4. This is WHY overriding `equals()` without `hashCode()` breaks HashMap — inconsistent hashing means the object might be searched in the wrong bucket entirely.

## 5. Comparable vs Comparator — heavily tested comparison

| Feature | Comparable | Comparator |
|---|---|---|
| Package | java.lang | java.util |
| Method | `compareTo(Object o)` | `compare(Object o1, Object o2)` |
| Where implemented | Inside the class itself | Separate class (or lambda) |
| Number of sort orders | Only ONE natural ordering per class | Multiple different orderings possible |
| Used by | `Collections.sort(list)` | `Collections.sort(list, comparator)` |
| Modifies original class? | Yes (class must implement it) | No (external, doesn't touch original class) |

```java
class Employee implements Comparable<Employee> {
    int salary;
    public int compareTo(Employee other) {
        return this.salary - other.salary; // natural ordering by salary
    }
}

// Comparator - external, multiple possible:
Comparator<Employee> byName = (e1, e2) -> e1.name.compareTo(e2.name);
Collections.sort(employeeList, byName);
```

**Trap:** `compareTo()` returns: negative if `this < other`, 0 if equal, positive if `this > other`. Common trap: `return this.salary - other.salary;` can cause **integer overflow** for very large/small salary values — safer to use `Integer.compare(this.salary, other.salary)`.

## 6. Iterator vs ListIterator vs Enhanced for-loop

| Feature | Iterator | ListIterator | for-each |
|---|---|---|---|
| Direction | Forward only | Forward AND backward | Forward only |
| Can remove elements during iteration? | Yes, via `iterator.remove()` | Yes, via `listIterator.remove()`/`set()`/`add()` | **No — throws ConcurrentModificationException** |
| Works on | All Collections | List only | All Iterables |

**Trap — ConcurrentModificationException:**
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
for (Integer i : list) {
    if (i == 2) list.remove(i); // ConcurrentModificationException!
}
// CORRECT way: use Iterator explicitly
Iterator<Integer> it = list.iterator();
while (it.hasNext()) {
    if (it.next() == 2) it.remove(); // safe
}
```
Modifying a Collection directly while iterating with for-each (or a raw Iterator without using `.remove()`) throws `ConcurrentModificationException`.

## 7. Queue & Stack

- `Queue` — FIFO (First In First Out). Key methods: `offer()`, `poll()`, `peek()`.
- `Deque` (Double-Ended Queue) — can act as both Queue (FIFO) and Stack (LIFO). `ArrayDeque` is the preferred modern replacement for legacy `Stack` class.
- `PriorityQueue` — elements ordered by natural ordering or Comparator, NOT insertion order; `poll()` always returns the smallest (or highest priority) element.

**Trap:** The legacy `Stack` class extends `Vector` (synchronized, slower) — modern code prefers `ArrayDeque` for stack-like LIFO behavior.

## 8. Generics (brief, MCQ-relevant basics)
- `List<String>` — type safety at compile time, avoids `ClassCastException` at runtime.
- Diamond operator: `List<String> list = new ArrayList<>();` (Java 7+, infers type).
- Wildcards: `<? extends T>` (upper bound, read-only-ish), `<? super T>` (lower bound, write-ish) — PECS: **P**roducer **E**xtends, **C**onsumer **S**uper.

## One-Page Revision
- Map does NOT extend Collection — separate hierarchy.
- ArrayList: fast random access (O(1)), slow middle insert/delete (O(n)). LinkedList: opposite.
- HashSet: no order. LinkedHashSet: insertion order. TreeSet: sorted, NO null allowed.
- HashMap: one null key allowed, no order. TreeMap: NO null key, sorted by key.
- HashMap internals: hashCode() → bucket, equals() → exact match within bucket. Broken hashCode/equals contract = broken HashMap behavior.
- Comparable = compareTo(), internal, ONE natural order. Comparator = compare(), external, MULTIPLE orders possible.
- for-each + direct list.remove() = ConcurrentModificationException. Use Iterator.remove() instead.
- Deque/ArrayDeque preferred over legacy synchronized Stack class.
- PriorityQueue orders by priority (natural/Comparator), not insertion order.

---

# MCQs — Chapter 8 (25 Questions)

**Q1.** [Easy | Hierarchy] Does `Map` extend `Collection`?
A) Yes B) No, Map is a separate hierarchy C) Only HashMap does D) Only in Java 8+
**Answer: B**
*Explanation:* Map is fundamentally about key-value pairs, structurally and conceptually distinct from Collection's single-element model — it's a top-level interface of its own.
*Why others wrong:* A) A very common false assumption — directly incorrect. C) No implementation of Map extends Collection, including HashMap. D) This has never changed across any Java version — Map has always been separate.

**Q2.** [Easy | ArrayList vs LinkedList] Which is faster for random access via `get(index)`?
A) LinkedList B) ArrayList C) Both are equal D) Neither supports get(index)
**Answer: B**
*Explanation:* ArrayList uses a backing array with direct index-based access (O(1)); LinkedList must traverse node-by-node from an end (O(n)).
*Why others wrong:* A) LinkedList is significantly slower for this exact operation. C) Their performance for get(index) is clearly different. D) Both support get(index) via the List interface — the difference is purely about efficiency.

**Q3.** [Medium | TreeSet null] What happens when you try `treeSet.add(null)` on an empty TreeSet?
A) Adds null successfully B) Throws NullPointerException C) Silently ignores the add D) Throws ClassCastException
**Answer: B**
*Explanation:* TreeSet requires comparing elements to maintain sort order; comparing against null triggers a NullPointerException internally.
*Why others wrong:* A) Directly contradicted by TreeSet's null-handling restriction. C) It's not silently ignored — an actual exception is thrown. D) ClassCastException would apply to incompatible-type comparisons, not null handling specifically.

**Q4.** [Medium | HashMap null key] How many null keys does HashMap allow?
A) Zero B) Exactly one C) Unlimited D) Depends on initial capacity
**Answer: B**
*Explanation:* HashMap explicitly supports exactly one null key (stored in a special bucket), though it allows multiple null VALUES.
*Why others wrong:* A) HashMap does support a null key, unlike TreeMap. C) Unlike values, only one null key can exist (since a key is unique by definition anyway). D) Not related to initial capacity settings at all.

**Q5.** [Medium | Comparable vs Comparator] Which interface's method is `compareTo()`?
A) Comparator B) Comparable C) Iterable D) Iterator
**Answer: B**
*Explanation:* Comparable defines a class's own single natural ordering via the compareTo(Object) method, implemented within the class itself.
*Why others wrong:* A) Comparator uses `compare(o1, o2)` instead, as an external comparison strategy. C, D) Both relate to iteration mechanics, unrelated to ordering/comparison logic.

**Q6.** [Medium | ConcurrentModificationException] What happens here?
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1,2,3));
for (Integer i : list) {
    if (i == 2) list.remove(i);
}
```
A) Compiles and runs fine, removes 2 B) ConcurrentModificationException at runtime C) Compile error D) Silently does nothing
**Answer: B**
*Explanation:* Directly modifying a List while iterating over it with for-each (which uses an internal Iterator) invalidates the iterator's expected modification count, triggering this exception.
*Why others wrong:* A) The removal attempt actually throws an exception rather than completing successfully. C) This is a runtime issue, not a compile-time one — the code is syntactically valid. D) An active exception is thrown, not silent failure.

**Q7.** [Hard | HashMap internals] What role does `hashCode()` play in HashMap?
A) It determines which bucket a key-value pair is stored in B) It directly returns the stored value C) It sorts entries alphabetically D) It has no role, only equals() matters
**Answer: A**
*Explanation:* The hashCode determines the bucket index where an entry is placed/looked up; equals() is then used to resolve exact key matching within that bucket (in case of collisions).
*Why others wrong:* B) hashCode() returns an int hash, not the actual stored value. C) HashMap does NOT sort entries at all (that's TreeMap's behavior). D) equals() and hashCode() work together — both matter, contradicting this option's dismissal of hashCode().

**Q8.** [Easy | LinkedHashSet] What ordering does LinkedHashSet maintain?
A) No particular order B) Sorted natural order C) Insertion order D) Reverse insertion order
**Answer: C**
*Explanation:* LinkedHashSet combines HashSet's uniqueness with a linked list structure that preserves the order elements were originally inserted.
*Why others wrong:* A) That describes plain HashSet, not LinkedHashSet. B) That describes TreeSet's behavior. D) LinkedHashSet preserves forward insertion order, not reversed.

**Q9.** [Medium | PriorityQueue] What does `poll()` return on a PriorityQueue?
A) The most recently added element B) A random element C) The element with the highest priority (smallest by natural order, by default) D) Always null
**Answer: C**
*Explanation:* PriorityQueue orders elements by natural ordering (or a provided Comparator) internally, and `poll()` retrieves/removes the head — which is the smallest element by default (min-heap behavior).
*Why others wrong:* A) It's not FIFO/insertion-based like a regular Queue — it's priority-based. B) The retrieval is deterministic based on priority, not random. D) Only returns null if the queue is empty, not as a default behavior.

**Q10.** [Hard | Comparable integer overflow trap] What's the risk with this compareTo implementation?
```java
public int compareTo(Employee other) {
    return this.salary - other.salary;
}
```
A) No risk, this is standard practice B) Potential integer overflow for very large/small salary values, producing incorrect comparison results C) Compile error D) It only works for salaries under 100
**Answer: B**
*Explanation:* Subtracting two ints can overflow the int range if the values are large enough (or have opposite extreme signs), silently producing an incorrect sign and thus a wrong comparison result — `Integer.compare()` is the safer alternative.
*Why others wrong:* A) It's a commonly seen but genuinely risky pattern — a real trap Capgemini likes to test. C) Compiles perfectly fine — the issue is a runtime/logical correctness problem, not a syntax error. D) No such arbitrary threshold restriction exists — the overflow risk depends on the actual magnitude and sign combinations involved.

**Q11.** [Medium | Iterator.remove] Which is the SAFE way to remove elements while iterating?
A) Using for-each with list.remove() directly B) Using Iterator's own `.remove()` method C) Using a regular for loop with increasing index while removing D) There is no safe way to remove during iteration
**Answer: B**
*Explanation:* Iterator.remove() is specifically designed to safely modify the underlying collection during iteration without corrupting the iterator's internal state/modCount tracking.
*Why others wrong:* A) This exact pattern throws ConcurrentModificationException, as shown in Q6. C) Removing while incrementing an index in a regular for-loop causes skipped elements due to shifting indices (a different, subtler bug) — not a recommended safe pattern. D) A safe way DOES exist — Iterator.remove() is exactly that.

**Q12.** [Medium | Set uniqueness] What happens when you add a duplicate element to a HashSet?
A) Throws an exception B) The duplicate is silently ignored (add() returns false) C) Replaces the existing element D) Creates a second copy
**Answer: B**
*Explanation:* Set enforces uniqueness; attempting to add an already-present element (per equals()/hashCode()) simply does nothing and add() returns false to signal this.
*Why others wrong:* A) No exception is thrown for this normal, expected scenario. C) Nothing is "replaced" since the equal element already effectively represents the same logical entry. D) Sets explicitly prevent duplicate copies from existing — that's their defining characteristic.

**Q13.** [Hard | TreeMap] Which exception is thrown when you try `treeMap.put(null, "value")`?
A) No exception, works fine B) NullPointerException C) IllegalArgumentException D) ClassCastException
**Answer: B**
*Explanation:* TreeMap needs to compare keys to maintain sorted order, and comparing against a null key isn't possible, resulting in NullPointerException.
*Why others wrong:* A) Directly contradicts TreeMap's explicit null-key restriction (unlike HashMap). C) Not the specific exception type Java throws here. D) ClassCastException relates to incompatible type comparisons, not null handling specifically.

**Q14.** [Medium | Deque vs Stack] Which class is the MODERN preferred choice for LIFO stack operations?
A) java.util.Stack B) ArrayDeque C) Vector D) LinkedHashSet
**Answer: B**
*Explanation:* ArrayDeque is faster and non-synchronized (unless you need thread-safety), making it the recommended modern replacement over the legacy, synchronized Stack class.
*Why others wrong:* A) Legacy class, extends Vector, carries unnecessary synchronization overhead for typical single-threaded use. C) Vector itself is legacy and synchronized, not stack-specific at all. D) LinkedHashSet is a Set implementation, entirely unrelated to LIFO/stack behavior.

**Q15.** [Medium | Generics wildcard] What does `<? extends T>` represent in generics?
A) A lower bound wildcard (Consumer) B) An upper bound wildcard (Producer) C) An exact type match only D) No relation to bounds at all
**Answer: B**
*Explanation:* `<? extends T>` restricts to T or its subtypes, commonly used for "producer" scenarios (reading/retrieving values) — remember mnemonic PECS: Producer Extends, Consumer Super.
*Why others wrong:* A) That describes `<? super T>` instead. C) Wildcards specifically allow flexibility across a type hierarchy, not exact matching. D) It directly relates to type bound restriction — that's its entire purpose.

**Q16.** [Hard | Collision handling] Since Java 8, what happens to a HashMap bucket when too many entries collide (same hash)?
A) The bucket keeps using a plain linked list indefinitely B) The bucket structure converts from linked list to a Red-Black Tree for better worst-case performance C) HashMap throws an exception D) The HashMap automatically becomes a TreeMap
**Answer: B**
*Explanation:* Java 8 introduced treeification of heavily-collided buckets (converting from O(n) linked list lookup to O(log n) tree lookup) once a threshold (default 8 entries) is reached, improving worst-case performance under hash collision attacks.
*Why others wrong:* A) This was the pre-Java-8 behavior; Java 8 specifically improved on this. C) No exception is thrown for normal collision handling — it's handled gracefully internally. D) The HashMap itself doesn't change type; only the internal bucket structure changes.

**Q17.** [Medium | Queue methods] Which Queue method returns null instead of throwing an exception when the queue is empty?
A) `remove()` B) `element()` C) `poll()` D) `add()`
**Answer: C**
*Explanation:* `poll()` is the "safe" retrieval-and-remove method that returns null on an empty queue, contrasted with `remove()` which throws NoSuchElementException in that case.
*Why others wrong:* A) `remove()` throws an exception on empty queue rather than returning null. B) `element()` also throws NoSuchElementException on empty queue (it's the "unsafe" peek equivalent). D) `add()` is for insertion, throws an exception on failure (e.g., capacity-restricted queues), unrelated to this null-return behavior.

**Q18.** [Medium | List vs Set] Which allows duplicate elements?
A) HashSet B) TreeSet C) ArrayList D) LinkedHashSet
**Answer: C**
*Explanation:* List implementations explicitly allow duplicate elements since they're ordered by index/position, unlike Set implementations which enforce uniqueness.
*Why others wrong:* A, B, D) All are Set implementations, which by definition disallow duplicate elements.

**Q19.** [Hard | Comparator lambda] Which correctly creates a Comparator sorting Strings by length?
A) `Comparator<String> c = (s1, s2) -> s1.length() - s2.length();` B) `Comparable<String> c = (s1, s2) -> s1.length() - s2.length();` C) `Comparator<String> c = s -> s.length();` D) `Comparator c = String::length;`
**Answer: A**
*Explanation:* This correctly implements Comparator's functional interface method `compare(T o1, T o2)` returning an int via a two-argument lambda.
*Why others wrong:* B) Comparable's compareTo takes only ONE argument (comparing `this` to another) — this lambda signature doesn't match Comparable's functional shape correctly. C) Single-argument lambda doesn't match Comparator's two-argument `compare()` signature. D) Missing generic type and doesn't correctly form a valid two-argument comparison via simple method reference in this form.

**Q20.** [Medium | Vector vs ArrayList] What's the key difference between Vector and ArrayList?
A) Vector is synchronized (thread-safe); ArrayList is not B) ArrayList is synchronized; Vector is not C) They are functionally identical in every way D) Vector doesn't support generics
**Answer: A**
*Explanation:* Vector is a legacy synchronized class from Java 1.0; ArrayList (introduced later) is unsynchronized, making it faster for single-threaded use — the primary reason ArrayList is generally preferred today.
*Why others wrong:* B) Reverses the actual synchronization relationship. C) They differ meaningfully in synchronization and performance characteristics. D) Vector fully supports generics like any modern Java collection.

**Q21.** [Hard | Trap] Which of the following is TRUE about `Collections.sort(list)` when the list contains custom objects?
A) It works automatically for any object type B) The object's class MUST implement Comparable, or a Comparator must be passed C) It always throws an exception for custom objects D) It sorts based on hashCode() by default
**Answer: B**
*Explanation:* Without a natural ordering (Comparable) defined on the class, or an explicit Comparator supplied as a second argument, Collections.sort() has no way to determine element ordering and throws a ClassCastException at runtime.
*Why others wrong:* A) Only true for built-in types with natural ordering (like String, Integer) — custom classes need explicit Comparable/Comparator support. C) It only throws an exception if NEITHER Comparable nor a Comparator is provided — it works correctly when either is properly implemented. D) hashCode() has nothing to do with sorting order.

**Q22.** [Medium | Iterable vs Collection] Which interface does `Collection` extend?
A) Iterable B) Comparable C) Serializable D) Cloneable
**Answer: A**
*Explanation:* Collection extends Iterable, which is precisely why all Collection implementations (List, Set, Queue) can be used in enhanced for-each loops.
*Why others wrong:* B, C, D) None of these are part of the Collection interface's direct inheritance chain — they're unrelated interfaces that specific classes might separately implement.

**Q23.** [Hard | Scenario] What's the output?
```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("A");
set.add("B");
System.out.println(set.size());
```
A) 3 B) 2 C) 1 D) Compile error
**Answer: B**
*Explanation:* HashSet rejects the duplicate "A" (second add() call returns false silently), leaving only two unique elements: "A" and "B".
*Why others wrong:* A) Would only be true if duplicates were allowed, contradicting Set's core uniqueness constraint. C) Both "A" and "B" (distinct values) are genuinely present — not just one. D) Perfectly valid, compiling code.

**Q24.** [Medium | LinkedList as Deque] Why can LinkedList be used both as a List and a Queue/Deque?
A) It's a coincidence with no real design reason B) LinkedList implements both the List interface and the Deque interface C) LinkedList secretly converts internally depending on usage D) Only ArrayList can do this, not LinkedList
**Answer: B**
*Explanation:* LinkedList's class declaration explicitly implements both `List<E>` and `Deque<E>`, giving it dual capability — ordered indexed access AND double-ended queue operations.
*Why others wrong:* A) This is a deliberate, well-documented design decision, not coincidental. C) No hidden internal conversion happens — it's simply a single class implementing multiple interfaces. D) It's specifically LinkedList (not ArrayList) that implements Deque — ArrayList does not.

**Q25.** [Hard | Trap Summary] Why must classes override BOTH equals() and hashCode() together when used as HashMap/HashSet keys?
A) It's only a style guideline with no functional impact B) Because inconsistent equals()/hashCode() implementations break correct bucket lookup, potentially causing "lost" or duplicate entries C) hashCode() is deprecated and unnecessary in modern Java D) Only equals() matters; hashCode() is automatically generated correctly regardless
**Answer: B**
*Explanation:* HashMap/HashSet use hashCode() to locate a bucket, then equals() to confirm the exact match within that bucket — if two "equal" objects produce different hashCodes, they may end up in different buckets entirely, breaking lookups and causing subtle, hard-to-diagnose bugs (a recurring theme from Chapter 6, reinforced here in Collections).
*Why others wrong:* A) This has real, demonstrable functional consequences, not just stylistic preference. C) hashCode() remains fully relevant and required in all modern Java collection usage. D) The default Object.hashCode() is based on memory address/identity, which won't align with a custom equals() based on content — you cannot rely on the default being "automatically correct."

---

## Chapter 8 Complete ✅
This was a dense chapter — watch out for: Map-not-a-Collection (Q1), null handling differences across HashMap/TreeMap/TreeSet (Q3, Q4, Q13), ConcurrentModificationException (Q6, Q11), and the equals/hashCode-breaks-HashMap theme that spans Chapters 6 and 8 (Q7, Q25) — it's asked from multiple angles across the whole exam.

**Next up: Chapter 9 — Java 8 Features (Lambda, Streams, Functional Interfaces, Optional).** This is also heavily weighted for Capgemini. Say "next chapter" to continue.

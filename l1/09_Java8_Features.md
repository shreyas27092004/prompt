# MODULE 1 — Chapter 9: Java 8 Features

---

## 9.1 Why Java 8 Matters (Interview Angle)

Before Java 8, Java was purely object-oriented and imperative — you told the computer *how* to do something step by step. Java 8 (2014) introduced **functional programming** ideas: you now tell the computer *what* you want, and it figures out how.

**Why Capgemini loves this topic:** Almost every modern Java codebase (including things like your GymPro services) uses streams and lambdas for filtering/transforming collections. It's the single most MCQ-dense topic in "Modern Java" sections.

---

## 9.2 Lambda Expressions

### What is it?
A lambda is an **anonymous function** — a block of code with no name, that you can pass around like a value.

**Before Java 8:**
```java
Runnable r = new Runnable() {
    public void run() {
        System.out.println("Running");
    }
};
```

**With Lambda:**
```java
Runnable r = () -> System.out.println("Running");
```

### Syntax
```
(parameters) -> expression
(parameters) -> { statements; }
```

| Case | Example |
|---|---|
| No parameters | `() -> System.out.println("Hi")` |
| One parameter (parentheses optional) | `x -> x * 2` |
| Multiple parameters | `(x, y) -> x + y` |
| Block body | `(x, y) -> { int sum = x+y; return sum; }` |

### Why lambdas work — the real reason
A lambda can ONLY be assigned to a **functional interface** (an interface with exactly one abstract method). The lambda body becomes the implementation of that one method. The compiler infers everything else via **target typing**.

**⚠️ Trap:** Lambda has no interface of its own — its "type" is decided entirely by the context (the variable/parameter type it's assigned to).

---

## 9.3 Functional Interfaces

### Definition
An interface with **exactly one abstract method** (it can have any number of default/static methods). Annotated with `@FunctionalInterface` (optional but recommended — compiler will error if you break the rule).

```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}
```

### Built-in Functional Interfaces (java.util.function) — MEMORIZE THIS TABLE

| Interface | Abstract Method | Input | Output | Use Case |
|---|---|---|---|---|
| `Function<T,R>` | `R apply(T t)` | T | R | Transform data |
| `Predicate<T>` | `boolean test(T t)` | T | boolean | Condition check |
| `Consumer<T>` | `void accept(T t)` | T | none | Perform action (e.g. print) |
| `Supplier<T>` | `T get()` | none | T | Generate/supply a value |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | T,U | R | Two-input transform |
| `UnaryOperator<T>` | `T apply(T t)` | T | T | Same-type transform |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | T,T | T | Combine two same-type values |

**Trick to remember:** 
- **Predicate** → "Predict true/false"
- **Consumer** → "Consumes input, gives nothing back" (like eating)
- **Supplier** → "Supplies output, takes nothing" (like a vending machine with no coin slot)
- **Function** → "Transforms input → output"

### Common Trap
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
Consumer<String> print = s -> System.out.println(s);
Supplier<String> greet = () -> "Hello";
Function<Integer, Integer> square = n -> n * n;
```
❌ Common mistake: writing `Predicate<Integer> p = n -> n;` — this fails to compile because `test()` must return `boolean`, not `int`.

---

## 9.4 Method References

A **shorthand** for a lambda that just calls one existing method.

| Type | Syntax | Example | Equivalent Lambda |
|---|---|---|---|
| Static method | `Class::staticMethod` | `Math::sqrt` | `x -> Math.sqrt(x)` |
| Instance method (particular object) | `obj::instanceMethod` | `str::toUpperCase` | `() -> str.toUpperCase()` |
| Instance method (arbitrary object of a type) | `Class::instanceMethod` | `String::toUpperCase` | `s -> s.toUpperCase()` |
| Constructor reference | `Class::new` | `ArrayList::new` | `() -> new ArrayList<>()` |

**Interviewer trap:** `String::toUpperCase` looks like a static reference but it's actually "instance method of an arbitrary object" — the first lambda parameter becomes the object the method is called on.

```java
List<String> names = Arrays.asList("john", "amy");
names.forEach(System.out::println);   // instance method reference on System.out
names.replaceAll(String::toUpperCase); // arbitrary-object instance method
```

---

## 9.5 Streams API

### What is a Stream?
A **pipeline of operations** on a source of data (collection, array, etc.) — it does NOT store data itself, and does NOT modify the source. Streams process elements **lazily** and can only be **consumed once**.

### Stream Pipeline = Source → Intermediate Operations → Terminal Operation

```java
List<String> names = Arrays.asList("Shreyas", "Amit", "Sara", "Zoe");

List<String> result = names.stream()
        .filter(n -> n.length() > 3)   // intermediate
        .map(String::toUpperCase)      // intermediate
        .sorted()                      // intermediate
        .collect(Collectors.toList()); // terminal
```

### Intermediate vs Terminal Operations

| Type | Behavior | Examples |
|---|---|---|
| Intermediate | Returns a new Stream, is **lazy** (not executed until terminal op runs) | `filter`, `map`, `sorted`, `distinct`, `limit`, `skip`, `peek` |
| Terminal | Triggers execution, produces a result or side-effect, stream is "consumed" | `collect`, `forEach`, `count`, `reduce`, `anyMatch`, `findFirst`, `toArray` |

**⚠️ Big Trap (frequently asked):**
```java
Stream<String> s = names.stream().filter(n -> n.length() > 3);
s.forEach(System.out::println);
s.forEach(System.out::println); // ❌ IllegalStateException: stream has already been operated upon or closed
```
A stream can be traversed **only once**.

### Laziness Trap
```java
Stream<Integer> s = Stream.of(1,2,3).filter(n -> {
    System.out.println("filtering " + n);
    return n > 1;
});
// Nothing printed yet! No terminal operation called.
s.forEach(System.out::println); // NOW filtering happens
```

### Common Stream Methods

| Method | Purpose |
|---|---|
| `filter(Predicate)` | Keep elements matching condition |
| `map(Function)` | Transform each element |
| `sorted()` / `sorted(Comparator)` | Sort elements |
| `distinct()` | Remove duplicates (uses `equals()`) |
| `limit(n)` | Take first n elements |
| `skip(n)` | Skip first n elements |
| `collect(Collectors.toList()/toSet()/toMap())` | Convert stream back to a collection |
| `reduce(identity, BinaryOperator)` | Combine elements into one result |
| `count()` | Number of elements |
| `anyMatch/allMatch/noneMatch(Predicate)` | Boolean checks |
| `findFirst()/findAny()` | Returns `Optional<T>` |

### `reduce()` — commonly confused
```java
int sum = Stream.of(1,2,3,4).reduce(0, (a,b) -> a+b); // 10
```
`0` is the identity (starting value); `(a,b)->a+b` combines accumulator with next element.

### Collectors — Frequently Asked
```java
List<String> list = stream.collect(Collectors.toList());
Set<String> set = stream.collect(Collectors.toSet());
String joined = stream.collect(Collectors.joining(", "));
Map<String,Integer> map = stream.collect(Collectors.toMap(s -> s, String::length));
Map<Boolean, List<String>> partition = stream.collect(Collectors.partitioningBy(s -> s.length() > 3));
Map<Integer, List<String>> grouped = stream.collect(Collectors.groupingBy(String::length));
```

### Stream vs Collection

| Feature | Collection | Stream |
|---|---|---|
| Storage | Stores data | No storage, pipeline only |
| Traversal | Multiple times | Only ONCE |
| Iteration | External (you write the loop) | Internal (stream handles it) |
| Modification | Can add/remove elements | Cannot modify source |
| Laziness | N/A | Intermediate ops are lazy |

### Sequential vs Parallel Streams
```java
list.stream()          // single thread
list.parallelStream()  // multiple threads (uses ForkJoinPool)
```
**Trap:** Parallel streams don't always improve performance — overhead can make small collections *slower*. Also, order is not guaranteed unless you use `forEachOrdered`.

---

## 9.6 Optional

### Why it exists
To avoid `NullPointerException` by explicitly representing "a value that might be absent."

```java
Optional<String> opt = Optional.of("Hello");       // must be non-null
Optional<String> empty = Optional.empty();
Optional<String> nullable = Optional.ofNullable(getName()); // can be null
```

### Common Methods

| Method | Behavior |
|---|---|
| `isPresent()` | true if value exists |
| `isEmpty()` (Java 11+) | true if value absent |
| `get()` | returns value, throws `NoSuchElementException` if empty — **avoid using directly** |
| `orElse(default)` | returns value or default |
| `orElseGet(Supplier)` | returns value or computes default lazily |
| `orElseThrow()` | throws exception if empty |
| `ifPresent(Consumer)` | runs code only if value present |
| `map(Function)` | transforms value if present |

**⚠️ Trap:** `Optional.of(null)` throws `NullPointerException` immediately. Use `Optional.ofNullable(null)` instead — that's safe and returns an empty Optional.

**Best Practice Trap:** Calling `.get()` without checking `isPresent()` defeats the whole purpose of Optional — Capgemini loves asking "what's wrong with this code" questions here.

---

## 9.7 Date and Time API (java.time)

Java 8 replaced the old buggy `Date`/`Calendar` classes with an **immutable, thread-safe** API.

| Class | Represents |
|---|---|
| `LocalDate` | Date only (yyyy-MM-dd) |
| `LocalTime` | Time only (HH:mm:ss) |
| `LocalDateTime` | Date + Time, no timezone |
| `ZonedDateTime` | Date + Time + Timezone |
| `Period` | Difference in years/months/days |
| `Duration` | Difference in hours/minutes/seconds |
| `DateTimeFormatter` | Formatting/parsing dates |

```java
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(2000, 5, 15);
Period p = Period.between(birthday, today);

LocalDateTime dt = LocalDateTime.now();
Duration d = Duration.ofMinutes(90);
```

**Key trap:** All `java.time` classes are **immutable** — methods like `plusDays()` return a NEW object, they don't modify the original.
```java
LocalDate d1 = LocalDate.now();
d1.plusDays(5);       // ❌ does nothing to d1 — return value discarded!
LocalDate d2 = d1.plusDays(5); // ✅ correct
```

---

## 9.8 Default and Static Methods in Interfaces

Java 8 allowed interfaces to have method **bodies** (before this, interfaces could only have abstract methods).

```java
interface Vehicle {
    void drive(); // abstract

    default void honk() {           // default method
        System.out.println("Beep!");
    }

    static Vehicle create() {       // static method
        return () -> System.out.println("Driving");
    }
}
```

**Why default methods were added:** To add new methods to existing interfaces (like `List`, `Map`) **without breaking** all classes that already implement them. This is called **interface evolution**.

### Diamond Problem with Default Methods
If a class implements two interfaces with the same default method, it's ambiguous — the class MUST override the method to resolve it.
```java
interface A { default void show() { System.out.println("A"); } }
interface B { default void show() { System.out.println("B"); } }
class C implements A, B {
    public void show() {   // ✅ MUST override — otherwise compile error
        A.super.show();    // explicitly calling A's version
    }
}
```

---

## 9.9 One-Page Revision — Java 8 Features

| Concept | Key Point |
|---|---|
| Lambda | Anonymous function; needs a functional interface target |
| Functional Interface | Exactly 1 abstract method; `@FunctionalInterface` |
| Predicate/Function/Consumer/Supplier | test/apply/accept/get |
| Method Reference | Shorthand for lambda calling existing method (`Class::method`) |
| Stream | Pipeline; not storage; lazy intermediate ops; single-use |
| Intermediate ops | filter, map, sorted, distinct — lazy, return Stream |
| Terminal ops | collect, forEach, reduce, count — trigger execution |
| Optional | Wrapper to avoid NPE; use `ofNullable`, avoid raw `.get()` |
| java.time | Immutable, thread-safe; `plusDays()` returns new object |
| default/static in interfaces | Enables interface evolution without breaking implementers |

---

## 9.10 MCQ Practice — Java 8 Features

**Difficulty distribution below: 12 Easy, 16 Medium, 12 Hard (40 total for this batch)**

### EASY

**Q1.** What is a lambda expression in Java?
A) A named function B) An anonymous function C) A class D) A loop
**Answer:** B
**Explanation:** A lambda is a function without a name that can be treated as a value.
**Why others wrong:** A) Named functions are regular methods. C) It's not a class definition. D) Unrelated to loops.
**Difficulty:** Easy | **Topic:** Lambda

---

**Q2.** How many abstract methods can a functional interface have?
A) 0 B) 1 C) 2 D) Unlimited
**Answer:** B
**Explanation:** Exactly one abstract method is the defining rule of a functional interface (default/static methods don't count).
**Why others wrong:** A) Zero abstract methods means it's not functional (no method to implement via lambda). C, D) Would violate the single-method contract, causing a compile error with `@FunctionalInterface`.
**Difficulty:** Easy | **Topic:** Functional Interface

---

**Q3.** Which functional interface returns a boolean?
A) Function B) Consumer C) Predicate D) Supplier
**Answer:** C
**Explanation:** `Predicate<T>` has method `boolean test(T t)`.
**Why others wrong:** A) Function returns R (generic type). B) Consumer returns void. D) Supplier returns T, takes no input.
**Difficulty:** Easy | **Topic:** Functional Interfaces

---

**Q4.** Which of these correctly creates an empty Optional?
A) `Optional.of(null)` B) `Optional.empty()` C) `new Optional()` D) `Optional.null()`
**Answer:** B
**Explanation:** `Optional.empty()` is the standard factory method for an Optional with no value.
**Why others wrong:** A) Throws NullPointerException. C) Optional has no public constructor. D) Not a valid method.
**Difficulty:** Easy | **Topic:** Optional

---

**Q5.** What does a Stream's `filter()` method return?
A) A boolean B) A List C) A new Stream D) void
**Answer:** C
**Explanation:** `filter()` is an intermediate operation, so it returns a new Stream.
**Why others wrong:** A) That's what a Predicate returns internally, not filter() itself. B) Only `collect()` converts to a List. D) filter is not a terminal/void operation.
**Difficulty:** Easy | **Topic:** Streams

---

**Q6.** Which class represents date without time in java.time?
A) LocalTime B) LocalDate C) LocalDateTime D) Instant
**Answer:** B
**Explanation:** `LocalDate` stores only year-month-day, no time component.
**Why others wrong:** A) Time only, no date. C) Both date and time. D) A machine timestamp, not human-readable date.
**Difficulty:** Easy | **Topic:** Date-Time API

---

**Q7.** Which operator symbol is used in lambda expressions?
A) `->` B) `=>` C) `::` D) `-->`
**Answer:** A
**Explanation:** The arrow token `->` separates parameters from the body.
**Why others wrong:** B) Used in other languages, not Java. C) That's the method reference operator. D) Not valid Java syntax.
**Difficulty:** Easy | **Topic:** Lambda Syntax

---

**Q8.** What is `String::toUpperCase` an example of?
A) Constructor reference B) Static method reference C) Instance method reference of an arbitrary object D) Instance method reference of a particular object
**Answer:** C
**Explanation:** Since `toUpperCase()` is called on the first lambda argument (any String instance), it's an "arbitrary object" reference.
**Why others wrong:** A) Constructor references use `Class::new`. B) toUpperCase is not static in String. D) A "particular object" reference uses a specific instance like `str::toUpperCase`.
**Difficulty:** Easy | **Topic:** Method References

---

**Q9.** Which annotation is recommended (not mandatory) for functional interfaces?
A) `@Override` B) `@FunctionalInterface` C) `@Functional` D) `@Lambda`
**Answer:** B
**Explanation:** `@FunctionalInterface` triggers a compile-time check ensuring only one abstract method exists.
**Why others wrong:** A) Used for overriding methods, unrelated. C, D) Not real Java annotations.
**Difficulty:** Easy | **Topic:** Functional Interface

---

**Q10.** What does `stream.count()` return?
A) A Stream B) An int C) A long D) A boolean
**Answer:** C
**Explanation:** `count()` returns a `long` representing the number of elements.
**Why others wrong:** A) It's a terminal operation, doesn't return a Stream. B) Return type is long, not int. D) Not a boolean-returning method.
**Difficulty:** Easy | **Topic:** Streams

---

**Q11.** Which method converts a Stream back into a List?
A) `toList()` directly on Stream in older Java only B) `collect(Collectors.toList())` C) `asList()` D) `stream.list()`
**Answer:** B
**Explanation:** `collect(Collectors.toList())` is the standard, portable way to gather stream elements into a List.
**Why others wrong:** A) `stream.toList()` exists only from Java 16+, not part of core Java 8. C) `asList()` belongs to `Arrays`, not Stream. D) Not a valid method.
**Difficulty:** Easy | **Topic:** Streams/Collectors

---

**Q12.** In `Function<T,R>`, what does R represent?
A) Return type B) Recursive type C) Reference type D) Random type
**Answer:** A
**Explanation:** R is the output/return type produced by `apply()`.
**Why others wrong:** B, C, D) Not standard Java generics naming conventions.
**Difficulty:** Easy | **Topic:** Functional Interfaces

---

### MEDIUM

**Q13.** What will this print?
```java
Predicate<Integer> isPositive = n -> n > 0;
System.out.println(isPositive.test(-5));
```
A) true B) false C) Compile error D) Runtime exception
**Answer:** B
**Explanation:** -5 is not greater than 0, so `test()` evaluates to false.
**Why others wrong:** A) Would only be true for positive numbers. C, D) Code is syntactically and logically valid.
**Difficulty:** Medium | **Topic:** Predicate

---

**Q14.** What is wrong with this code?
```java
Stream<Integer> s = Stream.of(1,2,3);
long count = s.count();
s.forEach(System.out::println);
```
A) Nothing, it works fine B) `count()` returns int not long, compile error C) Stream already consumed, throws IllegalStateException at forEach D) forEach must come before count
**Answer:** C
**Explanation:** Once a terminal operation (`count()`) runs, the stream is considered "closed"; reusing it in `forEach` throws `IllegalStateException`.
**Why others wrong:** A) It will throw at runtime. B) count() correctly returns long, no compile error there. D) Order doesn't matter — the issue is reuse, not sequence.
**Difficulty:** Medium | **Topic:** Streams

---

**Q15.** What does this output?
```java
List<Integer> nums = Arrays.asList(1,2,3,4,5);
int sum = nums.stream().reduce(0, Integer::sum);
System.out.println(sum);
```
A) 0 B) 15 C) Compile error D) 5
**Answer:** B
**Explanation:** `reduce` starts at identity 0 and adds each element: 0+1+2+3+4+5 = 15.
**Why others wrong:** A) That's just the identity, ignoring the actual sum. C) Valid, compiles fine. D) That's the count of elements, not the sum.
**Difficulty:** Medium | **Topic:** Streams - reduce

---

**Q16.** Which statement about `Optional.get()` is TRUE?
A) It's always safe to call B) It throws `NoSuchElementException` if the Optional is empty C) It returns null if empty D) It automatically wraps in try-catch
**Answer:** B
**Explanation:** Calling `.get()` on an empty Optional throws `NoSuchElementException` — that's why it should be guarded with `isPresent()` or replaced with `orElse()`.
**Why others wrong:** A) Unsafe if not checked first. C) It throws, doesn't silently return null. D) Java doesn't auto-wrap exception handling.
**Difficulty:** Medium | **Topic:** Optional

---

**Q17.** What will this print?
```java
LocalDate date = LocalDate.of(2024, 1, 1);
date.plusDays(10);
System.out.println(date);
```
A) 2024-01-11 B) 2024-01-01 C) Compile error D) null
**Answer:** B
**Explanation:** `LocalDate` is immutable; `plusDays(10)` returns a new object, but since it's not assigned back, `date` is unchanged.
**Why others wrong:** A) Would be correct only if reassigned: `date = date.plusDays(10)`. C, D) No compile error, and date isn't null.
**Difficulty:** Medium | **Topic:** Date-Time API (immutability trap)

---

**Q18.** Which grouping produces `Map<Integer, List<String>>` keyed by string length?
A) `Collectors.toMap(String::length, s -> s)` B) `Collectors.groupingBy(String::length)` C) `Collectors.partitioningBy(String::length)` D) `Collectors.joining()`
**Answer:** B
**Explanation:** `groupingBy` classifies elements into groups by a classifier function, producing a `Map<K, List<T>>`.
**Why others wrong:** A) toMap maps each key to a SINGLE value, not a list — would throw `IllegalStateException` on duplicate keys. C) partitioningBy only works with a boolean Predicate, not length (int). D) joining just merges strings, doesn't group.
**Difficulty:** Medium | **Topic:** Collectors

---

**Q19.** Which is the correct order of execution for this pipeline?
```java
list.stream().filter(x -> x > 2).map(x -> x * 2).collect(Collectors.toList());
```
A) All filters run first on entire list, then all maps run B) Each element flows through filter then map one at a time (element-by-element) C) map runs before filter internally D) Order is undefined
**Answer:** B
**Explanation:** Java streams process elements one at a time through the whole pipeline (like an assembly line) rather than doing one operation across the whole collection before the next.
**Why others wrong:** A) This describes how a Collection-based loop-per-stage approach works, not Streams. C) The pipeline as coded runs filter first per element. D) Order is well-defined for sequential streams.
**Difficulty:** Medium | **Topic:** Stream internals

---

**Q20.** What's the output?
```java
Supplier<String> s = () -> "Hello";
System.out.println(s.get());
```
A) Hello B) Compile error, Supplier needs an argument C) null D) get() 
**Answer:** A
**Explanation:** `Supplier<T>` takes no arguments and returns a value via `get()`.
**Why others wrong:** B) Supplier is defined to take zero arguments — that's correct usage. C) Not null, it returns "Hello". D) Not literal method name output.
**Difficulty:** Medium | **Topic:** Supplier

---

**Q21.** Which best describes `Comparator.comparing(String::length).thenComparing(Comparator.naturalOrder())`?
A) Sorts only by length B) Sorts by length, then alphabetically for equal lengths C) Sorts alphabetically only D) Compile error
**Answer:** B
**Explanation:** `thenComparing` provides a secondary sort criterion applied when the primary comparator considers two elements equal.
**Why others wrong:** A) Ignores the chained comparator. C) Length is still the primary sort key. D) Valid, standard Comparator chaining syntax.
**Difficulty:** Medium | **Topic:** Comparator + Streams

---

**Q22.** What does `IntStream.rangeClosed(1, 5)` produce?
A) 1,2,3,4 B) 1,2,3,4,5 C) 0,1,2,3,4,5 D) 2,3,4,5
**Answer:** B
**Explanation:** `rangeClosed` includes both endpoints, so 1 through 5 inclusive.
**Why others wrong:** A) That's what plain `range(1,5)` (exclusive end) gives. C) Doesn't start at 0 here. D) Doesn't skip the start value.
**Difficulty:** Medium | **Topic:** IntStream

---

**Q23.** Which interface would you use to combine two Integers into one Integer?
A) Function<Integer, Integer> B) BinaryOperator<Integer> C) Consumer<Integer> D) Supplier<Integer>
**Answer:** B
**Explanation:** `BinaryOperator<T>` takes two arguments of the same type and returns the same type — perfect for combining values (used internally by `reduce`).
**Why others wrong:** A) Function takes one input. C) Consumer takes input but returns nothing. D) Supplier takes no input.
**Difficulty:** Medium | **Topic:** Functional Interfaces

---

**Q24.** What is the purpose of `peek()` in a stream pipeline?
A) Terminal operation to end the stream B) Intermediate operation mainly for debugging, doesn't change elements C) Removes duplicate elements D) Sorts the stream
**Answer:** B
**Explanation:** `peek()` lets you observe elements as they flow through (e.g., for logging) without altering them, and is intermediate (lazy).
**Why others wrong:** A) It's intermediate, not terminal. C) That's `distinct()`. D) That's `sorted()`.
**Difficulty:** Medium | **Topic:** Streams

---

**Q25.** Which statement about default methods is TRUE?
A) They must be static B) They provide a body inside an interface C) They cannot be overridden D) They are abstract
**Answer:** B
**Explanation:** Default methods have implementation bodies directly in the interface, enabling interface evolution.
**Why others wrong:** A) Default methods are instance-level, called on objects, unlike static methods. C) Implementing classes CAN override them. D) They are the opposite of abstract — they have implementation.
**Difficulty:** Medium | **Topic:** Interface default methods

---

**Q26.** What happens when a class implements two interfaces having the same default method signature and doesn't override it?
A) Runs interface A's version automatically B) Runs interface B's version automatically C) Compile error — ambiguous, must override D) Runtime exception
**Answer:** C
**Explanation:** Java cannot decide which default implementation to use, so it forces the developer to explicitly override and resolve the conflict.
**Why others wrong:** A, B) Java doesn't guess/prioritize either interface automatically. D) The error is caught at compile time, not runtime.
**Difficulty:** Medium | **Topic:** Interface diamond problem

---

**Q27.** What is the correct way to safely get a value or a default from Optional?
A) `opt.get() != null ? opt.get() : "default"` B) `opt.orElse("default")` C) `if(opt == null) "default"` D) `opt.getOrDefault("default")`
**Answer:** B
**Explanation:** `orElse()` is the idiomatic, null-safe way to provide a fallback value.
**Why others wrong:** A) Calling `.get()` on empty Optional throws before the null-check ever happens. C) Optional reference itself is never null (use isPresent/isEmpty instead). D) Not a real Optional method (that's a Map method).
**Difficulty:** Medium | **Topic:** Optional

---

**Q28.** Given `list.parallelStream().forEach(System.out::println);` on `[1,2,3,4,5]`, what can we say about the output order?
A) Always printed in order 1-5 B) Order is not guaranteed C) Always reverse order D) Throws an exception
**Answer:** B
**Explanation:** Parallel streams process elements across multiple threads, so completion (and print) order isn't guaranteed.
**Why others wrong:** A, C) Neither strict order is guaranteed with parallel execution. D) No exception — it's valid code, just unordered.
**Difficulty:** Medium | **Topic:** Parallel Streams

---

### HARD

**Q29.** What is the output?
```java
List<Integer> nums = Arrays.asList(5, 3, 8, 1);
Optional<Integer> max = nums.stream().max(Integer::compareTo);
System.out.println(max.get());
```
A) 1 B) 8 C) 5 D) Compile error
**Answer:** B
**Explanation:** `max()` with `Integer::compareTo` finds the largest element in natural order, which is 8.
**Why others wrong:** A) That's the minimum, not max. C) That's just the first element. D) Code is valid and compiles correctly.
**Difficulty:** Hard | **Topic:** Streams - max/min

---

**Q30.** What does this print?
```java
Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> square = x -> x * x;
Function<Integer, Integer> combo = addOne.andThen(square);
System.out.println(combo.apply(2));
```
A) 5 B) 9 C) 6 D) 4
**Answer:** B
**Explanation:** `andThen` applies `addOne` first (2+1=3), then feeds that result into `square` (3*3=9).
**Why others wrong:** A) Would be the result of only addOne applied twice incorrectly. C) Doesn't match either function's math. D) That's just square(2) alone, skipping addOne.
**Difficulty:** Hard | **Topic:** Function composition (andThen/compose)

---

**Q31.** What's the difference in output if we use `compose` instead of `andThen` in Q30 (`addOne.compose(square)`, called with `apply(2)`)?
A) Same result, 9 B) 5, because square runs first (2*2=4), then addOne (4+1=5) C) 4, no change D) Compile error
**Answer:** B
**Explanation:** `compose` reverses the order: the argument function (`square`) runs FIRST, then the caller (`addOne`) runs on that result. So square(2)=4, then addOne(4)=5.
**Why others wrong:** A) Order is reversed compared to andThen, so result differs. C) Ignores addOne entirely. D) Valid, compiles fine.
**Difficulty:** Hard | **Topic:** Function composition (compose vs andThen — frequently confused)

---

**Q32.** What will happen here?
```java
List<String> list = new ArrayList<>(Arrays.asList("a","b","c"));
list.stream().forEach(s -> list.remove(s));
```
A) Removes all elements safely B) Throws ConcurrentModificationException C) Compile error D) Works fine because stream copies the list
**Answer:** B
**Explanation:** Modifying the backing collection (`list.remove`) while a stream is actively iterating over it triggers `ConcurrentModificationException`, just like doing this with a regular iterator.
**Why others wrong:** A) It doesn't complete safely — it throws mid-iteration. C) It's a runtime issue, not a compile-time error. D) Streams don't copy the source collection; they operate on it directly.
**Difficulty:** Hard | **Topic:** Streams + Collection modification trap

---

**Q33.** What is the result?
```java
Stream<String> s = Stream.of("a","bb","ccc");
int total = s.mapToInt(String::length).sum();
System.out.println(total);
```
A) 3 B) 6 C) Compile error, mapToInt not valid on Stream<String> D) 5
**Answer:** B
**Explanation:** `mapToInt` converts to an `IntStream` using each string's length (1+2+3=6), and `sum()` adds them.
**Why others wrong:** A) That's just the count of elements, not sum of lengths. C) `mapToInt` is a valid Stream method for converting to a primitive int stream. D) Doesn't match the actual sum (1+2+3=6, not 5).
**Difficulty:** Hard | **Topic:** Stream to IntStream conversion

---

**Q34.** Which statement about `Collectors.toMap()` is TRUE when duplicate keys occur?
A) It silently keeps the first value B) It silently keeps the last value C) It throws `IllegalStateException` unless a merge function is provided D) It creates a List as the value automatically
**Answer:** C
**Explanation:** Without a merge function, `toMap()` throws `IllegalStateException: Duplicate key` when two elements map to the same key — you must supply a third argument (merge function) to resolve it.
**Why others wrong:** A, B) It doesn't silently pick one — it throws unless handled. D) Auto-listing values is the behavior of `groupingBy`, not `toMap`.
**Difficulty:** Hard | **Topic:** Collectors.toMap edge case

---

**Q35.** What does this output?
```java
Optional<String> opt = Optional.ofNullable(null);
String result = opt.map(String::toUpperCase).orElse("EMPTY");
System.out.println(result);
```
A) null B) NullPointerException C) EMPTY D) Compile error
**Answer:** C
**Explanation:** `ofNullable(null)` creates an empty Optional; `map()` on an empty Optional just returns empty (skips the function), and `orElse("EMPTY")` supplies the fallback.
**Why others wrong:** A) It never prints literal null — orElse kicks in. B) map() safely short-circuits on empty Optional, no exception thrown. D) Fully valid, compiles correctly.
**Difficulty:** Hard | **Topic:** Optional.map chaining

---

**Q36.** What's wrong (if anything) with this default method usage?
```java
interface Shape {
    default double area() { return 0; }
}
class Circle implements Shape {
    // no override
}
Shape s = new Circle();
System.out.println(s.area());
```
A) Compile error, Circle must implement area() B) Prints 0.0, since Circle inherits the default implementation C) Runtime exception D) Prints nothing
**Answer:** B
**Explanation:** Since there's only ONE default method (no conflict), Circle can simply inherit it without overriding — legal and common.
**Why others wrong:** A) Overriding is optional when there's no ambiguity. C) No exception; area() executes fine returning 0.0. D) It does print a value (0.0).
**Difficulty:** Hard | **Topic:** Default methods (non-diamond case)

---

**Q37.** Given `IntStream.range(1, 5).boxed().collect(Collectors.toList())`, what is the result?
A) [1, 2, 3, 4] B) [1, 2, 3, 4, 5] C) [0, 1, 2, 3, 4] D) Compile error, boxed() doesn't exist
**Answer:** A
**Explanation:** `range(1,5)` is exclusive of the upper bound, producing 1,2,3,4; `boxed()` converts `int` primitives to `Integer` objects so `collect` can build a `List<Integer>`.
**Why others wrong:** B) Would require `rangeClosed`. C) Doesn't start at 0 here. D) `boxed()` is a real, commonly-used IntStream method.
**Difficulty:** Hard | **Topic:** IntStream + boxed()

---

**Q38.** What is the output?
```java
List<Integer> nums = Arrays.asList(1,2,3,4,5,6);
long evenCount = nums.stream().filter(n -> n % 2 == 0).count();
System.out.println(evenCount);
```
A) 3 B) 6 C) 2 D) Compile error, count() needs an argument
**Answer:** A
**Explanation:** Even numbers in the list are 2, 4, 6 → count = 3.
**Why others wrong:** B) That's the total list size, not just evens. C) Undercounts — misses one even number. D) `count()` takes no arguments; it's a no-arg terminal operation.
**Difficulty:** Hard | **Topic:** Streams filter+count

---

**Q39.** Why does this fail to compile?
```java
Function<Integer, Integer> f = (Integer x, int y) -> x + y;
```
A) Lambdas can't mix parameter type styles B) Function only accepts one parameter, but this lambda has two C) int can't be added to Integer D) Missing return statement
**Answer:** B
**Explanation:** `Function<T,R>`'s abstract method `apply(T t)` takes exactly ONE parameter — this lambda supplies two, causing a mismatch with the functional interface's method signature.
**Why others wrong:** A) Actually a separate real rule (you can't mix explicit and inferred types among parameters), but the primary/decisive issue here is the wrong parameter COUNT for Function. C) Integer auto-unboxes fine in arithmetic with int. D) Expression-bodied lambdas don't need explicit return.
**Difficulty:** Hard | **Topic:** Functional interface method signature matching

---

**Q40.** What does this print?
```java
List<String> names = Arrays.asList("Tom", "Jerry", "Spike");
String result = names.stream()
    .filter(n -> n.length() > 3)
    .findFirst()
    .orElse("None");
System.out.println(result);
```
A) Tom B) Jerry C) None D) Spike
**Answer:** B
**Explanation:** Filtering keeps names longer than 3 chars: "Jerry" (5) and "Spike" (5); `findFirst()` returns the first one encountered in stream order, which is "Jerry".
**Why others wrong:** A) "Tom" has only 3 letters, filtered out. C) A match does exist, so orElse's fallback isn't triggered. D) "Spike" matches too but comes after "Jerry" in the list order.
**Difficulty:** Hard | **Topic:** Streams filter+findFirst

---

## Score Tracker
Out of 40: ___ / 40

- 36+ correct → You're exam-ready for this chapter ✅
- 28–35 → Solid, revise the traps marked ⚠️ above
- Below 28 → Re-read sections 9.3, 9.5, 9.6 before moving on

---

**Tell me your score and which question numbers you got wrong**, and I'll re-explain those concepts before we move to **Chapter 10: Multithreading, Synchronization & Executor Framework**.

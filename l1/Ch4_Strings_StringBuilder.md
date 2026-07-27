# JAVA CORE — Chapter 4: Strings, StringBuffer, StringBuilder

## 1. String Immutability — THE core concept
A `String` object, once created, can never be changed. Any operation that looks like "modifying" a String actually creates a **new** String object.

```java
String s = "Hello";
s.concat(" World");   // creates a new String but doesn't change s
System.out.println(s); // still prints "Hello"

s = s.concat(" World"); // now s is reassigned to the new object
System.out.println(s);  // "Hello World"
```

## 2. String Pool (String Constant Pool / SCP)
- String literals (`"abc"`) are stored in a special memory area called the **String Pool** (inside Heap since Java 7+).
- If a literal with the same value already exists, Java reuses it instead of creating a new object.

```java
String a = "hello";
String b = "hello";
System.out.println(a == b); // TRUE — both point to same pooled object

String c = new String("hello");
System.out.println(a == c); // FALSE — new String() forces heap allocation, bypasses pool
System.out.println(a.equals(c)); // TRUE — equals() compares content
```

**Capgemini trap:** `new String("x")` always creates a new object outside the pool (or at least a new reference), even if `"x"` already exists in the pool. Use `.intern()` to force it into the pool.

## 3. String vs StringBuffer vs StringBuilder — Comparison table (heavily tested)

| Feature | String | StringBuffer | StringBuilder |
|---|---|---|---|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safety | N/A (immutable, safe) | **Synchronized (thread-safe)** | **Not synchronized (not thread-safe)** |
| Performance | Slow for repeated modification | Slower (sync overhead) | **Fastest** |
| Introduced | Java 1.0 | Java 1.0 | Java 5 |
| Use case | Fixed/rarely-changed text | Multi-threaded string building | Single-threaded string building |

**Trap:** "Which is faster, StringBuffer or StringBuilder?" → **StringBuilder** (no synchronization overhead). "Which is thread-safe?" → **StringBuffer**.

## 4. Why String Concatenation in Loops is Bad
```java
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // creates a NEW String object every single iteration!
}
```
Each `+=` creates a new String object (since String is immutable), wasting memory and time — O(n²) behavior. Correct approach: use `StringBuilder`.
```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i); // modifies the SAME internal buffer, no new object
}
String result = sb.toString();
```

## 5. Important String Methods (know exact behavior)

| Method | Behavior |
|---|---|
| `length()` | Returns number of characters (method, with parens) |
| `charAt(i)` | Character at index i |
| `substring(begin)` | From begin to end |
| `substring(begin, end)` | begin inclusive, end **exclusive** |
| `equals()` | Content comparison |
| `equalsIgnoreCase()` | Content comparison, ignoring case |
| `compareTo()` | Lexicographic comparison, returns int |
| `indexOf()` | First occurrence index, -1 if not found |
| `trim()` / `strip()` | Removes leading/trailing whitespace |
| `replace()` | Replaces chars/substrings |
| `split()` | Splits into String[] by regex |
| `toUpperCase()`/`toLowerCase()` | Case conversion, returns new String |
| `intern()` | Forces string into the String Pool |

**Trap — substring bounds:**
```java
String s = "HelloWorld";
System.out.println(s.substring(0, 5)); // "Hello" — index 5 is EXCLUDED
```

## 6. StringBuffer / StringBuilder Key Methods
`append()`, `insert()`, `delete()`, `deleteCharAt()`, `reverse()`, `replace()`, `capacity()`.

**Trap:** `reverse()` exists on StringBuilder/StringBuffer but NOT on String directly — you must convert first: `new StringBuilder(str).reverse().toString()`.

## One-Page Revision
- String is immutable; every "modification" creates a new object.
- String literals go in the String Pool; `new String()` bypasses it and creates a separate heap object.
- `==` compares references; `.equals()` compares content. Always use `.equals()` for String value comparison.
- StringBuffer = mutable + thread-safe (synchronized) = slower.
- StringBuilder = mutable + NOT thread-safe = faster. Prefer StringBuilder unless multithreading requires safety.
- Looping with `String +=` is inefficient (O(n²)); use StringBuilder.append() instead.
- `substring(begin, end)`: end index is exclusive.
- No `.reverse()` on String directly — only on StringBuilder/StringBuffer.

---

# MCQs — Chapter 4 (20 Questions)

**Q1.** [Easy | Immutability] What happens when you call `.concat()` on a String without reassigning it?
A) The original String changes B) A new String is created but the original variable is unaffected C) Compile error D) Runtime exception
**Answer: B**
*Explanation:* Strings are immutable — `.concat()` returns a new String object; without reassignment, the original reference still points to the unchanged original.
*Why others wrong:* A) Strings cannot be changed in place, ever. C, D) Perfectly valid, non-error-throwing code.

**Q2.** [Easy | Pool] What does this print?
```java
String a = "test";
String b = "test";
System.out.println(a == b);
```
A) true B) false C) Compile error D) Depends on JVM
**Answer: A**
*Explanation:* Both literals reference the same object in the String Pool since they have identical content.
*Why others wrong:* B) Would only be true for `new String()` cases. C, D) Deterministic, valid, compiling behavior — not JVM-dependent.

**Q3.** [Medium | new String] What does this print?
```java
String a = "test";
String b = new String("test");
System.out.println(a == b);
System.out.println(a.equals(b));
```
A) true, true B) false, true C) true, false D) false, false
**Answer: B**
*Explanation:* `new String()` forces creation of a separate object outside/independent from the pool reference, so `==` is false; but `.equals()` compares content, which is identical, so true.
*Why others wrong:* A) `==` is false here, not true. C) `.equals()` is true, not false, since content matches. D) `.equals()` correctly returns true for matching content.

**Q4.** [Medium | Comparison] Which of these correctly compares Strings by VALUE?
A) `==` B) `.equals()` C) `.hashCode() == hashCode()` D) `.compareTo() == null`
**Answer: B**
*Explanation:* `.equals()` is explicitly overridden in String to compare character content, unlike `==` which compares object references.
*Why others wrong:* A) Compares references, unreliable for value comparison across different objects. C) Not idiomatic and can theoretically have edge-case collisions, not the standard approach. D) `compareTo` returns int, not usable this way — nonsensical syntax.

**Q5.** [Medium | Table] Which class provides thread-safe, synchronized string modification?
A) String B) StringBuilder C) StringBuffer D) StringPool
**Answer: C**
*Explanation:* StringBuffer's methods are synchronized, making it safe for concurrent access by multiple threads (at the cost of performance).
*Why others wrong:* A) Immutable, not about thread-safety of modification since it can't be modified. B) Explicitly NOT synchronized — faster but not thread-safe. D) Not a real Java class.

**Q6.** [Medium | Performance] Which is generally FASTER for single-threaded string-building operations?
A) StringBuffer B) StringBuilder C) String with += D) They're identical in performance
**Answer: B**
*Explanation:* StringBuilder lacks synchronization overhead present in StringBuffer, making it faster when thread-safety isn't required.
*Why others wrong:* A) Synchronization overhead makes it slower than StringBuilder. C) String concatenation via += creates new objects repeatedly — the slowest of the three for repeated modification. D) Clear performance differences exist; they are not identical.

**Q7.** [Hard | substring] What does this print?
```java
String s = "HelloWorld";
System.out.println(s.substring(2, 5));
```
A) "llo" B) "lloW" C) "elloW" D) "Hell"
**Answer: A**
*Explanation:* substring(2,5) takes characters starting at index 2 up to (but not including) index 5: indices 2,3,4 = 'l','l','o' = "llo".
*Why others wrong:* B, C, D) Incorrectly include or exclude boundary characters, misapplying the exclusive end-index rule.

**Q8.** [Medium | Loop inefficiency] Why is `String += ` inside a loop considered inefficient?
A) It throws exceptions frequently B) It creates a new String object on every iteration due to immutability C) It requires explicit garbage collection calls D) It only works with StringBuilder internally
**Answer: B**
*Explanation:* Every concatenation allocates a brand-new String object (since Strings can't be modified in place), leading to O(n²) time and memory churn across many iterations.
*Why others wrong:* A) No exceptions are thrown; it's a performance issue, not a correctness one. C) Java's GC is automatic, never requires manual invocation, and isn't the root inefficiency cause. D) Under the hood javac DOES sometimes use StringBuilder for compile-time constant concatenation in simple cases, but repeated loop concatenation on a variable is still inefficient per-iteration object creation.

**Q9.** [Hard | reverse()] Which of these correctly reverses a String?
A) `str.reverse()` B) `new StringBuilder(str).reverse().toString()` C) `Collections.reverse(str)` D) `str.flip()`
**Answer: B**
*Explanation:* String has no `.reverse()` method; you must wrap it in a StringBuilder, call reverse(), then convert back with toString().
*Why others wrong:* A) String doesn't have this method — compile error. C) Collections.reverse() works on List, not String. D) Not a real Java method.

**Q10.** [Medium | intern] What does `.intern()` do?
A) Converts a String to lowercase B) Forces a String into the String Pool, returning the pooled reference C) Deletes a String from memory D) Compares two strings for equality
**Answer: B**
*Explanation:* `.intern()` checks if an equal String already exists in the pool; if so, returns that reference, otherwise adds this String to the pool and returns it.
*Why others wrong:* A) Unrelated to case conversion. C) No explicit deletion mechanism like this exists for Strings. D) That's `.equals()`'s job, not `.intern()`.

**Q11.** [Hard | Code Trace] What does this print?
```java
String a = new String("java");
String b = a.intern();
String c = "java";
System.out.println(b == c);
```
A) true B) false C) Compile error D) Runtime exception
**Answer: A**
*Explanation:* `.intern()` returns the pooled reference for "java" (creating it in the pool if not present), which is the SAME object that literal `c` points to.
*Why others wrong:* B) Would be true only WITHOUT calling intern() (comparing `a == c` directly). C, D) Fully valid, non-throwing code.

**Q12.** [Medium | StringBuffer methods] Which StringBuffer method removes a character at a specific index?
A) `remove(index)` B) `deleteCharAt(index)` C) `pop(index)` D) `trim(index)`
**Answer: B**
*Explanation:* `deleteCharAt(int index)` is the correct StringBuffer/StringBuilder API method for single-character removal.
*Why others wrong:* A, C) Not real methods on StringBuffer. D) `trim()` exists but only removes leading/trailing whitespace, takes no index argument.

**Q13.** [Easy | length] For `String s = "Capgemini";`, what does `s.length()` return?
A) 8 B) 9 C) 10 D) Compile error, should be `s.length`
**Answer: B**
*Explanation:* "Capgemini" has 9 characters (C-a-p-g-e-m-i-n-i); String's length() is a proper method call with parentheses.
*Why others wrong:* A, C) Simple miscounts of the actual character count. D) String correctly uses `.length()` with parens (unlike arrays, which use `.length` without parens) — this is the reverse of the Chapter 3 array trap.

**Q14.** [Hard | Equality trap] What does this print?
```java
String a = "Hello";
String b = "Hel" + "lo"; // compile-time constant expression
System.out.println(a == b);
```
A) true B) false C) Compile error D) NullPointerException
**Answer: A**
*Explanation:* Since both operands of `+` are compile-time constants, the compiler folds them into a single literal `"Hello"` at compile time, which resolves to the same pooled object as `a`.
*Why others wrong:* B) This would be true for runtime-computed concatenation (e.g., using a variable), but here it's a compile-time constant, so it's actually interned automatically. C, D) Valid, non-throwing code.

**Q15.** [Hard | Equality trap contrast] What does this print?
```java
String x = "Hel";
String a = "Hello";
String b = x + "lo"; // x is a variable, NOT a compile-time constant
System.out.println(a == b);
```
A) true B) false C) Compile error D) NullPointerException
**Answer: B**
*Explanation:* Since `x` is a variable (not a compile-time constant), the concatenation happens at RUNTIME, producing a new String object outside the pool — so `==` fails even though content matches.
*Why others wrong:* A) This is the key contrast to Q14 — runtime concatenation does NOT get automatically interned. C, D) Valid, non-throwing code — just a reference inequality, not an error.

**Q16.** [Medium | compareTo] What does `"apple".compareTo("banana")` return?
A) A positive number B) A negative number C) 0 D) Throws exception
**Answer: B**
*Explanation:* compareTo does lexicographic comparison based on Unicode values; 'a' (97) comes before 'b' (98), so "apple" is "less than" "banana", returning negative.
*Why others wrong:* A) Would indicate "apple" is lexicographically greater, which is false here. C) 0 would mean equal strings, which these aren't. D) No exception scenario applies to simple compareTo calls.

**Q17.** [Medium | split] What does `"a,b,,c".split(",")` produce (length of resulting array)?
A) 3 B) 4 C) 5 D) 2
**Answer: B**
*Explanation:* Splitting "a,b,,c" by comma produces ["a", "b", "", "c"] — the empty string between two consecutive commas is preserved as an element (unless trailing empties are stripped, but this isn't trailing).
*Why others wrong:* A) Undercounts by ignoring the empty string element. C, D) Miscount the actual segments produced.

**Q18.** [Hard | StringBuilder vs StringBuffer trap] Which statement is FALSE?
A) StringBuilder was introduced in Java 5 B) StringBuffer methods are synchronized C) StringBuilder is thread-safe by default D) StringBuffer predates StringBuilder
**Answer: C**
*Explanation:* StringBuilder is explicitly NOT thread-safe (no synchronization) — that's precisely why it's faster than StringBuffer; this statement is false, so it's the correct MCQ answer.
*Why others wrong:* A) True — StringBuilder was introduced in Java 5 as a faster alternative to StringBuffer. B) True — StringBuffer's methods use `synchronized`. D) True — StringBuffer has existed since Java 1.0, long before StringBuilder.

**Q19.** [Medium | Immutability benefit] Why is String immutability useful for HashMap keys?
A) It makes lookups slower but safer B) The hashCode can be cached/computed once and remains valid forever since content never changes C) Immutable objects cannot be stored in HashMap D) It has nothing to do with HashMap
**Answer: B**
*Explanation:* Since String content never changes, its hashCode is computed once and cached internally — critical for consistent, reliable HashMap bucket placement (a mutable key's changing hashCode would break the map).
*Why others wrong:* A) Actually makes lookups FASTER due to hashCode caching, not slower. C) Immutable objects are ideal HashMap keys, not prohibited. D) Directly relevant — this is a commonly asked "why" conceptual question.

**Q20.** [Hard | Trap Summary] Which comparison approach should you ALWAYS use for comparing String content in production code?
A) `==` B) `.equals()` C) `.hashCode()` D) `.intern() == `
**Answer: B**
*Explanation:* `.equals()` is the reliable, guaranteed-correct method for content comparison regardless of pooling/object-identity nuances covered throughout this chapter.
*Why others wrong:* A) Unreliable due to pool vs. new String() vs. runtime concatenation inconsistencies (Q3, Q14, Q15). C) hashCode alone doesn't guarantee equality (collisions possible in theory). D) Works but is unnecessarily convoluted compared to simply calling `.equals()`.

---

## Chapter 4 Complete ✅
Watch out for: String pool vs `new String()` (Q3, Q11), compile-time-constant vs runtime concatenation (Q14 vs Q15 — this exact contrast is a favorite Capgemini pairing), and StringBuilder-vs-StringBuffer thread-safety/performance (Q5, Q6, Q18).

**Next up: Chapter 5 — Methods, Overloading, Overriding, Constructors, this/super keywords.** Say "next chapter" to continue.

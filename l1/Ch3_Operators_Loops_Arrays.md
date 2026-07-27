# JAVA CORE — Chapter 3: Operators, Control Statements, Loops & Arrays

## 1. Operators — Precedence Traps

| Category | Operators | Notes |
|---|---|---|
| Arithmetic | `+ - * / %` | `%` works on floats/doubles too |
| Relational | `< > <= >= == !=` | Returns boolean |
| Logical | `&& \|\| !` | Short-circuit |
| Bitwise | `& \| ^ ~ << >> >>>` | Operate on bits |
| Assignment | `= += -= *= /=` | Compound assignment auto-casts |
| Ternary | `condition ? a : b` | Single expression if-else |
| instanceof | `obj instanceof Type` | Type check |

**Trap #1 — Short-circuit vs non-short-circuit:**
```java
int x = 5;
if (x > 10 && (x++ > 2)) {} // x++ NEVER runs, x stays 5 (short-circuit skips right side)
if (x > 10 & (x++ > 2)) {}  // x++ ALWAYS runs, x becomes 6 (non-short-circuit & evaluates both sides)
```
`&&`/`||` skip evaluating the right side when the result is already determined. `&`/`|` (used as logical operators) always evaluate both sides — a favorite "predict the output" trap.

**Trap #2 — Compound assignment implicit cast:**
```java
byte b = 10;
b = b + 1;   // COMPILE ERROR — int result can't auto-assign to byte
b += 1;      // OK — compiler inserts implicit cast: b = (byte)(b + 1)
```

**Trap #3 — Integer division:**
```java
System.out.println(5 / 2);    // 2 (int division truncates)
System.out.println(5.0 / 2);  // 2.5 (double division)
System.out.println(5 % 2);    // 1
```

**Trap #4 — Pre vs Post increment:**
```java
int a = 5;
int b = a++ + ++a; // a++ uses 5 then a becomes 6; ++a makes a 7 then uses 7 → b = 5+7 = 12; a = 7
```
Post-increment (`a++`) uses the current value THEN increments. Pre-increment (`++a`) increments THEN uses the value. Frequently tested in "predict output" MCQs.

## 2. Control Statements

- `if / else if / else`
- `switch` — works on int, char, String (Java 7+), enum. **Falls through** without `break`.
- **switch trap:** Missing `break` causes execution to "fall through" to next case(s) until a break or end.

```java
int x = 2;
switch(x) {
    case 1: System.out.println("One");
    case 2: System.out.println("Two");   // prints
    case 3: System.out.println("Three"); // ALSO prints — fall-through, no break!
    default: System.out.println("Default"); // ALSO prints
}
// Output: Two, Three, Default
```

## 3. Loops

| Loop | Use case |
|---|---|
| `for` | Known iteration count |
| `while` | Condition checked before body |
| `do-while` | Body runs **at least once**, condition checked after |
| Enhanced `for` (for-each) | Iterating collections/arrays, read-only access |

**Trap:** `do-while` always executes the body at least once even if the condition is false from the start.
```java
int i = 10;
do { System.out.println(i); } while (i < 5); // prints 10 once, then exits
```

**break vs continue:**
- `break` — exits the loop entirely.
- `continue` — skips to the next iteration.
- **Labeled break/continue** — can target an outer loop in nested loops:
```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) continue outer; // skips to next i, not just next j
        System.out.println(i + "," + j);
    }
}
```

## 4. Arrays

- Fixed size, declared with `[]`, index starts at **0**.
- Default values: `int[]` → 0, `boolean[]` → false, `Object[]`/`String[]` → null.
- `array.length` — property (no parentheses), NOT `array.length()`. That's the #1 array trap.
- **String.length()** — method (with parentheses). This int-array-vs-String distinction is heavily tested.

```java
int[] arr = new int[5];  // all elements default to 0
arr.length   // 5 → correct
arr.length() // COMPILE ERROR
```

### 2D Arrays
```java
int[][] matrix = new int[3][4]; // 3 rows, 4 columns
int[][] jagged = new int[3][];  // jagged array — rows can have different lengths
jagged[0] = new int[2];
jagged[1] = new int[5];
```

**Trap:** `ArrayIndexOutOfBoundsException` is a **runtime** exception (unchecked), not caught at compile time — accessing `arr[5]` on a 5-element array (valid indices 0-4) throws it.

## One-Page Revision
- `&&`/`||` short-circuit (skip right side if already determined); `&`/`|` always evaluate both.
- Compound assignment (`+=`) auto-casts; plain `b = b+1` on byte/short does NOT compile.
- Integer division truncates; use a double operand for decimal results.
- Post-increment uses-then-increments; pre-increment increments-then-uses.
- switch falls through without `break`; works on int/char/String/enum.
- do-while always runs at least once.
- Labeled break/continue can control outer loops.
- Array: fixed size, `.length` is a field (no parens); index from 0; out-of-bounds access = runtime `ArrayIndexOutOfBoundsException`.
- String uses `.length()` (method) — opposite of array's `.length` (field). **Classic trap.**

---

# MCQs — Chapter 3 (20 Questions)

**Q1.** [Easy | Operators] What is `5 / 2` in Java (both int)?
A) 2.5 B) 2 C) 3 D) 2.0
**Answer: B**
*Explanation:* Integer division truncates the decimal part; both operands are int, so result is int.
*Why others wrong:* A, D) Would require at least one operand to be float/double. C) No rounding occurs, only truncation.

**Q2.** [Easy | Array] How do you get the length of an array `arr`?
A) `arr.length()` B) `arr.size()` C) `arr.length` D) `arr.getLength()`
**Answer: C**
*Explanation:* Array length is a public final field, accessed without parentheses — unlike String's `.length()` method.
*Why others wrong:* A) That's the classic trap — confusing it with String's method syntax. B) `.size()` is used by Collections (List, Set), not arrays. D) Not a real array method.

**Q3.** [Easy | Loop] Which loop guarantees the body executes at least once?
A) for B) while C) do-while D) for-each
**Answer: C**
*Explanation:* do-while checks its condition AFTER executing the body, so the first execution is unconditional.
*Why others wrong:* A, B, D) All check the condition BEFORE the first execution, so they may run zero times.

**Q4.** [Medium | Short-circuit] What is the output?
```java
int x = 5;
if (x > 10 && (x++ > 2)) {}
System.out.println(x);
```
A) 5 B) 6 C) 7 D) Compile error
**Answer: A**
*Explanation:* `&&` short-circuits — since `x > 10` is false, the right-hand `x++` expression is never evaluated, so x remains 5.
*Why others wrong:* B, C) Would only happen if x++ actually executed. D) Valid, compiling code.

**Q5.** [Medium | Non-short-circuit] What is the output?
```java
int x = 5;
if (x > 10 & (x++ > 2)) {}
System.out.println(x);
```
A) 5 B) 6 C) 7 D) Compile error
**Answer: B**
*Explanation:* `&` (single ampersand, used as boolean operator) always evaluates BOTH sides regardless of the left result, so `x++` executes, incrementing x to 6.
*Why others wrong:* A) Would be true only if short-circuiting occurred, but `&` doesn't short-circuit. C) x++ only increments once. D) Valid code.

**Q6.** [Medium | Increment] What is the output?
```java
int a = 5;
int b = a++ + ++a;
System.out.println(a + " " + b);
```
A) 7 11 B) 7 12 C) 6 11 D) 7 10
**Answer: B**
*Explanation:* `a++` returns 5 (then a becomes 6); `++a` increments a to 7 then returns 7; so b = 5 + 7 = 12; final a = 7.
*Why others wrong:* A, C, D) Miscalculate the order of increment application — this is exactly the trap tested.

**Q7.** [Medium | switch] What does this print?
```java
int x = 2;
switch(x) {
    case 1: System.out.println("One");
    case 2: System.out.println("Two");
    case 3: System.out.println("Three");
    default: System.out.println("Default");
}
```
A) Two B) Two Three Default C) One Two Three Default D) Compile error
**Answer: B**
*Explanation:* Without `break`, execution falls through from the matched case (2) through all subsequent cases including default.
*Why others wrong:* A) Would only be correct if a `break` existed after case 2. C) Case 1 doesn't match x=2, so "One" is skipped entirely — matching starts at case 2. D) Valid code, no error.

**Q8.** [Medium | Compound Assignment] Which compiles successfully?
```java
byte b = 10;
// Option A: b = b + 1;
// Option B: b += 1;
```
A) Only Option A B) Only Option B C) Both compile D) Neither compiles
**Answer: B**
*Explanation:* `b += 1` has an implicit cast inserted by the compiler `(byte)(b+1)`; plain `b = b + 1` fails because `b + 1` promotes to int, which can't be auto-assigned back to byte.
*Why others wrong:* A) `b + 1` is int; assigning int to byte without an explicit cast is a compile error. C) Option A does NOT compile. D) Option B does compile.

**Q9.** [Medium | Array Default] What is the default value of elements in `new boolean[3]`?
A) true B) false C) null D) 0
**Answer: B**
*Explanation:* boolean arrays default every element to `false`, same as boolean's standalone default.
*Why others wrong:* A) Not the default — Java defaults to false, not true. C) null applies to reference type arrays, not boolean. D) 0 applies to numeric type arrays.

**Q10.** [Hard | ArrayIndexOutOfBounds] What happens?
```java
int[] arr = new int[5];
System.out.println(arr[5]);
```
A) Prints 0 B) Compile error C) ArrayIndexOutOfBoundsException at runtime D) Prints null
**Answer: C**
*Explanation:* Valid indices for a 5-element array are 0-4; index 5 is out of bounds, thrown as an unchecked runtime exception.
*Why others wrong:* A) No default value is returned for an invalid index — it throws instead. B) Compiler cannot detect this at compile time (bounds aren't statically known in general). D) int arrays never contain null.

**Q11.** [Hard | Labeled Loop] What does this print?
```java
outer:
for (int i = 0; i < 2; i++) {
    for (int j = 0; j < 2; j++) {
        if (j == 1) continue outer;
        System.out.println(i + "," + j);
    }
}
```
A) 0,0 0,1 1,0 1,1 B) 0,0 1,0 C) 0,0 0,1 1,0 D) 0,0 1,0 1,1
**Answer: B**
*Explanation:* `continue outer` skips directly to the next iteration of the outer loop as soon as j==1, so only j=0 ever prints for each i.
*Why others wrong:* A) Would be the output with no continue at all. C, D) Incorrectly include j=1 outputs or omit an i iteration.

**Q12.** [Medium | 2D Array] What does `int[][] jagged = new int[3][];` create?
A) A 3x3 matrix with all zeros B) An array of 3 rows where each row's length is not yet defined C) Compile error D) A 3-element 1D array
**Answer: B**
*Explanation:* This creates a "jagged array" — 3 row references initialized to null, where each row must be separately allocated with potentially different lengths.
*Why others wrong:* A) No column dimension was specified, so it's not a fixed matrix. C) This is valid Java syntax for jagged arrays. D) It's a 2D structure (array of arrays), not flat 1D.

**Q13.** [Easy | Ternary] What is the value of `x` in `int x = (5 > 3) ? 10 : 20;`?
A) 5 B) 3 C) 10 D) 20
**Answer: C**
*Explanation:* Since `5 > 3` is true, the ternary operator evaluates and returns the value before the colon (10).
*Why others wrong:* A, B) Not related to the ternary's actual return values. D) 20 would only apply if the condition were false.

**Q14.** [Hard | Bitwise] What is `5 & 3` in Java?
A) 1 B) 7 C) 8 D) 2
**Answer: A**
*Explanation:* In binary: 5 = 101, 3 = 011. Bitwise AND compares each bit: 101 & 011 = 001 = 1.
*Why others wrong:* B) 7 would be the result of OR (`5 | 3` = 111 = 7). C, D) Don't correspond to correct bitwise AND arithmetic.

**Q15.** [Medium | switch with String] Is this valid Java (Java 7+)?
```java
String s = "hello";
switch(s) {
    case "hello": System.out.println("Hi"); break;
    default: System.out.println("Bye");
}
```
A) Valid, prints "Hi" B) Invalid — switch doesn't support String C) Valid, prints "Bye" D) Compile error, switch only supports primitives
**Answer: A**
*Explanation:* Since Java 7, switch statements support String as a valid selector type (internally uses hashCode + equals), in addition to int, char, and enum.
*Why others wrong:* B, D) Outdated assumption — pre-Java 7 this was true, but current Java fully supports String switches. C) "hello" matches the case, so "Hi" prints, not the default branch.

**Q16.** [Medium | do-while trap] What does this print?
```java
int i = 10;
do {
    System.out.println(i);
} while (i < 5);
```
A) Nothing prints B) 10 (once) C) Infinite loop D) Compile error
**Answer: B**
*Explanation:* do-while always executes the body once before checking the condition; since `10 < 5` is false, it exits after just one print.
*Why others wrong:* A) Contradicts do-while's guaranteed first execution. C) The loop correctly terminates after checking the false condition. D) Perfectly valid syntax.

**Q17.** [Hard | Operator precedence] What is the output?
```java
int result = 10 + 5 * 2 - 3 / 3;
System.out.println(result);
```
A) 19 B) 9 C) 10 D) 29
**Answer: A**
*Explanation:* Following precedence (`*` and `/` before `+`/`-`, left to right): 5*2=10, 3/3=1, then 10+10-1 = 19.
*Why others wrong:* B, C, D) Result from ignoring correct operator precedence (e.g., evaluating strictly left-to-right without respecting `*`/`/` priority).

**Q18.** [Medium | for-each limitation] Which of these CANNOT be done with an enhanced for-each loop?
A) Reading array elements B) Iterating a List C) Modifying the underlying array's element values by index during iteration D) Iterating a Set
**Answer: C**
*Explanation:* for-each provides a copy of each element in the loop variable (for primitives) or a reference (for objects) — it has no access to the index, so you can't directly reassign an array slot through the loop variable itself.
*Why others wrong:* A, B, D) All are exactly what for-each is designed for — straightforward iteration/reading.

**Q19.** [Hard | Trap] What is the output?
```java
System.out.println(1 + 2 + "3" + 4 + 5);
```
A) 12345 B) "33" then 4 then 5 concatenated as 3345 C) 15 D) Compile error
**Answer: B — printed as: 3345**
*Explanation:* Left-to-right evaluation: `1 + 2` = 3 (still int, arithmetic since both are numbers), then `3 + "3"` = "33" (string concat begins once a String is involved), then `"33" + 4` = "334", then `"334" + 5` = "3345".
*Why others wrong:* A) Would only happen if all values concatenated as strings from the start (no leading numeric int addition). C) Would be the result if everything summed numerically, ignoring the String literal entirely. D) Fully valid, compiling expression.

**Q20.** [Medium | Trap Summary] Which statement about `break` vs `continue` is TRUE?
A) break skips to the next iteration; continue exits the loop B) break exits the loop; continue skips to the next iteration C) Both exit the loop entirely D) Both skip to the next iteration
**Answer: B**
*Explanation:* This is the standard, correct behavior definition — break terminates the entire loop, continue just moves on to the next cycle.
*Why others wrong:* A) Reverses the definitions. C) break does exit, but continue does NOT exit — it continues. D) continue does skip forward, but break does NOT just skip — it stops the loop entirely.

---

## Chapter 3 Complete ✅
Watch out for: increment/decrement trace questions (Q6), short-circuit vs bitwise-as-logical (Q4/Q5), and string-concatenation-with-numbers (Q19) — these show up constantly as "predict the output" MCQs in the actual exam.

**Next up: Chapter 4 — Strings, StringBuffer, StringBuilder & Methods.** Say "next chapter" to continue.

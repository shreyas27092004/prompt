# JAVA CORE — Chapter 2: Variables, Data Types, Wrapper Classes, Autoboxing

## 1. Variable Types
- **Local variable** — declared inside a method/block, no default value, must be initialized before use, lives on Stack.
- **Instance variable** — declared inside a class, outside methods; gets a default value; one copy per object; lives on Heap.
- **Static variable** — declared with `static`; one copy shared by all objects; lives in Method Area; gets default value.

**Trap:** Local variables are **never** auto-initialized. Using an uninitialized local variable → **compile-time error**, not runtime.

## 2. Primitive Data Types (8 total) — MUST memorize sizes

| Type | Size | Default | Range/Notes |
|---|---|---|---|
| byte | 1 byte | 0 | -128 to 127 |
| short | 2 bytes | 0 | -32,768 to 32,767 |
| int | 4 bytes | 0 | ~-2.1B to 2.1B |
| long | 8 bytes | 0L | needs `L` suffix for literals beyond int range |
| float | 4 bytes | 0.0f | needs `f` suffix |
| double | 8 bytes | 0.0d | default for decimal literals |
| char | 2 bytes | '\u0000' | single Unicode character |
| boolean | JVM-dependent (not fixed) | false | true/false only |

**Capgemini trap #1:** `boolean` size is **NOT specified by the JVM spec** — it's often asked as a trick "what is the size of boolean?" Correct answer: **JVM-dependent / not precisely defined** (commonly stated as 1 bit conceptually, 1 byte in practice — but the exam usually wants you to know it's not fixed like the others).

**Capgemini trap #2:** `char` is unsigned, range 0 to 65535 (not negative like byte/short).

**Capgemini trap #3:**
```java
long l = 2147483648; // ERROR — too big for int literal, even though target is long
long l2 = 2147483648L; // OK — L suffix makes it a long literal
```
Any integer literal is `int` by default. Without `L`, a value beyond int range fails to compile **even when assigned to a long**.

## 3. Primitive vs Non-Primitive (Reference Types)

| | Primitive | Non-Primitive (Reference) |
|---|---|---|
| Examples | int, char, boolean, etc. | String, Array, Class objects, Interface |
| Storage | Value directly in Stack (if local) | Reference in Stack, object in Heap |
| Default value | 0/false/etc. | null |
| Methods | Cannot call methods on them | Can call methods |
| Size | Fixed | Depends on object |

**Trap:** `String` is NOT a primitive even though it's used like one constantly — it's a class (reference type), and it's **immutable**.

## 4. Wrapper Classes

Every primitive has a corresponding wrapper class:
`int`→`Integer`, `char`→`Character`, `boolean`→`Boolean`, `double`→`Double`, `long`→`Long`, `float`→`Float`, `byte`→`Byte`, `short`→`Short`.

Why wrappers exist: Collections (`ArrayList`, `HashMap`) can only store **objects**, not primitives. Wrapper classes let primitives be used where objects are required.

## 5. Autoboxing & Unboxing

- **Autoboxing**: automatic conversion of primitive → wrapper object.
  `Integer i = 10;` (compiler does `Integer.valueOf(10)` internally)
- **Unboxing**: automatic conversion of wrapper → primitive.
  `int x = i;` (compiler does `i.intValue()` internally)

### THE most famous Capgemini trap — Integer caching:
```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b); // TRUE

Integer c = 200;
Integer d = 200;
System.out.println(c == d); // FALSE
```
**Why:** Java caches `Integer` objects for values **-128 to 127** (`IntegerCache`). Values in that range reuse the same object, so `==` (reference comparison) returns true. Outside that range, new objects are created each time, so `==` returns false. Always use `.equals()` to compare wrapper values safely.

## One-Page Revision
- Local vars: no default, must initialize, Stack. Instance vars: default value, Heap. Static vars: default value, Method Area, shared.
- 8 primitives: byte(1) short(2) int(4) long(8) float(4) double(8) char(2) boolean(JVM-dependent).
- char is unsigned (0–65535). Integer literals default to `int`; use `L` for long literals beyond int range.
- Primitive = value type, stored directly. Reference type = pointer to Heap object, default `null`.
- String is a class, not primitive, and is immutable.
- Wrapper classes let primitives work in Collections/Generics.
- Autoboxing = primitive→wrapper (compiler-inserted `valueOf`). Unboxing = wrapper→primitive (`xxxValue()`).
- **Integer cache range -128 to 127**: `==` gives true inside range, false outside. Always use `.equals()` for wrapper value comparison.

---

# MCQs — Chapter 2 (20 Questions)

**Q1.** [Easy | Variables] What is the default value of an uninitialized local `int` variable?
A) 0 B) null C) Compile-time error if used before initialization D) Random garbage value
**Answer: C**
*Explanation:* Unlike instance/static variables, local variables get NO default value; the compiler forces you to initialize before use.
*Why others wrong:* A) That's the default for instance/static int, not local. B) null applies to references, not int. D) Java doesn't allow garbage values like C/C++ — it's a compile error.

**Q2.** [Easy | Data Types] Which of these requires an explicit suffix in its literal form?
A) int B) long (when exceeding int range) C) short D) byte
**Answer: B**
*Explanation:* Literal integers default to `int` type; to represent a long literal beyond int's range you must append `L` (e.g., `9999999999L`).
*Why others wrong:* A) int is the default literal type, no suffix needed. C, D) short/byte values are assigned from int literals directly (within range), no suffix exists for them.

**Q3.** [Easy | char] What is the range of `char` in Java?
A) -128 to 127 B) -32768 to 32767 C) 0 to 65535 D) 0 to 255
**Answer: C**
*Explanation:* char is a 2-byte **unsigned** type representing Unicode characters, range 0–65535.
*Why others wrong:* A) That's byte's range. B) That's short's (signed) range. D) That's the old ASCII/extended range, not Java's char.

**Q4.** [Medium | Wrapper Caching] What does this print?
```java
Integer a = 127, b = 127;
System.out.println(a == b);
```
A) true B) false C) Compile error D) Runtime exception
**Answer: A**
*Explanation:* 127 is within the Integer cache range (-128 to 127), so both references point to the same cached object.
*Why others wrong:* B) Would be correct only outside the cache range. C, D) No error — this is valid, running code.

**Q5.** [Medium | Wrapper Caching] What does this print?
```java
Integer a = 128, b = 128;
System.out.println(a == b);
```
A) true B) false C) Compile error D) Runtime exception
**Answer: B**
*Explanation:* 128 exceeds the Integer cache range, so `a` and `b` are separate objects — `==` compares references, not values.
*Why others wrong:* A) Only true within cache range -128 to 127. C, D) Perfectly valid, compiling and running code.

**Q6.** [Medium | Trap] Which statement is TRUE about String in Java?
A) String is a primitive type B) String is a mutable class C) String is an immutable reference type D) String has a fixed size of 2 bytes
**Answer: C**
*Explanation:* String is a class (not primitive) whose internal character data cannot be changed once created — any "modification" creates a new String object.
*Why others wrong:* A) It's a class, not one of the 8 primitives. B) It's immutable, not mutable (that's what StringBuilder is for). D) String size varies based on content, not fixed.

**Q7.** [Medium | Autoboxing] What happens internally when you write `Integer i = 5;`?
A) Nothing, direct assignment B) Compiler calls `Integer.valueOf(5)` C) Compiler calls `new Integer(5)` always D) Runtime error
**Answer: B**
*Explanation:* Autoboxing uses `valueOf()`, which internally may return a cached object (for -128 to 127) rather than always creating new instances.
*Why others wrong:* A) It IS a conversion, not direct — int and Integer are different types. C) `new Integer()` is actually deprecated/avoided precisely because it bypasses caching. D) Perfectly valid code.

**Q8.** [Medium | Static vs Instance] Which memory area holds a `static` variable?
A) Stack B) Heap C) Method Area D) PC Register
**Answer: C**
*Explanation:* Static (class-level) data is stored once in the Method Area, shared across all instances.
*Why others wrong:* A) Stack is for local variables/method frames. B) Heap holds instance-level object data. D) PC Register just tracks the current instruction.

**Q9.** [Hard | Trap] What's the size of `boolean` in Java, per the JVM specification?
A) 1 bit B) 1 byte C) 4 bytes D) Not precisely defined by the spec
**Answer: D**
*Explanation:* Unlike other primitives, the JVM spec does not mandate an exact boolean size — it's implementation-dependent (commonly treated as 1 byte in practice, but not formally specified like int=4 bytes).
*Why others wrong:* A, B, C) These are commonly assumed answers, but none are formally guaranteed by the JVM spec — this is exactly the kind of "gotcha" Capgemini tests.

**Q10.** [Hard | Code Trace] What is the output?
```java
public class Test {
    static int x;
    public static void main(String[] args) {
        System.out.println(x);
    }
}
```
A) Compile error, x not initialized B) 0 C) null D) Garbage value
**Answer: B**
*Explanation:* Static variables get automatic default values (0 for int), so this compiles and prints 0 without explicit initialization.
*Why others wrong:* A) Only local variables need explicit initialization, not static/instance. C) null is for reference types, not int. D) Java guarantees deterministic defaults, unlike C.

**Q11.** [Hard | Scenario] Which comparison is SAFE and always correct for comparing two Integer objects by value?
A) `a == b` B) `a.equals(b)` C) `a.hashCode() == b.hashCode()` D) `(int)a == (int)b`
**Answer: B**
*Explanation:* `.equals()` compares actual values regardless of caching, unlike `==` which compares references (unreliable outside -128 to 127).
*Why others wrong:* A) Unreliable due to caching behavior shown in Q4/Q5. C) hashCode equality doesn't strictly guarantee value equality (though for Integer it usually aligns, it's not idiomatic/guaranteed practice). D) Not valid cast syntax for unboxing comparison (would need `.intValue()`), and generally unnecessary when `.equals()` exists.

**Q12.** [Medium | Non-Primitive] Which of these is a non-primitive (reference) type?
A) int B) char C) int[] (array) D) boolean
**Answer: C**
*Explanation:* Arrays are objects in Java, stored on the Heap with a reference on the Stack — making them reference types, unlike the 8 primitives.
*Why others wrong:* A, B, D) All are primitive types with direct value storage.

**Q13.** [Easy | Wrapper] Which wrapper class corresponds to primitive `char`?
A) Char B) Character C) Chars D) CharacterType
**Answer: B**
*Explanation:* The wrapper class name is `Character` (not a direct capitalization of `char`), unlike most other primitive-to-wrapper mappings.
*Why others wrong:* A, C, D) Not real Java class names — this is a common naming trap since char→Character breaks the "just capitalize it" pattern that works for int→Integer.

**Q14.** [Medium | Trap] Which primitive-to-wrapper mapping is NOT simply capitalized?
A) int → Integer B) char → Character C) boolean → Boolean D) double → Double
**Answer: A**
*Explanation:* `int` maps to `Integer`, not "Int" — this breaks the simple capitalization pattern (along with char→Character).
*Why others wrong:* B) Actually char→Character is ALSO an exception (worth noting both), but among the options, A is the standard trap answer expected. C, D) boolean→Boolean and double→Double DO follow simple capitalization.

**Q15.** [Hard | Unboxing/NPE] What happens here?
```java
Integer i = null;
int x = i;
```
A) x becomes 0 B) Compile error C) NullPointerException at runtime D) x becomes null
**Answer: C**
*Explanation:* Unboxing `null` calls `i.intValue()` internally, which throws NPE since you can't call a method on a null reference.
*Why others wrong:* A) No silent fallback to 0 happens. B) This compiles fine — the error is a runtime one. D) int can never hold null; that's not valid for a primitive.

**Q16.** [Medium | Data Types] Which is the correct way to declare a `long` literal larger than `Integer.MAX_VALUE`?
A) `long x = 5000000000;` B) `long x = 5000000000L;` C) `long x = (long)5000000000;` D) `Long x = 5000000000;`
**Answer: B**
*Explanation:* The `L` suffix marks the literal itself as type long before assignment, avoiding the "integer literal too large" compile error.
*Why others wrong:* A) Fails to compile — 5000000000 is treated as int literal first, which overflows int range. C) Casting doesn't help; the literal has already failed to parse as int before the cast could apply. D) Same literal overflow issue applies regardless of the Long wrapper target.

**Q17.** [Medium | Instance Variable] Which of the following gets automatically initialized to a default value WITHOUT explicit assignment?
A) Local variable in a method B) Instance variable of a class C) Both A and B D) Neither A nor B
**Answer: B**
*Explanation:* Only instance (and static) variables get default values automatically; local variables must always be explicitly initialized before use.
*Why others wrong:* A) Local variables are the specific exception with NO default value. C, D) Only instance variables (not local) get this treatment.

**Q18.** [Hard | Scenario] What does this output?
```java
byte b = 127;
b++;
System.out.println(b);
```
A) 128 B) -128 C) Compile error D) 127
**Answer: B**
*Explanation:* byte overflows past its max (127) and wraps around to its minimum value (-128), since byte is an 8-bit signed type — this is integer overflow wraparound.
*Why others wrong:* A) 128 exceeds byte's range entirely, so it can't hold that value. C) `b++` compiles fine (compound operators implicitly cast). D) Value does change due to overflow, doesn't stay the same.

**Q19.** [Easy | Terminology] What is "unboxing" in Java?
A) Converting wrapper object to primitive automatically B) Converting primitive to wrapper automatically C) Removing an object from a Collection D) Casting one primitive to another
**Answer: A**
*Explanation:* Unboxing is the automatic conversion from a wrapper object (e.g., Integer) back to its primitive form (int) by the compiler.
*Why others wrong:* B) That's autoboxing, the reverse process. C) Unrelated to Collections removal operations. D) That's primitive widening/narrowing, a different concept.

**Q20.** [Hard | Comprehensive] Which statement is FALSE?
A) Wrapper classes are immutable B) Autoboxing can cause NullPointerException during unboxing of null C) All primitive wrapper objects are cached regardless of value D) String, being a reference type, defaults to null when uninitialized as instance variable
**Answer: C**
*Explanation:* Only Integer (and similarly Byte, Short, Long, Character, Boolean) values within specific small ranges (like -128 to 127 for Integer) are cached — NOT all wrapper objects for all values, making this statement false.
*Why others wrong:* A) True — wrapper classes are genuinely immutable. B) True, as demonstrated in Q15. D) True — reference type instance variables default to null.

---

## Chapter 2 Complete ✅
Focus areas if you missed questions: Integer caching (Q4/Q5/Q9/Q20) and unboxing NPE (Q15) are Capgemini's favorite traps in this chapter — re-read section 5 if these tripped you up.

**Next up: Chapter 3 — Operators, Control Statements, Loops & Arrays.** Say "next chapter" to continue.

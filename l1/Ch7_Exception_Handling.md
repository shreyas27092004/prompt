# JAVA CORE — Chapter 7: Exception Handling

## 1. Exception Hierarchy — Must memorize

```
Throwable
├── Error (serious, unrecoverable — e.g., OutOfMemoryError, StackOverflowError)
└── Exception
    ├── Checked Exceptions (must be handled/declared — e.g., IOException, SQLException)
    └── RuntimeException (Unchecked — e.g., NullPointerException, ArithmeticException, ArrayIndexOutOfBoundsException, ClassCastException)
```

**Trap #1 — Checked vs Unchecked:**
- **Checked** exceptions: MUST be handled (try-catch) or declared (`throws`) — checked at COMPILE time. Examples: `IOException`, `SQLException`, `FileNotFoundException`, `ClassNotFoundException`.
- **Unchecked** (RuntimeException and its subclasses): NOT required to handle/declare — checked only at RUNTIME. Examples: `NullPointerException`, `ArithmeticException`, `ArrayIndexOutOfBoundsException`, `NumberFormatException`, `ClassCastException`.
- **Error** is NEVER meant to be caught/handled (e.g., `OutOfMemoryError`, `StackOverflowError`) — represents unrecoverable JVM-level problems.

**Trap #2:** `Error` and `RuntimeException` are both unchecked, but they are NOT the same thing — `Error` is a sibling of `Exception` under `Throwable`, not a subclass of `Exception`.

## 2. try / catch / finally

- `try` block: code that might throw an exception.
- `catch` block: handles a specific exception type. Multiple catch blocks allowed (must go from **most specific to most general**).
- `finally` block: ALWAYS executes, regardless of whether an exception occurred, was caught, or even if there's a `return` in try/catch — **except** if JVM exits via `System.exit()` or the JVM crashes.

**Trap #3 — finally with return:**
```java
public static int test() {
    try {
        return 1;
    } finally {
        System.out.println("finally runs");
    }
}
// Output: "finally runs" is printed, THEN 1 is returned. finally executes even with a return in try.
```

**Trap #4 — finally OVERRIDING return:**
```java
public static int test() {
    try {
        return 1;
    } finally {
        return 2; // This OVERRIDES the try's return value!
    }
}
// Returns 2, NOT 1 — a return in finally always wins. (Considered bad practice, but tested heavily.)
```

**Trap #5 — Catch block order:**
```java
try {
    // code
} catch (ArithmeticException e) {
    // specific
} catch (Exception e) {
    // general — must come AFTER specific subclasses
}
// Reversing this order (Exception first, then ArithmeticException) = COMPILE ERROR: "unreachable code"
```

## 3. throw vs throws — commonly confused

| | `throw` | `throws` |
|---|---|---|
| Purpose | Actually throws an exception instance | Declares that a method MIGHT throw an exception |
| Location | Inside method body | In method signature |
| Followed by | An exception object (`throw new IOException()`) | Exception class name(s) (`throws IOException`) |
| Count | Only ONE exception thrown at a time | Multiple exceptions can be declared, comma-separated |

```java
void readFile() throws IOException {   // throws — declaration
    throw new IOException("File not found"); // throw — actual throw
}
```

## 4. Custom Exceptions

```java
class InsufficientBalanceException extends Exception {  // checked custom exception
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
// To make it UNCHECKED instead: extend RuntimeException instead of Exception
```

**Trap:** Extending `Exception` → custom exception is CHECKED (must be declared/handled). Extending `RuntimeException` → custom exception is UNCHECKED.

## 5. Multi-catch (Java 7+)
```java
try {
    // code
} catch (IOException | SQLException e) {  // single block handles multiple exception types
    System.out.println(e.getMessage());
}
```
**Trap:** In multi-catch, the exception variable `e` is implicitly `final` — you cannot reassign it inside the catch block.

## 6. try-with-resources (Java 7+)
```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    // use br
} // br.close() called AUTOMATICALLY, even if exception occurs
```
Only works with classes implementing `AutoCloseable`/`Closeable`. Eliminates the need for explicit `finally { br.close(); }`.

## One-Page Revision
- Hierarchy: Throwable → Error (unrecoverable) + Exception → Checked (compile-time enforced) + RuntimeException (unchecked, runtime-only).
- Checked examples: IOException, SQLException. Unchecked examples: NullPointerException, ArithmeticException, ArrayIndexOutOfBoundsException, ClassCastException, NumberFormatException.
- finally ALWAYS runs (except System.exit() or JVM crash) — even with a return in try.
- return in finally OVERRIDES return in try/catch (trap, bad practice but testable).
- catch blocks: specific subclass exceptions before general ones, or compile error (unreachable code).
- throw = actually throwing (needs an instance). throws = declaring possibility (in signature, class names only).
- Custom exception extends Exception (checked) or RuntimeException (unchecked).
- Multi-catch: `catch (A | B e)` — e is implicitly final.
- try-with-resources auto-closes AutoCloseable resources, no explicit finally needed.

---

# MCQs — Chapter 7 (20 Questions)

**Q1.** [Easy | Hierarchy] Which of these is a checked exception?
A) NullPointerException B) ArithmeticException C) IOException D) ArrayIndexOutOfBoundsException
**Answer: C**
*Explanation:* IOException extends Exception directly (not RuntimeException), making it a checked exception that must be handled or declared.
*Why others wrong:* A, B, D) All three extend RuntimeException, making them unchecked — no compile-time handling requirement.

**Q2.** [Easy | finally] When does the `finally` block execute?
A) Only if an exception occurs B) Only if no exception occurs C) Always, regardless of exception occurrence (except System.exit()/JVM crash) D) Only if the exception is caught
**Answer: C**
*Explanation:* finally is guaranteed to run in virtually all circumstances — its entire purpose is reliable cleanup code, independent of whether an exception was thrown, caught, or neither.
*Why others wrong:* A, B, D) All describe conditional execution, but finally's defining feature is its UNconditional execution.

**Q3.** [Medium | throw vs throws] Which is used to declare that a method might throw an exception?
A) throw B) throws C) catch D) try
**Answer: B**
*Explanation:* `throws` appears in the method signature to declare potential exceptions the caller must handle or propagate.
*Why others wrong:* A) `throw` actually triggers an exception instance, doesn't just declare possibility. C, D) Both are for handling exceptions, not declaring them.

**Q4.** [Medium | Catch order] What happens with this code?
```java
try {
    int x = 5/0;
} catch (Exception e) {
    System.out.println("General");
} catch (ArithmeticException e) {
    System.out.println("Specific");
}
```
A) Prints "Specific" B) Prints "General" C) Compile error — unreachable code D) Runtime exception, uncaught
**Answer: C**
*Explanation:* Since `Exception` is a superclass of `ArithmeticException`, placing the general catch FIRST makes the more specific catch unreachable — Java catches this at compile time.
*Why others wrong:* A, B) Neither can print because the code fails to compile in the first place. D) The exception WOULD be catchable if the catch order were correct — this is purely a compile-time ordering issue.

**Q5.** [Hard | finally with return] What does this return?
```java
public static int test() {
    try {
        return 1;
    } finally {
        System.out.println("finally");
    }
}
```
A) 1, without printing "finally" B) 1, after printing "finally" C) Compile error D) 0
**Answer: B**
*Explanation:* finally executes BEFORE the method actually returns, even though the return value (1) was already determined in the try block — so "finally" prints, then 1 is returned.
*Why others wrong:* A) finally always executes when this code path runs, contradicting "without printing." C) Perfectly valid, compiling code. D) The try block explicitly returns 1, not 0.

**Q6.** [Hard | finally overriding return] What does this return?
```java
public static int test() {
    try {
        return 1;
    } finally {
        return 2;
    }
}
```
A) 1 B) 2 C) Compile error D) Both 1 and 2 are returned
**Answer: B**
*Explanation:* A `return` statement inside `finally` completely overrides/discards any pending return from try or catch — this is a well-known Java gotcha.
*Why others wrong:* A) The try's return of 1 gets discarded once finally's own return executes. C) Valid (if poor-practice) syntax that compiles fine. D) A method can only return one value; finally's return wins exclusively.

**Q7.** [Medium | Custom Exception] To create an UNCHECKED custom exception, you should extend:
A) Exception B) RuntimeException C) Error D) Throwable
**Answer: B**
*Explanation:* Extending RuntimeException makes your custom exception unchecked, meaning callers aren't compiler-forced to catch or declare it.
*Why others wrong:* A) Extending Exception directly creates a CHECKED exception, the opposite of what's asked. C) Error is reserved for serious, unrecoverable JVM-level problems — not appropriate for custom application exceptions. D) Throwable is the root of everything; extending it directly is unconventional and not the standard unchecked-exception pattern.

**Q8.** [Medium | Unchecked examples] Which of these is NOT a RuntimeException subclass?
A) NullPointerException B) ClassCastException C) FileNotFoundException D) NumberFormatException
**Answer: C**
*Explanation:* FileNotFoundException extends IOException, which extends Exception directly (not RuntimeException) — making it a checked exception.
*Why others wrong:* A, B, D) All three genuinely extend RuntimeException and are unchecked.

**Q9.** [Hard | Multi-catch] What is TRUE about the exception variable in a multi-catch block like `catch (IOException | SQLException e)`?
A) It can be reassigned freely inside the block B) It is implicitly final and cannot be reassigned C) It must be explicitly declared final or it won't compile D) Multi-catch doesn't allow a shared variable name
**Answer: B**
*Explanation:* Java automatically treats the multi-catch parameter as effectively `final`, preventing reassignment within the catch block, even without writing the `final` keyword yourself.
*Why others wrong:* A) Directly contradicts the implicit-final restriction. C) It's implicit — you don't need to write `final` explicitly for this rule to apply. D) Multi-catch syntax explicitly supports one shared variable name across the listed exception types.

**Q10.** [Medium | try-with-resources] What interface must a resource implement to be used in try-with-resources?
A) Serializable B) Comparable C) AutoCloseable (or Closeable) D) Cloneable
**Answer: C**
*Explanation:* try-with-resources automatically calls `.close()` at the end of the block, which requires the resource to implement AutoCloseable (or its subinterface Closeable).
*Why others wrong:* A) Serializable relates to object serialization, unrelated to resource closing. B) Comparable is for ordering/sorting objects. D) Cloneable relates to object copying, not resource management.

**Q11.** [Hard | Error vs Exception] Which statement about `Error` is TRUE?
A) Error is a subclass of Exception B) Error and Exception are siblings, both extending Throwable directly C) Error should always be caught and handled in application code D) Error is a checked type requiring explicit handling
**Answer: B**
*Explanation:* Both Error and Exception directly extend Throwable as separate branches — Error is NOT nested under Exception, a common misconception.
*Why others wrong:* A) A very common trap — Error is NOT a subclass of Exception; they're siblings. C) Errors represent serious, typically unrecoverable conditions (like OutOfMemoryError) that applications generally shouldn't try to catch/handle. D) Error is unchecked, not subject to compile-time handling enforcement.

**Q12.** [Medium | ClassCastException scenario] What throws a ClassCastException?
```java
Object obj = "Hello";
Integer i = (Integer) obj;
```
A) Compile error, not a runtime exception B) ClassCastException at runtime C) No exception, i becomes null D) NumberFormatException
**Answer: B**
*Explanation:* Attempting to cast a String object to Integer at runtime fails because the actual object type is incompatible with the target cast type — this is caught only at runtime, not compile time (since obj is declared as Object).
*Why others wrong:* A) The compiler allows the cast syntactically (since Object could theoretically be any subtype) — the actual type mismatch is only detected at runtime. C) No silent null fallback occurs; an exception is actively thrown. D) NumberFormatException applies to invalid String-to-number parsing (e.g., `Integer.parseInt("abc")`), not object casting.

**Q13.** [Medium | throw syntax] Which is valid syntax for actually throwing an exception?
A) `throw new IOException("error");` B) `throws new IOException("error");` C) `throw IOException;` D) `throws IOException("error");`
**Answer: A**
*Explanation:* `throw` requires an actual exception INSTANCE (created via `new`), which is the correctly formed statement here.
*Why others wrong:* B) `throws` is a declaration keyword used in method signatures, not for actually throwing — mixing them is invalid syntax. C) Missing the required `new` instantiation — you can't throw a class name directly. D) Same declaration-vs-throwing keyword confusion as B.

**Q14.** [Hard | Scenario] What is the output?
```java
public static void main(String[] args) {
    try {
        System.out.println("A");
        throw new RuntimeException("error");
    } catch (RuntimeException e) {
        System.out.println("B");
    } finally {
        System.out.println("C");
    }
    System.out.println("D");
}
```
A) A B C B) A B C D C) A C D D) A B D
**Answer: B**
*Explanation:* "A" prints, exception thrown and caught printing "B", finally always runs printing "C", then execution continues normally after the try-catch-finally block, printing "D".
*Why others wrong:* A) Misses that execution continues to "D" after the whole try-catch-finally completes normally (since the exception was caught, not re-thrown). C) Skips "B" — but the exception IS successfully caught, so "B" must print. D) Misses "C" — finally always executes regardless.

**Q15.** [Medium | Checked exception handling] If a method declares `throws IOException`, what must the CALLER do?
A) Nothing, it's optional B) Either catch the IOException or declare it further with its own `throws` clause C) The caller cannot call this method at all D) Only unchecked exceptions require caller action
**Answer: B**
*Explanation:* Checked exceptions enforce a compile-time contract — the calling code must either handle it directly (try-catch) or propagate the obligation upward via its own throws declaration.
*Why others wrong:* A) Checked exceptions are explicitly NOT optional to address — ignoring this causes a compile error. C) The method remains fully callable, just with this handling obligation attached. D) This statement is backwards — checked exceptions require this handling, not unchecked ones.

**Q16.** [Hard | Nested try-finally] What is the output?
```java
public static void main(String[] args) {
    System.out.println(test());
}
static int test() {
    int x = 10;
    try {
        return x;
    } finally {
        x = 20; // modifying x AFTER return value was already captured
    }
}
```
A) 10 B) 20 C) Compile error D) 0
**Answer: A**
*Explanation:* Java captures/evaluates the return value (10) at the point of the `return` statement in try, BEFORE finally executes; modifying `x` afterward in finally doesn't retroactively change the already-captured return value (since finally doesn't itself return here).
*Why others wrong:* B) Would only apply if finally itself contained a `return x;` re-reading the updated value — it doesn't here, just a plain reassignment. C) Perfectly valid, compiling code. D) x is never actually 0 at any point in this flow.

**Q17.** [Medium | Multiple exceptions in throws] Is this valid syntax?
```java
void process() throws IOException, SQLException {
    // ...
}
```
A) No, only one exception can be declared B) Yes, multiple exceptions can be comma-separated in throws C) Compile error, needs a semicolon between them D) Only valid if both extend the same parent
**Answer: B**
*Explanation:* The `throws` clause explicitly supports declaring multiple exception types, comma-separated, when a method might throw any of several checked exceptions.
*Why others wrong:* A) Directly false — this is standard, common syntax. C) Commas are correct syntax; semicolons would be invalid here. D) No shared-parent restriction exists; IOException and SQLException are unrelated types and both can be listed together freely.

**Q18.** [Hard | StackOverflowError] What causes a `StackOverflowError`?
A) Heap memory exhaustion B) Excessive/infinite recursion exceeding stack depth C) Too many static variables D) A caught but unhandled checked exception
**Answer: B**
*Explanation:* Each recursive call adds a new frame to the thread's stack; unbounded recursion without a proper base case eventually exhausts the fixed stack size, triggering this Error.
*Why others wrong:* A) Heap exhaustion causes OutOfMemoryError, a related but distinct issue (different memory region entirely). C) Static variable count doesn't directly relate to stack depth. D) Unrelated concept entirely — this describes exception handling, not stack memory exhaustion.

**Q19.** [Medium | try-with-resources vs finally] Which is TRUE about try-with-resources compared to traditional try-finally for resource cleanup?
A) try-with-resources requires manual `.close()` calls too B) try-with-resources automatically closes resources implementing AutoCloseable, eliminating manual finally cleanup C) try-with-resources cannot handle multiple resources D) try-with-resources is only available for File-related classes
**Answer: B**
*Explanation:* This is precisely its purpose — introduced in Java 7 to reduce verbose, error-prone manual resource cleanup code in finally blocks.
*Why others wrong:* A) The entire point is AUTOMATIC closing — manual calls aren't needed. C) It fully supports multiple resources, separated by semicolons within the parentheses. D) Works with any class implementing AutoCloseable/Closeable, not limited to File-related types.

**Q20.** [Hard | Trap Summary] Which best describes the relationship between RuntimeException and checked exceptions regarding compiler enforcement?
A) Both are enforced identically by the compiler B) RuntimeException (and subclasses) are unchecked — no compile-time handling requirement; checked exceptions require explicit handling or declaration C) RuntimeException requires handling; checked exceptions do not D) Neither requires any handling at any point
**Answer: B**
*Explanation:* This is the fundamental checked-vs-unchecked distinction underpinning this entire chapter — RuntimeException subclasses bypass compile-time enforcement entirely, while checked exceptions (direct Exception subclasses, excluding RuntimeException) are strictly enforced by the compiler.
*Why others wrong:* A) They are explicitly treated differently by the compiler — this is the core distinction, not a similarity. C) Reverses the actual rule completely. D) Checked exceptions absolutely require handling/declaration or the code fails to compile.

---

## Chapter 7 Complete ✅
Watch out for: finally-with-return and finally-overriding-return (Q5, Q6, Q16) — these appear as "predict the output" questions constantly. Also nail the Error-vs-Exception sibling relationship (Q11) and catch-block-ordering compile error (Q4).

**Next up: Chapter 8 — Collections Framework (List, Set, Map, Queue, Comparable/Comparator).** This is one of the heaviest-weighted topics — say "next chapter" to continue.

# JAVA CORE — Chapter 12: File Handling, Serialization, Annotations & Reflection

## PART A — File Handling

### 1. Why File Handling? (Core idea)
Programs often need to read/write data that survives after the program ends (persisted to disk). Java's `java.io` and `java.nio` packages provide classes for this.

### 2. Key Classes — Byte Streams vs Character Streams

| Type | Base classes | Use for |
|---|---|---|
| **Byte Streams** | `InputStream` / `OutputStream` (abstract) | Binary data (images, audio, any raw bytes) |
| **Character Streams** | `Reader` / `Writer` (abstract) | Text data (handles character encoding correctly) |

| Common Class | Type | Purpose |
|---|---|---|
| `FileInputStream` / `FileOutputStream` | Byte | Read/write raw bytes from/to a file |
| `FileReader` / `FileWriter` | Character | Read/write text from/to a file |
| `BufferedReader` / `BufferedWriter` | Character (wrapper) | Adds a buffer → far fewer actual disk I/O calls → much faster |
| `BufferedInputStream` / `BufferedOutputStream` | Byte (wrapper) | Same buffering benefit, for byte streams |

**Capgemini trap:** Using `FileReader` directly (without wrapping in `BufferedReader`) to read line-by-line is inefficient and `FileReader` doesn't even have a `readLine()` method — only `BufferedReader` does. `FileReader` only has basic `read()` (single char) or `read(char[])`.

```java
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

### 3. try-with-resources — WHY it matters
Any class implementing `AutoCloseable` (all Streams/Readers/Writers do) can be declared inside `try(...)` parentheses — Java automatically calls `.close()` at the end, **even if an exception occurs**, without needing an explicit `finally` block.

**Trap:** Before Java 7 (no try-with-resources), forgetting to close a stream in a `finally` block was the #1 cause of **file handle leaks**. MCQs often ask "why is try-with-resources preferred?" → **Answer: guarantees resource closure automatically, reduces boilerplate, avoids leaks.**

### 4. `File` class vs Stream classes
`File` represents a **path/reference** to a file or directory (metadata: exists, isDirectory, length, etc.) — it does NOT itself read/write file content.

```java
File f = new File("test.txt");
System.out.println(f.exists());       // boolean
System.out.println(f.isDirectory());  // boolean
f.createNewFile();                    // creates empty file, throws IOException
```

**Trap:** `File.delete()` returns a `boolean` (true/false), it does NOT throw an exception if deletion fails — a common trick question asking "what happens if delete() fails on a locked file?" → returns `false`, no exception thrown.

## PART B — Serialization

### 5. Why Serialization? (Core idea)
Serialization converts a Java object into a **byte stream** so it can be saved to a file, sent over a network, or stored in a database — and later reconstructed (**deserialization**) back into an object.

```java
class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    String name;
    transient int salary;  // NOT serialized
}
```

### 6. Key Rules (very frequently asked)

| Rule | Detail |
|---|---|
| Class must implement | `java.io.Serializable` (a **marker interface** — no methods to implement) |
| `transient` keyword | Fields marked `transient` are **skipped** during serialization (e.g., passwords, sensitive/derived data) — deserialized as default value (0, null, false) |
| `static` fields | Never serialized (they belong to the class, not the instance) — this is automatic, no keyword needed |
| `serialVersionUID` | A version identifier; if not declared explicitly, JVM auto-generates one based on class structure — **best practice: always declare it explicitly** to avoid `InvalidClassException` when the class changes slightly between serialization and deserialization |
| Non-serializable field in a serializable class | Throws `NotSerializableException` at runtime UNLESS that field is marked `transient` |

**Capgemini trap #1:** `Serializable` is a **marker interface** — it has **zero methods**. Just implementing it is enough to signal "this class can be serialized"; you don't override anything (contrast with `Comparable`, which DOES require implementing `compareTo()`).

**Capgemini trap #2:** If a class has a field whose type does NOT implement `Serializable`, and that field is NOT `transient`, serialization throws `NotSerializableException` at **runtime** (not compile time).

**Capgemini trap #3:** `static` fields are automatically excluded from serialization — you do NOT need to (and cannot meaningfully) mark them `transient`; the exclusion happens because serialization only deals with instance state.

### 7. Serialization Example

```java
// Serialize
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("emp.ser"));
oos.writeObject(employeeObj);
oos.close();

// Deserialize
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("emp.ser"));
Employee e = (Employee) ois.readObject();  // throws ClassNotFoundException, IOException
ois.close();
```

## PART C — Annotations

### 8. What are Annotations? (Core idea)
Annotations are **metadata** attached to code (classes, methods, fields) — they don't change program logic directly but are read by the compiler, tools, or at runtime (via Reflection) to trigger behavior.

### 9. Built-in Annotations (frequently asked)

| Annotation | Purpose |
|---|---|
| `@Override` | Tells compiler "this method must override a superclass/interface method" — compile error if it doesn't actually override anything (e.g., typo in method name/signature) |
| `@Deprecated` | Marks a method/class as outdated; generates compiler warning when used |
| `@SuppressWarnings("...")` | Tells compiler to suppress specific warning types (e.g., `"unchecked"`) |
| `@FunctionalInterface` | Marks an interface as having exactly one abstract method (for lambda use); compile error if more than one abstract method exists |

**Trap:** `@Override` on a method that does NOT actually override anything from a parent (e.g., you misspelled the method name, so it's really a brand new method) causes a **compile-time error** — this is precisely WHY `@Override` is recommended: it catches typo bugs early.

### 10. Custom Annotations & Meta-Annotations

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnnotation {
    String value() default "test";
}
```

| Meta-Annotation | Purpose |
|---|---|
| `@Retention` | How long the annotation is kept: `SOURCE` (discarded by compiler), `CLASS` (in .class file, not available at runtime — default), `RUNTIME` (available via Reflection at runtime) |
| `@Target` | Where it can be applied: `METHOD`, `FIELD`, `TYPE` (class/interface), `PARAMETER`, etc. |

**Trap:** To read a custom annotation at runtime using Reflection, `@Retention(RetentionPolicy.RUNTIME)` is **mandatory** — if retention is `SOURCE` or `CLASS` (the default), Reflection cannot see it at runtime at all.

## PART D — Reflection (Basics)

### 11. What is Reflection? (Core idea)
Reflection lets a Java program **inspect and manipulate classes, methods, fields, and constructors at runtime** — even private ones — without knowing them at compile time. Frameworks like Spring and Hibernate rely heavily on Reflection internally (e.g., to inject dependencies, read annotations).

```java
Class<?> cls = Class.forName("com.example.Employee");   // or Employee.class, or obj.getClass()
Method[] methods = cls.getDeclaredMethods();             // all methods, including private
Field field = cls.getDeclaredField("salary");
field.setAccessible(true);                                // bypass private access check
Object value = field.get(employeeObj);
```

### 12. Three ways to get a `Class` object

| Method | Example | Note |
|---|---|---|
| `.class` literal | `Employee.class` | Compile-time known, no exception |
| `getClass()` | `obj.getClass()` | Needs an existing instance |
| `Class.forName(String)` | `Class.forName("com.example.Employee")` | Fully-qualified name as String; throws checked `ClassNotFoundException` |

**Trap:** `getDeclaredMethods()` returns ALL methods **declared in that class only** (including `private`), but NOT inherited ones. `getMethods()` returns all **public** methods, including inherited ones, but NOT private ones. This public-vs-declared, own-vs-inherited distinction is a classic trap.

| Method | Includes private? | Includes inherited? |
|---|---|---|
| `getMethods()` | No | Yes (public inherited included) |
| `getDeclaredMethods()` | Yes | No (own class only) |

### 13. Why Reflection is "double-edged"
- **Pros:** Enables frameworks (Spring's `@Autowired`, JUnit's `@Test` discovery, JSON libraries like Jackson) to work generically without hard-coding every class.
- **Cons:** Slower than direct code (no compile-time optimization), can break encapsulation (`setAccessible(true)` bypasses `private`), and errors surface only at runtime instead of compile time.

## One-Page Revision — Chapter 12
- Byte Streams (`InputStream`/`OutputStream`) = binary data; Character Streams (`Reader`/`Writer`) = text data.
- `BufferedReader`/`BufferedWriter` wrap raw streams to reduce actual disk I/O calls (performance). Only `BufferedReader` has `readLine()`, not plain `FileReader`.
- try-with-resources auto-closes any `AutoCloseable` (streams/readers/writers) even on exception — no manual `finally` needed.
- `File` class = path/metadata only (exists, isDirectory, delete — returns boolean, no exception on failure); it does NOT read/write content itself.
- Serialization = converting object → byte stream (and back). Class must implement `Serializable` (marker interface, zero methods).
- `transient` fields are skipped during serialization; `static` fields are automatically excluded (not instance state).
- Missing `serialVersionUID` → JVM auto-generates one; best practice: declare explicitly to avoid `InvalidClassException` on class evolution.
- Non-serializable non-transient field in a Serializable class → `NotSerializableException` at runtime.
- Annotations = metadata; `@Override` catches "doesn't actually override" typos at compile time; `@FunctionalInterface` enforces exactly one abstract method.
- Custom annotations need `@Retention(RUNTIME)` to be readable via Reflection; `@Target` restricts where they can be applied.
- Reflection inspects/manipulates classes at runtime — 3 ways to get `Class` object: `.class`, `getClass()`, `Class.forName(String)` (throws `ClassNotFoundException`).
- `getDeclaredMethods()` = own class only, includes private. `getMethods()` = public only, includes inherited. Frameworks (Spring/Hibernate) rely on Reflection internally.

---

# MCQs — Chapter 12 (25 Questions)

**Q1.** [Easy | Streams] Which pair correctly represents byte stream base classes?
A) `Reader` / `Writer` B) `InputStream` / `OutputStream` C) `BufferedReader` / `BufferedWriter` D) `File` / `Files`
**Answer: B**
*Explanation:* `InputStream` and `OutputStream` are the abstract base classes for all byte-oriented I/O in Java.
*Why others wrong:* A) Those are character stream base classes. C) Those are buffered character stream wrappers, not the base byte classes. D) Those relate to file metadata/paths, not stream I/O.

**Q2.** [Easy | Character Streams] `FileReader` and `FileWriter` are used for:
A) Binary data only B) Text/character data C) Serialized objects only D) Network sockets only
**Answer: B**
*Explanation:* They are character streams designed to read/write text data, handling character encoding.
*Why others wrong:* A) Binary data uses `FileInputStream`/`FileOutputStream`. C) Object serialization uses `ObjectOutputStream`/`ObjectInputStream`. D) Sockets have their own stream types, not directly FileReader/Writer.

**Q3.** [Medium | Trap] Which class provides the `readLine()` method for reading a text file line by line?
A) `FileReader` B) `BufferedReader` C) `FileInputStream` D) `InputStreamReader`
**Answer: B**
*Explanation:* `BufferedReader` wraps a `Reader` and adds `readLine()` along with internal buffering for efficiency; plain `FileReader` lacks this method.
*Why others wrong:* A) FileReader only offers basic `read()`/`read(char[])`, no `readLine()`. C) That's a byte stream, no line-based text reading. D) It bridges byte-to-character streams but doesn't itself add `readLine()`.

**Q4.** [Medium | try-with-resources] What is the main benefit of try-with-resources over manually closing streams in a `finally` block?
A) It runs faster at the CPU level B) It guarantees `.close()` is called automatically, even if an exception occurs, reducing boilerplate and leak risk C) It removes the need for exception handling entirely D) It only works with `File` objects
**Answer: B**
*Explanation:* Any `AutoCloseable` resource declared in the `try(...)` parentheses is automatically closed when the block exits, whether normally or via exception — no manual `finally` cleanup needed.
*Why others wrong:* A) No inherent CPU performance boost, it's about resource safety. C) You still need to catch/handle checked exceptions like `IOException`. D) It works with any `AutoCloseable`, not just File (which isn't even AutoCloseable itself).

**Q5.** [Medium | File class] What does `File.delete()` return if the file cannot be deleted (e.g., it's locked)?
A) Throws `IOException` B) Returns `false` C) Returns `null` D) Throws `FileNotFoundException`
**Answer: B**
*Explanation:* `delete()` returns a `boolean` indicating success/failure; it does not throw a checked exception on failure.
*Why others wrong:* A, D) No exception is thrown for a failed delete. C) It returns a primitive `boolean`, which can't be `null`.

**Q6.** [Easy | File class] Does the `File` class itself read or write file content?
A) Yes, directly B) No — it only represents a path/metadata; Streams/Readers/Writers handle actual content C) Only for text files D) Only for binary files
**Answer: B**
*Explanation:* `File` represents an abstract path with metadata operations (exists, isDirectory, length, delete); actual reading/writing requires stream/reader/writer classes.
*Why others wrong:* A) It doesn't read/write content at all. C, D) Neither — content I/O is entirely delegated to other classes regardless of file type.

**Q7.** [Easy | Serialization] Which interface must a class implement to be serializable?
A) `Cloneable` B) `Serializable` C) `Comparable` D) `Runnable`
**Answer: B**
*Explanation:* `java.io.Serializable` is the marker interface signaling that instances of the class can be converted to a byte stream.
*Why others wrong:* A) Cloneable enables `clone()`, unrelated to serialization. C) Comparable is for ordering objects. D) Runnable is for multithreading tasks.

**Q8.** [Medium | Trap] How many methods does the `Serializable` interface declare?
A) One — `serialize()` B) Two — `readObject()` and `writeObject()` C) Zero — it's a marker interface D) Three, matching the ObjectStream methods
**Answer: C**
*Explanation:* `Serializable` is a marker interface with no methods; simply implementing it flags the class as eligible for serialization to the JVM.
*Why others wrong:* A, B, D) Serializable itself has no such declared methods; `readObject()`/`writeObject()` can optionally be *defined* by the class for custom serialization but aren't required by the interface.

**Q9.** [Medium | transient] What happens to a `transient` field during serialization?
A) It is serialized normally B) It is skipped; on deserialization it gets the default value (0/null/false) C) It causes a compile error D) It is serialized but encrypted
**Answer: B**
*Explanation:* `transient` explicitly excludes a field from the serialized byte stream; when deserialized, that field is reset to its type's default value.
*Why others wrong:* A) That's the opposite of what `transient` does. C) It's valid, compiles fine. D) Java doesn't auto-encrypt transient fields — they're just omitted.

**Q10.** [Medium | static fields] Are `static` fields included in object serialization?
A) Yes, always B) No — static fields belong to the class, not the instance, so they're automatically excluded C) Only if marked `transient` D) Only if `serialVersionUID` is declared
**Answer: B**
*Explanation:* Serialization captures instance state; since static fields are class-level (shared), they are inherently excluded without needing `transient`.
*Why others wrong:* A) They are not included. C) `transient` is unnecessary/meaningless for statics since they're already excluded. D) `serialVersionUID` is unrelated to which fields get serialized.

**Q11.** [Hard | Trap] A `Serializable` class has a non-transient field whose type does NOT implement `Serializable`. What happens when you try to serialize an instance?
A) Compile-time error B) `NotSerializableException` at runtime C) The field is silently skipped D) `ClassCastException`
**Answer: B**
*Explanation:* The JVM only discovers the problem when actually attempting to serialize that field's value, throwing `NotSerializableException` at runtime — the compiler cannot catch this ahead of time.
*Why others wrong:* A) The compiler has no way to verify deep serializability of every field type at compile time. C) It's not silently skipped — an exception is thrown (unless marked transient). D) Wrong exception type — this isn't a casting issue.

**Q12.** [Hard | serialVersionUID] What is the purpose of `serialVersionUID`?
A) It encrypts serialized data B) It's a version identifier used to verify sender/receiver class compatibility during deserialization C) It determines which fields are transient D) It sets the file size limit for serialization
**Answer: B**
*Explanation:* During deserialization, the JVM compares the `serialVersionUID` of the serialized data against the current class; a mismatch throws `InvalidClassException`, protecting against incompatible class version changes.
*Why others wrong:* A) No encryption involved. C) Transient status is a separate field modifier, unrelated to serialVersionUID. D) It has nothing to do with file size limits.

**Q13.** [Easy | Annotations] `@Override` is used to:
A) Suppress compiler warnings B) Indicate a method overrides a superclass/interface method; causes a compile error if it doesn't actually override anything C) Mark a method as deprecated D) Enable Reflection access
**Answer: B**
*Explanation:* It tells the compiler to verify the method actually overrides something; a mismatch (e.g., typo in signature) becomes a compile-time error instead of a silent bug.
*Why others wrong:* A) That's `@SuppressWarnings`'s job. C) That's `@Deprecated`'s job. D) Retention/reflection access isn't related to `@Override`.

**Q14.** [Medium | @FunctionalInterface] What does `@FunctionalInterface` enforce?
A) Exactly one abstract method in the interface (compile error otherwise) B) At least one static method C) No default methods allowed D) The interface must be public
**Answer: A**
*Explanation:* It's a compile-time check ensuring the interface qualifies for lambda expressions by having exactly one abstract method (default/static methods are still allowed).
*Why others wrong:* B) Static methods aren't required. C) Default methods are explicitly allowed alongside the single abstract method. D) Visibility (public) isn't enforced by this annotation.

**Q15.** [Hard | Retention] To read a custom annotation via Reflection at runtime, which meta-annotation setting is REQUIRED?
A) `@Target(ElementType.METHOD)` B) `@Retention(RetentionPolicy.RUNTIME)` C) `@Retention(RetentionPolicy.SOURCE)` D) `@Documented`
**Answer: B**
*Explanation:* Only `RetentionPolicy.RUNTIME` keeps annotation data available in the class file AND loaded into the JVM at runtime, making it visible to Reflection APIs like `getAnnotation()`.
*Why others wrong:* A) `@Target` restricts where the annotation applies, unrelated to runtime visibility. C) `SOURCE` retention discards the annotation after compilation — invisible even in the .class file. D) `@Documented` only affects Javadoc generation, not runtime visibility.

**Q16.** [Medium | Retention Policies] Which `RetentionPolicy` is the DEFAULT if none is specified?
A) `SOURCE` B) `CLASS` C) `RUNTIME` D) `NONE`
**Answer: B**
*Explanation:* `RetentionPolicy.CLASS` is the default — the annotation is retained in the `.class` file but is NOT available via Reflection at runtime.
*Why others wrong:* A) SOURCE must be explicitly specified; it's more restrictive than the default. C) RUNTIME must be explicitly specified; it's not automatic. D) `NONE` isn't a valid RetentionPolicy value.

**Q17.** [Medium | Reflection] Which method retrieves a `Class` object using a fully-qualified class name as a `String`, throwing a checked exception if not found?
A) `.class` literal B) `getClass()` C) `Class.forName(String)` D) `new Class()`
**Answer: C**
*Explanation:* `Class.forName("com.example.MyClass")` dynamically loads the class by name and throws checked `ClassNotFoundException` if it can't be located.
*Why others wrong:* A) Requires compile-time known class reference, no String lookup. B) Requires an existing object instance, not a class name string. D) `Class` has no public constructor; you cannot instantiate it directly.

**Q18.** [Hard | getMethods vs getDeclaredMethods] Which statement is TRUE?
A) `getMethods()` returns private methods of the class B) `getDeclaredMethods()` returns only public methods C) `getDeclaredMethods()` returns methods declared in that class only (including private), excluding inherited ones D) `getMethods()` excludes inherited public methods
**Answer: C**
*Explanation:* `getDeclaredMethods()` gives everything declared directly in that class — public, private, protected, package — but not anything inherited from superclasses.
*Why others wrong:* A) `getMethods()` only returns public methods (own + inherited), never private. B) `getDeclaredMethods()` returns methods of ALL access levels, not just public. D) `getMethods()` DOES include inherited public methods, it doesn't exclude them.

**Q19.** [Medium | Reflection] `field.setAccessible(true)` is used to:
A) Make a field `static` B) Bypass Java's access control checks (e.g., access a `private` field via Reflection) C) Serialize the field D) Delete the field
**Answer: B**
*Explanation:* It suppresses the default Java language access checks so Reflection code can read/write even `private`/`protected` members — commonly used by frameworks.
*Why others wrong:* A) It doesn't change field modifiers like static. C) Unrelated to serialization mechanics. D) Fields can't be "deleted" via Reflection at runtime.

**Q20.** [Medium | Real-world use] Which of the following frameworks commonly relies on Reflection internally?
A) Only command-line tools B) Spring (dependency injection) and Hibernate (ORM mapping) C) Only compilers D) None — Reflection is rarely used in real frameworks
**Answer: B**
*Explanation:* Spring uses Reflection to instantiate beans and inject dependencies (`@Autowired`); Hibernate uses it to map object fields to database columns dynamically without hard-coded class references.
*Why others wrong:* A) Not specific to CLI tools. C) Compilers work at compile-time and don't use runtime Reflection for this purpose. D) Reflection is heavily used in real-world enterprise Java frameworks.

**Q21.** [Hard | Code Trace] What happens when this code runs, given `Employee` implements `Serializable` but has a field `Address address;` where `Address` does NOT implement `Serializable` and is NOT marked `transient`?
```java
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("e.ser"));
oos.writeObject(new Employee("John", new Address("MG Road")));
```
A) Compiles and runs fine B) Compile-time error C) Throws `NotSerializableException` at runtime for the `Address` field D) Silently skips the `address` field
**Answer: C**
*Explanation:* Since `address` isn't `transient` and its type isn't `Serializable`, the JVM throws `NotSerializableException` (naming the offending class) when it tries to serialize that field's value.
*Why others wrong:* A) It will fail at runtime, not run fine. B) The compiler can't detect this — deep serializability isn't a compile-time check. D) It doesn't silently skip; an exception interrupts serialization.

**Q22.** [Medium | Byte vs Character] Which scenario is MORE appropriate for a byte stream (`InputStream`/`OutputStream`) rather than a character stream?
A) Reading a plain text `.txt` log file line by line B) Copying an image (`.jpg`) file C) Reading a CSV file to parse rows D) Writing a formatted text report
**Answer: B**
*Explanation:* Images are binary data with no meaningful character encoding — byte streams handle raw bytes directly without interpretation.
*Why others wrong:* A, C, D) All involve text data, best handled via character streams (`Reader`/`Writer`) which correctly manage character encoding.

**Q23.** [Easy | Annotations] `@Deprecated` on a method causes:
A) A compile-time error whenever the method is called B) A compiler warning when the method is used, without preventing compilation C) The method to be automatically removed at compile time D) No effect at all
**Answer: B**
*Explanation:* It signals "outdated, avoid using" and triggers a compiler warning at call sites, but the code still compiles and runs.
*Why others wrong:* A) It's a warning, not a hard error — code still compiles. C) Nothing is removed automatically. D) It does have a visible effect (the warning).

**Q24.** [Hard | Scenario] You want a custom annotation to be applied ONLY to methods (not fields or classes) and readable at runtime via Reflection. Which combination is correct?
A) `@Retention(RetentionPolicy.SOURCE)` + `@Target(ElementType.METHOD)` B) `@Retention(RetentionPolicy.RUNTIME)` + `@Target(ElementType.METHOD)` C) `@Retention(RetentionPolicy.CLASS)` + `@Target(ElementType.FIELD)` D) No annotations needed, this is default Java behavior
**Answer: B**
*Explanation:* `@Retention(RUNTIME)` ensures Reflection can read it at runtime; `@Target(ElementType.METHOD)` restricts valid usage to methods only, causing a compile error if applied elsewhere.
*Why others wrong:* A) SOURCE retention would make it invisible at runtime, defeating the Reflection requirement. C) CLASS retention is also invisible at runtime, and FIELD target contradicts "methods only". D) Custom behavior always requires explicit meta-annotations; there's no such default.

**Q25.** [Medium | Trap] Which statement about Reflection is FALSE?
A) Reflection can access private fields/methods using `setAccessible(true)` B) Reflection-based code generally runs as fast as direct compiled code, with zero overhead C) Reflection is used by frameworks like Spring for dependency injection D) Reflection lets you inspect class structure (methods, fields, constructors) at runtime
**Answer: B**
*Explanation:* Reflection introduces additional runtime overhead (type-checking, security checks, no JIT-level inlining benefits) compared to direct compiled method calls — it is NOT zero-overhead.
*Why others wrong:* A, C, D) All are true, accurate descriptions of Reflection's real capabilities and use cases — only the "zero overhead" claim in B is false.

---

## Chapter 12 Complete ✅
Score yourself: if you missed Q8, Q11, Q15, Q18, or Q21 — reread Serialization rules (§6) and Reflection method distinctions (§12), the two highest-trap-density sections in this chapter.

**Java Core Multithreading → GC → File Handling/Serialization/Annotations/Reflection block is now complete.** Say "next chapter" or tell me which module/chapter to continue with (e.g., Java 8 Features, Collections deep-dive, or Module 2 - DSA).

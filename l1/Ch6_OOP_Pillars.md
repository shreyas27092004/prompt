# JAVA CORE — Chapter 6: OOP Pillars — Encapsulation, Inheritance, Polymorphism, Abstraction, Interfaces

## 1. The Four Pillars — Quick Definitions

| Pillar | Definition | How Java implements it |
|---|---|---|
| Encapsulation | Bundling data + methods, hiding internal state | private fields + public getters/setters |
| Inheritance | Acquiring properties/behavior of another class | `extends` (class), `implements` (interface) |
| Polymorphism | One interface, many forms | Method overloading (compile-time) + overriding (runtime) |
| Abstraction | Hiding implementation, showing only essential features | abstract classes, interfaces |

**Trap:** Encapsulation ≠ Abstraction. Encapsulation is about **data hiding** (protecting state via access modifiers). Abstraction is about **hiding implementation complexity** (showing what an object does, not how). They're related but tested as distinct concepts.

## 2. Inheritance

- `class Child extends Parent` — single inheritance for classes (Java does NOT support multiple class inheritance — diamond problem avoidance).
- `class MyClass implements InterfaceA, InterfaceB` — multiple interface implementation IS allowed.
- Constructors are not inherited; private members are not inherited (technically present in memory but not accessible directly).
- `Object` class is the implicit root of all classes.

**Trap — Why no multiple class inheritance?** The "Diamond Problem": if class C extends both A and B, and both A and B have a method `foo()`, the compiler can't decide which `foo()` C should inherit. Java sidesteps this by disallowing multiple class inheritance, but ALLOWS implementing multiple interfaces (with default methods resolved via explicit override rules).

## 3. Polymorphism — Two Types

| Type | Also called | Binding | Example |
|---|---|---|---|
| Compile-time | Static / Overloading | Resolved at compile time | `add(int,int)` vs `add(double,double)` |
| Runtime | Dynamic / Overriding | Resolved at runtime via actual object type | `Animal a = new Dog(); a.sound();` |

**Trap — Upcasting and method resolution:**
```java
class Animal { void sound() { System.out.println("Some sound"); } }
class Dog extends Animal { void sound() { System.out.println("Bark"); } }

Animal a = new Dog(); // Upcasting
a.sound(); // prints "Bark" — JVM checks ACTUAL object type at runtime, not reference type
```
This is the opposite of static method hiding (Chapter 5) — instance methods use the OBJECT's actual type, static methods use the REFERENCE's declared type.

## 4. Abstraction — Abstract Class vs Interface (heavily tested comparison)

| Feature | Abstract Class | Interface |
|---|---|---|
| Keyword | `abstract class` | `interface` |
| Methods | Can have both abstract AND concrete methods | Traditionally all abstract; since Java 8 can have `default` and `static` methods; Java 9+ `private` methods |
| Variables | Any type (instance, static, final, etc.) | Implicitly `public static final` (constants only) |
| Constructor | CAN have a constructor | CANNOT have a constructor |
| Multiple inheritance | A class can extend only ONE abstract class | A class can implement MULTIPLE interfaces |
| Access modifiers on methods | Any (public, protected, etc.) | Implicitly `public` (for abstract methods) |
| When to use | Related classes sharing common code/state | Unrelated classes needing a common contract/capability |

**Trap #1:** Interface variables are ALWAYS `public static final` implicitly — even if you don't write those keywords, they're added by the compiler. So interface "variables" are really constants; you can't reassign them.

**Trap #2:** Since Java 8, interfaces CAN have method bodies via `default` and `static` methods:
```java
interface Vehicle {
    default void start() { System.out.println("Starting..."); } // has a body!
    static void info() { System.out.println("This is a vehicle"); }
}
```
This is frequently tested as "Can interfaces have method bodies?" → **Yes, since Java 8 (default/static), and Java 9 (private methods)**.

**Trap #3:** A class that implements an interface but doesn't implement ALL abstract methods must itself be declared `abstract`.

## 5. Abstract Class Rules
- Declared with `abstract` keyword; CANNOT be instantiated directly (`new AbstractClass()` → compile error).
- Can have 0 or more abstract methods (methods with no body, ending in `;`).
- If a class has even ONE abstract method, the class itself MUST be declared abstract.
- Subclass must implement all abstract methods, OR itself be declared abstract.

## 6. Object Class Methods (root of all Java classes)
- `equals(Object o)` — default checks reference equality (`==`); override for content comparison.
- `hashCode()` — returns int hash; **must be overridden consistently with equals()** — this is a top Capgemini trap.
- `toString()` — default returns `ClassName@hashCodeHex`; override for readable output.

**Trap — equals/hashCode contract:** If two objects are `.equals()`, they MUST have the same `hashCode()`. Violating this breaks HashMap/HashSet behavior (objects "disappear" when used as keys).
```java
// If you override equals() but NOT hashCode(), you break the contract:
// two "equal" objects could end up in different HashMap buckets
```

## One-Page Revision
- 4 Pillars: Encapsulation (data hiding via access modifiers), Inheritance (extends/implements), Polymorphism (overload=compile-time, override=runtime), Abstraction (abstract class/interface, hides implementation).
- No multiple class inheritance (diamond problem); multiple interface implementation IS allowed.
- Instance method overriding resolved by actual OBJECT type at runtime (opposite of static method hiding, resolved by reference type).
- Abstract class: can have constructors, any variable type, mix of abstract/concrete methods, single inheritance only.
- Interface: no constructor, variables are implicitly public static final, methods implicitly public; since Java 8 supports default/static methods with bodies; multiple interfaces can be implemented.
- A class with any abstract method must itself be abstract.
- Always override hashCode() when overriding equals() — the equals/hashCode contract is critical for correct HashMap/HashSet behavior.

---

# MCQs — Chapter 6 (20 Questions)

**Q1.** [Easy | Pillars] Which OOP pillar is implemented using private fields with public getters/setters?
A) Inheritance B) Polymorphism C) Encapsulation D) Abstraction
**Answer: C**
*Explanation:* Encapsulation is specifically about bundling and protecting internal state via controlled access (private fields + public accessor methods).
*Why others wrong:* A) Inheritance is about extending classes, unrelated to field access control. B) Polymorphism is about multiple forms of a method/interface. D) Abstraction hides implementation complexity, a related but distinct concept from data hiding.

**Q2.** [Easy | Inheritance] Does Java support multiple inheritance of classes?
A) Yes, fully supported B) No, but multiple interface implementation is allowed C) Yes, but only with abstract classes D) No, inheritance isn't supported at all in Java
**Answer: B**
*Explanation:* Java disallows extending multiple classes to avoid the Diamond Problem, but explicitly permits implementing multiple interfaces.
*Why others wrong:* A) Directly false — Java has never supported multiple class inheritance. C) No such abstract-class-specific exception exists. D) Java fully supports single-class inheritance and interface implementation — inheritance absolutely exists in Java.

**Q3.** [Medium | Runtime Polymorphism] What does this print?
```java
class Animal { void sound() { System.out.println("Some sound"); } }
class Dog extends Animal { void sound() { System.out.println("Bark"); } }

Animal a = new Dog();
a.sound();
```
A) Some sound B) Bark C) Compile error D) Runtime exception
**Answer: B**
*Explanation:* Instance method calls are resolved based on the actual object's type at runtime (Dog), not the reference type (Animal) — this is dynamic method dispatch/runtime polymorphism.
*Why others wrong:* A) Would only apply to static method hiding, not instance method overriding. C, D) Valid, non-throwing, compiling code — upcasting and overriding are standard Java features.

**Q4.** [Medium | Interface Variables] What are interface variables implicitly declared as?
A) private final B) public static final C) protected static D) public volatile
**Answer: B**
*Explanation:* Any variable declared in an interface is automatically `public static final`, regardless of whether you write those keywords explicitly — making them true constants.
*Why others wrong:* A) private contradicts interfaces' inherently public nature. C) protected isn't applicable/used for interface members. D) volatile isn't automatically applied; final (immutability) is the actual implicit modifier.

**Q5.** [Medium | Abstract class instantiation] What happens when you try `new AbstractClassName()` directly on an abstract class?
A) Creates an object with default values B) Compile error — cannot instantiate an abstract class C) Runtime exception D) Creates a null object
**Answer: B**
*Explanation:* Abstract classes are explicitly designed to be incomplete/non-instantiable — attempting direct instantiation is a compile-time error.
*Why others wrong:* A) No object creation of any kind is permitted directly. C) Caught at compile time, never reaches runtime. D) "Null object" isn't a valid outcome of a failed instantiation attempt.

**Q6.** [Hard | Interface default method] Can interfaces have method bodies in modern Java (8+)?
A) No, never B) Yes, via `default` and `static` methods (and `private` methods since Java 9) C) Only in abstract classes, never interfaces D) Only if the interface has zero abstract methods
**Answer: B**
*Explanation:* Java 8 introduced default and static methods with implementations directly in interfaces; Java 9 further added private interface methods for internal code reuse.
*Why others wrong:* A) Outdated pre-Java-8 assumption — a common trap for people who learned old Java rules. C) Interfaces themselves gained this capability, independent of abstract classes. D) No such restriction exists — interfaces can mix abstract methods and default/static methods freely.

**Q7.** [Hard | equals/hashCode contract] What breaks if you override `equals()` but NOT `hashCode()`?
A) Nothing, they're independent B) HashMap/HashSet may behave inconsistently — "equal" objects could map to different buckets C) Compile error D) The equals() method stops working entirely
**Answer: B**
*Explanation:* Hash-based collections rely on the contract that equal objects must produce equal hash codes; violating this can cause duplicates in Sets or failed lookups in Maps.
*Why others wrong:* A) They are explicitly linked by contract — overriding one without the other is a common bug source. C) No compiler enforcement catches this logical violation — it's a runtime/logical issue, not compile-time. D) equals() itself still works in isolation; the problem manifests specifically in hash-based collection behavior.

**Q8.** [Medium | Abstract class vs Interface] Which of these CAN have a constructor?
A) Interface only B) Abstract class only C) Both D) Neither
**Answer: B**
*Explanation:* Abstract classes can define constructors (called via `super()` from subclasses during object construction); interfaces cannot have constructors since they're never directly instantiated and don't participate in object initialization the same way.
*Why others wrong:* A) Interfaces cannot have constructors at all. C) Only abstract classes support this, not both. D) Abstract classes DO support constructors, contradicting "neither."

**Q9.** [Medium | Diamond Problem] Why doesn't Java allow multiple class inheritance?
A) Performance reasons only B) To avoid ambiguity when two parent classes have conflicting method implementations (Diamond Problem) C) Because classes can only have one constructor D) Because Java doesn't support inheritance
**Answer: B**
*Explanation:* If a class inherited from two classes with the same method, the compiler couldn't unambiguously decide which implementation to use — Java avoids this entirely by design.
*Why others wrong:* A) Not primarily a performance consideration — it's a design/ambiguity resolution issue. C) Unrelated to constructor count. D) Java clearly supports single-class inheritance and multiple interface implementation.

**Q10.** [Hard | Trap] A class implements an interface but doesn't implement all its abstract methods. What must be true about this class?
A) It will not compile under any circumstance B) The class itself must be declared abstract C) Java auto-implements the missing methods D) The interface methods become optional automatically
**Answer: B**
*Explanation:* If a class doesn't provide implementations for all inherited abstract methods, it must itself be marked abstract, deferring full implementation to a further subclass.
*Why others wrong:* A) It CAN compile, but only if properly declared abstract. C) Java never auto-generates method implementations. D) Interface methods remain mandatory contracts — they don't become optional.

**Q11.** [Medium | toString] What does the default `toString()` (inherited from Object, unless overridden) return?
A) An empty string B) null C) ClassName@HashCodeInHex D) The object's field values as a comma-separated string
**Answer: C**
*Explanation:* Object's default toString() implementation returns the fully qualified class name concatenated with '@' and the hex representation of the hashCode.
*Why others wrong:* A) Never returns empty by default. B) toString() never returns null by default. D) That's what a properly overridden toString() might produce, but it's not automatic default behavior.

**Q12.** [Medium | Encapsulation vs Abstraction] Which best distinguishes Encapsulation from Abstraction?
A) They are identical concepts B) Encapsulation hides data (via access modifiers); Abstraction hides implementation complexity (via abstract classes/interfaces) C) Encapsulation is only for interfaces; Abstraction is only for classes D) Abstraction is a subset of Inheritance
**Answer: B**
*Explanation:* This is the precise, commonly-tested distinction — encapsulation is about protecting state, abstraction is about simplifying what's exposed to the user of a class/API.
*Why others wrong:* A) They're related but distinctly different concepts, frequently confused in MCQs specifically because of this. C) No such class/interface-exclusive restriction exists for either pillar. D) Abstraction is its own pillar, not a subset of Inheritance.

**Q13.** [Hard | Static vs Instance dispatch] Which statement correctly contrasts static method hiding with instance method overriding?
A) Both are resolved using the object's actual runtime type B) Static hiding uses reference type; instance overriding uses actual object type C) Static hiding uses actual object type; instance overriding uses reference type D) Both are resolved using the reference's declared type
**Answer: B**
*Explanation:* This exact contrast (covered across Chapters 5 & 6) is one of Capgemini's favorite "gotcha" pairings — static methods are hidden (reference-type resolved), instance methods are overridden (object-type resolved, i.e., true polymorphism).
*Why others wrong:* A) Only true for instance method overriding, not static hiding. C) Reverses the correct mapping entirely. D) Only true for static method hiding, not instance overriding.

**Q14.** [Medium | Interface multiple implementation] Can a class implement more than one interface?
A) No, only one interface per class B) Yes, a class can implement multiple interfaces C) Only abstract classes can implement multiple interfaces D) Only if the interfaces have no methods
**Answer: B**
*Explanation:* Java explicitly supports implementing any number of interfaces via comma-separated `implements InterfaceA, InterfaceB`, providing multiple-inheritance-like capability without the Diamond Problem for classes.
*Why others wrong:* A) Directly contradicts this well-established Java capability. C) No such restriction limiting this to abstract classes exists — any class (abstract or concrete) can implement multiple interfaces. D) No such restriction based on whether interfaces have methods.

**Q15.** [Hard | equals() default behavior] What does the default `equals()` method (from Object, unoverridden) actually compare?
A) Field-by-field content equality B) Reference equality (same as `==`) C) hashCode values only D) Always returns true
**Answer: B**
*Explanation:* Object's default equals() implementation simply performs a reference comparison (`this == obj`), identical to `==`, unless a subclass overrides it for content-based comparison.
*Why others wrong:* A) Field-by-field comparison only happens if you explicitly override equals() to do so — not the default behavior. C) Default equals() doesn't consult hashCode() at all. D) It only returns true when comparing an object to itself (or an identical reference), not universally.

**Q16.** [Medium | Abstract method rule] If a class contains even ONE abstract method, what must be true?
A) All other methods must also be abstract B) The class itself must be declared abstract C) The class cannot have a constructor D) The class must implement an interface
**Answer: B**
*Explanation:* Java enforces that any class containing at least one abstract (unimplemented) method must itself be marked `abstract`, since it's inherently incomplete.
*Why others wrong:* A) A class can freely mix abstract and fully-implemented (concrete) methods. C) Abstract classes CAN have constructors (called via subclasses' super()). D) No requirement to implement an interface exists — this is purely about the class's own abstract methods.

**Q17.** [Hard | Polymorphism type identification] `int add(int a, int b)` and `double add(double a, double b)` existing in the same class demonstrates:
A) Runtime polymorphism B) Compile-time polymorphism (overloading) C) Abstraction D) Encapsulation
**Answer: B**
*Explanation:* Different parameter types create genuinely different overloaded signatures, resolved at compile time based on argument types — the definition of compile-time/static polymorphism.
*Why others wrong:* A) Runtime polymorphism specifically refers to overriding with dynamic dispatch, not overloading. C, D) Unrelated pillars — this scenario is purely about polymorphism through overloading.

**Q18.** [Medium | Object class] Which of these is NOT a method inherited from the `Object` class?
A) `toString()` B) `equals()` C) `hashCode()` D) `compareTo()`
**Answer: D**
*Explanation:* `compareTo()` belongs to the `Comparable` interface, which classes must explicitly implement — it is NOT part of the universal Object class.
*Why others wrong:* A, B, C) All three are genuine Object class methods, inherited automatically by every Java class.

**Q19.** [Hard | Scenario] Which is TRUE regarding abstract class fields vs interface fields?
A) Both are implicitly public static final B) Abstract class fields can be any access modifier and mutability; interface fields are always public static final C) Interface fields can be private; abstract class fields cannot D) Neither can have fields at all
**Answer: B**
*Explanation:* Abstract classes behave like regular classes regarding fields (any modifier, any mutability) since they support full state and constructors; interfaces restrict all fields to implicit constants.
*Why others wrong:* A) Only true for interface fields, not abstract class fields — abstract classes have full flexibility. C) Interface fields cannot be private (they're always public); this reverses the actual restriction. D) Both explicitly CAN have fields — abstract classes freely, interfaces as constants only.

**Q20.** [Hard | Trap Summary] Which combination is a valid design choice in Java?
A) A class extending two abstract classes B) A class implementing two interfaces with conflicting default method signatures, without resolving the conflict C) A class implementing multiple interfaces and extending one abstract class simultaneously D) An interface extending a class
**Answer: C**
*Explanation:* Java explicitly allows a class to both extend ONE class (abstract or concrete) and implement MULTIPLE interfaces at the same time — this is a very common, valid, and frequently used pattern.
*Why others wrong:* A) Multiple class extension (even of abstract classes) is not allowed — same Diamond Problem restriction applies. B) If two implemented interfaces have conflicting default methods with the same signature, the implementing class MUST explicitly override and resolve the conflict, or it won't compile. D) Interfaces can only extend other interfaces, never classes.

---

## Chapter 6 Complete ✅
Watch out for: static-hiding vs instance-overriding dispatch (Q3, Q13) — this recurring theme spans Chapters 5-6 and is a Capgemini favorite. Also nail down the equals/hashCode contract (Q7) and the abstract-class-vs-interface comparison table — these get tested from multiple angles.

**Next up: Chapter 7 — Exception Handling (try/catch/finally, throw, throws, custom exceptions).** Say "next chapter" to continue.

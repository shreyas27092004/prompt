# JAVA CORE — Chapter 5: Methods, Overloading, Overriding, Constructors, this & super

## 1. Method Overloading vs Overriding — THE most-tested comparison in this chapter

| Feature | Overloading | Overriding |
|---|---|---|
| Definition | Same method name, different parameter list, **same class** (or subclass adding new signatures) | Same method name + same signature, subclass redefines parent's method |
| Parameters | MUST differ (number/type/order) | MUST be identical |
| Return type | Can differ (as long as params differ) | Must be same or **covariant** (subtype) |
| Access modifier | Can be anything | Cannot be MORE restrictive than parent's |
| Binding | **Compile-time** (static binding) | **Runtime** (dynamic binding) — polymorphism |
| `static` methods | Can be overloaded | Cannot be overridden (only hidden) |
| `private`/`final` methods | Can be overloaded | Cannot be overridden |
| Throws clause | No restriction | Cannot throw new/broader checked exceptions |

**Trap #1:** Changing ONLY the return type (keeping same name + same params) is **NOT** valid overloading → compile error ("method already defined").
```java
int add(int a, int b) { return a+b; }
double add(int a, int b) { return a+b; } // COMPILE ERROR — same signature, different return type only
```

**Trap #2:** Overriding access modifier can only stay the same or become **less** restrictive (more public), never more restrictive.
```java
class Parent { public void show() {} }
class Child extends Parent {
    void show() {} // COMPILE ERROR — reducing public to default/package-private
}
```

**Trap #3:** Static methods are NOT overridden — they're "hidden" (method hiding). Which version runs depends on the **reference type**, not the object type (opposite of instance method overriding, which depends on actual object type).

```java
class Parent { static void greet() { System.out.println("Parent"); } }
class Child extends Parent { static void greet() { System.out.println("Child"); } }

Parent p = new Child();
p.greet(); // prints "Parent" — static methods resolved by REFERENCE type, not object type
```

## 2. Constructors

- Same name as class, no return type (not even void).
- Called automatically when `new` is used.
- If you don't write ANY constructor, Java provides a default no-arg constructor. If you write ANY constructor (even parameterized), the default one is NOT auto-generated anymore.
- Constructors can be overloaded (constructor overloading) but NOT overridden (they're not inherited).
- `this()` — calls another constructor in the SAME class (constructor chaining). Must be the **first statement**.
- `super()` — calls the parent class's constructor. Also must be the **first statement**. If omitted, Java inserts an implicit `super()` call automatically.

**Trap:** You cannot use BOTH `this()` and `super()` in the same constructor — only one first-statement is allowed.

```java
class A {
    A() { System.out.println("A constructor"); }
}
class B extends A {
    B() {
        System.out.println("B constructor");
        // implicit super() inserted here automatically BEFORE this print in reality —
        // actual output order: "A constructor" THEN "B constructor"
    }
}
```

## 3. `this` keyword — 4 uses
1. Refer to current object's instance variable (resolving shadowing): `this.name = name;`
2. Call another constructor in same class: `this();`
3. Pass current object as argument: `someMethod(this);`
4. Return current object: `return this;` (method chaining)

## 4. `super` keyword — 3 uses
1. Call parent constructor: `super();`
2. Access parent's instance variable (if shadowed): `super.name`
3. Call parent's overridden method: `super.methodName();`

## 5. static, final Keywords

| Keyword | On variable | On method | On class |
|---|---|---|---|
| static | Class-level, shared, one copy | Belongs to class, no `this`, cannot access instance members directly | (N/A, only nested classes can be static) |
| final | Constant, cannot be reassigned | Cannot be overridden | Cannot be extended/subclassed |

**Trap:** A `static` method cannot directly call a non-static (instance) method or access an instance variable, because static methods don't have an implicit `this` — no object context exists.

## 6. Access Modifiers (must memorize exactly)

| Modifier | Same class | Same package | Subclass (diff package) | Different package |
|---|---|---|---|---|
| private | ✅ | ❌ | ❌ | ❌ |
| default (no modifier) | ✅ | ✅ | ❌ | ❌ |
| protected | ✅ | ✅ | ✅ | ❌ |
| public | ✅ | ✅ | ✅ | ✅ |

## One-Page Revision
- Overloading = compile-time, same class, different params, return type alone insufficient to differentiate.
- Overriding = runtime, subclass, identical signature, access can only widen (not narrow), can't override static/private/final methods.
- Static method "hiding" is resolved by reference type; instance method overriding is resolved by actual object type.
- No constructor written → default no-arg constructor auto-generated. Any constructor written → default is NOT auto-generated.
- `this()`/`super()` must be first line; can't use both in one constructor.
- `this` → current object reference. `super` → parent class reference.
- static method: no `this`, can't directly access instance members.
- Access modifiers order (most to least restrictive): private < default < protected < public.

---

# MCQs — Chapter 5 (20 Questions)

**Q1.** [Easy | Overloading] Which of these is a valid method overload of `void show(int a)`?
A) `void show(int b)` B) `int show(int a)` C) `void show(int a, int b)` D) `void show(int a) throws Exception`
**Answer: C**
*Explanation:* Overloading requires a different parameter list; adding a second parameter creates a genuinely distinct signature.
*Why others wrong:* A) Same signature, parameter name doesn't matter — this is a duplicate, compile error. B) Same params, only return type differs — NOT valid overloading (Trap #1). D) Same signature; throws clause differences don't count as overloading.

**Q2.** [Easy | Overriding] Which binding type is used for method overriding?
A) Compile-time (static) B) Runtime (dynamic) C) Both equally D) Neither — overriding uses no binding
**Answer: B**
*Explanation:* Overriding relies on the actual object type at runtime to determine which version executes — this is the foundation of runtime polymorphism.
*Why others wrong:* A) That's overloading's binding mechanism, not overriding's. C, D) Overriding is specifically and exclusively runtime-bound.

**Q3.** [Medium | Access Modifier Trap] What happens here?
```java
class Parent { public void greet() {} }
class Child extends Parent {
    void greet() {} // package-private
}
```
A) Compiles fine, valid override B) Compile error — reducing visibility C) Runtime exception D) Valid overload
**Answer: B**
*Explanation:* Overriding cannot reduce the access level of the parent's method; `public` cannot become package-private in the child.
*Why others wrong:* A) This directly violates the access-widening-only rule. C) The error is caught at compile time, not runtime. D) The signature is identical, so it's an override attempt, not an overload.

**Q4.** [Medium | Static Method Hiding] What does this print?
```java
class Parent { static void greet() { System.out.println("Parent"); } }
class Child extends Parent { static void greet() { System.out.println("Child"); } }

Parent p = new Child();
p.greet();
```
A) Parent B) Child C) Compile error D) Runtime exception
**Answer: A**
*Explanation:* Static methods are resolved based on the REFERENCE type (`Parent p`), not the actual object type, because static "overriding" is really just method hiding, not polymorphic dispatch.
*Why others wrong:* B) Would be correct if this were instance method overriding (polymorphism), but static methods don't behave this way. C, D) Valid, compiling, non-throwing code.

**Q5.** [Medium | Constructor] If class `Test` has NO explicitly defined constructor, what happens when you call `new Test()`?
A) Compile error B) Java provides a default no-arg constructor automatically C) Runtime exception D) Object created with null constructor
**Answer: B**
*Explanation:* The compiler auto-generates a public no-arg constructor with an empty body if no constructor is explicitly declared.
*Why others wrong:* A) Perfectly valid — this is exactly what enables `new Test()` to compile. C) No exception involved; it's standard, well-defined behavior. D) Not meaningful terminology — objects don't have "null constructors."

**Q6.** [Hard | Constructor Trap] What happens here?
```java
class Test {
    Test(int x) { System.out.println(x); }
}
// elsewhere:
Test t = new Test();
```
A) Compiles fine, calls default constructor B) Compile error — no matching constructor found C) Runtime exception D) Prints 0
**Answer: B**
*Explanation:* Since a parameterized constructor was explicitly defined, Java does NOT auto-generate the no-arg default constructor anymore — `new Test()` has no matching constructor.
*Why others wrong:* A) This is the exact trap — the "free" default constructor disappears once ANY constructor is written. C) The error is caught at compile time, before any code runs. D) Nothing runs since compilation fails.

**Q7.** [Hard | this/super] Which statement about `this()` and `super()` is TRUE?
A) Both can be used in the same constructor, in any order B) Only one of them can be used per constructor, and it must be the first statement C) Neither needs to be the first statement D) super() is optional and never called automatically
**Answer: B**
*Explanation:* Only one constructor-chaining call (either this() OR super()) is allowed per constructor, and Java requires it to be the very first statement.
*Why others wrong:* A) Using both simultaneously is a compile error. C) Both, when used, are strictly required to be first. D) If omitted, Java automatically inserts an implicit no-arg `super()` call — it's not optional in effect.

**Q8.** [Medium | static restriction] Why can't a static method directly access an instance (non-static) variable?
A) Because static methods run before the JVM starts B) Because static methods have no `this` reference / no associated object context C) Because instance variables are private by default D) Because static methods can only return primitives
**Answer: B**
*Explanation:* Static methods belong to the class itself, not any object instance — without an object, there's no `this` to resolve instance-level data.
*Why others wrong:* A) Static methods run within normal JVM execution, not before startup. C) Access modifier isn't the issue here — even public instance variables can't be accessed directly from a static context without an object reference. D) Return type has nothing to do with this restriction.

**Q9.** [Medium | Covariant return] Overriding allows the return type to be:
A) Completely different and unrelated B) Exactly the same type only C) The same type or a subtype (covariant return) D) Any primitive type regardless of parent's return type
**Answer: C**
*Explanation:* Since Java 5, covariant return types are allowed — an overriding method can return a subtype of the original return type.
*Why others wrong:* A) Return types must maintain a type relationship, not be arbitrary. B) Overly restrictive — subtypes are explicitly permitted too. D) No such blanket primitive-only rule exists.

**Q10.** [Hard | this keyword] What does `this` refer to inside a constructor?
A) The parent class B) The class itself (as a type) C) The current object instance being constructed D) A static reference shared by all objects
**Answer: C**
*Explanation:* `this` always refers to the specific object instance on which the current method/constructor is operating.
*Why others wrong:* A) That's what `super` refers to. B) `this` is an object reference, not a type/class reference. D) `this` is per-instance, explicitly NOT shared/static.

**Q11.** [Medium | final method] Can a `final` method be overridden in a subclass?
A) Yes, always B) No, never C) Only if the subclass is in the same package D) Only if explicitly annotated with @Override
**Answer: B**
*Explanation:* `final` on a method explicitly prevents any subclass from overriding it — this is a compile-time enforced restriction.
*Why others wrong:* A) Directly contradicts the purpose of `final`. C) Package location is irrelevant; final blocks overriding universally. D) The @Override annotation doesn't bypass the final restriction — attempting to override still fails to compile.

**Q12.** [Medium | Overloading resolution] Given `void test(int x)` and `void test(double x)`, which is called for `test(5)`?
A) test(double x), because int is widened to double B) test(int x), exact match preferred over widening C) Compile error, ambiguous D) Runtime decides based on JVM version
**Answer: B**
*Explanation:* Java's overload resolution always prefers an exact type match over an implicit widening conversion when both are available.
*Why others wrong:* A) Widening would only apply if no exact int match existed. C) Not ambiguous — Java has clear, deterministic resolution rules. D) Resolved at compile time, not runtime, and not JVM-version-dependent.

**Q13.** [Hard | super chaining] What is the output?
```java
class A {
    A() { System.out.println("A"); }
}
class B extends A {
    B() { System.out.println("B"); }
}
public class Main {
    public static void main(String[] args) {
        new B();
    }
}
```
A) B B) A C) A then B D) B then A
**Answer: C**
*Explanation:* Java automatically inserts an implicit `super()` as the first line of B's constructor, so A's constructor runs completely before B's body executes.
*Why others wrong:* A) Ignores that A's constructor also runs. B) Ignores that B's own constructor body still executes after A's. D) Reverses the actual parent-first execution order.

**Q14.** [Medium | Overloading with array/varargs] Which pair represents valid method overloading?
A) `void f(int a)` and `void f(int b)` B) `void f(int... a)` and `void f(int a)` C) `void f(int a)` and `int f(int a)` D) `void f(int a) {}` twice in the same class body
**Answer: B**
*Explanation:* Varargs (`int...`) and a single int parameter have genuinely different signatures at the bytecode level, making this valid overloading (though calling with exactly one int arg can create ambiguity resolved by preferring the non-varargs version).
*Why others wrong:* A) Identical signature (parameter name doesn't matter) — duplicate declaration, compile error. C) Same params, differing only by return type — invalid, as established in Trap #1. D) Literal duplicate method — compile error.

**Q15.** [Hard | Access modifier widening] Which override is VALID?
```java
class Parent { protected void show() {} }
```
A) `private void show()` in child B) `void show()` (default) in child C) `public void show()` in child D) None are valid
**Answer: C**
*Explanation:* protected → public is widening (less restrictive), which is explicitly allowed by overriding rules.
*Why others wrong:* A) private is MORE restrictive than protected — invalid narrowing. B) default (package-private) is also more restrictive than protected — invalid narrowing. D) Option C is valid, so this is incorrect.

**Q16.** [Medium | Constructor overloading] Can constructors be overloaded?
A) No, constructors can only exist once per class B) Yes, multiple constructors with different parameter lists are allowed C) Only if they're all private D) Only in abstract classes
**Answer: B**
*Explanation:* Constructor overloading is a standard, common pattern — multiple constructors differentiated by parameter lists, enabling flexible object creation.
*Why others wrong:* A) Directly false — overloading constructors is a core Java feature. C) No such restriction tied to private access exists. D) Not restricted to abstract classes at all — applies to any class.

**Q17.** [Hard | Trap] Can a constructor be inherited by a subclass?
A) Yes, always B) No — constructors are never inherited, though subclass constructors can invoke parent constructors via super() C) Only default constructors are inherited D) Only if marked protected
**Answer: B**
*Explanation:* Constructors are tied specifically to their declaring class and are never inherited outright; subclasses must define their own, though they implicitly or explicitly invoke the parent's via `super()`.
*Why others wrong:* A) Contradicts fundamental constructor behavior in Java. C) No such partial-inheritance exception exists for default constructors. D) Access modifiers don't affect constructor inheritance rules — they're never inherited regardless.

**Q18.** [Medium | this() chaining] What's the purpose of `this(args)` inside a constructor?
A) Calls the superclass constructor B) Calls another constructor in the same class (constructor chaining) C) Creates a new object of the same class D) Refers to a static variable
**Answer: B**
*Explanation:* `this(...)` redirects to a different constructor overload within the SAME class, useful for reducing duplicate initialization code.
*Why others wrong:* A) That's `super(...)`'s job, not `this(...)`. C) It doesn't create a new object — it's still initializing the current one being constructed. D) Unrelated to static variables entirely.

**Q19.** [Hard | Overriding + exceptions] If a parent method declares `throws IOException`, can the overriding method throw a broader checked exception like `Exception`?
A) Yes, any exception is allowed B) No — the overriding method cannot throw a broader/new checked exception than the parent declares C) Only if it's a RuntimeException D) Only if annotated with @SuppressWarnings
**Answer: B**
*Explanation:* Overriding rules restrict checked exceptions to being the same, narrower, or none at all compared to the parent's declared exceptions — broadening violates the overriding contract and fails to compile.
*Why others wrong:* A) Directly violates checked-exception overriding restrictions. C) Unchecked (Runtime) exceptions aren't subject to this restriction at all, but that's a separate point from the given restrictive answer needed here. D) Annotations don't override this fundamental language rule.

**Q20.** [Medium | Trap Summary] Which of these CANNOT be overridden?
A) public methods B) protected methods C) static methods D) All of the above can be overridden
**Answer: C**
*Explanation:* Static methods are hidden, not overridden — since they don't participate in runtime polymorphism/dynamic dispatch, they don't fit the technical definition of "overriding."
*Why others wrong:* A, B) Both public and protected instance methods are prime candidates for standard overriding. D) Contradicted directly by the correct answer C.

---

## Chapter 5 Complete ✅
Watch out for: static method "hiding" vs instance overriding (Q4, Q20) — this distinction is asked constantly. Also re-check the constructor-disappears-when-you-write-your-own trap (Q6) and access-modifier-can-only-widen rule (Q3, Q15).

**Next up: Chapter 6 — OOP Pillars: Encapsulation, Inheritance, Polymorphism, Abstraction, Interfaces & Abstract Classes.** Say "next chapter" to continue.

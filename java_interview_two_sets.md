# Java Interview Preparation — Two Sets

**Coverage:** Java Fundamentals, OOPS, Arrays, Strings, Exception Handling  
**Format:** 25 questions per set, plus only 2 coding programs in total.

---

# Set 1 — 25 Questions and Answers

## Java Fundamentals

### 1. What is Java?
Java is a high-level, class-based programming language. Java source code is compiled into bytecode, which runs on a compatible Java Virtual Machine (JVM).

### 2. What are JDK, JRE, and JVM?
- **JDK:** Tools for developing Java applications, including the compiler.
- **JRE:** Traditionally, the runtime components needed to run Java applications, including the JVM and libraries.
- **JVM:** Executes Java bytecode.

*Note: Modern Java distributions may not offer a separate JRE installation.*

### 3. Why is Java platform-independent?
Java is compiled into platform-independent bytecode. The bytecode can run on any operating system that has a compatible JVM.

### 4. What are Java's eight primitive data types?
`byte`, `short`, `int`, `long`, `float`, `double`, `char`, and `boolean`.

### 5. What is the difference between `==` and `equals()`?
For primitives, `==` compares values. For objects, `==` compares references. `equals()` compares object contents when the class implements an appropriate equality check.

## OOPS

### 6. What is OOPS?
Object-Oriented Programming is a programming approach organized around classes and objects. Its four commonly taught principles are encapsulation, inheritance, polymorphism, and abstraction.

### 7. What is a class and an object?
A **class** defines a type's fields and methods. An **object** is an instance of a class.

### 8. What is encapsulation?
Encapsulation combines data and methods in a class and controls access to data, commonly by using `private` fields and public methods.

### 9. What is inheritance?
Inheritance allows a class to acquire accessible members and behavior from a superclass. In Java, class inheritance uses `extends`.

### 10. What is polymorphism?
Polymorphism allows the same method name or interface to represent different behaviors. Method overloading is resolved at compile time; overridden instance methods are dynamically dispatched at runtime.

### 11. What is abstraction?
Abstraction exposes essential functionality while hiding implementation details. Java supports abstraction through abstract classes and interfaces.

### 12. What is the difference between an abstract class and an interface?
An abstract class can contain instance state, constructors, abstract methods, and concrete methods. An interface defines a contract and can contain abstract, default, static, and private methods; its fields are implicitly `public static final`. A class can extend one class but implement multiple interfaces.

## Arrays

### 13. What is an array in Java?
An array is an object that stores a fixed number of elements of one component type. Its elements are accessed by zero-based indexes.

### 14. Can an array's size change after creation?
No. An array's length is fixed when it is created. To change capacity, create another array or use a resizable collection such as `ArrayList`.

### 15. What happens when an invalid array index is accessed?
Java throws an `ArrayIndexOutOfBoundsException`.

### 16. What are the default values in an array?
Numeric primitive elements default to zero (or the appropriate zero value), `boolean` elements to `false`, `char` elements to `'\u0000'`, and reference elements to `null`.

### 17. What is the difference between an array and an ArrayList?
An array has a fixed length and can hold primitives or references. `ArrayList` is resizable, belongs to the Collections Framework, and stores objects (primitive values are represented by wrapper classes).

## Strings

### 18. What is a String in Java?
`String` is a Java class used to represent text as a sequence of characters. String objects are immutable.

### 19. What does immutable mean for String?
It means a String object's contents cannot be changed after creation. Operations that appear to modify a String return a String value rather than changing the original object.

### 20. What is the difference between String, StringBuilder, and StringBuffer?
`String` is immutable. `StringBuilder` is mutable and generally preferred for repeated modifications in single-threaded code. `StringBuffer` is mutable and its methods are synchronized.

## Exception Handling

### 21. What is exception handling?
Exception handling is a mechanism for responding to exceptional conditions so the program can handle errors appropriately rather than terminating unexpectedly.

### 22. What is the difference between checked and unchecked exceptions?
Checked exceptions must be caught or declared, as required by the compiler. Unchecked exceptions (subclasses of `RuntimeException`) do not have that requirement.

### 23. What is the purpose of `try` and `catch`?
The `try` block contains code that may throw an exception. A matching `catch` block handles an exception if one is thrown.

### 24. What is the purpose of `finally`?
`finally` is commonly used for cleanup and normally executes whether or not an exception occurs, except in situations such as abrupt JVM termination.

### 25. What is the difference between `throw` and `throws`?
`throw` explicitly throws an exception object. `throws` declares exceptions a method may pass to its caller.

---

# Set 2 — 25 Questions and Answers

## Java Fundamentals

### 1. What is bytecode?
Bytecode is the platform-independent instruction format produced by the Java compiler and stored in `.class` files. A compatible JVM executes it.

### 2. What is the difference between local, instance, and static variables?
- **Local variable:** Declared inside a method, constructor, or block; it must be initialized before use.
- **Instance variable:** Belongs to an object.
- **Static variable:** Belongs to the class and is shared by its instances.

### 3. What are relational operators in Java?
They compare values and produce a boolean result. Examples include `==`, `!=`, `<`, `>`, `<=`, and `>=`.

### 4. What is the difference between `break` and `continue`?
`break` exits the nearest loop or switch. `continue` skips the remainder of the current loop iteration and proceeds to the next iteration.

### 5. What is the `static` keyword?
`static` declares a member associated with the class rather than an individual instance. A top-level class cannot be declared `static`, but a nested class can.

## OOPS

### 6. What is a constructor?
A constructor initializes a newly created object. It has the same name as the class and no return type.

### 7. What is method overloading?
Method overloading means defining methods with the same name but different parameter lists. Changing only the return type is not sufficient.

### 8. What is method overriding?
Method overriding occurs when a subclass provides a compatible implementation of an inherited instance method, enabling runtime polymorphism.

### 9. What is multilevel inheritance?
Multilevel inheritance forms a chain: one class extends a second class, and that second class extends another class. Java supports this with classes.

### 10. Can Java support multiple inheritance?
A class cannot extend multiple classes. However, it can implement multiple interfaces.

### 11. What is the difference between `this` and `super`?
`this` refers to the current object and can invoke another constructor in the same class. `super` accesses superclass members and can invoke a superclass constructor.

### 12. What is a `final` class?
A `final` class cannot be extended. A `final` method cannot be overridden, and a `final` variable can be assigned only according to its final-variable initialization rules.

## Arrays

### 13. What is a multidimensional array?
It is an array whose elements are themselves arrays. Java supports nested arrays, including rows of different lengths.

### 14. How do you get an array's length?
Use the `length` field, for example `arr.length`. Unlike a String, an array does not use a `length()` method.

### 15. What is linear search?
Linear search examines elements one by one until the target is found or there are no elements left. Its worst-case time complexity is O(n).

### 16. Can an array store different data types?
An array has one declared component type. An array of `Object` can hold references to different object types, but a primitive array such as `int[]` holds only `int` values.

### 17. What is the difference between `arr.length` and `list.size()`?
`arr.length` is the field giving an array's fixed length. `list.size()` is a method returning the current number of elements in a collection such as a List.

## Strings

### 18. What is the String Constant Pool?
It is a JVM-managed area of the heap used to reuse interned String instances.

### 19. How do `equals()` and `equalsIgnoreCase()` differ for Strings?
`equals()` compares String contents with case sensitivity. `equalsIgnoreCase()` compares contents without treating letter case as significant.

### 20. What do `charAt()` and `substring()` do?
`charAt(index)` returns the character at a zero-based index. `substring(begin, end)` returns the portion from `begin` inclusive to `end` exclusive.

## Exception Handling

### 21. What is a `NullPointerException`?
It is an unchecked exception that can occur when code uses a null reference in a way that requires an actual object.

### 22. What is the difference between an Exception and an Error?
Both extend `Throwable`. Exceptions represent conditions an application may handle; Errors generally indicate serious JVM or environment problems that application code usually should not try to handle.

### 23. Can a `try` block exist without a `catch` block?
Yes. It can be followed by a `finally` block, or it can use try-with-resources with the appropriate syntax.

### 24. What is a custom exception?
A custom exception is a developer-defined exception class for application-specific error conditions. It can extend `Exception` or `RuntimeException`, depending on whether a checked or unchecked exception is desired.

### 25. Why should exceptions not be swallowed?
Ignoring an exception without handling or reporting it can hide failures and make debugging difficult. Handle it meaningfully, recover where possible, or propagate it to a suitable layer.

---

# Two Basic Java Coding Programs

These are the **only two coding programs** included in this document.

## Program 1: Reverse a String (Brute Force)

**Question:** Reverse a String without using `StringBuilder.reverse()`.

**Input:** `Shreyas`  
**Output:** `sayerhS`

```java
public class Main {
    public static void main(String[] args) {
        String str = "Shreyas";
        String rev = "";

        for (int i = str.length() - 1; i >= 0; i--) {
            rev = rev + str.charAt(i);
        }

        System.out.println(rev);
    }
}
```

**Explanation:** Start from the last character and append each character to a new String until the first character is reached.

**Time complexity:** O(n²) in the worst case due to repeated String concatenation.  
**Space complexity:** O(n) for the resulting String, excluding temporary allocations.

## Program 2: Check Whether a Number Is Prime (Brute Force)

**Question:** Check whether a number is prime.

**Input:** `7`  
**Output:** `Prime`

```java
public class Main {
    public static void main(String[] args) {
        int n = 7;
        boolean prime = true;

        if (n <= 1) {
            prime = false;
        } else {
            for (int i = 2; i < n; i++) {
                if (n % i == 0) {
                    prime = false;
                    break;
                }
            }
        }

        if (prime) {
            System.out.println("Prime");
        } else {
            System.out.println("Not Prime");
        }
    }
}
```

**Explanation:** Numbers less than or equal to 1 are not prime. Check every integer from 2 to `n - 1`; if any divides `n` evenly, it is not prime.

**Time complexity:** O(n)  
**Space complexity:** O(1)

---

## Quick Preparation Checklist

- Practise answering all 50 theory questions aloud.
- Understand the four OOPS principles and their differences.
- Review array indexing, String immutability, and exception keywords.
- Write the two coding programs from memory and explain their logic.

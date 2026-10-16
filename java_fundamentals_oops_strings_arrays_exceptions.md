# Java Fundamentals Interview Preparation Guide

A beginner-friendly guide to Java Fundamentals, OOPS, Strings, Arrays, Exception Handling, and five basic coding programs.

---

## 1. Java Fundamentals

### What is Java?
Java is a high-level, object-oriented programming language. Java source code is compiled into bytecode, which runs on the Java Virtual Machine (JVM).

### JVM, JRE, and JDK

| Term | Full form | Purpose |
|---|---|---|
| JVM | Java Virtual Machine | Executes Java bytecode. |
| JRE | Java Runtime Environment | Provides the JVM and libraries needed to run Java applications. |
| JDK | Java Development Kit | Provides tools to develop Java programs, including the compiler and runtime components. |

**Remember:** JDK is for developing; the runtime environment is for running; the JVM executes bytecode.

### Basic Java program

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

- `class` defines a class.
- `main()` is the program entry point.
- `System.out.println()` prints text followed by a new line.

### Operators

Operators perform operations on values and variables.

| Category | Operators | Example |
|---|---|---|
| Arithmetic | `+`, `-`, `*`, `/`, `%` | `10 % 3` gives `1` |
| Relational | `==`, `!=`, `>`, `<`, `>=`, `<=` | `a > b` |
| Logical | `&&`, `||`, `!` | `a > 0 && b > 0` |
| Assignment | `=`, `+=`, `-=`, `*=`, `/=` | `x += 2` |
| Unary | `++`, `--`, `+`, `-`, `!` | `i++` |
| Bitwise | `&`, `|`, `^`, `~` | `a & b` |
| Shift | `<<`, `>>`, `>>>` | `n >> 1` |
| Conditional | `?:` | `int max = a > b ? a : b;` |
| Type comparison | `instanceof` | `obj instanceof String` |

**Important:** `=` assigns a value. `==` compares primitive values; for objects, it checks whether references point to the same object. Use `equals()` to compare object contents when appropriate.

### Access modifiers

Access modifiers control where classes, methods, and fields can be accessed.

| Modifier | Same class | Same package | Subclass in another package | Other packages |
|---|---:|---:|---:|---:|
| `private` | Yes | No | No | No |
| package-private (no modifier) | Yes | Yes | No, unless access is through package membership | No |
| `protected` | Yes | Yes | Yes, subject to Java's subclass access rules | No, except through permitted subclass access |
| `public` | Yes | Yes | Yes | Yes |

Top-level classes can generally be `public` or package-private; they cannot be `private` or `protected`.

---

## 2. OOPS Concepts

OOPS means Object-Oriented Programming System/Style. Java uses classes and objects to organize code.

### Class and object
- **Class:** A blueprint describing data and behavior.
- **Object:** An instance of a class.

```java
class Car {
    String color;

    void drive() {
        System.out.println("Car is moving");
    }
}

class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.color = "Red";
        car.drive();
    }
}
```

### Four main OOPS principles

#### 1. Encapsulation
Encapsulation bundles data and methods together and restricts direct access to internal data. A common approach is to make fields `private` and provide public methods.

```java
class Student {
    private int age;

    public void setAge(int age) {
        if (age >= 0) {
            this.age = age;
        }
    }

    public int getAge() {
        return age;
    }
}
```

#### 2. Inheritance
Inheritance allows one class to acquire accessible fields and methods from another class. Java uses `extends` for class inheritance.

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println("Barking");
    }
}
```

Java classes support single inheritance: a class can extend only one class. A class can implement multiple interfaces.

#### 3. Polymorphism
Polymorphism means “many forms.”

- **Compile-time polymorphism:** Usually achieved through method overloading—same method name, different parameter lists.
- **Runtime polymorphism:** Achieved through method overriding, where the method implementation is selected based on the actual object at runtime.

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    int add(int a, int b, int c) {
        return a + b + c;
    }
}
```

Changing only the return type does not overload a method.

#### 4. Abstraction
Abstraction exposes essential behavior while hiding implementation details. Java supports abstraction through abstract classes and interfaces.

```java
abstract class Shape {
    abstract double area();
}

class Circle extends Shape {
    private final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}
```

### More important OOPS terms

- **Constructor:** Initializes a new object. It has the class name and no return type.
- **`this`:** Refers to the current object.
- **`super`:** Refers to superclass members or invokes a superclass constructor.
- **`static`:** A member belongs to the class rather than to each instance.
- **`final` variable:** Can be assigned only once.
- **`final` method:** Cannot be overridden.
- **`final` class:** Cannot be extended.
- **Interface:** Defines a contract that implementing classes agree to fulfill. Interfaces can also contain permitted default, static, and private methods.
- **Overloading:** Same method name with different parameter lists.
- **Overriding:** A subclass provides a compatible implementation of an inherited instance method.

---

## 3. Strings

### What is a String?
A `String` represents a sequence of characters. Java strings are objects and are **immutable**: after a `String` object is created, its contents cannot be changed.

```java
String s = "Java";
s.concat(" Programming");
System.out.println(s); // Java
```

`concat()` returns a new string. To retain the result:

```java
s = s.concat(" Programming");
System.out.println(s); // Java Programming
```

### String Constant Pool
The String Constant Pool is a JVM-managed area used to reuse string literals.

```java
String a = "Java";
String b = "Java";
System.out.println(a == b); // true: same pooled literal reference
```

Using `new` creates a distinct `String` object:

```java
String c = new String("Java");
System.out.println(a == c);      // false: different references
System.out.println(a.equals(c)); // true: same text
```

- `==` compares references when used with objects.
- `equals()` compares string contents.

### Common String methods

| Method | Purpose | Example |
|---|---|---|
| `length()` | Returns number of characters | `"Java".length()` → `4` |
| `charAt(index)` | Returns character at an index | `"Java".charAt(0)` → `'J'` |
| `substring(start)` | Returns a substring from start index | `"Java".substring(2)` → `"va"` |
| `substring(start, end)` | Uses start-inclusive, end-exclusive indexes | `"Java".substring(1, 3)` → `"av"` |
| `equals()` | Compares contents, case-sensitive | `"java".equals("Java")` → `false` |
| `equalsIgnoreCase()` | Compares while ignoring case | `"java".equalsIgnoreCase("Java")` → `true` |
| `contains()` | Checks for a sequence | `"Java".contains("av")` → `true` |
| `indexOf()` | Finds first matching index, or `-1` | `"Java".indexOf('a')` → `1` |
| `toUpperCase()` | Converts to uppercase | `"java".toUpperCase()` → `"JAVA"` |
| `toLowerCase()` | Converts to lowercase | `"JAVA".toLowerCase()` → `"java"` |
| `trim()` | Removes leading/trailing characters recognized as <= U+0020 | `" Java ".trim()` → `"Java"` |
| `strip()` | Removes leading/trailing Unicode whitespace (Java 11+) | `" Java ".strip()` → `"Java"` |
| `replace()` | Replaces matching characters/sequences | `"banana".replace('a', 'o')` → `"bonono"` |
| `isEmpty()` | Checks whether length is zero | `"".isEmpty()` → `true` |

### String, StringBuilder, and StringBuffer

- **String:** Immutable; good when text does not need repeated modification.
- **StringBuilder:** Mutable and generally preferred for repeated string changes in single-threaded code.
- **StringBuffer:** Mutable and synchronized for thread-safe use, with additional synchronization overhead.

---

## 4. Arrays

### What is an array?
An array stores a fixed number of values of the same declared type. Array indexes start at `0`.

### Declaration and initialization

```java
int[] numbers = new int[3]; // [0, 0, 0]
numbers[0] = 10;
numbers[1] = 20;
numbers[2] = 30;

int[] values = {5, 10, 15};
System.out.println(values[0]); // 5
System.out.println(values.length); // 3
```

The last valid index is `array.length - 1`. Accessing an invalid index throws `ArrayIndexOutOfBoundsException`.

### Default values
When an array is created, its elements receive default values.

| Element type | Default value |
|---|---|
| Integral primitives (`byte`, `short`, `int`, `long`) | `0` |
| Floating-point primitives (`float`, `double`) | `0.0` |
| `char` | `'\u0000'` |
| `boolean` | `false` |
| Reference types (for example, `String`) | `null` |

Local variables inside methods do not automatically receive these defaults; they must be initialized before use.

### Loop through an array

```java
int[] numbers = {10, 20, 30};

for (int i = 0; i < numbers.length; i++) {
    System.out.println(numbers[i]);
}

for (int number : numbers) {
    System.out.println(number);
}
```

Use a traditional `for` loop when the index is needed; use the enhanced `for` loop for simple element-by-element traversal.

### Array vs ArrayList

| Feature | Array | `ArrayList` |
|---|---|---|
| Size | Fixed after creation | Can grow or shrink |
| Values | Can store primitives or references | Stores objects; use wrappers such as `Integer` for primitive values |
| Length/size | `array.length` | `list.size()` |
| Access | `array[index]` | `list.get(index)` |
| Add/remove | No built-in resizing methods | Supports `add()` and `remove()` |
| Package | Built into Java language | `java.util.ArrayList` |

Example:

```java
import java.util.ArrayList;

ArrayList<Integer> numbers = new ArrayList<>();
numbers.add(10);
numbers.add(20);
System.out.println(numbers.get(0)); // 10
System.out.println(numbers.size()); // 2
```

---

## 5. Exception Handling

### What is an exception?
An exception is an event that interrupts the normal flow of a program. Java lets code handle many exceptions so it can respond appropriately.

### Exception hierarchy overview
`Throwable` is the root type. Its main branches are:
- **`Error`:** Serious JVM or environment problems that applications usually do not try to handle.
- **`Exception`:** Conditions applications may handle. `RuntimeException` and its subclasses are unchecked; other exception subclasses are generally checked.

### `try` and `catch`
Put code that may throw an exception inside `try`. A matching `catch` handles the exception.

```java
try {
    int result = 10 / 0;
    System.out.println(result);
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero");
}
```

### `finally`
A `finally` block normally runs whether the `try` block succeeds or an exception is caught. It is commonly used for cleanup. For resources such as files, prefer try-with-resources when possible. A `finally` block is not guaranteed to execute if the JVM or process terminates abruptly.

```java
try {
    System.out.println("Try block");
} catch (Exception e) {
    System.out.println("Handled");
} finally {
    System.out.println("Cleanup");
}
```

### `throw` vs `throws`

| Keyword | Meaning | Example |
|---|---|---|
| `throw` | Actually throws an exception object | `throw new IllegalArgumentException("Invalid age");` |
| `throws` | Declares exceptions a method may pass to its caller | `void read() throws IOException { ... }` |

### Checked vs unchecked exceptions

- **Checked:** The compiler requires the code to catch or declare them, such as `IOException`.
- **Unchecked:** Subclasses of `RuntimeException`; the compiler does not require them to be caught or declared, such as `NullPointerException` and `ArithmeticException`.

### Custom exception example

```java
class InvalidAgeException extends Exception {
    InvalidAgeException(String message) {
        super(message);
    }
}

class Main {
    static void checkAge(int age) throws InvalidAgeException {
        if (age < 18) {
            throw new InvalidAgeException("Age must be at least 18");
        }
    }

    public static void main(String[] args) {
        try {
            checkAge(16);
        } catch (InvalidAgeException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

---

# 6. Five Basic Java Coding Programs

Each program is standalone. Save one program at a time as `Main.java` and run it.

## Program 1: Reverse a String

**Problem:** Reverse `"Shreyas"` to get `"sayerhS"`.

```java
public class Main {
    public static void main(String[] args) {
        String st = "Shreyas";
        String rev = "";

        for (int i = st.length() - 1; i >= 0; i--) {
            rev = rev + st.charAt(i);
        }

        System.out.println(rev);
    }
}
```

**Output**
```text
sayerhS
```

**Explanation:** Start from the last character and append each character to `rev`, moving backward until index `0`.

**Complexity:** This beginner-friendly concatenation approach can take O(n²) time because each concatenation may copy the current result. A `StringBuilder` approach can do the reversal in O(n) time.

## Program 2: Check Whether a Number Is Even or Odd

```java
public class Main {
    public static void main(String[] args) {
        int number = 7;

        if (number % 2 == 0) {
            System.out.println("Even");
        } else {
            System.out.println("Odd");
        }
    }
}
```

**Output**
```text
Odd
```

**Explanation:** `%` returns the remainder. An integer divisible by `2` has remainder `0`, so it is even; otherwise, it is odd.

## Program 3: Check Whether a Number Is Prime

A prime number is an integer greater than `1` with exactly two positive divisors: `1` and itself.

```java
public class Main {
    public static void main(String[] args) {
        int number = 13;
        boolean isPrime = number > 1;

        for (int i = 2; i * i <= number && isPrime; i++) {
            if (number % i == 0) {
                isPrime = false;
            }
        }

        if (isPrime) {
            System.out.println("Prime");
        } else {
            System.out.println("Not Prime");
        }
    }
}
```

**Output**
```text
Prime
```

**Explanation:** Numbers less than or equal to `1` are not prime. Test possible divisors from `2` while `i * i <= number`. If any divides evenly, the number is not prime. Checking only up to the square root is sufficient.

## Program 4: Find the Largest Element in an Array

```java
public class Main {
    public static void main(String[] args) {
        int[] numbers = {12, 45, 7, 89, 23};
        int largest = numbers[0];

        for (int i = 1; i < numbers.length; i++) {
            if (numbers[i] > largest) {
                largest = numbers[i];
            }
        }

        System.out.println("Largest: " + largest);
    }
}
```

**Output**
```text
Largest: 89
```

**Explanation:** Start by assuming the first element is the largest. Compare every remaining element with it and update `largest` whenever a bigger value is found. This example assumes the array is non-empty.

## Program 5: Check Whether a String Is a Palindrome

A palindrome reads the same forward and backward, such as `"madam"`.

```java
public class Main {
    public static void main(String[] args) {
        String text = "madam";
        boolean isPalindrome = true;

        for (int i = 0; i < text.length() / 2; i++) {
            if (text.charAt(i) != text.charAt(text.length() - 1 - i)) {
                isPalindrome = false;
                break;
            }
        }

        if (isPalindrome) {
            System.out.println("Palindrome");
        } else {
            System.out.println("Not Palindrome");
        }
    }
}
```

**Output**
```text
Palindrome
```

**Explanation:** Compare characters from the beginning with matching characters from the end. If any pair differs, the string is not a palindrome. This version is case-sensitive and does not ignore spaces or punctuation.

---

## Quick Interview Revision

1. **JDK vs JRE vs JVM:** Develop, run, execute bytecode.
2. **Encapsulation:** Protect data by controlling access through methods.
3. **Inheritance:** Build a class from another class using `extends`.
4. **Polymorphism:** Overloading at compile time; overriding for runtime dispatch.
5. **Abstraction:** Show essential behavior while hiding implementation details.
6. **String immutability:** String operations return new strings rather than changing the existing object.
7. **String Pool:** Reuses string literals; do not use `==` to compare text.
8. **Array indexing:** Starts at `0`; last index is `length - 1`.
9. **Array vs ArrayList:** Fixed size versus resizable collection.
10. **`throw` vs `throws`:** Throw an exception object versus declare possible exceptions.
11. **`finally`:** Usually runs for cleanup, but is not an absolute guarantee during abrupt process/JVM termination.
12. **Checked vs unchecked:** Checked exceptions must be caught or declared; unchecked exceptions do not have that compiler requirement.

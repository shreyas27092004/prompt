# Java Interview Preparation: Two Sets of Questions and Answers

This guide covers Java Fundamentals, OOPS, Arrays, Strings, Exception Handling, and two basic Java coding questions with brute-force solutions.

## Topics Covered

- Java Fundamentals
- OOPS (Object-Oriented Programming)
- Arrays
- Strings
- Exception Handling
- Two basic Java programming questions with brute-force solutions

The answers are designed for Java fresher interviews and technical evaluations.

---

# SET 1: Java Fundamentals, OOPS, Arrays, Strings & Exception Handling

## Part 1: Java Fundamentals

### Q1. What is Java?

**Answer:** Java is a high-level, object-oriented, class-based programming language developed by Sun Microsystems. It is platform-independent because Java code is compiled into bytecode that runs on a compatible Java Virtual Machine (JVM).

### Q2. What is the difference between JDK, JRE, and JVM?

**Answer:**

- **JDK (Java Development Kit):** Provides tools to develop and run Java applications, including the compiler.
- **JRE (Java Runtime Environment):** Traditionally refers to the runtime components needed to run Java applications, including the JVM and libraries.
- **JVM (Java Virtual Machine):** Executes Java bytecode.

**Remember:**
- JDK → Develop and run
- JRE → Run
- JVM → Execute bytecode

**Note:** Modern Java distributions may not provide a separate JRE installation.

### Q3. What are the primitive data types in Java?

**Answer:** Java has eight primitive data types.

| Data type | Example |
|---|---|
| `byte` | `byte a = 10;` |
| `short` | `short a = 100;` |
| `int` | `int a = 1000;` |
| `long` | `long a = 10000L;` |
| `float` | `float a = 10.5f;` |
| `double` | `double a = 20.5;` |
| `char` | `char a = 'A';` |
| `boolean` | `boolean a = true;` |

### Q4. What is the difference between `==` and `equals()`?

**Answer:**
- `==` compares primitive values or object references.
- `equals()` compares object contents when the class implements an appropriate equality check.

**Example:**

```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1 == s2);
System.out.println(s1.equals(s2));
```

**Output:**

```text
false
true
```

### Q5. What is the difference between `break` and `continue`?

**Answer:**
- `break` terminates the loop.
- `continue` skips the current iteration and proceeds to the next iteration.

---

## Part 2: OOPS

### Q6. What is OOPS?

**Answer:** OOPS stands for Object-Oriented Programming System. It is a programming paradigm based on classes and objects that promotes code reusability, maintainability, and modularity.

The four pillars are:
1. Encapsulation
2. Inheritance
3. Polymorphism
4. Abstraction

### Q7. What is Encapsulation?

**Answer:** Encapsulation combines data and methods within a class and restricts direct access to data using access modifiers.

**Example:**

```java
class Student {
    private int age;

    public void setAge(int age) {
        if (age > 0) {
            this.age = age;
        }
    }

    public int getAge() {
        return age;
    }
}
```

Here, `age` is private and accessed through public methods.

### Q8. What is Inheritance?

**Answer:** Inheritance allows one class to acquire accessible properties and methods from another class. It promotes code reusability.

**Example:**

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

### Q9. What is Polymorphism?

**Answer:** Polymorphism means one name or interface can represent different behaviors.

Two common types are:
- **Compile-time polymorphism:** Method overloading.
- **Runtime polymorphism:** Method overriding.

### Q10. What is the difference between an abstract class and an interface?

**Answer:**

| Abstract class | Interface |
|---|---|
| Declared using `abstract` | Declared using `interface` |
| Can contain abstract and concrete methods | Can contain abstract, default, static, and private methods, subject to Java version rules |
| Can have instance variables | Fields are implicitly `public static final` |
| A class can extend only one class | A class can implement multiple interfaces |

---

## Part 3: Arrays

### Q11. What is an array in Java?

**Answer:** An array is an object that stores a fixed number of elements of the same component type. Its elements are accessed using zero-based indexes.

**Example:**

```java
int[] numbers = {10, 20, 30};

System.out.println(numbers[0]);
```

**Output:**

```text
10
```

### Q12. What is the difference between an array and an `ArrayList`?

**Answer:**

| Array | `ArrayList` |
|---|---|
| Fixed length | Can grow or shrink |
| Supports primitives and references | Stores objects; primitives use wrapper classes |
| Uses `length` | Uses `size()` |
| Built-in array feature | Part of the Collections Framework |

### Q13. How do you find the largest element in an array?

**Answer:**

```java
int[] arr = {10, 50, 20, 80, 30};

int max = arr[0];

for (int i = 1; i < arr.length; i++) {
    if (arr[i] > max) {
        max = arr[i];
    }
}

System.out.println(max);
```

**Output:**

```text
80
```

### Q14. What is the index of the first element in an array?

**Answer:** The first element has index `0`, and the last element has index `length - 1`.

### Q15. What happens when we access an invalid array index?

**Answer:** Java throws an `ArrayIndexOutOfBoundsException`.

---

## Part 4: Strings

### Q16. What is a String in Java?

**Answer:** A `String` is an object of the `String` class used to represent a sequence of characters. Strings are immutable, meaning their contents cannot be changed after creation.

### Q17. Why is String immutable?

**Answer:** Immutability supports security, thread safety, reliable hashing, and String Constant Pool reuse.

### Q18. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?

**Answer:**

| Feature | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutability | Immutable | Mutable | Mutable |
| Modification | Operations return new string values | Can modify the same object | Can modify the same object |
| Typical use | Suitable for fixed text | Efficient for repeated modifications in single-threaded code | Provides synchronized methods for thread-safe use |

### Q19. How do you find the length of a String?

**Answer:**

```java
String name = "Java";

System.out.println(name.length());
```

**Output:**

```text
4
```

### Q20. What is a palindrome String?

**Answer:** A palindrome String reads the same forward and backward.

**Examples:** `madam`, `level`, `radar`.

---

## Part 5: Exception Handling

### Q21. What is exception handling?

**Answer:** Exception handling is a mechanism for dealing with exceptional conditions so that an application can respond appropriately rather than terminating unexpectedly.

Java provides `try`, `catch`, `finally`, `throw`, and `throws`.

### Q22. What is the difference between checked and unchecked exceptions?

**Answer:**

| Checked exception | Unchecked exception |
|---|---|
| Compiler requires handling or declaration | Compiler does not require handling or declaration |
| Examples: `IOException`, `SQLException` | Examples: `NullPointerException`, `ArithmeticException` |
| Often represent conditions an application may recover from | Often indicate programming errors, though not always |

### Q23. What is the difference between `throw` and `throws`?

**Answer:**
- `throw` explicitly throws an exception.
- `throws` declares exceptions that a method may pass to its caller.

**Example:**

```java
throw new IllegalArgumentException("Invalid age");

void readFile() throws IOException {
    // File-reading logic
}
```

The `IOException` example requires `import java.io.IOException;` if used in a complete class without a fully qualified name.

### Q24. What is the purpose of the `finally` block?

**Answer:** The `finally` block is commonly used for cleanup operations. It normally executes whether an exception occurs or not, except in circumstances such as abrupt JVM termination.

### Q25. Can we have multiple `catch` blocks?

**Answer:** Yes. A `try` block can be followed by multiple `catch` blocks to handle different exception types. More specific exceptions must be caught before more general ones.

---

## Part 6: Two Basic Java Coding Questions

### Coding Question 1: Reverse a String

**Question:** Write a Java program to reverse a String without using `StringBuilder.reverse()`.

**Input:**

```text
Shreyas
```

**Output:**

```text
sayerhS
```

#### Brute-Force Solution

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

#### Explanation

1. Start at the last character.
2. Read each character moving backward.
3. Append each character to `rev`.
4. Print the reversed String.

- **Time complexity:** `O(n²)` in the worst case because String concatenation repeatedly creates new String objects.
- **Space complexity:** `O(n)` for the resulting String, excluding temporary allocations.

### Coding Question 2: Check Whether a Number Is Prime

**Question:** Write a Java program to check whether a number is prime.

**Input:**

```text
7
```

**Output:**

```text
Prime
```

#### Brute-Force Solution

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

#### Explanation

1. Numbers less than or equal to `1` are not prime.
2. Check whether any number from `2` to `n - 1` divides `n`.
3. If a divisor is found, the number is not prime.
4. Otherwise, it is prime.

- **Time complexity:** `O(n)`.
- **Space complexity:** `O(1)`.

---

# SET 2: Java Fundamentals, OOPS, Arrays, Strings & Exception Handling

This set contains different questions to help you practise without simply memorizing Set 1.

## Part 1: Java Fundamentals

### Q1. Why is Java platform-independent?

**Answer:** Java source code is compiled into bytecode. The bytecode can run on any operating system that provides a compatible JVM.

### Q2. What is the difference between JDK and JVM?

**Answer:** The JDK provides development tools, including the Java compiler. The JVM executes Java bytecode.

### Q3. What are relational operators in Java?

**Answer:** Relational operators compare two values and return a boolean result.

- `==`
- `!=`
- `>`
- `<`
- `>=`
- `<=`

### Q4. What is the difference between local, instance, and static variables?

| Variable | Description |
|---|---|
| Local | Declared inside a method, constructor, or block |
| Instance | Belongs to an object |
| Static | Belongs to the class rather than an individual instance |

### Q5. What is the difference between `final`, `finally`, and `finalize()`?

- **`final`:** Keyword used to restrict reassignment, overriding, or inheritance, depending on its use.
- **`finally`:** Block used with exception handling.
- **`finalize()`:** Legacy garbage-collection-related method deprecated for removal; it should not be used for resource cleanup.

---

## Part 2: OOPS

### Q6. What is a class and an object?

- **Class:** A blueprint that defines fields and methods.
- **Object:** An instance of a class.

**Example:**

```java
class Student {
    String name;
}

// Inside a method:
Student s = new Student();
```

Here, `Student` is the class and `s` refers to the created object.

### Q7. What is the difference between method overloading and method overriding?

| Overloading | Overriding |
|---|---|
| Same method name with different parameter lists | Subclass provides a compatible implementation of an inherited method |
| Resolved at compile time | Instance method dispatch is generally resolved at runtime |
| Can occur in the same class | Requires inheritance or interface implementation |

### Q8. What is multilevel inheritance?

**Answer:** Multilevel inheritance occurs when a class inherits from another class that itself inherits from a third class.

```java
class A {
    void displayA() {
        System.out.println("A");
    }
}

class B extends A {
    void displayB() {
        System.out.println("B");
    }
}

class C extends B {
    void displayC() {
        System.out.println("C");
    }
}
```

An object of `C` can call accessible methods inherited from `A` and `B`.

### Q9. Can we create an object of an abstract class?

**Answer:** No. An abstract class cannot be instantiated directly. A reference of its type can point to a subclass object.

```java
abstract class Animal {
    abstract void sound();
}

class Dog extends Animal {
    void sound() {
        System.out.println("Bark");
    }
}

// Inside a method:
Animal a = new Dog();
a.sound();
```

### Q10. What is the difference between `this` and `super`?

- **`this`:** Refers to the current object.
- **`super`:** Used to access accessible superclass members or invoke a superclass constructor.

---

## Part 3: Arrays

### Q11. How do you calculate the sum of array elements?

```java
int[] arr = {10, 20, 30, 40};
int sum = 0;

for (int num : arr) {
    sum += num;
}

System.out.println(sum);
```

**Output:**

```text
100
```

### Q12. What are the default values of array elements?

**Answer:** Java initializes array elements with default values.

| Type | Default value |
|---|---|
| `int` | `0` |
| `double` | `0.0` |
| `boolean` | `false` |
| `char` | `'\u0000'` |
| Object references | `null` |

### Q13. What is a multidimensional array?

**Answer:** A multidimensional array is an array whose elements are themselves arrays.

```java
int[][] matrix = {
    {1, 2},
    {3, 4}
};

System.out.println(matrix[1][0]);
```

**Output:**

```text
3
```

### Q14. What is linear search?

**Answer:** Linear search checks each element sequentially until the target is found or the array ends. Its worst-case time complexity is **O(n)**.

### Q15. Can an array's size be changed after creation?

**Answer:** No. An array's length is fixed when created. To change capacity, create another array or use a dynamic collection such as `ArrayList`.

---

## Part 4: Strings

### Q16. What is the String Constant Pool?

**Answer:** The String Constant Pool is a JVM-managed area of the heap used to reuse interned String instances.

```java
String s1 = "Java";
String s2 = "Java";

System.out.println(s1 == s2);
```

**Output:**

```text
true
```

Both variables refer to the same interned String instance in this example.

### Q17. How do you compare two Strings without considering case?

**Answer:** Use `equalsIgnoreCase()`.

```java
String a = "JAVA";
String b = "java";

System.out.println(a.equalsIgnoreCase(b));
```

**Output:**

```text
true
```

### Q18. What does `charAt()` do?

**Answer:** Returns the character at a specified index in a String.

```java
String s = "Java";
System.out.println(s.charAt(2));
```

**Output:**

```text
v
```

### Q19. What is the difference between `substring()` and `subSequence()`?

- **`substring()`:** Returns a String containing the specified portion.
- **`subSequence()`:** Returns a `CharSequence` representing the specified portion.

Both use a starting index inclusive and an ending index exclusive.

### Q20. How do you convert a String to uppercase?

```java
String s = "java";
System.out.println(s.toUpperCase());
```

**Output:**

```text
JAVA
```

---

## Part 5: Exception Handling

### Q21. What is an exception?

**Answer:** An exception is an event represented by an object that disrupts the normal flow of program execution.

### Q22. What is the difference between an exception and an error?

- **Exception:** An exceptional condition that an application may be able to handle.
- **Error:** Generally represents a serious problem involving the JVM or execution environment that applications usually should not attempt to recover from.

Both are subclasses of `Throwable`.

### Q23. What is a `NullPointerException`?

**Answer:** A `NullPointerException` occurs when code performs an operation requiring an object reference, but the reference is `null`.

```java
String s = null;
System.out.println(s.length());
```

This throws a `NullPointerException`.

### Q24. Can we use `try` without `catch`?

**Answer:** Yes. A `try` statement can be followed by a `finally` block without a `catch` block. Java also supports try-with-resources for automatic resource management.

### Q25. What is a custom exception?

**Answer:** A custom exception is an exception class created by a developer to represent an application-specific error.

```java
class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}
```

---

## Part 6: Two Basic Java Coding Questions

### Coding Question 1: Find the Largest Element in an Array

**Question:** Find the largest element without built-in sorting methods.

**Input:**

```text
10 50 20 80 30
```

**Output:**

```text
80
```

#### Brute-Force Solution

```java
public class Main {
    public static void main(String[] args) {
        int[] arr = {10, 50, 20, 80, 30};
        int max = arr[0];

        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) {
                max = arr[i];
            }
        }

        System.out.println(max);
    }
}
```

**Explanation:**

1. Assume the first element is largest.
2. Compare each remaining element with `max`.
3. Update `max` when a larger value is found.
4. Print the maximum.

- **Time complexity:** O(n)
- **Space complexity:** O(1)

### Coding Question 2: Check Whether a String Is a Palindrome

**Question:** Check a String without using `StringBuilder.reverse()`.

**Input:**

```text
madam
```

**Output:**

```text
Palindrome
```

#### Brute-Force Solution

```java
public class Main {
    public static void main(String[] args) {
        String str = "madam";
        boolean palindrome = true;

        for (int i = 0; i < str.length() / 2; i++) {
            if (str.charAt(i) != str.charAt(str.length() - 1 - i)) {
                palindrome = false;
                break;
            }
        }

        if (palindrome) {
            System.out.println("Palindrome");
        } else {
            System.out.println("Not Palindrome");
        }
    }
}
```

**Explanation:**

1. Compare first and last characters.
2. Compare second and second-last characters.
3. Continue to the middle.
4. If any pair differs, it is not a palindrome.

- **Time complexity:** O(n)
- **Auxiliary space complexity:** O(1)

---

# Final Revision: Important Questions to Practise

| Topic | Set 1 | Set 2 |
|---|---|---|
| Java Fundamentals | JDK, JRE, JVM | Platform independence, `final` |
| OOPS | Four pillars | Overloading vs overriding |
| Arrays | Largest element | Array dimensions, searching |
| Strings | Reverse a String | Palindrome, String Pool |
| Exception Handling | `try`, `catch`, `finally` | Checked vs unchecked, custom exceptions |
| Coding | Reverse a String, prime number | Largest element, palindrome |

**Study tip:** Practise theory questions aloud, write all four coding programs from memory, and explain the logic and time complexity of each solution.

---

# SET 2: Java Fundamentals, OOPS, Arrays, Strings & Exception Handling

This set contains different questions to help you practise without simply memorizing Set 1.

## PART 1: Java Fundamentals

### Q1. Why is Java platform-independent?

**Answer:** Java source code is compiled into bytecode. The bytecode can run on any operating system that provides a compatible JVM.

### Q2. What is the difference between JDK and JVM?

**Answer:** The JDK provides development tools, including the Java compiler. The JVM executes Java bytecode.

### Q3. What are relational operators in Java?

**Answer:** Relational operators compare two values and return a boolean result.

- `==`
- `!=`
- `>`
- `<`
- `>=`
- `<=`

### Q4. What is the difference between local, instance, and static variables?

| Variable | Description |
|---|---|
| Local | Declared inside a method, constructor, or block |
| Instance | Belongs to an object |
| Static | Belongs to the class rather than an individual instance |

### Q5. What is the difference between `final`, `finally`, and `finalize()`?

- **`final`:** A keyword used to restrict reassignment, overriding, or inheritance, depending on its use.
- **`finally`:** A block used with exception handling.
- **`finalize()`:** A legacy garbage-collection-related method deprecated for removal; it should not be used for resource cleanup.

---

## PART 2: OOPS

### Q6. What is a class and an object?

- **Class:** A blueprint that defines fields and methods.
- **Object:** An instance of a class.

```java
class Student {
    String name;
}

// Inside a method:
Student s = new Student();
```

Here, `Student` is the class and `s` refers to the created object.

### Q7. What is the difference between method overloading and method overriding?

| Overloading | Overriding |
|---|---|
| Same method name with different parameter lists | Subclass provides a compatible implementation of an inherited method |
| Resolved at compile time | Instance method dispatch is generally resolved at runtime |
| Can occur in the same class | Requires inheritance or interface implementation |

### Q8. What is multilevel inheritance?

**Answer:** Multilevel inheritance occurs when a class inherits from another class that itself inherits from a third class.

```java
class A {
    void displayA() {
        System.out.println("A");
    }
}

class B extends A {
    void displayB() {
        System.out.println("B");
    }
}

class C extends B {
    void displayC() {
        System.out.println("C");
    }
}
```

An object of `C` can call the accessible methods inherited from `A` and `B`.

### Q9. Can we create an object of an abstract class?

**Answer:** No. An abstract class cannot be instantiated directly. However, we can create a reference of its type that refers to a subclass object.

```java
abstract class Animal {
    abstract void sound();
}

class Dog extends Animal {
    void sound() {
        System.out.println("Bark");
    }
}

// Inside a method:
Animal a = new Dog();
a.sound();
```

### Q10. What is the difference between `this` and `super`?

- **`this`:** Refers to the current object.
- **`super`:** Used to access accessible superclass members or invoke a superclass constructor.

---

## PART 3: Arrays

### Q11. How do you calculate the sum of array elements?

```java
int[] arr = {10, 20, 30, 40};
int sum = 0;

for (int num : arr) {
    sum += num;
}

System.out.println(sum);
```

**Output:**

```text
100
```

### Q12. What are the default values of array elements?

**Answer:** Java initializes array elements with default values.

| Type | Default value |
|---|---|
| `int` | `0` |
| `double` | `0.0` |
| `boolean` | `false` |
| `char` | `'\u0000'` |
| Object references | `null` |

### Q13. What is a multidimensional array?

**Answer:** A multidimensional array is an array whose elements are themselves arrays.

```java
int[][] matrix = {
    {1, 2},
    {3, 4}
};

System.out.println(matrix[1][0]);
```

**Output:**

```text
3
```

### Q14. What is linear search?

**Answer:** Linear search checks each element sequentially until the target is found or the array ends. Its worst-case time complexity is **O(n)**.

### Q15. Can an array's size be changed after creation?

**Answer:** No. An array's length is fixed when the array is created. To change its capacity, create another array or use a dynamic collection such as `ArrayList`.

---

## PART 4: Strings

### Q16. What is the String Constant Pool?

**Answer:** The String Constant Pool is a JVM-managed area of the heap used to reuse interned String instances.

```java
String s1 = "Java";
String s2 = "Java";

System.out.println(s1 == s2);
```

**Output:**

```text
true
```

Both variables refer to the same interned String instance in this example.

### Q17. How do you compare two Strings without considering case?

**Answer:** Use `equalsIgnoreCase()`.

```java
String a = "JAVA";
String b = "java";

System.out.println(a.equalsIgnoreCase(b));
```

**Output:**

```text
true
```

### Q18. What does `charAt()` do?

**Answer:** The `charAt()` method returns the character at a specified index in a String.

```java
String s = "Java";

System.out.println(s.charAt(2));
```

**Output:**

```text
v
```

### Q19. What is the difference between `substring()` and `subSequence()`?

- **`substring()`:** Returns a `String` containing the specified portion.
- **`subSequence()`:** Returns a `CharSequence` representing the specified portion.

Both use a starting index inclusive and an ending index exclusive.

### Q20. How do you convert a String to uppercase?

```java
String s = "java";

System.out.println(s.toUpperCase());
```

**Output:**

```text
JAVA
```

---

## PART 5: Exception Handling

### Q21. What is an exception?

**Answer:** An exception is an event represented by an object that disrupts the normal flow of program execution.

### Q22. What is the difference between an exception and an error?

- **Exception:** An exceptional condition that an application may be able to handle.
- **Error:** Generally represents a serious problem involving the JVM or execution environment that applications usually should not attempt to recover from.

Both are subclasses of `Throwable`.

### Q23. What is a `NullPointerException`?

**Answer:** A `NullPointerException` occurs when code performs an operation requiring an object reference, but the reference is `null`.

```java
String s = null;
System.out.println(s.length());
```

This throws a `NullPointerException`.

### Q24. Can we use `try` without `catch`?

**Answer:** Yes. A `try` statement can be followed by a `finally` block without a `catch` block. Java also supports try-with-resources for automatic resource management.

### Q25. What is a custom exception?

**Answer:** A custom exception is an exception class created by a developer to represent an application-specific error.

```java
class InvalidAgeException extends Exception {
    public InvalidAgeException(String message) {
        super(message);
    }
}
```

---

## PART 6: Two Basic Java Coding Questions

### Coding Question 1: Find the Largest Element in an Array

**Question:** Write a Java program to find the largest element in an array without using built-in sorting methods.

**Input:**

```text
10 50 20 80 30
```

**Output:**

```text
80
```

#### Brute-Force Solution

```java
public class Main {
    public static void main(String[] args) {
        int[] arr = {10, 50, 20, 80, 30};
        int max = arr[0];

        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) {
                max = arr[i];
            }
        }

        System.out.println(max);
    }
}
```

**Explanation:**

1. Assume the first element is the largest.
2. Compare every remaining element with `max`.
3. Update `max` whenever a larger element is found.
4. Print the final maximum.

**Time complexity:** O(n)  
**Space complexity:** O(1)

---

### Coding Question 2: Check Whether a String Is a Palindrome

**Question:** Write a Java program to check whether a String is a palindrome without using `StringBuilder.reverse()`.

**Input:**

```text
madam
```

**Output:**

```text
Palindrome
```

#### Brute-Force Solution

```java
public class Main {
    public static void main(String[] args) {
        String str = "madam";
        boolean palindrome = true;

        for (int i = 0; i < str.length() / 2; i++) {
            if (str.charAt(i) != str.charAt(str.length() - 1 - i)) {
                palindrome = false;
                break;
            }
        }

        if (palindrome) {
            System.out.println("Palindrome");
        } else {
            System.out.println("Not Palindrome");
        }
    }
}
```

**Explanation:**

1. Compare the first character with the last character.
2. Compare the second character with the second-last character.
3. Continue until the middle of the String.
4. If any pair differs, the String is not a palindrome.

**Time complexity:** O(n)  
**Auxiliary space complexity:** O(1)

---

# Final Revision: Important Questions to Practise

| Topic | Set 1 | Set 2 |
|---|---|---|
| Java Fundamentals | JDK, JRE, JVM | Platform independence, `final` |
| OOPS | Four pillars | Overloading vs overriding |
| Arrays | Largest element | Array dimensions, searching |
| Strings | Reverse a String | Palindrome, String Pool |
| Exception Handling | `try`, `catch`, `finally` | Checked vs unchecked, custom exceptions |
| Coding | Reverse a String, prime number | Find largest element and palindrome |

**Study recommendation:** First practise answering the theory questions aloud without reading the answers. Then write all four coding programs from memory. Finally, practise explaining each program's logic and time complexity to prepare for technical interviews and coding evaluations.

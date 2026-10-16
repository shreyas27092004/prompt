# Capgemini Fresher Interview Preparation
## Two Basic Sets of Java Questions and Answers + Coding Programs

This guide is designed for a beginner preparing for a possible Capgemini fresher interview. These are practice questions based on the topics shared with you, **not confirmed questions from Capgemini**. The exact process and questions can vary by role and hiring drive.

Topics: Java Fundamentals, OOPS, Strings, Arrays, and Exception Handling.

---

# SET 1 — Basic Java Interview Questions

## Part 1: Java Fundamentals

### Q1. What is Java?
**Answer:** Java is a programming language used to build applications.

### Q2. What is JVM?
**Answer:** JVM stands for Java Virtual Machine. It runs Java bytecode.

### Q3. What is JDK?
**Answer:** JDK stands for Java Development Kit. It contains tools for developing and compiling Java programs.

### Q4. What is JRE?
**Answer:** JRE stands for Java Runtime Environment. Traditionally, it means the components needed to run Java applications.

**Remember:** JDK is for development, JRE is for running, and JVM executes bytecode.

### Q5. What are the eight primitive data types in Java?
**Answer:** `byte`, `short`, `int`, `long`, `float`, `double`, `char`, and `boolean`.

### Q6. What is a variable?
**Answer:** A variable is a named place to store a value.

```java
int age = 20;
```

### Q7. What is an operator?
**Answer:** An operator performs an operation on values.

Examples: `+`, `-`, `*`, `/`, `%`, `==`, `&&`, and `||`.

### Q8. What is the difference between `=` and `==`?
**Answer:** `=` assigns a value. `==` compares primitive values or checks whether two object references refer to the same object.

### Q9. What is the difference between `break` and `continue`?
**Answer:** `break` stops a loop. `continue` skips the current iteration and moves to the next one.

### Q10. What is an access modifier?
**Answer:** An access modifier controls where a class member can be accessed.
- `public`: Accessible from anywhere allowed by the program structure.
- `private`: Accessible only inside the same class.
- `protected`: Accessible within the same package and through permitted subclass access.
- No modifier: Accessible within the same package.

## Part 2: OOPS

### Q11. What is OOPS?
**Answer:** OOPS means Object-Oriented Programming. It organizes code using classes and objects.

### Q12. What is a class?
**Answer:** A class is a blueprint for creating objects.

```java
class Car {
    String color;
}
```

### Q13. What is an object?
**Answer:** An object is an instance of a class.

```java
Car myCar = new Car();
```

### Q14. What is encapsulation?
**Answer:** Encapsulation keeps data and methods together in a class and controls access to the data.

### Q15. What is inheritance?
**Answer:** Inheritance lets a child class use accessible features of a parent class.

```java
class Animal {
    void eat() {
        System.out.println("Eating");
    }
}

class Dog extends Animal {
}
```

### Q16. What is polymorphism?
**Answer:** Polymorphism means one method name or interface can have different behaviours.

### Q17. What is method overloading?
**Answer:** Overloading means having methods with the same name but different parameter lists.

```java
int add(int a, int b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}
```

### Q18. What is method overriding?
**Answer:** Overriding happens when a subclass provides its own implementation of an inherited method.

### Q19. What is abstraction?
**Answer:** Abstraction means showing what something does while hiding details of how it works.

### Q20. What is the difference between an abstract class and an interface?
**Answer:** An abstract class can contain instance variables and both abstract and concrete methods. An interface defines a contract and can contain abstract, default, static, and private methods, subject to Java version rules. A class can extend only one class but can implement multiple interfaces.

## Part 3: Strings

### Q21. What is a String?
**Answer:** A `String` is an object used to store text.

```java
String name = "Ravi";
```

### Q22. What does immutable mean?
**Answer:** Immutable means an object's contents cannot be changed after it is created. Java `String` objects are immutable.

### Q23. What is the String Constant Pool?
**Answer:** It is a JVM-managed area used to reuse interned strings.

```java
String a = "Java";
String b = "Java";
System.out.println(a == b); // true
```

### Q24. What is the difference between `==` and `equals()` for Strings?
**Answer:** `==` checks references. `equals()` compares text.

```java
String a = new String("Java");
String b = new String("Java");

System.out.println(a == b);      // false
System.out.println(a.equals(b)); // true
```

### Q25. How do you find the length of a String?
**Answer:** Use `length()`.

```java
System.out.println("Java".length()); // 4
```

### Q26. What does `charAt()` do?
**Answer:** It returns the character at a specified index.

```java
System.out.println("Java".charAt(0)); // J
```

### Q27. What is the difference between `String`, `StringBuilder`, and `StringBuffer`?
**Answer:**
- `String` is immutable.
- `StringBuilder` is mutable and useful for repeated changes in single-threaded code.
- `StringBuffer` is mutable and has synchronized methods.

## Part 4: Arrays

### Q28. What is an array?
**Answer:** An array stores a fixed number of values of the same component type.

```java
int[] numbers = {10, 20, 30};
```

### Q29. What is an array index?
**Answer:** An index is the position of an element. Java indexes start at `0`.

### Q30. How do you find the length of an array?
**Answer:** Use the `length` property.

```java
int[] numbers = {10, 20, 30};
System.out.println(numbers.length); // 3
```

### Q31. What happens if you use an invalid array index?
**Answer:** Java throws an `ArrayIndexOutOfBoundsException`.

### Q32. What is the difference between an array and an `ArrayList`?
**Answer:** An array has a fixed size. An `ArrayList` can grow or shrink. Arrays can store primitive values directly; an `ArrayList` stores objects, so wrapper classes such as `Integer` are used for numbers.

### Q33. What are the default values in an array?
**Answer:** Common defaults include `0` for `int`, `0.0` for `double`, `false` for `boolean`, `'\u0000'` for `char`, and `null` for object references.

## Part 5: Exception Handling

### Q34. What is an exception?
**Answer:** An exception is an event that interrupts the normal flow of a program.

### Q35. What is exception handling?
**Answer:** It lets a program respond to an exception rather than stopping unexpectedly.

### Q36. What is a `try` block?
**Answer:** A `try` block contains code that may cause an exception.

### Q37. What is a `catch` block?
**Answer:** A `catch` block handles a matching exception.

```java
try {
    int result = 10 / 0;
} catch (ArithmeticException e) {
    System.out.println("Cannot divide by zero");
}
```

### Q38. What is a `finally` block?
**Answer:** A `finally` block is commonly used for cleanup and normally runs whether an exception occurs or not.

### Q39. What is the difference between `throw` and `throws`?
**Answer:** `throw` explicitly throws an exception object. `throws` declares exceptions a method may pass to its caller.

### Q40. What is the difference between checked and unchecked exceptions?
**Answer:** Checked exceptions must be caught or declared, such as `IOException`. Unchecked exceptions, such as `NullPointerException`, do not have that compiler requirement.

---

## Set 1 — Two Basic Coding Programs

### Program 1: Reverse a String

**Problem:** Reverse the text `"Shreyas"` without using `StringBuilder.reverse()`.

```java
public class Main {
    public static void main(String[] args) {
        String text = "Shreyas";
        String reverse = "";

        for (int i = text.length() - 1; i >= 0; i--) {
            reverse = reverse + text.charAt(i);
        }

        System.out.println(reverse);
    }
}
```

**Output:**
```text
sayerhS
```

**Explanation:** Start from the last character and append each character while moving toward the first character.

**Time complexity:** `O(n²)` in the worst case due to repeated String concatenation.  
**Space complexity:** `O(n)` for the result, excluding temporary allocations.

### Program 2: Check Even or Odd

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

**Output:**
```text
Odd
```

**Explanation:** If the remainder after division by `2` is zero, the number is even. Otherwise, it is odd.

---

# SET 2 — Basic Java Interview Questions
*Some topics repeat from Set 1 on purpose, so you can practise answering them again.*

## Part 1: Java Fundamentals

### Q1. What is Java?
**Answer:** Java is a programming language used to build applications.

### Q2. Why is Java platform-independent?
**Answer:** Java compiles into bytecode that can run on any system with a compatible JVM.

### Q3. What is the difference between JDK and JVM?
**Answer:** JDK provides development tools, including the compiler. JVM executes Java bytecode.

### Q4. What is a data type?
**Answer:** A data type tells Java what kind of value a variable can store, such as an integer, decimal, character, or true/false value.

### Q5. What is a local variable?
**Answer:** A local variable is declared inside a method, constructor, or block and can be used within its scope.

### Q6. What is a static variable?
**Answer:** A `static` variable belongs to the class rather than to each individual object.

### Q7. What is a loop?
**Answer:** A loop repeats a block of code. Common loops are `for`, `while`, and `do-while`.

### Q8. What is the ternary operator?
**Answer:** It is a short form of an `if-else` expression.

```java
int age = 20;
String result = age >= 18 ? "Adult" : "Minor";
```

### Q9. What is a compile-time error?
**Answer:** An error found by the compiler, such as a missing semicolon or incompatible types.

### Q10. What is a runtime error?
**Answer:** An error that happens while the program runs, such as calling a method through a `null` reference.

## Part 2: OOPS

### Q11. What is OOPS?
**Answer:** OOPS is a way of writing programs using classes and objects.

### Q12. What is a constructor?
**Answer:** A constructor initializes a new object. It has the same name as the class and no return type.

### Q13. What is the `this` keyword?
**Answer:** `this` refers to the current object.

### Q14. What is the `super` keyword?
**Answer:** `super` is used to access accessible superclass members or call a superclass constructor.

### Q15. What is method overloading?
**Answer:** It means defining methods with the same name but different parameter lists.

### Q16. What is method overriding?
**Answer:** It means a subclass provides its own implementation of an inherited method.

### Q17. What is the difference between overloading and overriding?
**Answer:** Overloading uses the same method name with different parameters. Overriding changes the implementation of an inherited method in a subclass.

### Q18. Can a Java class extend two classes?
**Answer:** No. A Java class can extend only one class, but it can implement multiple interfaces.

### Q19. What is a `final` variable?
**Answer:** A `final` variable can be assigned only once.

### Q20. What is an interface?
**Answer:** An interface defines a contract that a class can implement.

## Part 3: Strings

### Q21. What is a String in Java?
**Answer:** A String is an object that represents text.

### Q22. Are Strings mutable?
**Answer:** No. String objects are immutable; operations that appear to change a String return a new String.

### Q23. What does `equals()` do?
**Answer:** It compares String contents.

### Q24. What does `equalsIgnoreCase()` do?
**Answer:** It compares two Strings without considering letter case.

```java
System.out.println("JAVA".equalsIgnoreCase("java")); // true
```

### Q25. What does `substring()` do?
**Answer:** It returns part of a String. The starting index is included and the ending index is excluded.

```java
System.out.println("Java".substring(1, 3)); // av
```

### Q26. What does `toLowerCase()` do?
**Answer:** It returns a lowercase version of a String.

### Q27. How can you check whether a String contains text?
**Answer:** Use `contains()`.

```java
System.out.println("Java Programming".contains("Java")); // true
```

## Part 4: Arrays

### Q28. How do you declare an integer array?
**Answer:**

```java
int[] numbers = new int[5];
```

This creates an array that can hold five integers, initially set to zero.

### Q29. How do you access the first array element?
**Answer:** Use index `0`, for example `numbers[0]`.

### Q30. Can an array store different primitive types in the same array?
**Answer:** A primitive array has one declared component type, such as `int[]` or `char[]`.

### Q31. How do you print all array elements?
**Answer:** Use a loop, such as an enhanced `for` loop.

```java
int[] numbers = {1, 2, 3};
for (int n : numbers) {
    System.out.println(n);
}
```

### Q32. What is a multidimensional array?
**Answer:** It is an array whose elements are themselves arrays, such as `int[][]`.

### Q33. What is linear search?
**Answer:** Linear search checks array elements one by one until the target is found or the array ends.

## Part 5: Exception Handling

### Q34. What is `NullPointerException`?
**Answer:** It occurs when code tries to use an object reference that is `null`.

### Q35. What is `ArithmeticException`?
**Answer:** It is an unchecked exception commonly caused by an invalid arithmetic operation, such as integer division by zero.

### Q36. Can a `try` block be used without `catch`?
**Answer:** Yes. A `try` statement can have a `finally` block without a `catch` block. Java also supports try-with-resources.

### Q37. Can we have multiple `catch` blocks?
**Answer:** Yes. Different `catch` blocks can handle different exception types. Put more specific exception types before more general types.

### Q38. What is a custom exception?
**Answer:** It is an exception class created by a programmer for an application-specific problem.

### Q39. What is the purpose of `finally`?
**Answer:** It is commonly used for cleanup and normally runs whether an exception occurs or not.

### Q40. What is the difference between an exception and an error?
**Answer:** An exception may be handled by application code. An error usually indicates a serious JVM or environment problem. Both belong to the `Throwable` hierarchy.

---

## Set 2 — Two Basic Coding Programs

### Program 1: Find the Largest Element in an Array

```java
public class Main {
    public static void main(String[] args) {
        int[] numbers = {10, 50, 20, 80, 30};
        int largest = numbers[0];

        for (int i = 1; i < numbers.length; i++) {
            if (numbers[i] > largest) {
                largest = numbers[i];
            }
        }

        System.out.println(largest);
    }
}
```

**Output:**
```text
80
```

**Explanation:** Start by treating the first element as the largest. Compare the other elements and update the value whenever a larger one is found. This example assumes the array is not empty.

**Time complexity:** `O(n)`.  
**Space complexity:** `O(1)` auxiliary space.

### Program 2: Check Whether a String Is a Palindrome

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

**Output:**
```text
Palindrome
```

**Explanation:** Compare the first character with the last, then the second with the second-last. If a pair differs, the String is not a palindrome. This version is case-sensitive and does not ignore spaces or punctuation.

**Time complexity:** `O(n)`.  
**Auxiliary space complexity:** `O(1)`.

---

# Last-Minute Revision Checklist

- Explain Java, JDK, JRE, and JVM in simple words.
- Learn the four OOPS principles.
- Know overloading versus overriding.
- Understand `==` versus `equals()` for Strings.
- Practise array indexes and `length`.
- Explain `try`, `catch`, `finally`, `throw`, and `throws`.
- Write the four coding programs without copying.
- Be ready to explain your code line by line.

**Reminder:** This is a beginner practice guide, not a verified list of Capgemini interview questions. Actual questions may vary by hiring drive and role.

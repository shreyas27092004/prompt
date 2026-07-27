# JAVA CORE — Chapter 1: JVM, JDK, JRE & Compilation

## 1. Why Java? (Core idea, not history trivia)
Java code compiles to **bytecode** (not machine code). Bytecode runs on the **JVM**, which exists for every OS. That's why Java is "Write Once, Run Anywhere" (WORA).

## 2. JDK vs JRE vs JVM — THE #1 asked concept

| | Full Form | Contains | Purpose |
|---|---|---|---|
| **JVM** | Java Virtual Machine | Class loader, bytecode interpreter/JIT, runtime memory areas | Executes bytecode |
| **JRE** | Java Runtime Environment | JVM + core libraries (rt.jar etc.) | Lets you **run** Java programs |
| **JDK** | Java Development Kit | JRE + compiler (`javac`) + dev tools (`javadoc`, `jdb`) | Lets you **write + compile + run** Java |

**Memory trick:** JDK ⊃ JRE ⊃ JVM (each one contains the previous).

**Capgemini trap:** "To run a .class file you only need ___" → Answer: **JRE** (not JDK). People wrongly pick JDK.

## 3. Compilation Process (asked as flow/sequence questions)

```
MyClass.java  →  [javac compiler]  →  MyClass.class (bytecode)  →  [JVM Class Loader]  →  [Bytecode Verifier]  →  [Interpreter / JIT Compiler]  →  Machine Code
```

- `javac` = compiler (Java → bytecode). Platform **independent** output.
- JVM interpreter converts bytecode → machine code at runtime. Platform **dependent** (different JVM per OS).
- **JIT (Just-In-Time compiler)**: speeds up execution by compiling hot bytecode directly to native code instead of interpreting repeatedly.

**Trap question pattern:** "Java is 100% platform independent — True/False?" → **False**. The `.class` bytecode is platform-independent; the JVM itself is platform-**dependent** (you need a different JVM binary per OS).

## 4. JVM Architecture (very frequently asked)

Three main subsystems:
1. **Class Loader Subsystem** — loads, links, initializes `.class` files (Loading → Linking(Verify, Prepare, Resolve) → Initialization)
2. **Runtime Data Areas** (memory):
   - **Method Area** – class-level data, static variables (shared)
   - **Heap** – all objects, instance variables (shared, GC happens here)
   - **Stack** – one per thread; method calls, local variables, frames
   - **PC Register** – per thread; address of currently executing instruction
   - **Native Method Stack** – for native (C/C++) method calls
3. **Execution Engine** — Interpreter + JIT Compiler + Garbage Collector

**Common trap:** Objects go in **Heap**, method call frames + local variables go in **Stack**. A question showing code with a local `int x` inside a method and asking "where is x stored" → **Stack**. If it's an instance variable → **Heap**.

## 5. Bytecode
- Intermediate, platform-independent instruction set stored in `.class` files.
- Not directly understood by hardware — JVM interprets/JIT-compiles it.
- View it conceptually as "half-compiled" code.

## One-Page Revision
- JDK = JRE + compiler + dev tools. JRE = JVM + libraries. JVM = executes bytecode.
- To just **run**: need JRE. To **develop**: need JDK.
- `javac` compiles `.java → .class` (bytecode, platform-independent).
- JVM interpreter/JIT converts bytecode → native code (platform-dependent step).
- JVM memory: Heap (objects, shared), Stack (per-thread, local vars/frames), Method Area (class data), PC Register, Native Stack.
- JIT = performance optimization, compiles hot code paths to native.
- "100% platform independent" statement about Java = **FALSE** (trap).

---

# MCQs — Chapter 1 (25 Questions)

**Q1.** [Easy | JDK/JRE/JVM] What do you need to only **run** a compiled `.class` file on a machine?
A) JDK B) JRE C) JIT D) IDE
**Answer: B**
*Explanation:* JRE contains the JVM + libraries needed to execute bytecode. JDK is only needed for development/compilation.
*Why others wrong:* A) JDK is a superset, unnecessary just to run. C) JIT is a component inside JVM, not a standalone runtime. D) IDE is a tool, unrelated to execution requirement.

**Q2.** [Easy | JVM] Which JVM memory area stores objects created using `new`?
A) Stack B) Method Area C) Heap D) PC Register
**Answer: C**
*Explanation:* All objects (and their instance variables) are allocated on the Heap; it's also where Garbage Collection operates.
*Why others wrong:* A) Stack holds method frames/local primitives & references, not the objects themselves. B) Method Area holds class-level structure/static data. D) PC Register holds the address of the currently executing instruction.

**Q3.** [Easy | Compilation] What is the output of the `javac` compiler?
A) Machine code B) Bytecode (.class) C) Executable .exe D) Source map
**Answer: B**
*Explanation:* `javac` compiles `.java` source into platform-independent bytecode stored in `.class` files.
*Why others wrong:* A) Machine code is produced by JVM's interpreter/JIT at runtime, not by javac. C) Java doesn't produce native .exe by default. D) Not a Java compilation artifact.

**Q4.** [Medium | JVM] True or False: "Java bytecode is platform-independent but the JVM itself is platform-dependent."
A) True B) False
**Answer: A**
*Explanation:* The same `.class` file runs anywhere, but each OS needs its own JVM implementation to interpret that bytecode.
*Why others wrong:* B is the classic Capgemini trap — people assume "WORA" means everything including JVM is platform-independent, which is false.

**Q5.** [Medium | JVM Architecture] Local variables declared inside a method are stored in:
A) Heap B) Stack C) Method Area D) Native stack
**Answer: B**
*Explanation:* Each thread has its own stack; method invocation creates a frame holding local variables and partial results.
*Why others wrong:* A) Heap is for objects, not primitive locals. C) Method Area is for class-level static/shared data. D) Native stack is for native (non-Java) method calls specifically.

**Q6.** [Medium | JIT] What is the main purpose of the JIT compiler?
A) Convert .java to .class B) Improve runtime performance by compiling hot bytecode to native code C) Perform garbage collection D) Load classes into memory
**Answer: B**
*Explanation:* JIT identifies frequently executed ("hot") bytecode and compiles it directly to native machine code, avoiding repeated interpretation overhead.
*Why others wrong:* A) That's javac's job. C) GC is a separate execution engine component. D) That's the Class Loader's job.

**Q7.** [Medium | Class Loading] What are the three phases of class linking, in order?
A) Verify → Load → Initialize B) Load → Verify → Prepare → Resolve C) Verify → Prepare → Resolve D) Prepare → Verify → Load
**Answer: C**
*Explanation:* Linking = Verify (check bytecode correctness) → Prepare (allocate memory for static fields with default values) → Resolve (replace symbolic references with direct references). Loading itself is a separate earlier phase.
*Why others wrong:* A, B, D scramble the correct sequence or merge Loading into Linking incorrectly.

**Q8.** [Hard | Memory] A static variable is stored in which JVM memory area?
A) Heap B) Stack C) Method Area D) PC Register
**Answer: C**
*Explanation:* Static (class-level) variables live in the Method Area, shared across all instances of the class.
*Why others wrong:* A) Heap holds instance-level object data, not static/class-level data. B) Stack is per-thread/method-call specific. D) PC register just tracks instruction address.

**Q9.** [Hard | Trap] Which statement is TRUE?
A) JDK is a subset of JRE B) JVM is a subset of JRE C) JRE is a subset of JVM D) JDK contains no compiler
**Answer: B**
*Explanation:* JRE = JVM + libraries, so JVM is contained within (a subset of) JRE. JDK = JRE + dev tools, so JDK is the superset of everything.
*Why others wrong:* A) Reversed — JRE is a subset of JDK. C) Reversed — JVM is smaller than JRE. D) JDK explicitly includes `javac`, the compiler.

**Q10.** [Hard | Scenario] You compile Code.java on Windows and copy Code.class to a Linux machine that has a JVM installed. What happens?
A) It fails, must recompile on Linux B) It runs fine — bytecode is portable C) It runs but very slowly D) It needs a JDK, not JRE, to run
**Answer: B**
*Explanation:* This is the core WORA guarantee — bytecode itself carries no OS dependency; the Linux JVM interprets it normally.
*Why others wrong:* A) Contradicts platform independence of bytecode. C) No inherent slowdown from cross-OS bytecode execution. D) Running only needs JRE (or a JDK, which includes a JRE) — JDK isn't strictly required.

**Q11.** [Easy | PC Register] The PC (Program Counter) Register keeps track of:
A) The heap memory address of the last created object B) The address of the instruction currently being executed by that thread C) The total number of loaded classes D) The size of the stack
**Answer: B**
*Explanation:* Each thread has its own PC Register pointing to the current instruction, enabling correct resumption after context switches.
*Why others wrong:* A) Unrelated to heap tracking. C) Class count is not tracked here. D) Stack size isn't stored in PC Register.

**Q12.** [Easy | Native Stack] Native Method Stack is used when:
A) A Java method calls another Java method B) A Java program calls native code written in C/C++ via JNI C) Garbage collection runs D) A class is loaded
**Answer: B**
*Explanation:* JNI (Java Native Interface) calls to non-Java code use the Native Method Stack, separate from the regular Java Stack.
*Why others wrong:* A) Pure Java-to-Java calls use the normal Java Stack. C) GC uses heap/method area, not native stack. D) Class loading uses the Class Loader Subsystem.

**Q13.** [Medium | GC] Garbage Collection in Java primarily operates on which memory area?
A) Stack B) Heap C) Method Area only D) PC Register
**Answer: B**
*Explanation:* GC reclaims memory of objects on the Heap that are no longer reachable.
*Why others wrong:* A) Stack memory is automatically freed when a method returns — no GC needed there. C) Method area is garbage collected far less frequently and isn't the "primary" target. D) PC Register isn't heap-managed memory at all.

**Q14.** [Medium | javac] Which command correctly compiles `Test.java`?
A) `java Test.java` B) `javac Test.java` C) `javac Test.class` D) `run Test.java`
**Answer: B**
*Explanation:* `javac` is the compiler command; it takes `.java` source files and produces `.class` bytecode.
*Why others wrong:* A) `java` is used to *run* a compiled class, not compile it. C) You compile `.java` files, not `.class` files. D) Not a valid Java tool command.

**Q15.** [Medium | Execution] Which command runs a compiled Java class named `Test`?
A) `javac Test` B) `java Test` C) `java Test.java` D) `run Test.class`
**Answer: B**
*Explanation:* `java ClassName` (no extension) launches the JVM and executes the class containing `public static void main`.
*Why others wrong:* A) That's the compiler, not the runner. C) You pass the class name, not the filename with extension. D) Not a valid command.

**Q16.** [Medium | Trap] Which of these is loaded FIRST by the Class Loader Subsystem?
A) Application classes B) Bootstrap classes (core Java classes like `java.lang.*`) C) Extension classes D) Custom user classes
**Answer: B**
*Explanation:* Class loading follows a hierarchy: Bootstrap ClassLoader loads core JDK classes first, then Extension/Platform ClassLoader, then Application ClassLoader for user code — this is the delegation model.
*Why others wrong:* A, C, D all load after Bootstrap in the standard delegation hierarchy.

**Q17.** [Medium | Scenario] If a `.class` file is corrupted or tampered with, which JVM phase catches this?
A) Loading B) Verification (part of Linking) C) Initialization D) Execution
**Answer: B**
*Explanation:* The Bytecode Verifier checks structural correctness and security constraints of bytecode during the Verify sub-phase of Linking, before any execution happens.
*Why others wrong:* A) Loading just locates and reads the file into memory. C) Initialization assigns actual values to static variables/runs static blocks, assuming bytecode is already valid. D) Execution happens only after successful verification.

**Q18.** [Hard | Memory] Which statement about Heap memory is TRUE?
A) It is thread-specific B) It is shared across all threads of the application C) It stores only static variables D) It is fixed in size and cannot grow
**Answer: B**
*Explanation:* Unlike Stack (per-thread), Heap is a single shared memory region across the whole JVM process, accessible by all threads.
*Why others wrong:* A) That describes Stack, not Heap. C) Static variables live in Method Area, not Heap. D) Heap size can grow/shrink dynamically (within `-Xms`/`-Xmx` bounds).

**Q19.** [Hard | Trap] "An `OutOfMemoryError` can only occur due to Heap exhaustion." True or False?
A) True B) False
**Answer: B**
*Explanation:* OutOfMemoryError can also occur due to Stack overflow scenarios in some JVMs, Method Area/Metaspace exhaustion (too many loaded classes), or exceeding native memory limits — not just Heap.
*Why others wrong:* A is the trap — Capgemini likes testing that OOM isn't Heap-exclusive; StackOverflowError is a *separate* error but Metaspace OOM is a real, distinct OOM cause.

**Q20.** [Hard | Code Trace] What does this print, and why is it a JVM-startup-relevant trap?
```java
public class Test {
    static { System.out.println("Static block"); }
    public static void main(String[] args) {
        System.out.println("Main method");
    }
}
```
A) "Main method" then "Static block" B) "Static block" then "Main method" C) Only "Main method" D) Compilation error
**Answer: B**
*Explanation:* Static blocks run during the **Initialization** phase of class loading, which happens before `main()` is invoked — this is why static initialization order matters for MCQs.
*Why others wrong:* A) Reverses actual JVM order. C) Static block always executes once the class is loaded, regardless of main. D) This is valid, compilable code.

**Q21.** [Hard | Scenario] Two classes, `A` and `B`, where `B extends A`. When you run `B`, in what order do static blocks execute if both A and B have static blocks?
A) B's static block, then A's B) A's static block, then B's C) Simultaneously D) Only B's static block runs
**Answer: B**
*Explanation:* JVM initializes the superclass fully (including static blocks) before initializing the subclass, since a subclass depends on its parent's static state.
*Why others wrong:* A) Reverses the correct parent-first order. C) JVM initialization is sequential, not simultaneous. D) Both classes' static blocks run since both are loaded/initialized.

**Q22.** [Medium | Bytecode] Bytecode is best described as:
A) Native machine instructions specific to one CPU B) An intermediate, platform-independent instruction set interpreted/JIT-compiled by the JVM C) Human-readable Java source code D) A compressed version of the .java file
**Answer: B**
*Explanation:* Bytecode sits between source code and machine code — it's the same across all platforms and JVM does the platform-specific translation.
*Why others wrong:* A) That's what bytecode gets translated *into*, not what it *is*. C) Source code is `.java`, bytecode is `.class`. D) It's not just compression — it's a different instruction format entirely.

**Q23.** [Easy | Terminology] JIT stands for:
A) Java Interpreted Translation B) Just-In-Time C) Java Integrated Tool D) Just Interpreted Type
**Answer: B**
*Explanation:* Just-In-Time compiler — compiles bytecode to native code "just in time" during execution rather than ahead-of-time.
*Why others wrong:* A, C, D are fabricated expansions that don't correspond to real Java terminology.

**Q24.** [Medium | Trap] Which of the following is NOT part of the JVM's Execution Engine?
A) Interpreter B) JIT Compiler C) Garbage Collector D) Class Loader
**Answer: D**
*Explanation:* Class Loader is its own separate subsystem (Class Loader Subsystem), not part of the Execution Engine, which handles Interpreter + JIT + GC.
*Why others wrong:* A, B, C are all legitimate components of the Execution Engine as per JVM architecture.

**Q25.** [Hard | Integration] Put these in correct execution order: (1) Bytecode Verification (2) javac compilation (3) JIT compilation of hot code (4) Class loading into memory
A) 2 → 4 → 1 → 3 B) 2 → 1 → 4 → 3 C) 1 → 2 → 4 → 3 D) 4 → 2 → 1 → 3
**Answer: A**
*Explanation:* Correct flow: source compiled by javac (2) → bytecode loaded into JVM memory (4) → verified for correctness (1) → hot paths JIT-compiled during execution (3).
*Why others wrong:* B, C, D all place verification or loading out of the actual sequence — loading must happen before verification, and compilation always happens first.

---

## Chapter 1 Complete ✅
Score yourself: if you missed any of Q8, Q9, Q17, Q19, Q21, or Q25 — reread the JVM Architecture and Class Loading sections above before moving on, since these are the highest-trap-density questions.

**Next up: Chapter 2 — Variables, Data Types, Primitive vs Non-Primitive, Wrapper Classes & Autoboxing.** Say "next chapter" to continue.

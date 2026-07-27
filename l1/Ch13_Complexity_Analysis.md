# DSA — Chapter 13: Complexity Analysis (Big O, Big Ω, Big Θ)

## 1. Why This Chapter Matters
Every DSA question eventually asks "what is the time complexity of this?" Capgemini tests whether you understand efficiency, not memorized syntax.

## 2. What Is Complexity Analysis?
We measure how time/space **grows** as input size `n` grows — not actual seconds/bytes (machine-dependent).

```java
for (int i = 0; i < n; i++) { System.out.println(i); }
```
n=10 → 10 iterations; n=1000 → 1000 iterations → grows linearly → **O(n)**.

## 3. Big O vs Big Ω vs Big Θ — THE #1 asked concept

| | Meaning | Analogy |
|---|---|---|
| **Big O** | Worst case (upper bound) | Worst traffic day — commute never exceeds 2h |
| **Big Ω** | Best case (lower bound) | Best traffic day — never less than 20 min |
| **Big Θ** | Tight/average bound | Typical day — usually exactly 40 min |

**Memory trick:** O = "Oh no, worst case." Ω looks like a bowl (holds the minimum). Θ has a line through the middle (balanced/average).

**Capgemini trap:** If a question just says "time complexity of this algorithm" with no qualifier, assume **Big O (worst case)**.

## 4. Common Complexities (Best → Worst)

| Complexity | Name | Example |
|---|---|---|
| O(1) | Constant | Array index access |
| O(log n) | Logarithmic | Binary Search |
| O(n) | Linear | Simple loop |
| O(n log n) | Linearithmic | Merge/Quick Sort (avg) |
| O(n²) | Quadratic | Nested loops, Bubble Sort |
| O(2ⁿ) | Exponential | Naive recursive Fibonacci |
| O(n!) | Factorial | Brute-force permutations |

**Order to memorize:** O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)

## 5. Calculating Complexity — Rules

1. **Drop constants:** `O(2n)` → `O(n)`
2. **Drop lower-order terms:** `n² + n + 1` → `O(n²)`
3. **Nested loops multiply:** loop inside loop, both size n → `O(n²)`
4. **Sequential loops add, keep the dominant term:** `O(n) + O(n²)` → `O(n²)`
5. **Halving/doubling index → O(log n):**
```java
for (int i = 1; i < n; i = i * 2) { }   // O(log n)
```
6. **Recursion:** recognize common patterns — binary search O(log n), merge sort O(n log n), naive Fibonacci O(2ⁿ).

## 6. Space Complexity
```java
int[] arr = new int[n];  // O(n) space
int x = 5;                // O(1) space
```
**Trap:** Recursive calls consume stack frames even with no array — `recurse(n-1)` called n times = **O(n) space**, not O(1).

## 7. Algorithm Complexity Cheat Table

| Algorithm | Time (Worst) | Space |
|---|---|---|
| Linear Search | O(n) | O(1) |
| Binary Search | O(log n) | O(1) iterative |
| Bubble/Selection/Insertion Sort | O(n²) | O(1) |
| Merge Sort | O(n log n) | O(n) |
| Quick Sort | O(n²) worst, O(n log n) avg | O(log n) |
| HashMap get/put | O(1) avg, O(n) worst | O(n) |
| ArrayList access by index | O(1) | — |
| LinkedList access by index | O(n) | — |

## One-Page Revision
- Big O = worst case (most asked); Big Ω = best case; Big Θ = tight/average.
- Order: O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)
- Nested loops multiply; sequential loops add (keep dominant term).
- Multiplying/dividing loop index → O(log n).
- Recursion always costs stack space too — count call frames.
- HashMap: O(1) avg, O(n) worst (collisions).
- Binary Search O(log n); Merge Sort always O(n log n); Quick Sort O(n²) worst.

---

# MCQs — Chapter 13 (18 Questions)

**Q1.** [Easy | Big O Basics] Time complexity of accessing an array element by index?
A) O(n) B) O(log n) C) O(1) D) O(n²)
**Answer: C**
*Explanation:* Direct memory address calculation from index gives constant-time access.
*Why others wrong:* A, B, D all imply traversal/comparison, unnecessary for direct indexing.

**Q2.** [Easy | Nested Loops] Complexity of a loop inside a loop, both running `n` times?
A) O(n) B) O(n log n) C) O(n²) D) O(2n)
**Answer: C**
*Explanation:* n × n = n² total iterations.
*Why others wrong:* A, D underestimate; B applies to divide-and-conquer, not simple nesting.

**Q3.** [Easy | Searching] Time complexity of Binary Search?
A) O(n) B) O(log n) C) O(n log n) D) O(1)
**Answer: B**
*Explanation:* Search space halves each step → log₂(n) steps.
*Why others wrong:* O(n) is linear search; O(n log n) is for sorting; O(1) implies no searching at all.

**Q4.** [Easy | Ranking] Which grows fastest as n increases?
A) O(n log n) B) O(n²) C) O(2ⁿ) D) O(log n)
**Answer: C**
*Explanation:* Exponential growth outpaces all polynomial/logarithmic complexities.
*Why others wrong:* All grow slower than exponential.

**Q5.** [Easy | Notations] What does Big Ω represent?
A) Worst case B) Best case C) Average case D) Space only
**Answer: B**
*Explanation:* Big Omega is the lower bound — minimum time taken.
*Why others wrong:* Worst case = Big O; tight/average = Big Theta; Omega isn't space-specific.

**Q6.** [Medium | Loop Analysis] Complexity of:
```java
for (int i = 1; i < n; i = i * 2) { System.out.println(i); }
```
A) O(n) B) O(log n) C) O(n²) D) O(1)
**Answer: B**
*Explanation:* i doubles each time (1,2,4,8...) → ~log₂(n) iterations.
*Why others wrong:* O(n) applies to increment-by-1 loops; O(n²)/O(1) don't match doubling behavior.

**Q7.** [Medium | Recursion & Space] What is the space complexity of this function?
```java
void recurse(int n) {
    if (n == 0) return;
    recurse(n - 1);
}
```
A) O(1) B) O(n) C) O(log n) D) O(n²)
**Answer: B**
*Explanation:* Each call adds a stack frame; n calls deep = O(n) stack space.
*Why others wrong:* A is the classic trap — "no array = O(1)" ignores the call stack.

**Q8.** [Medium | HashMap] Worst-case time complexity of HashMap.get()?
A) O(1) B) O(log n) C) O(n) D) O(n log n)
**Answer: C**
*Explanation:* With many hash collisions, lookups degrade to O(n) (or O(log n) in Java 8+ treeified buckets, but classic MCQ answer is O(n)).
*Why others wrong:* O(1) is only the average case, not worst case.

**Q9.** [Medium | Sorting] Best-case time complexity of Bubble Sort with a "swapped" flag optimization on an already-sorted array?
A) O(n) B) O(n²) C) O(log n) D) O(1)
**Answer: A**
*Explanation:* With early-exit optimization, one pass with no swaps confirms sorted order → O(n).
*Why others wrong:* O(n²) is the worst/average case, not best case with optimization.

**Q10.** [Medium | Sequential Loops] Complexity of:
```java
for (int i = 0; i < n; i++) { }
for (int j = 0; j < n; j++) { for (int k = 0; k < n; k++) { } }
```
A) O(n) B) O(n²) C) O(n³) D) O(2n²)
**Answer: B**
*Explanation:* First loop O(n), second block O(n²); sequential loops add, keep dominant term → O(n²).
*Why others wrong:* C overestimates depth; D forgets constants are dropped.

**Q11.** [Hard | Merge Sort] Why is Merge Sort O(n log n) in all cases (best, worst, average)?
A) It uses recursion B) It always divides array in half (log n levels) and merges n elements at each level C) It uses extra space D) It's a comparison sort
**Answer: B**
*Explanation:* log n levels of division × O(n) merge work per level = O(n log n), regardless of input order.
*Why others wrong:* A, C, D are true facts about merge sort but don't explain the *complexity derivation*.

**Q12.** [Hard | Quick Sort] Why does Quick Sort have O(n²) worst case?
A) It uses more memory than merge sort B) Poor pivot selection (e.g., already sorted array with first-element pivot) causes unbalanced partitions C) It is not a stable sort D) It uses recursion
**Answer: B**
*Explanation:* Consistently unbalanced partitions (one side empty) lead to n levels of O(n) work = O(n²).
*Why others wrong:* Memory usage and stability are unrelated to time complexity degradation.

**Q13.** [Medium | Trap] True or False: "A loop running n/2 times is O(n/2)."
A) True B) False
**Answer: B**
*Explanation:* Constants are always dropped; O(n/2) simplifies to O(n).
*Why others wrong:* A is the classic trap Capgemini uses to test rule application.

**Q14.** [Easy | Space] Space complexity of `int[] arr = new int[n];`?
A) O(1) B) O(n) C) O(log n) D) O(n²)
**Answer: B**
*Explanation:* Array of size n requires n units of memory.
*Why others wrong:* O(1) ignores the array allocation entirely.

**Q15.** [Medium | ArrayList vs LinkedList] Time complexity to access the k-th element in a LinkedList?
A) O(1) B) O(log n) C) O(n) D) O(k²)
**Answer: C**
*Explanation:* LinkedList requires traversal from head (no direct indexing) → O(n).
*Why others wrong:* O(1) applies to ArrayList, not LinkedList.

**Q16.** [Hard | Big Theta] When are Big O and Big Ω considered equal, giving a Big Θ bound?
A) Never B) When best case and worst case have the same growth rate C) Only for O(1) algorithms D) Only for sorting algorithms
**Answer: B**
*Explanation:* Θ exists when upper and lower bounds match asymptotically, giving a "tight" bound.
*Why others wrong:* This can happen for many algorithm types, not just O(1) or sorting.

**Q17.** [Easy | Terminology] What does "asymptotic analysis" mean?
A) Analyzing code style B) Studying how an algorithm's resource usage behaves as input size approaches infinity C) Testing code on real hardware D) Counting exact number of operations
**Answer: B**
*Explanation:* Asymptotic analysis studies growth trends for large n, ignoring machine-specific constants.
*Why others wrong:* D describes exact operation counting, which is different from Big-O style analysis.

**Q18.** [Hard | Scenario] An algorithm does `n` work to split the problem, then recurses on two halves of size n/2 each with O(1) combine cost. What recurrence best represents this, and what pattern (not exact solving) should you recognize?
A) T(n) = T(n-1) + O(1) → O(n) B) T(n) = 2T(n/2) + O(n) → O(n log n) (like Merge Sort) C) T(n) = T(n/2) + O(1) → O(log n) (like Binary Search) D) T(n) = 2T(n/2) + O(1) → O(n)
**Answer: B**
*Explanation:* Two recursive calls on half input plus O(n) work per level = classic Merge-Sort-style recurrence → O(n log n).
*Why others wrong:* A/C/D describe different recursion patterns (linear recursion, binary search, or divide-without-merge-cost) with different complexities.

---

## Chapter 13 Complete ✅
If you missed Q7, Q8, Q11, Q12, or Q18 — reread sections 5 and 6 before moving on, since recursion-space and derivation-style questions are the highest-trap-density here.

**Next up: Chapter 14 — Searching & Sorting.**

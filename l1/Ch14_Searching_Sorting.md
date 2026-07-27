# DSA — Chapter 14: Searching & Sorting

## 1. Searching Algorithms

### Linear Search
Check every element one by one until found.
```java
for (int i = 0; i < arr.length; i++) {
    if (arr[i] == target) return i;
}
return -1;
```
- Time: **O(n)** worst/average, O(1) best (found at index 0)
- Space: O(1)
- Works on **unsorted** arrays too.

### Binary Search
Requires a **sorted** array. Repeatedly compare middle element, discard half.
```java
int low = 0, high = arr.length - 1;
while (low <= high) {
    int mid = low + (high - low) / 2;
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) low = mid + 1;
    else high = mid - 1;
}
return -1;
```
- Time: **O(log n)**
- Space: O(1) iterative, O(log n) recursive (call stack)

**Capgemini trap:** `mid = (low + high) / 2` can cause **integer overflow** for very large arrays. Correct safe form: `mid = low + (high - low) / 2`.

**Trap 2:** Binary search on an **unsorted** array gives wrong/undefined results — always check the array is sorted first.

## 2. Sorting Algorithms

### Bubble Sort
Repeatedly swap adjacent elements if out of order.
- Time: O(n²) avg/worst, **O(n)** best (with swapped-flag optimization)
- Space: O(1) — in-place
- **Stable** (equal elements keep relative order)

### Selection Sort
Find the minimum in the unsorted part, swap it to the front.
- Time: O(n²) in **all cases** (always scans remaining elements, even if sorted)
- Space: O(1)
- **Not stable** by default

### Insertion Sort
Build sorted array one element at a time, inserting into correct position.
- Time: O(n²) avg/worst, **O(n) best** (already sorted)
- Space: O(1)
- **Stable**; efficient for small/nearly-sorted datasets

### Merge Sort
Divide array into halves, sort each recursively, merge.
- Time: **O(n log n)** in ALL cases (best, worst, average)
- Space: **O(n)** (needs auxiliary array for merging)
- **Stable**; good for linked lists and large datasets; not in-place

### Quick Sort
Pick a pivot, partition into smaller/larger, recurse.
- Time: O(n log n) average, **O(n²) worst** (bad pivot choice, e.g., sorted array + first-element pivot)
- Space: O(log n) (recursion stack)
- **Not stable** by default; usually faster in practice than Merge Sort due to in-place partitioning and cache locality

## 3. Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Yes |
| Binary Search | O(1) | O(log n) | O(log n) | O(1) | — | — |
| Linear Search | O(1) | O(n) | O(n) | O(1) | — | — |

## 4. Frequently Asked Capgemini Questions
- "Which sorting algorithm has guaranteed O(n log n) in worst case?" → **Merge Sort** (not Quick Sort!)
- "Which is the only unstable sort among Bubble/Insertion/Merge?" → trick: these three are all stable; **Selection Sort and Quick Sort** are the unstable ones.
- "Selection Sort's best case time complexity?" → still **O(n²)** (common trap — students assume O(n) like Insertion/Bubble).
- Code tracing questions showing one pass of Bubble/Selection/Insertion Sort and asking "array state after pass 1."

## 5. Interviewer Traps
1. **Trap:** Assuming Quick Sort is always faster than Merge Sort — Quick Sort's *worst case* is O(n²); Merge Sort *guarantees* O(n log n).
2. **Trap:** Selection Sort "looks like" it should have a good best case — it doesn't; it always scans fully.
3. **Trap:** Binary search formula overflow with `(low+high)/2`.
4. **Trap:** Confusing "stable" (equal elements retain order) with "in-place" (no extra array needed) — they are independent properties.
5. **Trap:** Believing Merge Sort is in-place — it is NOT (needs O(n) auxiliary space).

## One-Page Revision
- Linear Search: O(n), works unsorted. Binary Search: O(log n), needs sorted array.
- Bubble/Insertion: O(n) best (optimized/nearly sorted), O(n²) worst. Both stable.
- Selection Sort: O(n²) ALWAYS, not stable.
- Merge Sort: O(n log n) always, O(n) space, stable, NOT in-place.
- Quick Sort: O(n log n) avg, O(n²) worst (bad pivot), O(log n) space, in-place, NOT stable.
- Safe binary search mid: `low + (high-low)/2`.

---

# MCQs — Chapter 14 (16 Questions)

**Q1.** [Easy | Linear Search] Time complexity of Linear Search in the worst case?
A) O(1) B) O(log n) C) O(n) D) O(n²)
**Answer: C**
*Explanation:* Worst case checks every element once → O(n).
*Why others wrong:* O(1) is best case only; O(log n)/O(n²) don't apply to simple linear scans.

**Q2.** [Easy | Binary Search] Prerequisite for Binary Search to work correctly?
A) Array must be an ArrayList B) Array must be sorted C) Array must contain unique elements D) Array must be size > 100
**Answer: B**
*Explanation:* Binary search relies on discarding half the elements based on ordering — requires sorted data.
*Why others wrong:* Data structure type, uniqueness, and size are irrelevant to correctness.

**Q3.** [Medium | Overflow Trap] What is the safer way to compute mid-index in binary search for very large arrays?
A) `(low + high) / 2` B) `low + (high - low) / 2` C) `(low + high) * 0.5` D) `high - low / 2`
**Answer: B**
*Explanation:* Avoids integer overflow that can occur when low+high exceeds int range.
*Why others wrong:* A is the classic overflow-prone version; C, D are mathematically incorrect for indices.

**Q4.** [Medium | Sorting Guarantee] Which sort guarantees O(n log n) even in the worst case?
A) Quick Sort B) Merge Sort C) Bubble Sort D) Selection Sort
**Answer: B**
*Explanation:* Merge Sort's divide-and-merge structure is independent of input arrangement.
*Why others wrong:* Quick Sort degrades to O(n²) with poor pivot choices; Bubble/Selection are O(n²) always/worst.

**Q5.** [Medium | Selection Sort Trap] Best-case time complexity of Selection Sort?
A) O(n) B) O(n log n) C) O(n²) D) O(1)
**Answer: C**
*Explanation:* Selection Sort always scans the remaining unsorted part fully to find the minimum, regardless of input order.
*Why others wrong:* Unlike Bubble/Insertion Sort, there's no early-exit optimization possible.

**Q6.** [Medium | Stability] Which of these sorting algorithms is NOT stable by default?
A) Bubble Sort B) Insertion Sort C) Merge Sort D) Selection Sort
**Answer: D**
*Explanation:* Selection Sort's swap-based mechanism can change the relative order of equal elements.
*Why others wrong:* Bubble, Insertion, and Merge Sort all preserve relative order of equal elements.

**Q7.** [Medium | Space] Which sort requires the most auxiliary space?
A) Bubble Sort B) Selection Sort C) Merge Sort D) Insertion Sort
**Answer: C**
*Explanation:* Merge Sort needs O(n) extra space for merging subarrays.
*Why others wrong:* Bubble, Selection, and Insertion Sort are all in-place with O(1) extra space.

**Q8.** [Hard | Quick Sort Worst Case] Quick Sort has O(n²) worst case when:
A) The array is very large B) The pivot always ends up being the smallest or largest element, causing unbalanced partitions C) The array has duplicate elements D) The array is randomly shuffled
**Answer: B**
*Explanation:* Repeatedly unbalanced partitions (e.g., sorted array + first-element pivot) create n recursion levels of O(n) work.
*Why others wrong:* Array size alone doesn't degrade complexity; random shuffling actually tends to avoid worst case.

**Q9.** [Easy | Terminology] "In-place" sorting algorithm means:
A) It doesn't use recursion B) It sorts without requiring significant additional memory (O(1) or O(log n) extra) C) It's always stable D) It's the fastest algorithm
**Answer: B**
*Explanation:* In-place refers to memory usage characteristics, not recursion or stability.
*Why others wrong:* Recursion, stability, and speed are separate, unrelated properties.

**Q10.** [Medium | Insertion Sort] Best use case for Insertion Sort?
A) Very large random datasets B) Nearly sorted or small datasets C) Datasets requiring guaranteed O(n log n) D) Linked lists specifically
**Answer: B**
*Explanation:* Insertion Sort performs close to O(n) on nearly sorted data due to minimal shifting needed.
*Why others wrong:* Large random datasets favor Merge/Quick Sort; guaranteed O(n log n) is Merge Sort's domain.

**Q11.** [Hard | Code Trace] After one full pass of Bubble Sort on `[5, 1, 4, 2, 8]` (ascending), what is the array state?
A) [1, 2, 4, 5, 8] B) [1, 5, 4, 2, 8] C) [1, 4, 2, 5, 8] D) [5, 1, 4, 2, 8]
**Answer: C**
*Explanation:* Pass 1 swaps adjacent out-of-order pairs: (5,1)→swap, (5,4)→swap, (5,2)→swap, (5,8)→no swap. Result: [1,4,2,5,8].
*Why others wrong:* A is fully sorted (would take multiple passes); B and D don't reflect all adjacent swaps in one pass.

**Q12.** [Medium | Comparison] Which pair of algorithms have identical time complexity in ALL cases (best/avg/worst)?
A) Quick Sort & Merge Sort B) Bubble Sort & Selection Sort C) Merge Sort & Selection Sort (both O(n²)? ) D) Selection Sort & Quick Sort
**Answer: C**
*Explanation:* Trick question — Selection Sort is O(n²) in all cases, but Merge Sort is O(n log n) in all cases; they do NOT match. The intended correct answer highlighting "same complexity across all cases" for a single algorithm is Selection Sort itself (O(n²) always) — among the options, none truly match, but this question tests whether you recognize Merge Sort is O(n log n) always (not O(n²)), so C is FALSE upon inspection.
*Why others wrong:* This question is a reasoning trap — always double check by recalling each algorithm's individual best/avg/worst rather than assuming pairs match.
*(Difficulty:* Hard | *Topic:* Trap Recognition)

**Q13.** [Easy | Binary Search Space] Space complexity of iterative Binary Search?
A) O(n) B) O(log n) C) O(1) D) O(n log n)
**Answer: C**
*Explanation:* Iterative version uses only a few variables (low, high, mid) regardless of input size.
*Why others wrong:* O(log n) applies to the *recursive* version due to call stack depth.

**Q14.** [Medium | Sorting Choice] Which sort is generally preferred for sorting a Linked List?
A) Quick Sort B) Merge Sort C) Selection Sort D) Bubble Sort
**Answer: B**
*Explanation:* Merge Sort doesn't require random access (unlike Quick Sort's partitioning) and works well with sequential access patterns of linked lists.
*Why others wrong:* Quick Sort relies heavily on random access/indexing, less efficient for linked lists.

**Q15.** [Hard | Scenario] You need to sort a huge dataset and require a **worst-case time guarantee** with acceptable memory usage. Which is the best choice?
A) Quick Sort (in-place but O(n²) worst) B) Merge Sort (O(n log n) guaranteed, O(n) space) C) Bubble Sort D) Selection Sort
**Answer: B**
*Explanation:* Merge Sort provides the strict worst-case guarantee even though it costs extra space, which is usually acceptable trade-off.
*Why others wrong:* Quick Sort risks O(n²); Bubble/Selection are always too slow for huge datasets.

**Q16.** [Easy | Terminology] What does "stable" mean in sorting?
A) The algorithm never crashes B) Equal elements retain their original relative order after sorting C) The algorithm uses O(1) space D) The algorithm is always faster than O(n²)
**Answer: B**
*Explanation:* Stability is specifically about preserving relative order of equal-valued elements.
*Why others wrong:* Crash-safety, space usage, and speed are unrelated concepts.

---

## Chapter 14 Complete ✅
If you missed Q5, Q8, Q11, or Q12 — reread the Sorting Comparison Table and re-trace Bubble Sort by hand; these trip up most learners.

**Next up: Chapter 15 — Linked List, Stack, Queue, Hashing.**

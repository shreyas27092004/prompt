# DSA — Chapter 17: Dynamic Programming, Greedy, Backtracking & Sliding Window/Two Pointer (Basics)

## 1. Why this chapter matters
These four patterns cover almost every "optimize/count/find subsequence" MCQ in L1 tests. They're often confused with each other — Capgemini loves testing "which technique fits this problem" questions. Master the **decision rule** below and 80% of these MCQs become easy.

## 2. The Big Decision Rule (memorize this first)

| Clue in the problem | Technique |
|---|---|
| "Contiguous subarray/substring" + condition on sum/length/distinct chars | **Sliding Window** |
| "Sorted array" / "pair with target sum" / "container with most water" | **Two Pointer** |
| "Overlapping subproblems" + "optimal substructure" (min/max/count ways) | **Dynamic Programming (DP)** |
| "Choose locally best option at each step, never reconsider" | **Greedy** |
| "Try all possibilities, undo choice if it fails" (permutations, N-Queens, Sudoku) | **Backtracking** |

**Capgemini trap:** Students confuse DP and Greedy. Rule of thumb — if the **greedy (locally best) choice can fail** to give the global optimum, you need DP. If greedy always works (provably), use Greedy (it's faster: usually O(n log n) vs DP's O(n²)).

## 3. Dynamic Programming (DP) — Basics

**Core idea:** Break a problem into overlapping subproblems, solve each subproblem **once**, and store the result (memoize) to avoid recomputation.

### 3.1 Two required properties
1. **Overlapping Subproblems** — the same subproblem is solved multiple times in a naive recursive solution (e.g., Fibonacci recomputes `fib(2)` many times).
2. **Optimal Substructure** — the optimal solution to the problem can be built from optimal solutions of its subproblems.

**Trap:** If a problem has overlapping subproblems but NO optimal substructure, DP doesn't apply directly.

### 3.2 Two approaches to DP

| | Approach | How it works | Space |
|---|---|---|---|
| **Top-Down** | Memoization | Normal recursion + a cache (array/map) storing already-computed results | O(n) recursion stack + cache |
| **Bottom-Up** | Tabulation | Build solution iteratively from base cases upward, filling a table | O(n) table (stack-free) |

**Capgemini trap:** "Memoization" = top-down (recursive + cache). "Tabulation" = bottom-up (iterative table). Questions often swap these definitions as wrong options.

### 3.3 Classic example — Fibonacci

```java
// Naive recursion: O(2^n) — recomputes same values repeatedly
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

// Top-down (Memoization): O(n)
int fibMemo(int n, int[] dp) {
    if (n <= 1) return n;
    if (dp[n] != -1) return dp[n];
    return dp[n] = fibMemo(n-1, dp) + fibMemo(n-2, dp);
}

// Bottom-up (Tabulation): O(n), O(1) space possible
int fibTab(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1, curr = 0;
    for (int i = 2; i <= n; i++) {
        curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}
```

### 3.4 Common beginner DP patterns (know these names — frequently named directly in MCQs)
- **0/1 Knapsack** — each item picked at most once; choice = include or exclude.
- **Unbounded Knapsack** — item can be picked unlimited times (e.g., Coin Change).
- **Longest Common Subsequence (LCS)** — 2D DP comparing two strings.
- **Longest Increasing Subsequence (LIS)** — O(n²) DP or O(n log n) with binary search.
- **Coin Change (min coins / count ways)** — classic unbounded knapsack variant.
- **Matrix Path problems** (min cost path, unique paths) — 2D grid DP.

**Trick:** Number of subproblems in most 2D DP problems ≈ (rows × cols), so time complexity is usually O(m×n).

## 4. Greedy Algorithms — Basics

**Core idea:** At every step, pick the option that looks **best right now** (locally optimal), without reconsidering past choices, hoping it leads to a globally optimal answer.

### 4.1 When Greedy works
Greedy gives the correct (globally optimal) answer **only when the problem has the "greedy choice property"** — i.e., a locally optimal choice at each step leads to a globally optimal solution. This must be proven per-problem; it's not automatic.

### 4.2 Classic Greedy examples
- **Activity Selection** — pick the activity that finishes earliest first, repeat.
- **Fractional Knapsack** — pick items by highest value/weight ratio first (unlike 0/1 Knapsack, fractional items are allowed, so Greedy works here but NOT for 0/1 Knapsack).
- **Huffman Coding** — build optimal prefix codes by always merging two lowest-frequency nodes.
- **Dijkstra's Algorithm** — greedily picks the nearest unvisited vertex (only works correctly with non-negative weights).
- **Coin Change (Greedy version)** — works for standard currency systems (like Indian/US coins) but **fails for arbitrary denominations** (e.g., coins {1, 3, 4}, target 6 → Greedy picks 4+1+1=3 coins, but optimal is 3+3=2 coins).

**Capgemini trap #1:** "0/1 Knapsack can be solved using Greedy" → **False**. 0/1 Knapsack needs DP because the greedy (best ratio first) choice can be suboptimal when items can't be split.

**Capgemini trap #2:** "Greedy algorithms always give the optimal solution" → **False**. Only when greedy-choice property + optimal substructure both hold (proven case by case).

## 5. Backtracking — Basics

**Core idea:** Build a solution incrementally, and **abandon ("backtrack")** a path as soon as it's determined that path cannot lead to a valid solution — then try the next option.

### 5.1 The pattern (mental template)
```
function backtrack(state):
    if state is a complete valid solution:
        record it
        return
    for each choice available at this state:
        make the choice
        backtrack(new state)
        undo the choice   // <-- this "undo" step IS backtracking
```

### 5.2 Classic Backtracking problems (frequently named in MCQs)
- **N-Queens** — place N queens on an N×N board so none attack each other; backtrack when a placement conflicts.
- **Sudoku Solver** — try digits 1–9 in each cell, backtrack on constraint violation.
- **Permutations / Combinations / Subsets generation** — try each element in/out, backtrack.
- **Rat in a Maze** — try each direction, backtrack on dead end.

**Trap:** Backtracking is essentially **DFS + pruning**. It explores the full search tree conceptually but cuts off (prunes) invalid branches early — this is why it's more efficient than brute force, but still can be exponential in worst case.

**Backtracking vs DP:** Backtracking usually explores **all valid combinations** (used for enumeration: "find all ways"), while DP is used to find **one optimal value** (min/max/count) by reusing overlapping subproblem results. Backtracking typically does NOT memoize; DP does.

## 6. Sliding Window — Basics

**Core idea:** Maintain a "window" (a contiguous subarray/substring) defined by two pointers (`start`, `end`). Instead of recomputing from scratch for every subarray (O(n²) or worse), slide the window forward, adding/removing one element at a time — O(n) overall.

### 6.1 Two types

| Type | Use case | Window size |
|---|---|---|
| **Fixed-size window** | "Max sum of subarray of size k" | Constant k |
| **Variable-size window** | "Smallest subarray with sum ≥ target" / "Longest substring without repeating characters" | Grows/shrinks based on a condition |

### 6.2 Fixed-size window example
```java
// Max sum of subarray of size k
int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0, maxSum;
    for (int i = 0; i < k; i++) windowSum += arr[i];   // build first window
    maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];  // slide: add new, remove old
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```
**Trick:** This turns an O(n×k) brute-force into O(n) — each element is added once and removed once.

### 6.3 Variable-size window example
```java
// Longest substring without repeating characters
int longestUniqueSubstring(String s) {
    Set<Character> seen = new HashSet<>();
    int left = 0, maxLen = 0;
    for (int right = 0; right < s.length(); right++) {
        while (seen.contains(s.charAt(right))) {
            seen.remove(s.charAt(left));
            left++;   // shrink window from the left
        }
        seen.add(s.charAt(right));
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

## 7. Two Pointer — Basics

**Core idea:** Use two indices (usually moving toward each other, or one fast/one slow) to avoid nested loops — typically on a **sorted** array or when checking pairs/palindromes.

### 7.1 Classic patterns
- **Opposite ends closing in** — e.g., "Pair with given sum in sorted array": `left` starts at 0, `right` at end; if sum too small, move `left++`; if too big, move `right--`.
- **Fast & slow pointer** — e.g., detecting a cycle in a linked list (Floyd's algorithm), finding the middle of a list.
- **Same-direction two pointer** — e.g., removing duplicates from a sorted array in-place.

```java
// Pair with target sum in a SORTED array
boolean hasPairWithSum(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) return true;
        else if (sum < target) left++;
        else right--;
    }
    return false;
}
```

**Capgemini trap:** Two Pointer on an **unsorted** array for "pair sum" does NOT work directly — the array must be sorted first (or use a HashSet instead, which doesn't need sorting but uses O(n) extra space).

## 8. Sliding Window vs Two Pointer — often confused
| | Sliding Window | Two Pointer |
|---|---|---|
| Deals with | A contiguous **range/window** as a whole (size matters) | Two **specific indices**, not necessarily forming a "window" of interest |
| Typical question | Subarray/substring sum, length, distinct count | Pair sum, palindrome check, cycle detection |
| Data requirement | No sorting required | Often (not always) needs sorted data |

**Trap:** Many MCQs use these terms almost interchangeably in casual explanations, but technically Sliding Window is a **specialized case** often implemented *using* two pointers. If asked "which is more general," the answer is **Two Pointer** (Sliding Window is one specific application pattern of it).

## One-Page Revision

- **DP** = overlapping subproblems + optimal substructure → memoization (top-down, recursive+cache) OR tabulation (bottom-up, iterative table).
- **Greedy** = locally best choice each step; only correct if greedy-choice property holds (e.g., Fractional Knapsack ✅, 0/1 Knapsack ❌ needs DP).
- **Backtracking** = DFS + pruning; builds solution incrementally, undoes invalid choices; used for "find all ways" (N-Queens, permutations, Sudoku).
- **Sliding Window** = contiguous range tracked by moving start/end pointers; fixed-size (constant k) or variable-size (grows/shrinks on condition); turns O(n×k) or O(n²) into O(n).
- **Two Pointer** = two indices (opposite ends or fast/slow) reducing nested loops to O(n); classic use = pair-sum in **sorted** array, cycle detection.
- Key traps: JRE-style traps here are — "Greedy always optimal" (False), "0/1 Knapsack via Greedy" (False), "Two Pointer works on unsorted array for pair-sum" (False, needs sort or HashSet), "Memoization = bottom-up" (False, it's top-down).

---

# MCQs — Chapter 17 (25 Questions)

**Q1.** [Easy | DP Basics] Dynamic Programming is most suitable when a problem has:
A) Only optimal substructure B) Only overlapping subproblems C) Both overlapping subproblems and optimal substructure D) Neither property
**Answer: C**
*Explanation:* DP requires both properties — subproblems must repeat (overlapping) AND the optimal answer must be buildable from optimal subproblem answers (optimal substructure).
*Why others wrong:* A) Without overlapping subproblems, plain recursion/divide-and-conquer suffices, no need to cache. B) Without optimal substructure, caching won't help build a correct global optimum. D) Then DP doesn't apply at all.

**Q2.** [Easy | Terminology] Memoization refers to:
A) Bottom-up iterative table filling B) Top-down recursion with cached results C) A sorting technique D) A type of greedy algorithm
**Answer: B**
*Explanation:* Memoization stores results of recursive calls (top-down) so repeated subproblems return the cached value instead of recomputing.
*Why others wrong:* A) That's tabulation, the opposite approach. C) Unrelated to sorting. D) Memoization is a DP technique, not greedy.

**Q3.** [Easy | Terminology] Tabulation refers to:
A) Top-down recursive caching B) Bottom-up iterative table building starting from base cases C) Random selection of subproblems D) Backtracking with pruning
**Answer: B**
*Explanation:* Tabulation fills a DP table iteratively from the smallest subproblems upward until the final answer is reached.
*Why others wrong:* A) That's memoization. C) Tabulation is systematic, not random. D) Backtracking is a different technique entirely.

**Q4.** [Easy | Greedy] Greedy algorithms make choices based on:
A) Global optimum evaluated for all future steps B) What looks best at the current step only C) Random selection D) Exhaustive search of all possibilities
**Answer: B**
*Explanation:* Greedy picks the locally optimal choice at each step without reconsidering it later.
*Why others wrong:* A) That would require full lookahead, which greedy explicitly avoids (that's more like DP/exhaustive search). C) Greedy is deterministic, not random. D) Exhaustive search describes backtracking/brute force, not greedy.

**Q5.** [Medium | Greedy Trap] Which problem CANNOT be correctly solved using a simple Greedy approach?
A) Fractional Knapsack B) Activity Selection C) 0/1 Knapsack D) Huffman Coding
**Answer: C**
*Explanation:* 0/1 Knapsack requires trying combinations because items can't be split — the locally best ratio item may not fit optimally, so DP is needed.
*Why others wrong:* A, B, D all have a proven greedy-choice property and are correctly solved by Greedy.

**Q6.** [Medium | Backtracking] Backtracking is best described as:
A) Breadth-First Search with memoization B) Depth-First Search with pruning of invalid branches C) A greedy selection technique D) A sorting algorithm
**Answer: B**
*Explanation:* Backtracking explores choices depth-first, abandoning ("backtracking" from) a path as soon as it's known invalid, then trying the next option.
*Why others wrong:* A) BFS explores level by level, not depth-first with undo. C) Backtracking explores multiple options, not just one greedy pick. D) Unrelated to sorting.

**Q7.** [Medium | Backtracking] Which of these is a classic Backtracking problem?
A) Binary Search B) N-Queens C) Merge Sort D) Dijkstra's Algorithm
**Answer: B**
*Explanation:* N-Queens requires placing queens and undoing invalid placements — a textbook backtracking problem.
*Why others wrong:* A) Binary Search is a direct search technique, no backtracking. C) Merge Sort is divide-and-conquer sorting. D) Dijkstra's is a greedy shortest-path algorithm.

**Q8.** [Medium | Sliding Window] The main advantage of the Sliding Window technique over brute force for "max sum subarray of size k" is:
A) It reduces time complexity from O(n×k) to O(n) B) It reduces space complexity to O(1) always C) It sorts the array first D) It only works on sorted arrays
**Answer: A**
*Explanation:* Instead of recomputing the sum for every window from scratch, sliding window adds the new element and removes the old one — one pass, O(n).
*Why others wrong:* B) While often O(1) extra space, that's not the "main" advantage being tested here — time complexity is the key win. C) Sliding window doesn't require sorting. D) It works on any array, sorted or not.

**Q9.** [Medium | Two Pointer] Two Pointer technique for finding a pair with a given sum works correctly and efficiently on:
A) Any unsorted array directly B) A sorted array C) Only arrays with unique elements D) Only arrays of even length
**Answer: B**
*Explanation:* The classic O(n) two-pointer approach (moving left/right based on sum comparison) depends on the array being sorted so pointer movement is decisive.
*Why others wrong:* A) On unsorted arrays you'd need to sort first (O(n log n)) or use a HashSet instead. C) Works regardless of duplicates. D) Array length parity is irrelevant.

**Q10.** [Easy | DP Example] In the Fibonacci sequence computed via naive recursion (no memoization), the time complexity is:
A) O(n) B) O(n log n) C) O(2^n) D) O(n²)
**Answer: C**
*Explanation:* Naive recursive Fibonacci branches into 2 calls per call without caching, leading to exponential O(2^n) time due to massive repeated recomputation.
*Why others wrong:* A, B, D are all achievable only with memoization/tabulation/matrix exponentiation optimizations, not the naive version.

**Q11.** [Medium | Code Output] What does this print?
```java
int[] dp = new int[10];
Arrays.fill(dp, -1);
System.out.println(fibMemo(6, dp));
// where fibMemo is standard top-down Fibonacci memoization
```
A) 5 B) 6 C) 8 D) 13
**Answer: C**
*Explanation:* Fibonacci sequence: fib(0)=0, fib(1)=1, fib(2)=1, fib(3)=2, fib(4)=3, fib(5)=5, fib(6)=8.
*Why others wrong:* A) That's fib(5). B) Not a Fibonacci value at that index. D) That's fib(7).

**Q12.** [Hard | Greedy Trap] Coin denominations {1, 3, 4}, target sum = 6. What does a Greedy (largest coin first) approach produce, and is it optimal?
A) 4+1+1 = 3 coins; optimal B) 3+3 = 2 coins; optimal C) 4+1+1 = 3 coins; NOT optimal (2 coins exist) D) 1+1+1+3 = 4 coins; optimal
**Answer: C**
*Explanation:* Greedy picks 4 first (largest ≤ 6), leaving 2, then two 1s → 3 coins. But 3+3=6 uses only 2 coins — Greedy fails here, proving Greedy isn't always optimal for coin change with arbitrary denominations.
*Why others wrong:* A) Correct coin count for greedy but wrongly labeled optimal. B) That's the true optimal, but not what Greedy actually produces. D) Neither the greedy result nor optimal.

**Q13.** [Medium | Sliding Window Type] "Find the smallest subarray with sum ≥ a target value" is an example of:
A) Fixed-size sliding window B) Variable-size sliding window C) Two Pointer on sorted array only D) Backtracking
**Answer: B**
*Explanation:* The window size isn't fixed — it grows (adding elements) until the condition is met, then shrinks from the left to minimize, which is the variable-size window pattern.
*Why others wrong:* A) Fixed-size implies a constant k, not applicable here. C) No sorting is involved or required. D) No need to undo choices; this is a linear scan, not exploratory search.

**Q14.** [Medium | Backtracking vs DP] The key difference between Backtracking and DP for optimization problems is:
A) Backtracking always memoizes results; DP never does B) DP finds one optimal value by reusing overlapping subproblems; Backtracking typically enumerates all valid solutions without memoization C) They are identical techniques D) Backtracking is always faster than DP
**Answer: B**
*Explanation:* DP is designed to avoid recomputation for optimal value/count problems; Backtracking is designed for enumeration (find all ways) and generally explores fresh each branch.
*Why others wrong:* A) Reversed — DP is the one associated with memoization. C) They solve different problem types and work differently. D) Backtracking can be exponential and slower for optimization-type problems where DP applies.

**Q15.** [Easy | Two Pointer] Floyd's Cycle Detection algorithm for linked lists uses which pointer strategy?
A) Two pointers moving in opposite directions B) Fast and slow pointer moving in the same direction at different speeds C) Random pointer jumps D) A single pointer with a visited-set
**Answer: B**
*Explanation:* The "slow" pointer moves one step, the "fast" pointer moves two steps; if there's a cycle, they eventually meet.
*Why others wrong:* A) Opposite-direction closing pointers are used for sorted-array pair-sum problems, not cycle detection. C) Not how Floyd's algorithm works — it's deterministic. D) That's a different (HashSet-based) approach to cycle detection, not Floyd's two-pointer method.

**Q16.** [Hard | DP Pattern] 0/1 Knapsack DP table is typically built with dimensions:
A) O(n) — single array only, always B) O(items × capacity) C) O(items²) D) O(capacity²)
**Answer: B**
*Explanation:* The classic 0/1 Knapsack DP table has one dimension per item and one per possible weight/capacity value, giving O(n × W) time and space (space can be optimized to O(W) with a 1D array, but the fundamental table is 2D).
*Why others wrong:* A) A pure 1D array is only possible via space-optimization, not the "typical"/fundamental table structure being tested. C, D) Neither matches the actual (items × capacity) structure.

**Q17.** [Medium | Trap] True or False: "Sliding Window technique requires the input array to be sorted."
A) True B) False
**Answer: B**
*Explanation:* Sliding window works on the array/string in its given order (contiguous elements) — sorting is neither required nor typically desired, since it's about contiguous ranges, not sorted order.
*Why others wrong:* A is the trap — students confuse it with Two Pointer's sorted-array requirement, but they're different techniques with different prerequisites.

**Q18.** [Medium | Scenario] You need to generate ALL possible subsets of a set. The most appropriate technique is:
A) Dynamic Programming B) Greedy C) Backtracking D) Sliding Window
**Answer: C**
*Explanation:* Generating all subsets means exploring "include/exclude" choices for each element and enumerating every resulting combination — the classic backtracking (or recursive enumeration) pattern.
*Why others wrong:* A) DP would compute an optimal single value/count, not enumerate every subset. B) Greedy picks one path, not all combinations. D) Sliding window applies to contiguous ranges, not arbitrary subsets.

**Q19.** [Hard | Scenario] You need the MINIMUM number of coins to make a target amount using given denominations (not necessarily standard currency). What's the safest technique?
A) Greedy (largest coin first) always B) Dynamic Programming C) Two Pointer D) Sliding Window
**Answer: B**
*Explanation:* Since greedy can fail for arbitrary denominations (see Q12), DP (bottom-up, building up the min-coins table for every amount from 0 to target) guarantees the correct optimal answer.
*Why others wrong:* A) Only safe for specific "canonical" coin systems, not guaranteed in general — risky as a default. C, D) Not applicable to this counting/optimization problem structure.

**Q20.** [Medium | Code Trace] What is the time complexity of this sliding window code for finding max sum of subarray size k?
```java
int windowSum = 0;
for (int i = 0; i < k; i++) windowSum += arr[i];
int maxSum = windowSum;
for (int i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];
    maxSum = Math.max(maxSum, windowSum);
}
```
A) O(n × k) B) O(n) C) O(k²) D) O(n log n)
**Answer: B**
*Explanation:* Each element is visited a constant number of times (added once, subtracted once), giving linear O(n) time regardless of k.
*Why others wrong:* A) That would be the brute-force approach without sliding window optimization. C, D) Neither matches the simple single-pass structure shown.

**Q21.** [Hard | Trap] True or False: "Every problem solvable by Dynamic Programming can also be solved (less efficiently) by plain Backtracking/recursion without memoization."
A) True B) False
**Answer: A**
*Explanation:* DP is essentially recursion + caching; removing the cache still gives a correct (just exponentially slower) recursive/backtracking solution, since the recurrence relation itself is still valid.
*Why others wrong:* B is the trap — DP doesn't invent a new correctness relation, it just speeds up existing recursive relations via caching, so the un-cached version is still correct, just slow.

**Q22.** [Medium | Two Pointer Trap] For "check if a string is a palindrome," which technique is most natural?
A) Dynamic Programming B) Two Pointer (from both ends moving inward) C) Backtracking D) Sliding Window
**Answer: B**
*Explanation:* Compare characters at `left` and `right` indices, moving inward; mismatch means not a palindrome — classic two-pointer usage.
*Why others wrong:* A) Overkill for a simple linear check (DP is used for more complex palindrome problems like "longest palindromic substring"). C) No need to explore/undo choices for a direct comparison check. D) No contiguous "window" concept needed here, just direct index comparison.

**Q23.** [Easy | Terminology] "Optimal substructure" means:
A) The problem can only be solved with sorting B) An optimal solution to the problem can be constructed from optimal solutions of its subproblems C) The problem has no repeating subproblems D) The problem must be solved iteratively, never recursively
**Answer: B**
*Explanation:* This is the textbook definition — it's the property (alongside overlapping subproblems) that makes DP applicable.
*Why others wrong:* A) Unrelated to sorting. C) That describes the opposite of "overlapping subproblems," a separate DP requirement. D) DP can be solved either recursively (top-down) or iteratively (bottom-up).

**Q24.** [Hard | Scenario] A problem requires finding the longest substring with at most 2 distinct characters. Best technique?
A) Two Pointer on sorted string B) Variable-size Sliding Window with a character-frequency map C) Backtracking with pruning D) Greedy single pass without any window
**Answer: B**
*Explanation:* Expand the window (right pointer), track character frequencies; when distinct count exceeds 2, shrink from the left until valid again — a classic variable-size sliding window pattern.
*Why others wrong:* A) Strings aren't meant to be sorted here — order matters for "substring." C) No need for full exploratory backtracking; the window approach is linear and sufficient. D) A naive single greedy pass without window/frequency tracking can't correctly enforce the "at most 2 distinct" constraint.

**Q25.** [Hard | Integration] Match each technique to its typical time complexity class for an input of size n (best-known basic implementation): (1) Two Pointer pair-sum on sorted array (2) 0/1 Knapsack DP (n items, capacity W) (3) N-Queens Backtracking (n queens) (4) Sliding Window max-sum subarray
A) (1) O(n), (2) O(n×W), (3) exponential/factorial in worst case, (4) O(n) B) (1) O(n²), (2) O(n), (3) O(n), (4) O(n²) C) (1) O(log n), (2) O(n²), (3) O(n log n), (4) O(1) D) All are O(n) uniformly
**Answer: A**
*Explanation:* Two Pointer pair-sum is a single linear pass = O(n). 0/1 Knapsack DP fills an (items × capacity) table = O(n×W). N-Queens backtracking explores a search tree that can be exponential/factorial-like in the worst case despite pruning. Sliding window max-sum is a single linear pass = O(n).
*Why others wrong:* B, C, D all misassign at least one complexity class, mixing up techniques that have fundamentally different complexity behavior.

---

## Chapter 17 Complete ✅
Score yourself: if you missed any of Q5, Q12, Q16, Q19, Q21, or Q25 — reread the Greedy Traps (Section 4) and DP vs Backtracking distinction (Section 5) before moving on, since these are the highest-trap-density questions.

**Next up: Module 2 continues — Trees, Graphs (BST, AVL, Heap, DFS/BFS, Topological Sort) or say which chapter you'd like next.**

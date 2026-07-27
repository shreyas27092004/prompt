# DSA — Chapter 16: Trees, Heap, Graphs (BFS/DFS)

## 1. Trees — Basics

A **Tree** is a hierarchical, non-linear structure with a root node and child nodes; no cycles.

**Key terms:** root, parent, child, leaf (no children), height (longest path root→leaf), depth (distance from root to a node).

### Binary Tree
Each node has **at most 2 children** (left, right).

### Binary Search Tree (BST)
A Binary Tree where: **left subtree < node < right subtree** for every node.
- Search/Insert/Delete: **O(log n) average** (balanced), **O(n) worst** (skewed, e.g., inserting sorted data creates a linked-list-like tree)

### Tree Traversals (very frequently asked)

| Traversal | Order | Use case |
|---|---|---|
| **Inorder** (Left, Root, Right) | Gives sorted order for a BST | Retrieve sorted data |
| **Preorder** (Root, Left, Right) | Root visited first | Copying/serializing a tree |
| **Postorder** (Left, Right, Root) | Root visited last | Deleting a tree, evaluating expression trees |
| **Level Order** | Level by level (uses a Queue) | BFS-style traversal |

**Capgemini trap:** "Inorder traversal of a BST gives ___" → **sorted (ascending) order**. This is one of the most repeated questions.

### AVL Tree
A **self-balancing** BST where the height difference (balance factor) between left and right subtrees of any node is at most 1. Guarantees **O(log n)** for search/insert/delete even in worst case (unlike a plain BST).

**Rotations** (LL, RR, LR, RL) are used to rebalance after insertion/deletion.

## 2. Heap

A **Heap** is a complete binary tree satisfying the heap property:
- **Max-Heap:** parent ≥ children (root = maximum element)
- **Min-Heap:** parent ≤ children (root = minimum element)

Usually implemented using an **array** (no explicit pointers needed):
- For node at index `i`: left child = `2i+1`, right child = `2i+2`, parent = `(i-1)/2`

| Operation | Time Complexity |
|---|---|
| Get min/max (peek root) | O(1) |
| Insert | O(log n) |
| Delete root (extract min/max) | O(log n) |
| Build heap from array | O(n) |

**Use case:** Priority Queue, Heap Sort, finding k-th smallest/largest element.

**Capgemini trap:** "Building a heap from n elements takes O(n log n)" → **False**, it's actually **O(n)** using the bottom-up heapify approach (a commonly mis-stated fact).

## 3. Graphs

A **Graph** = set of vertices (nodes) + edges (connections). Can be:
- **Directed** (edges have direction) vs **Undirected**
- **Weighted** (edges have costs) vs **Unweighted**
- **Cyclic** vs **Acyclic**

### Representations
| Representation | Space | Edge lookup |
|---|---|---|
| Adjacency Matrix | O(V²) | O(1) |
| Adjacency List | O(V + E) | O(degree of vertex) |

**Capgemini trap:** Adjacency Matrix is better for **dense** graphs; Adjacency List is better for **sparse** graphs (most real-world graphs are sparse).

### BFS (Breadth-First Search)
Explores level by level, using a **Queue**.
- Time: O(V + E)
- Use case: shortest path in **unweighted** graphs, level-order tree traversal

```java
Queue<Integer> queue = new LinkedList<>();
boolean[] visited = new boolean[n];
queue.add(start); visited[start] = true;
while (!queue.isEmpty()) {
    int node = queue.poll();
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            visited[neighbor] = true;
            queue.add(neighbor);
        }
    }
}
```

### DFS (Depth-First Search)
Explores as deep as possible before backtracking, using a **Stack** (or recursion).
- Time: O(V + E)
- Use case: cycle detection, topological sort, connected components, path existence

**Capgemini trap:** BFS uses a **Queue**, DFS uses a **Stack**/recursion — one of the most repeated MCQ facts. Mixing these up is the #1 mistake.

### Topological Sort
Only valid for a **Directed Acyclic Graph (DAG)**. Orders vertices such that for every directed edge u→v, u comes before v. Used for task scheduling with dependencies.

### Shortest Path Basics
| Algorithm | Use case |
|---|---|
| BFS | Shortest path in unweighted graphs |
| Dijkstra's Algorithm | Shortest path in weighted graphs with **non-negative** weights |
| Bellman-Ford | Shortest path, handles **negative** weights too |

## 4. Frequently Asked Capgemini Questions
- "Inorder traversal of BST gives?" → Sorted ascending order
- "Which traversal is used to delete a tree safely (children before parent)?" → Postorder
- "BFS uses which data structure?" → Queue
- "DFS uses which data structure?" → Stack (or recursion)
- "Time complexity to build a heap from an array?" → O(n)
- "Worst case time complexity of BST search?" → O(n) (skewed tree)

## 5. Interviewer Traps
1. Plain BST worst case is O(n) (skewed like a linked list) — only **AVL/Red-Black Trees** guarantee O(log n) worst case.
2. Confusing Min-Heap and Max-Heap root property.
3. Assuming Dijkstra's works with negative weights — it does **NOT**; use Bellman-Ford instead.
4. Forgetting that Topological Sort requires the graph to be a **DAG** (fails/undefined with cycles).
5. Heap is NOT a BST — heap only guarantees parent-child order, not full left-right ordering.

## One-Page Revision
- Tree traversals: Inorder (sorted for BST), Preorder (copy), Postorder (delete), Level-order (BFS-style, uses Queue).
- BST: O(log n) avg, O(n) worst (skewed). AVL: O(log n) guaranteed via rotations.
- Heap: array-based complete binary tree; O(1) peek, O(log n) insert/delete, O(n) build.
- Graph: Adjacency List for sparse (common), Matrix for dense.
- BFS → Queue, shortest path unweighted. DFS → Stack/recursion, cycle detection/topological sort.
- Dijkstra: non-negative weights only. Bellman-Ford: handles negative weights.
- Topological Sort: only valid on DAGs.

---

# MCQs — Chapter 16 (18 Questions)

**Q1.** [Easy | Traversal] What does Inorder traversal of a BST produce?
A) Random order B) Sorted ascending order C) Reverse sorted order D) Level-by-level order
**Answer: B**
*Explanation:* Left-Root-Right visiting order naturally yields ascending sorted values for a BST.
*Why others wrong:* Level-by-level describes level order; reverse order would need Right-Root-Left.

**Q2.** [Easy | BFS/DFS] Which data structure does BFS use internally?
A) Stack B) Queue C) Heap D) Array only
**Answer: B**
*Explanation:* BFS processes nodes level by level using FIFO order via a Queue.
*Why others wrong:* Stack is used by DFS; Heap is for priority-based structures.

**Q3.** [Easy | BFS/DFS] Which data structure does DFS use internally?
A) Queue B) Stack (or recursion, which uses the call stack) C) Heap D) Priority Queue
**Answer: B**
*Explanation:* DFS explores deeply before backtracking, matching LIFO stack behavior (explicit or via recursion).
*Why others wrong:* Queue is BFS's structure; heap/priority queue relate to weighted shortest path algorithms.

**Q4.** [Medium | BST Worst Case] Worst-case time complexity for search in a plain (unbalanced) BST?
A) O(1) B) O(log n) C) O(n) D) O(n log n)
**Answer: C**
*Explanation:* A skewed BST (e.g., built from sorted input) degenerates into a linked-list-like structure.
*Why others wrong:* O(log n) only holds for balanced trees like AVL.

**Q5.** [Medium | Heap Build] Time complexity to build a heap from an unsorted array of n elements?
A) O(n log n) B) O(n) C) O(log n) D) O(n²)
**Answer: B**
*Explanation:* Bottom-up heapify achieves linear time, a commonly mis-stated fact (many assume O(n log n)).
*Why others wrong:* O(n log n) would be the cost of n individual insertions, not the optimized build-heap approach.

**Q6.** [Medium | Heap Property] In a Max-Heap, which statement is true?
A) Root is always the smallest element B) Root is always the largest element C) Left child is always greater than right child D) It must be a complete BST
**Answer: B**
*Explanation:* Max-Heap property ensures parent ≥ children, so root holds the maximum value.
*Why others wrong:* Left/right child ordering isn't guaranteed in a heap; a heap is not a BST.

**Q7.** [Medium | Graph Representation] Which graph representation is more space-efficient for sparse graphs?
A) Adjacency Matrix B) Adjacency List C) Both are equal D) Neither works for sparse graphs
**Answer: B**
*Explanation:* Adjacency List uses O(V+E) space, avoiding the O(V²) overhead of a matrix when edges are few.
*Why others wrong:* Adjacency Matrix wastes space representing non-existent edges in sparse graphs.

**Q8.** [Hard | Shortest Path] Which algorithm correctly handles graphs with negative edge weights?
A) Dijkstra's Algorithm B) BFS C) Bellman-Ford Algorithm D) DFS
**Answer: C**
*Explanation:* Bellman-Ford can detect and correctly compute shortest paths even with negative weights (and detects negative cycles).
*Why others wrong:* Dijkstra's fails with negative weights; BFS/DFS don't account for weighted edges at all.

**Q9.** [Medium | Topological Sort] Topological Sort is only valid for:
A) Any graph B) Undirected graphs C) Directed Acyclic Graphs (DAG) D) Weighted graphs only
**Answer: C**
*Explanation:* Ordering based on dependencies (u before v for edge u→v) requires direction and no cycles.
*Why others wrong:* Cycles make a valid linear ordering impossible; undirected graphs don't have the directional dependency needed.

**Q10.** [Easy | Tree Terms] A node with no children is called a:
A) Root B) Parent C) Leaf D) Sibling
**Answer: C**
*Explanation:* Leaf nodes are the terminal nodes at the bottom of a tree.
*Why others wrong:* Root is the topmost node; parent/sibling describe relational positions, not childlessness.

**Q11.** [Medium | Traversal Use Case] Which traversal should you use to safely delete a tree (free children before the parent)?
A) Preorder B) Inorder C) Postorder D) Level order
**Answer: C**
*Explanation:* Postorder visits children before the parent (Left, Right, Root), ensuring nodes are freed bottom-up safely.
*Why others wrong:* Preorder visits root first (parent would be deleted before children), which is unsafe.

**Q12.** [Hard | AVL Trees] What guarantees O(log n) worst-case operations in an AVL Tree, unlike a plain BST?
A) It stores data in an array B) Self-balancing via rotations keeps height difference ≤ 1 between subtrees C) It doesn't allow duplicate values D) It uses a hash function
**Answer: B**
*Explanation:* Rotations (LL, RR, LR, RL) maintain balance after every insertion/deletion, preventing skewed height.
*Why others wrong:* Array storage and hash functions are unrelated to AVL's balancing mechanism.

**Q13.** [Medium | Graph Traversal Application] Which traversal (BFS or DFS) is best suited to find the shortest path in an unweighted graph?
A) DFS B) BFS C) Either works equally well D) Neither
**Answer: B**
*Explanation:* BFS explores nodes level by level, guaranteeing the first time a node is reached is via the shortest path (unweighted).
*Why others wrong:* DFS may find a longer path first since it explores depth-first without level guarantees.

**Q14.** [Easy | Complete Binary Tree] What defines a "complete" binary tree (relevant to Heaps)?
A) All levels are fully filled except possibly the last, which fills left to right B) Every node has exactly 2 children C) It must be a BST D) All leaves are at the same depth
**Answer: A**
*Explanation:* This property allows heaps to be efficiently represented as arrays.
*Why others wrong:* D describes a "perfect" binary tree, a stricter subset of complete trees.

**Q15.** [Hard | Scenario] You insert values 1, 2, 3, 4, 5 in that order into a plain BST. What is the resulting structure's shape and search time complexity?
A) Balanced tree, O(log n) B) Skewed tree (like a linked list), O(n) C) Complete binary tree, O(1) D) It becomes a heap automatically
**Answer: B**
*Explanation:* Inserting sorted ascending values into a plain BST creates a right-skewed chain, degrading search to O(n).
*Why others wrong:* Only self-balancing trees (AVL, Red-Black) would prevent this degeneration.

**Q16.** [Medium | Graph Terms] A graph with weighted edges and no cycles that connects all vertices with minimum total edge weight is called a:
A) Spanning Tree B) Minimum Spanning Tree (MST) C) Binary Tree D) Complete Graph
**Answer: B**
*Explanation:* MST connects all vertices with the least possible total edge weight, without cycles.
*Why others wrong:* A Spanning Tree alone doesn't guarantee minimum weight; the other options are unrelated structures.

**Q17.** [Medium | Heap vs BST] Which statement correctly distinguishes a Heap from a BST?
A) A Heap is always sorted left-to-right like a BST B) A Heap only guarantees parent-child ordering, not full left-right ordering like a BST C) A Heap cannot be represented as an array D) A BST always has O(log n) worst case
**Answer: B**
*Explanation:* Heaps enforce parent ≥ (or ≤) children only; there's no guarantee about relative order between siblings or subtrees, unlike BST's strict left<root<right rule.
*Why others wrong:* Heaps ARE typically array-represented; plain BSTs can degrade to O(n) worst case.

**Q18.** [Hard | Dijkstra Trap] Why can't Dijkstra's Algorithm handle negative edge weights correctly?
A) It runs out of memory B) It greedily finalizes the shortest distance to a node once visited, which can be invalidated later by a negative-weight edge C) It only works on undirected graphs D) It's actually fine with negative weights
**Answer: B**
*Explanation:* Dijkstra's greedy approach assumes once a node's shortest distance is finalized, it can't improve — a false assumption when negative weights exist.
*Why others wrong:* Memory isn't the issue, and it works on both directed/undirected graphs (just not negative weights).

---

## Chapter 16 Complete ✅
If you missed Q5, Q8, Q15, or Q18 — reread sections 2 and 3 on Heap-build complexity and shortest-path algorithm limitations; these are the highest-trap-density areas.

**Next up: Chapter 17 — DP, Greedy, Backtracking, Sliding Window & Two Pointer.**

# Big-O Notation & Complexity Analysis

## Common Complexities (fastest → slowest)

| Notation   | Name         | Example                          |
|-----------|-------------|----------------------------------|
| O(1)      | Constant    | Hash map lookup, array index     |
| O(log n)  | Logarithmic | Binary search, balanced BST ops  |
| O(n)      | Linear      | Single loop, linear search       |
| O(n log n)| Log-linear  | Merge sort, heap sort            |
| O(n²)     | Quadratic   | Nested loops, bubble sort        |
| O(2ⁿ)     | Exponential | Subsets, naive recursive Fibonacci|
| O(n!)     | Factorial   | Permutations, brute-force TSP    |

---

## Rules for Analyzing Code

1. **Single loop** over input → O(n)
2. **Nested loops** → O(n²) (or O(n × m) for different-sized inputs)
3. **Halving each step** → O(log n)
4. **Recursion** → draw the recursion tree; total work = nodes × work per node
5. **Drop constants** — O(3n) → O(n)
6. **Drop non-dominant terms** — O(n² + n) → O(n²)
7. **Different variables stay separate** — O(a + b) is not O(n)

---

## Analyzing Loops

### Single Loop — O(n)

```javascript
function sum(arr) {
  let total = 0;
  for (let i = 0; i < arr.length; i++) {
    total += arr[i]; // O(1) work, n iterations
  }
  return total;
}
```

```python
def total(arr):
    result = 0
    for num in arr:     # O(1) work, n iterations
        result += num
    return result
```

```csharp
public int Sum(int[] arr)
{
    int total = 0;
    for (int i = 0; i < arr.Length; i++)
        total += arr[i]; // O(1) work, n iterations
    return total;
}
```

### Nested Loops — O(n²)

```javascript
function printPairs(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      console.log(arr[i], arr[j]); // n*(n-1)/2 pairs → O(n²)
    }
  }
}
```

```python
def print_pairs(arr):
    for i in range(len(arr)):
        for j in range(i + 1, len(arr)):
            print(arr[i], arr[j])   # n*(n-1)/2 pairs → O(n²)
```

```csharp
public void PrintPairs(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
        for (int j = i + 1; j < arr.Length; j++)
            Console.WriteLine($"{arr[i]} {arr[j]}"); // O(n²)
}
```

### Halving — O(log n)

```javascript
function countHalves(n) {
  let count = 0;
  while (n > 1) {
    n = Math.floor(n / 2); // halves each step → O(log n)
    count++;
  }
  return count;
}
```

```python
def count_halves(n):
    count = 0
    while n > 1:
        n //= 2       # halves each step → O(log n)
        count += 1
    return count
```

```csharp
public int CountHalves(int n)
{
    int count = 0;
    while (n > 1)
    {
        n /= 2;       // halves each step → O(log n)
        count++;
    }
    return count;
}
```

---

## Space Complexity

| Concept            | Space      | Example                          |
|-------------------|------------|----------------------------------|
| In-place algorithm | O(1)       | Reverse array with two pointers  |
| Hash map of input  | O(n)       | Two Sum — store seen values      |
| Recursion stack    | O(depth)   | DFS — each call adds a frame     |
| 2D DP table        | O(n × m)   | Edit Distance tabulation         |
| Adjacency list     | O(V + E)   | Graph storage                    |

Key rules:
- **Auxiliary space** = extra memory your algorithm uses (excluding input)
- **Recursion** — each call adds O(1) to the stack; `n` calls = O(n) space
- **Always report both** time and space in interviews

---

## Amortized Analysis

Some operations are *usually* cheap but *occasionally* expensive. Amortized analysis averages the cost over a sequence of operations.

| Data Structure       | Operation        | Worst Case | Amortized |
|---------------------|-----------------|------------|-----------|
| Dynamic array        | Push             | O(n)       | O(1)      |
| Hash map             | Insert / lookup  | O(n)       | O(1)      |
| Splay tree           | Search           | O(n)       | O(log n)  |

**Dynamic array resize:** Most pushes are O(1). When the array is full, it doubles in size — copying all `n` elements is O(n). But this happens only every ~n pushes, so the amortized cost per push is O(1).

---

## Recurrence Relations

For recursive algorithms, express the runtime as a recurrence and solve:

| Recurrence                  | Solution     | Example              |
|----------------------------|-------------|----------------------|
| T(n) = T(n−1) + O(1)       | O(n)         | Linear recursion     |
| T(n) = T(n−1) + O(n)       | O(n²)        | Selection sort       |
| T(n) = 2T(n/2) + O(n)      | O(n log n)   | Merge sort           |
| T(n) = 2T(n/2) + O(1)      | O(n)         | Binary tree traversal|
| T(n) = T(n/2) + O(1)       | O(log n)     | Binary search        |
| T(n) = 2T(n−1) + O(1)      | O(2ⁿ)        | Naive Fibonacci      |

### Master Theorem (quick reference)

For recurrences of the form **T(n) = aT(n/b) + O(nᵈ)**:

- If d < log_b(a) → **O(n^log_b(a))**
- If d = log_b(a) → **O(nᵈ log n)**
- If d > log_b(a) → **O(nᵈ)**

---

## Data Structure Operations Cheat Sheet

| Data Structure     | Access | Search | Insert | Delete | Notes               |
|-------------------|--------|--------|--------|--------|---------------------|
| Array              | O(1)   | O(n)   | O(n)   | O(n)   | O(1) insert at end  |
| Linked List        | O(n)   | O(n)   | O(1)   | O(1)   | With pointer to node |
| Hash Map           | —      | O(1)*  | O(1)*  | O(1)*  | *Amortized average   |
| BST (balanced)     | —      | O(log n)| O(log n)| O(log n)| AVL, Red-Black   |
| BST (worst)        | —      | O(n)   | O(n)   | O(n)   | Degenerate / skewed  |
| Heap               | —      | O(n)   | O(log n)| O(log n)| Access min/max O(1) |
| Stack / Queue      | —      | O(n)   | O(1)   | O(1)   |                     |
| Trie               | —      | O(k)   | O(k)   | O(k)   | k = key length       |

## Sorting Algorithms Cheat Sheet

| Algorithm      | Best       | Average    | Worst      | Space   | Stable |
|---------------|-----------|-----------|-----------|---------|--------|
| Bubble Sort   | O(n)      | O(n²)     | O(n²)     | O(1)    | Yes    |
| Selection Sort| O(n²)     | O(n²)     | O(n²)     | O(1)    | No     |
| Insertion Sort| O(n)      | O(n²)     | O(n²)     | O(1)    | Yes    |
| Merge Sort    | O(n log n)| O(n log n)| O(n log n)| O(n)    | Yes    |
| Quick Sort    | O(n log n)| O(n log n)| O(n²)     | O(log n)| No     |
| Heap Sort     | O(n log n)| O(n log n)| O(n log n)| O(1)    | No     |
| Counting Sort | O(n+k)    | O(n+k)   | O(n+k)   | O(k)    | Yes    |
| Radix Sort    | O(nk)     | O(nk)    | O(nk)    | O(n+k)  | Yes    |

---

## Common Interview Questions

| Question | Answer |
|----------|--------|
| Time complexity of binary search? | O(log n) |
| Why is hash map lookup O(1)? | Hash function maps key directly to index |
| Space complexity of DFS? | O(h) where h = height of tree/graph |
| Why is merge sort O(n log n)? | O(log n) levels × O(n) merge work per level |
| Quick sort worst case — when? | O(n²) — pivot is always min or max |
| What does amortized O(1) mean? | Average over many operations, even if one is slow |
| Difference between O(n) and O(n²)? | Linear vs quadratic — at n=1000: 1K ops vs 1M ops |
| Can you do better than O(n log n) sort? | Only with constraints: counting sort O(n+k) |

---

## Recognizing Complexity by Pattern

| Pattern You See                | Likely Complexity |
|-------------------------------|-------------------|
| Single pass / two pointers     | O(n)              |
| Sorting then scanning          | O(n log n)        |
| Binary search / halving        | O(log n)          |
| Two nested loops               | O(n²)             |
| Generate all subsets           | O(2ⁿ)             |
| Generate all permutations      | O(n!)             |
| BFS/DFS on a graph             | O(V + E)          |
| DP with 1D table               | O(n)              |
| DP with 2D table               | O(n × m)          |
| Divide and conquer (balanced)  | O(n log n)        |

---

## LeetCode Practice

- [LeetCode 1 — Two Sum](https://leetcode.com/problems/two-sum/) — analyze O(n²) brute force vs O(n) hash map
- [LeetCode 704 — Binary Search](https://leetcode.com/problems/binary-search/) — classic O(log n)
- [LeetCode 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/) — implement O(n log n) sort
- [LeetCode 70 — Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) — O(2ⁿ) recursion vs O(n) DP
- [LeetCode 53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) — O(n) Kadane's
- [LeetCode 15 — 3Sum](https://leetcode.com/problems/3sum/) — O(n³) brute force vs O(n²) two pointers

# Advanced Tree Structures

## Overview

| Tree Type     | Key Property                       | Lookup  | Insert  | Use Case                       |
|--------------|------------------------------------|---------|---------|--------------------------------|
| AVL Tree      | Height-balanced (diff ≤ 1)         | O(log n)| O(log n)| Frequent lookups               |
| Red-Black Tree| Color-balanced (no two red in a row)| O(log n)| O(log n)| Java TreeMap, Linux kernel     |
| Segment Tree  | Range queries on arrays            | O(log n)| O(log n)| Range sum / min / max          |
| Fenwick Tree  | Prefix sums (simpler than segment) | O(log n)| O(log n)| Cumulative frequency           |
| B-Tree        | Multi-way, disk-optimized          | O(log n)| O(log n)| Database indexes (MySQL, Postgres) |

---

## AVL Tree

A self-balancing BST where the height difference between left and right subtrees is **at most 1**. After every insert/delete, perform **rotations** to rebalance.

### Rotations

| Imbalance       | Rotation         | When                                    |
|----------------|-----------------|------------------------------------------|
| Left-Left       | Right rotation   | Left child's left subtree is taller      |
| Right-Right     | Left rotation    | Right child's right subtree is taller    |
| Left-Right      | Left-Right rotation | Left child's right subtree is taller  |
| Right-Left      | Right-Left rotation | Right child's left subtree is taller  |

```javascript
class AVLNode {
  constructor(val) {
    this.val = val;
    this.left = this.right = null;
    this.height = 1;
  }
}

function getHeight(node) {
  return node ? node.height : 0;
}

function getBalance(node) {
  return node ? getHeight(node.left) - getHeight(node.right) : 0;
}

function rightRotate(y) {
  const x = y.left;
  const T2 = x.right;
  x.right = y;
  y.left = T2;
  y.height = 1 + Math.max(getHeight(y.left), getHeight(y.right));
  x.height = 1 + Math.max(getHeight(x.left), getHeight(x.right));
  return x;
}

function leftRotate(x) {
  const y = x.right;
  const T2 = y.left;
  y.left = x;
  x.right = T2;
  x.height = 1 + Math.max(getHeight(x.left), getHeight(x.right));
  y.height = 1 + Math.max(getHeight(y.left), getHeight(y.right));
  return y;
}

function insert(node, val) {
  if (!node) return new AVLNode(val);

  if (val < node.val) node.left = insert(node.left, val);
  else if (val > node.val) node.right = insert(node.right, val);
  else return node; // no duplicates

  node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));
  const balance = getBalance(node);

  if (balance > 1 && val < node.left.val) return rightRotate(node);         // LL
  if (balance < -1 && val > node.right.val) return leftRotate(node);        // RR
  if (balance > 1 && val > node.left.val) {                                 // LR
    node.left = leftRotate(node.left);
    return rightRotate(node);
  }
  if (balance < -1 && val < node.right.val) {                               // RL
    node.right = rightRotate(node.right);
    return leftRotate(node);
  }

  return node;
}
```

```python
class AVLNode:
    def __init__(self, val):
        self.val = val
        self.left = self.right = None
        self.height = 1

def get_height(node):
    return node.height if node else 0

def get_balance(node):
    return get_height(node.left) - get_height(node.right) if node else 0

def right_rotate(y):
    x = y.left
    y.left = x.right
    x.right = y
    y.height = 1 + max(get_height(y.left), get_height(y.right))
    x.height = 1 + max(get_height(x.left), get_height(x.right))
    return x

def left_rotate(x):
    y = x.right
    x.right = y.left
    y.left = x
    x.height = 1 + max(get_height(x.left), get_height(x.right))
    y.height = 1 + max(get_height(y.left), get_height(y.right))
    return y

def insert(node, val):
    if not node:
        return AVLNode(val)

    if val < node.val:
        node.left = insert(node.left, val)
    elif val > node.val:
        node.right = insert(node.right, val)
    else:
        return node

    node.height = 1 + max(get_height(node.left), get_height(node.right))
    balance = get_balance(node)

    if balance > 1 and val < node.left.val:      # LL
        return right_rotate(node)
    if balance < -1 and val > node.right.val:     # RR
        return left_rotate(node)
    if balance > 1 and val > node.left.val:       # LR
        node.left = left_rotate(node.left)
        return right_rotate(node)
    if balance < -1 and val < node.right.val:     # RL
        node.right = right_rotate(node.right)
        return left_rotate(node)

    return node
```

```csharp
public class AVLNode
{
    public int Val;
    public AVLNode Left, Right;
    public int Height = 1;
    public AVLNode(int val) => Val = val;
}

public int GetHeight(AVLNode node) => node?.Height ?? 0;
public int GetBalance(AVLNode node) =>
    node == null ? 0 : GetHeight(node.Left) - GetHeight(node.Right);

public AVLNode RightRotate(AVLNode y)
{
    var x = y.Left;
    y.Left = x.Right;
    x.Right = y;
    y.Height = 1 + Math.Max(GetHeight(y.Left), GetHeight(y.Right));
    x.Height = 1 + Math.Max(GetHeight(x.Left), GetHeight(x.Right));
    return x;
}

public AVLNode LeftRotate(AVLNode x)
{
    var y = x.Right;
    x.Right = y.Left;
    y.Left = x;
    x.Height = 1 + Math.Max(GetHeight(x.Left), GetHeight(x.Right));
    y.Height = 1 + Math.Max(GetHeight(y.Left), GetHeight(y.Right));
    return y;
}

public AVLNode Insert(AVLNode node, int val)
{
    if (node == null) return new AVLNode(val);
    if (val < node.Val) node.Left = Insert(node.Left, val);
    else if (val > node.Val) node.Right = Insert(node.Right, val);
    else return node;

    node.Height = 1 + Math.Max(GetHeight(node.Left), GetHeight(node.Right));
    int balance = GetBalance(node);

    if (balance > 1 && val < node.Left.Val) return RightRotate(node);
    if (balance < -1 && val > node.Right.Val) return LeftRotate(node);
    if (balance > 1 && val > node.Left.Val)
    { node.Left = LeftRotate(node.Left); return RightRotate(node); }
    if (balance < -1 && val < node.Right.Val)
    { node.Right = RightRotate(node.Right); return LeftRotate(node); }

    return node;
}
```

---

## Segment Tree

Answers **range queries** (sum, min, max) in O(log n) and supports **point updates** in O(log n). The tree has 4n nodes for an array of size n.

```javascript
class SegmentTree {
  constructor(nums) {
    this.n = nums.length;
    this.tree = new Array(4 * this.n).fill(0);
    this.build(nums, 1, 0, this.n - 1);
  }

  build(nums, node, start, end) {
    if (start === end) { this.tree[node] = nums[start]; return; }
    const mid = (start + end) >> 1;
    this.build(nums, 2 * node, start, mid);
    this.build(nums, 2 * node + 1, mid + 1, end);
    this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1];
  }

  update(idx, val, node = 1, start = 0, end = this.n - 1) {
    if (start === end) { this.tree[node] = val; return; }
    const mid = (start + end) >> 1;
    if (idx <= mid) this.update(idx, val, 2 * node, start, mid);
    else this.update(idx, val, 2 * node + 1, mid + 1, end);
    this.tree[node] = this.tree[2 * node] + this.tree[2 * node + 1];
  }

  query(l, r, node = 1, start = 0, end = this.n - 1) {
    if (r < start || end < l) return 0;
    if (l <= start && end <= r) return this.tree[node];
    const mid = (start + end) >> 1;
    return this.query(l, r, 2 * node, start, mid)
         + this.query(l, r, 2 * node + 1, mid + 1, end);
  }
}
```

```python
class SegmentTree:
    def __init__(self, nums):
        self.n = len(nums)
        self.tree = [0] * (4 * self.n)
        self._build(nums, 1, 0, self.n - 1)

    def _build(self, nums, node, start, end):
        if start == end:
            self.tree[node] = nums[start]
            return
        mid = (start + end) // 2
        self._build(nums, 2 * node, start, mid)
        self._build(nums, 2 * node + 1, mid + 1, end)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def update(self, idx, val, node=1, start=0, end=None):
        if end is None: end = self.n - 1
        if start == end:
            self.tree[node] = val
            return
        mid = (start + end) // 2
        if idx <= mid: self.update(idx, val, 2 * node, start, mid)
        else: self.update(idx, val, 2 * node + 1, mid + 1, end)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def query(self, l, r, node=1, start=0, end=None):
        if end is None: end = self.n - 1
        if r < start or end < l: return 0
        if l <= start and end <= r: return self.tree[node]
        mid = (start + end) // 2
        return (self.query(l, r, 2 * node, start, mid)
              + self.query(l, r, 2 * node + 1, mid + 1, end))
```

```csharp
public class SegmentTree
{
    private int[] tree;
    private int n;

    public SegmentTree(int[] nums)
    {
        n = nums.Length;
        tree = new int[4 * n];
        Build(nums, 1, 0, n - 1);
    }

    private void Build(int[] nums, int node, int start, int end)
    {
        if (start == end) { tree[node] = nums[start]; return; }
        int mid = (start + end) / 2;
        Build(nums, 2 * node, start, mid);
        Build(nums, 2 * node + 1, mid + 1, end);
        tree[node] = tree[2 * node] + tree[2 * node + 1];
    }

    public void Update(int idx, int val, int node = 1, int start = 0, int end = -1)
    {
        if (end == -1) end = n - 1;
        if (start == end) { tree[node] = val; return; }
        int mid = (start + end) / 2;
        if (idx <= mid) Update(idx, val, 2 * node, start, mid);
        else Update(idx, val, 2 * node + 1, mid + 1, end);
        tree[node] = tree[2 * node] + tree[2 * node + 1];
    }

    public int Query(int l, int r, int node = 1, int start = 0, int end = -1)
    {
        if (end == -1) end = n - 1;
        if (r < start || end < l) return 0;
        if (l <= start && end <= r) return tree[node];
        int mid = (start + end) / 2;
        return Query(l, r, 2 * node, start, mid)
             + Query(l, r, 2 * node + 1, mid + 1, end);
    }
}
```

---

## Fenwick Tree (Binary Indexed Tree)

Simpler alternative to Segment Tree for **prefix sum queries** and **point updates**, both in O(log n).

```javascript
class FenwickTree {
  constructor(n) {
    this.tree = new Array(n + 1).fill(0);
  }

  update(i, delta) {
    for (i++; i < this.tree.length; i += i & (-i))
      this.tree[i] += delta;
  }

  query(i) {
    let sum = 0;
    for (i++; i > 0; i -= i & (-i))
      sum += this.tree[i];
    return sum;
  }

  rangeQuery(l, r) {
    return this.query(r) - (l > 0 ? this.query(l - 1) : 0);
  }
}
```

```python
class FenwickTree:
    def __init__(self, n):
        self.tree = [0] * (n + 1)

    def update(self, i, delta):
        i += 1
        while i < len(self.tree):
            self.tree[i] += delta
            i += i & (-i)

    def query(self, i):
        s = 0
        i += 1
        while i > 0:
            s += self.tree[i]
            i -= i & (-i)
        return s

    def range_query(self, l, r):
        return self.query(r) - (self.query(l - 1) if l > 0 else 0)
```

```csharp
public class FenwickTree
{
    private int[] tree;

    public FenwickTree(int n) => tree = new int[n + 1];

    public void Update(int i, int delta)
    {
        for (i++; i < tree.Length; i += i & (-i))
            tree[i] += delta;
    }

    public int Query(int i)
    {
        int sum = 0;
        for (i++; i > 0; i -= i & (-i))
            sum += tree[i];
        return sum;
    }

    public int RangeQuery(int l, int r) =>
        Query(r) - (l > 0 ? Query(l - 1) : 0);
}
```

---

## B-Trees (Conceptual)

Multi-way balanced search trees optimized for **disk-based storage**. Each node can hold many keys, minimizing disk I/O.

| Property                | Value                                        |
|------------------------|----------------------------------------------|
| Keys per node           | Between ⌈m/2⌉−1 and m−1 (for order m)       |
| Children per node       | Between ⌈m/2⌉ and m                          |
| All leaves at same depth| Yes — perfectly balanced                     |
| Used in                 | MySQL InnoDB, PostgreSQL indexes, file systems|

**Why databases use B-trees:** Each node read is one disk page. A B-tree with order 1000 and 1 billion keys needs only ~3 disk reads to find any key.

---

## Red-Black Trees (Conceptual)

Self-balancing BST with these invariants:
1. Every node is **red or black**
2. Root is always **black**
3. No two adjacent **red** nodes
4. Every path from root to null has the **same number of black nodes**

This guarantees O(log n) height. Used in Java `TreeMap`/`TreeSet`, C++ `std::map`, and the Linux CFS scheduler.

**AVL vs Red-Black:**

| Property      | AVL              | Red-Black         |
|--------------|------------------|-------------------|
| Balance       | Stricter (±1)    | Looser            |
| Lookup speed  | Slightly faster  | Slightly slower   |
| Insert/Delete | More rotations   | Fewer rotations   |
| Best for      | Read-heavy       | Write-heavy       |

---

## LeetCode Practice

- [LeetCode 307 — Range Sum Query Mutable](https://leetcode.com/problems/range-sum-query-mutable/) — Segment Tree / Fenwick Tree
- [LeetCode 315 — Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) — Fenwick Tree
- [LeetCode 327 — Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/) — Segment Tree
- [LeetCode 1649 — Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/) — Fenwick Tree

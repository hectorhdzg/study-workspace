# Trees & Binary Search Trees

## Tree Terminology

- **Root** — Top node.
- **Leaf** — Node with no children.
- **Height** — Length of longest path from root to leaf.
- **Depth** — Distance from root to a node.
- **BST Property** — Left subtree values < node value < right subtree values.

## Complexity (Balanced BST)

| Operation | Average  | Worst (unbalanced) |
|-----------|----------|--------------------|
| Search    | O(log n) | O(n)               |
| Insert    | O(log n) | O(n)               |
| Delete    | O(log n) | O(n)               |

---

## Tree Node

```javascript
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}
```

---

## Tree Traversals

Traversals define the order in which you visit every node. The naming refers to when you process the **root** relative to its subtrees:

- **In-order** (Left → Root → Right) — Yields sorted output for BSTs. Use for retrieving elements in order.
- **Pre-order** (Root → Left → Right) — Processes the root first. Use for serializing/cloning a tree.
- **Post-order** (Left → Right → Root) — Processes children before the parent. Use for deleting a tree or computing subtree sizes.
- **Level-order** (BFS) — Visits nodes level by level using a queue. Use for shortest-path in unweighted trees, or when you need to process one level at a time.

```javascript
// In-order: Left → Root → Right (sorted for BST)
function inOrder(root) {
  if (!root) return [];
  return [...inOrder(root.left), root.val, ...inOrder(root.right)];
}

// Pre-order: Root → Left → Right (serialize tree)
function preOrder(root) {
  if (!root) return [];
  return [root.val, ...preOrder(root.left), ...preOrder(root.right)];
}

// Post-order: Left → Right → Root (delete tree)
function postOrder(root) {
  if (!root) return [];
  return [...postOrder(root.left), ...postOrder(root.right), root.val];
}

// Level-order (BFS)
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length > 0) {
    const level = [];
    const size = queue.length;

    for (let i = 0; i < size; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }

    result.push(level);
  }

  return result;
}
```

```python
from collections import deque

def in_order(root):
    return in_order(root.left) + [root.val] + in_order(root.right) if root else []

def pre_order(root):
    return [root.val] + pre_order(root.left) + pre_order(root.right) if root else []

def level_order(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result
```

```csharp
public IList<IList<int>> LevelOrder(TreeNode root)
{
    var result = new List<IList<int>>();
    if (root == null) return result;
    var queue = new Queue<TreeNode>();
    queue.Enqueue(root);

    while (queue.Count > 0)
    {
        var level = new List<int>();
        int size = queue.Count;
        for (int i = 0; i < size; i++)
        {
            var node = queue.Dequeue();
            level.Add(node.val);
            if (node.left != null) queue.Enqueue(node.left);
            if (node.right != null) queue.Enqueue(node.right);
        }
        result.Add(level);
    }
    return result;
}
```

---

## Common Tree Problems

### Max Depth

The depth (or height) of a tree is the length of the longest path from root to leaf. This is a classic recursive problem: the depth of a node is 1 + the maximum depth of its children. The base case is a null node, which has depth 0.

```javascript
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

```python
def max_depth(root):
    if not root: return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

```csharp
public int MaxDepth(TreeNode root)
{
    if (root == null) return 0;
    return 1 + Math.Max(MaxDepth(root.left), MaxDepth(root.right));
}
```

### Check if Balanced

A tree is **height-balanced** if for every node, the heights of its left and right subtrees differ by at most 1. A naive approach checks balance at each node separately (O(n²)). The optimized approach computes height bottom-up and returns -1 early if an imbalance is found, giving O(n) time.

```javascript
function isBalanced(root) {
  function height(node) {
    if (!node) return 0;
    const left = height(node.left);
    const right = height(node.right);
    if (left === -1 || right === -1 || Math.abs(left - right) > 1) return -1;
    return 1 + Math.max(left, right);
  }
  return height(root) !== -1;
}
```

### Lowest Common Ancestor (BST)

In a BST, the LCA of two nodes is the first node you encounter (from the root) whose value falls **between** the two target values. If both targets are smaller, go left; if both are larger, go right; otherwise, you've found the LCA. The BST property makes this O(h) instead of O(n).

```javascript
function lcaBST(root, p, q) {
  if (p.val < root.val && q.val < root.val) return lcaBST(root.left, p, q);
  if (p.val > root.val && q.val > root.val) return lcaBST(root.right, p, q);
  return root;
}
```

### Lowest Common Ancestor (Binary Tree)

For a general binary tree (not a BST), recursively search both subtrees. If the current node matches either target, return it. If both left and right subtrees return non-null, the current node is the LCA. Otherwise, propagate whichever side found a match.

```javascript
function lca(root, p, q) {
  if (!root || root === p || root === q) return root;
  const left = lca(root.left, p, q);
  const right = lca(root.right, p, q);
  if (left && right) return root; // p and q on different sides
  return left || right;
}
```

### Validate BST

A BST is valid if every node's value falls within the range defined by its ancestors. Pass down `min` and `max` bounds as you recurse: left children must be less than the parent, right children must be greater. A common mistake is only checking immediate parent-child relationships — you must enforce the range from all ancestors.

```javascript
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

```python
def is_valid_bst(root, lo=float('-inf'), hi=float('inf')):
    if not root: return True
    if root.val <= lo or root.val >= hi: return False
    return is_valid_bst(root.left, lo, root.val) and \
           is_valid_bst(root.right, root.val, hi)
```

```csharp
public bool IsValidBST(TreeNode root, long min = long.MinValue, long max = long.MaxValue)
{
    if (root == null) return true;
    if (root.val <= min || root.val >= max) return false;
    return IsValidBST(root.left, min, root.val) &&
           IsValidBST(root.right, root.val, max);
}
```

### Path Sum

Determine if any root-to-leaf path sums to a target value. Subtract the current node's value at each step; when you reach a leaf, check if the remaining target is zero. This is a natural DFS problem — explore one path, backtrack, try another.

```javascript
function hasPathSum(root, targetSum) {
  if (!root) return false;
  if (!root.left && !root.right) return root.val === targetSum;
  return hasPathSum(root.left, targetSum - root.val) ||
         hasPathSum(root.right, targetSum - root.val);
}
```

---

## Practice Problems

- [LeetCode 104 — Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
- [LeetCode 226 — Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/)
- [LeetCode 100 — Same Tree](https://leetcode.com/problems/same-tree/)
- [LeetCode 98 — Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
- [LeetCode 102 — Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
- [LeetCode 235 — Lowest Common Ancestor of BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)
- [LeetCode 124 — Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)
- [LeetCode 105 — Construct Binary Tree from Preorder and Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

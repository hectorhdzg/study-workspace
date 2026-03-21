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

---

## Common Tree Problems

### Max Depth

```javascript
function maxDepth(root) {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### Check if Balanced

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

```javascript
function lcaBST(root, p, q) {
  if (p.val < root.val && q.val < root.val) return lcaBST(root.left, p, q);
  if (p.val > root.val && q.val > root.val) return lcaBST(root.right, p, q);
  return root;
}
```

### Lowest Common Ancestor (Binary Tree)

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

```javascript
function isValidBST(root, min = -Infinity, max = Infinity) {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

### Path Sum

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

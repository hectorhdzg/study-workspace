# Recursion & Backtracking

## Recursion Basics

Every recursive function needs:
1. **Base case** — When to stop.
2. **Recursive case** — Break the problem into smaller subproblems.
3. **Combine** — Use the result of recursive calls to build the answer.

```javascript
// Classic: Factorial
function factorial(n) {
  if (n <= 1) return 1;          // base case
  return n * factorial(n - 1);  // recursive case
}
```

---

## Backtracking Template

Backtracking explores all possibilities and **undoes** choices that don't lead to a valid solution.

```javascript
function backtrack(state, choices) {
  if (isSolution(state)) {
    recordSolution(state);
    return;
  }

  for (const choice of choices) {
    if (isValid(state, choice)) {
      makeChoice(state, choice);       // choose
      backtrack(state, nextChoices);   // explore
      undoChoice(state, choice);       // unchoose (backtrack)
    }
  }
}
```

---

## Subsets (Power Set)

```javascript
function subsets(nums) {
  const result = [];

  function backtrack(start, current) {
    result.push([...current]);

    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop(); // backtrack
    }
  }

  backtrack(0, []);
  return result;
}

console.log(subsets([1, 2, 3]));
// [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

---

## Permutations

```javascript
function permutations(nums) {
  const result = [];

  function backtrack(current, used) {
    if (current.length === nums.length) {
      result.push([...current]);
      return;
    }

    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      used[i] = true;
      current.push(nums[i]);
      backtrack(current, used);
      current.pop();
      used[i] = false;
    }
  }

  backtrack([], new Array(nums.length).fill(false));
  return result;
}

console.log(permutations([1, 2, 3]));
// [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

---

## Combinations

```javascript
function combinations(n, k) {
  const result = [];

  function backtrack(start, current) {
    if (current.length === k) {
      result.push([...current]);
      return;
    }

    for (let i = start; i <= n; i++) {
      current.push(i);
      backtrack(i + 1, current);
      current.pop();
    }
  }

  backtrack(1, []);
  return result;
}

console.log(combinations(4, 2));
// [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```

---

## N-Queens

```javascript
function solveNQueens(n) {
  const result = [];
  const board = Array.from({ length: n }, () => new Array(n).fill('.'));

  function isValid(row, col) {
    // Check column
    for (let r = 0; r < row; r++) {
      if (board[r][col] === 'Q') return false;
    }
    // Check upper-left diagonal
    for (let r = row - 1, c = col - 1; r >= 0 && c >= 0; r--, c--) {
      if (board[r][c] === 'Q') return false;
    }
    // Check upper-right diagonal
    for (let r = row - 1, c = col + 1; r >= 0 && c < n; r--, c++) {
      if (board[r][c] === 'Q') return false;
    }
    return true;
  }

  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }
    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = 'Q';
        backtrack(row + 1);
        board[row][col] = '.';
      }
    }
  }

  backtrack(0);
  return result;
}
```

---

## Practice Problems

- [LeetCode 78 — Subsets](https://leetcode.com/problems/subsets/)
- [LeetCode 46 — Permutations](https://leetcode.com/problems/permutations/)
- [LeetCode 77 — Combinations](https://leetcode.com/problems/combinations/)
- [LeetCode 39 — Combination Sum](https://leetcode.com/problems/combination-sum/)
- [LeetCode 51 — N-Queens](https://leetcode.com/problems/n-queens/)
- [LeetCode 79 — Word Search](https://leetcode.com/problems/word-search/)
- [LeetCode 131 — Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)

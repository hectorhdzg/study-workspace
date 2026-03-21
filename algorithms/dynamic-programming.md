# Dynamic Programming

## What is Dynamic Programming?

Dynamic Programming (DP) solves problems by breaking them into overlapping subproblems and storing results to avoid redundant computation. There are two main approaches:

- **Top-Down (Memoization):** Recursive + cache results.
- **Bottom-Up (Tabulation):** Iterative, fill table from base cases.

## When to Use DP

- Problem asks for **optimal value** (max, min, longest, shortest, count).
- Problem has **overlapping subproblems** (same subproblem solved multiple times).
- Problem has **optimal substructure** (optimal solution built from optimal subproblems).

---

## Classic Patterns

### 1. Fibonacci (Basic Memoization)

```javascript
// Top-Down
function fib(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n]) return memo[n];
  memo[n] = fib(n - 1, memo) + fib(n - 2, memo);
  return memo[n];
}

// Bottom-Up
function fibDP(n) {
  if (n <= 1) return n;
  const dp = [0, 1];
  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
}

console.log(fib(10));   // 55
console.log(fibDP(10)); // 55
```

---

### 2. 0/1 Knapsack

**Problem:** Given items with weights and values, maximize value within a weight limit.

```javascript
function knapsack(weights, values, capacity) {
  const n = weights.length;
  // dp[i][w] = max value using first i items with capacity w
  const dp = Array.from({ length: n + 1 }, () => new Array(capacity + 1).fill(0));

  for (let i = 1; i <= n; i++) {
    for (let w = 0; w <= capacity; w++) {
      // Don't take item i
      dp[i][w] = dp[i - 1][w];
      // Take item i (if it fits)
      if (weights[i - 1] <= w) {
        dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
      }
    }
  }

  return dp[n][capacity];
}

// Example
console.log(knapsack([1, 3, 4, 5], [1, 4, 5, 7], 7)); // 9
```

---

### 3. Longest Common Subsequence (LCS)

```javascript
function lcs(s1, s2) {
  const m = s1.length, n = s2.length;
  const dp = Array.from({ length: m + 1 }, () => new Array(n + 1).fill(0));

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (s1[i - 1] === s2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }

  return dp[m][n];
}

console.log(lcs("abcde", "ace")); // 3 ("ace")
```

---

### 4. Coin Change (Min Coins)

```javascript
function coinChange(coins, amount) {
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0;

  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }

  return dp[amount] === Infinity ? -1 : dp[amount];
}

console.log(coinChange([1, 5, 6, 9], 11)); // 2 (5+6)
```

---

### 5. Longest Increasing Subsequence (LIS)

```javascript
// O(n²) DP
function lis(nums) {
  const dp = new Array(nums.length).fill(1);

  for (let i = 1; i < nums.length; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        dp[i] = Math.max(dp[i], dp[j] + 1);
      }
    }
  }

  return Math.max(...dp);
}

console.log(lis([10, 9, 2, 5, 3, 7, 101, 18])); // 4 ([2,3,7,101])
```

---

## DP Problem-Solving Template

1. **Define subproblem** — What does `dp[i]` (or `dp[i][j]`) represent?
2. **Write recurrence relation** — How does `dp[i]` depend on smaller subproblems?
3. **Identify base cases** — What are the simplest inputs?
4. **Determine order** — Bottom-up: which order to fill the table?
5. **Extract answer** — Where in the table is the final answer?

---

## Practice Problems

- [LeetCode 70 — Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)
- [LeetCode 322 — Coin Change](https://leetcode.com/problems/coin-change/)
- [LeetCode 300 — Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/)
- [LeetCode 1143 — Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/)
- [LeetCode 72 — Edit Distance](https://leetcode.com/problems/edit-distance/)
- [LeetCode 518 — Coin Change II](https://leetcode.com/problems/coin-change-ii/)
- [LeetCode 198 — House Robber](https://leetcode.com/problems/house-robber/)
- [LeetCode 312 — Burst Balloons](https://leetcode.com/problems/burst-balloons/)

# Problem-Solving Framework

## The UMPIRE Method

A structured approach to tackling coding problems in interviews:

| Step | Description |
|------|-------------|
| **U** — Understand | Restate the problem, clarify ambiguities |
| **M** — Match | Identify the problem pattern |
| **P** — Plan | Outline your approach before coding |
| **I** — Implement | Write clean, working code |
| **R** — Review | Test with examples and edge cases |
| **E** — Evaluate | State time and space complexity |

---

## Step 1: Understand

Before writing a single line of code:

- Repeat the problem in your own words.
- Ask clarifying questions:
  - What are the input/output types?
  - What are the constraints? (n ≤ 10^5? Negative numbers? Duplicates?)
  - What edge cases should I handle? (empty input, single element, all same values)
  - Are there any space or time constraints?

```
Example questions to ask:
- "Can the array be empty?"
- "Are the values always integers, or could they be floats?"
- "Is the input always sorted?"
- "Can I modify the input in place?"
```

---

## Step 2: Match

Identify what **pattern** or **data structure** applies:

| Pattern               | Trigger Words                                              |
|----------------------|-----------------------------------------------------------|
| Two Pointers          | "sorted array", "pairs", "sum to target"                  |
| Sliding Window        | "subarray", "substring", "contiguous", "window"           |
| Binary Search         | "sorted", "find target", "minimize/maximize in sorted space"|
| BFS                   | "shortest path", "level by level", "minimum steps"        |
| DFS / Backtracking    | "all combinations", "all paths", "permutations"           |
| Dynamic Programming   | "optimal", "count ways", "max/min value", "overlapping"   |
| Hash Map              | "fast lookup", "frequency", "two sum", "seen before"      |
| Heap                  | "K largest/smallest", "median", "merge sorted"            |
| Monotonic Stack       | "next greater/smaller", "temperature", "histogram"        |
| Union-Find            | "connected components", "detect cycle", "group together"  |

---

## Step 3: Plan

Talk through your approach before coding:

1. State the brute-force solution and its complexity.
2. Explain the optimization and why it works.
3. Outline the algorithm step-by-step.

```
Example:
"A brute force approach would check all pairs in O(n²).
Instead, I can use a hash map to store each number's index.
For each element, I check if the complement exists in the map.
This gives O(n) time and O(n) space."
```

---

## Step 4: Implement

Write clean code:

- Use meaningful variable names (`left`, `right`, not `a`, `b`).
- Code one logical block at a time.
- Narrate what you're doing as you type.
- Don't silently delete and rewrite — announce changes.

---

## Step 5: Review

Test your code with:

1. **Given example** — Verify it produces the expected output.
2. **Edge cases:**
   - Empty input
   - Single element
   - All same elements
   - Negative numbers
   - Very large input
3. **Walk through line by line** with a small example.

---

## Step 6: Evaluate

Always state:

- **Time complexity:** O(?)
- **Space complexity:** O(?)
- Possible optimizations if time permits.

---

## If You're Stuck

1. Start with brute force — it shows you understand the problem.
2. Work through a small example by hand.
3. Think about what information you're throwing away on each step.
4. Ask yourself: "What data structure would make this fast?"
5. Tell the interviewer what you're thinking — they may give a hint.

# Stacks & Queues

## Stack

**LIFO** — Last In, First Out

**Use for:** Matching brackets, undo/redo, DFS, monotonic stack problems.

```javascript
class Stack {
  constructor() {
    this.items = [];
  }

  push(val) { this.items.push(val); }
  pop() { return this.items.pop(); }
  peek() { return this.items[this.items.length - 1]; }
  isEmpty() { return this.items.length === 0; }
  size() { return this.items.length; }
}
```

### Valid Parentheses

```javascript
function isValid(s) {
  const stack = [];
  const map = { ')': '(', ']': '[', '}': '{' };

  for (const c of s) {
    if ('([{'.includes(c)) {
      stack.push(c);
    } else {
      if (stack.pop() !== map[c]) return false;
    }
  }

  return stack.length === 0;
}

console.log(isValid("()[]{}")); // true
console.log(isValid("(]"));    // false
```

### Monotonic Stack (Next Greater Element)

```javascript
function nextGreaterElement(nums) {
  const result = new Array(nums.length).fill(-1);
  const stack = []; // stores indices

  for (let i = 0; i < nums.length; i++) {
    while (stack.length > 0 && nums[i] > nums[stack[stack.length - 1]]) {
      const idx = stack.pop();
      result[idx] = nums[i];
    }
    stack.push(i);
  }

  return result;
}

console.log(nextGreaterElement([2, 1, 2, 4, 3])); // [4, 2, 4, -1, -1]
```

### Min Stack

```javascript
class MinStack {
  constructor() {
    this.stack = [];
    this.minStack = [];
  }

  push(val) {
    this.stack.push(val);
    const min = this.minStack.length ? Math.min(val, this.getMin()) : val;
    this.minStack.push(min);
  }

  pop() {
    this.stack.pop();
    this.minStack.pop();
  }

  top() { return this.stack[this.stack.length - 1]; }
  getMin() { return this.minStack[this.minStack.length - 1]; }
}
```

---

## Queue

**FIFO** — First In, First Out

**Use for:** BFS, task scheduling, sliding window.

```javascript
class Queue {
  constructor() {
    this.items = [];
    this.front = 0;
  }

  enqueue(val) { this.items.push(val); }
  dequeue() {
    if (this.isEmpty()) return undefined;
    return this.items[this.front++];
  }
  peek() { return this.items[this.front]; }
  isEmpty() { return this.front >= this.items.length; }
  size() { return this.items.length - this.front; }
}
```

### Deque (Double-Ended Queue) — Sliding Window Maximum

```javascript
function maxSlidingWindow(nums, k) {
  const deque = []; // stores indices, front = max
  const result = [];

  for (let i = 0; i < nums.length; i++) {
    // Remove indices outside window
    while (deque.length && deque[0] < i - k + 1) deque.shift();

    // Remove smaller elements from back
    while (deque.length && nums[deque[deque.length - 1]] < nums[i]) deque.pop();

    deque.push(i);

    if (i >= k - 1) result.push(nums[deque[0]]);
  }

  return result;
}

console.log(maxSlidingWindow([1, 3, -1, -3, 5, 3, 6, 7], 3)); // [3,3,5,5,6,7]
```

---

## Practice Problems

- [LeetCode 20 — Valid Parentheses](https://leetcode.com/problems/valid-parentheses/)
- [LeetCode 155 — Min Stack](https://leetcode.com/problems/min-stack/)
- [LeetCode 739 — Daily Temperatures](https://leetcode.com/problems/daily-temperatures/)
- [LeetCode 84 — Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)
- [LeetCode 239 — Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)
- [LeetCode 232 — Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/)

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

A stack naturally solves bracket matching: push opening brackets, and when you see a closing bracket, pop and check that it matches. If the stack is empty at the end, every bracket was matched. This works because brackets must close in reverse order of opening — exactly LIFO behavior.

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

```python
def is_valid(s: str) -> bool:
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}
    for c in s:
        if c in '([{':
            stack.append(c)
        elif not stack or stack.pop() != pairs[c]:
            return False
    return len(stack) == 0

print(is_valid("()[]{}"))  # True
print(is_valid("(]"))      # False
```

```csharp
public bool IsValid(string s)
{
    var stack = new Stack<char>();
    var pairs = new Dictionary<char, char>
        { {')', '('}, {']', '['}, {'}', '{'} };

    foreach (char c in s)
    {
        if ("([{".Contains(c))
            stack.Push(c);
        else if (stack.Count == 0 || stack.Pop() != pairs[c])
            return false;
    }
    return stack.Count == 0;
}
```

### Monotonic Stack (Next Greater Element)

A **monotonic stack** maintains elements in sorted order (either increasing or decreasing). For the "next greater element" problem, maintain a stack of indices whose values haven't yet found a greater element. When a new value is larger than the stack's top, pop and record the answer — repeat until the stack's top is larger or empty.

This converts an O(n²) brute-force search into O(n) because each element is pushed and popped at most once.

**When to use:** Next greater/smaller element, daily temperatures, largest rectangle in histogram, stock span problems.

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

A **Min Stack** supports `push`, `pop`, `top`, and `getMin` — all in O(1). The trick: maintain a parallel stack that tracks the current minimum at each level. Every time you push, also push the new minimum (either the pushed value or the current min, whichever is smaller). When you pop, pop from both stacks.

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

A **deque** (double-ended queue) supports O(1) insertion and removal at both ends. For the sliding window maximum problem, maintain a deque of indices where values are **decreasing** from front to back. The front always holds the index of the current window’s maximum. When the window slides, remove indices that fall outside and remove from the back any values smaller than the new element.

**Time:** O(n) — each element enters and leaves the deque at most once.

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

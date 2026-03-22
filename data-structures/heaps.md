# Heaps / Priority Queues

## What is a Heap?

A **heap** is a complete binary tree satisfying the heap property:
- **Min-Heap** — Parent ≤ Children (root is the minimum).
- **Max-Heap** — Parent ≥ Children (root is the maximum).

Heaps are stored as arrays: for node at index `i`:
- Left child: `2i + 1`
- Right child: `2i + 2`
- Parent: `Math.floor((i - 1) / 2)`

## Complexity

| Operation    | Time     |
|-------------|----------|
| Insert      | O(log n) |
| Get min/max | O(1)     |
| Remove min/max | O(log n) |
| Build heap  | O(n)     |

---

## Min-Heap Implementation

```javascript
class MinHeap {
  constructor() {
    this.heap = [];
  }

  size() { return this.heap.length; }
  peek() { return this.heap[0]; }

  push(val) {
    this.heap.push(val);
    this._bubbleUp(this.heap.length - 1);
  }

  pop() {
    const min = this.heap[0];
    const last = this.heap.pop();
    if (this.heap.length > 0) {
      this.heap[0] = last;
      this._siftDown(0);
    }
    return min;
  }

  _bubbleUp(i) {
    while (i > 0) {
      const parent = Math.floor((i - 1) / 2);
      if (this.heap[parent] <= this.heap[i]) break;
      [this.heap[parent], this.heap[i]] = [this.heap[i], this.heap[parent]];
      i = parent;
    }
  }

  _siftDown(i) {
    const n = this.heap.length;
    while (true) {
      let smallest = i;
      const left = 2 * i + 1, right = 2 * i + 2;
      if (left < n && this.heap[left] < this.heap[smallest]) smallest = left;
      if (right < n && this.heap[right] < this.heap[smallest]) smallest = right;
      if (smallest === i) break;
      [this.heap[smallest], this.heap[i]] = [this.heap[i], this.heap[smallest]];
      i = smallest;
    }
  }
}
```

---

## Common Use Cases

### K Largest Elements

```javascript
function kLargest(nums, k) {
  const heap = new MinHeap();
  for (const num of nums) {
    heap.push(num);
    if (heap.size() > k) heap.pop();
  }
  return heap.heap;
}

console.log(kLargest([3, 1, 5, 12, 2, 11], 3)); // [5, 12, 11] (order may vary)
```

```python
import heapq

def k_largest(nums, k):
    # heapq is a min-heap by default — nlargest handles it
    return heapq.nlargest(k, nums)

# Manual approach with a min-heap of size k
def k_largest_manual(nums, k):
    heap = []
    for num in nums:
        heapq.heappush(heap, num)
        if len(heap) > k:
            heapq.heappop(heap)
    return heap

print(k_largest([3, 1, 5, 12, 2, 11], 3))  # [12, 11, 5]
```

```csharp
public int[] KLargest(int[] nums, int k)
{
    // .NET 6+ has PriorityQueue (min-heap)
    var heap = new PriorityQueue<int, int>();
    foreach (int num in nums)
    {
        heap.Enqueue(num, num);
        if (heap.Count > k) heap.Dequeue();
    }
    var result = new int[k];
    for (int i = 0; i < k; i++) result[i] = heap.Dequeue();
    return result;
}
```

### Merge K Sorted Lists

```javascript
// Uses a min-heap to always pick the smallest current element
function mergeKLists(lists) {
  const heap = new MinHeap(); // stores [val, listIndex, nodeRef]
  // (in practice, use a heap that compares by val)
  // ...
}
```

### Find Median from Data Stream

```javascript
class MedianFinder {
  constructor() {
    this.small = new MaxHeap(); // lower half
    this.large = new MinHeap(); // upper half
  }

  addNum(num) {
    this.small.push(num);
    this.large.push(this.small.pop());
    if (this.small.size() < this.large.size()) {
      this.small.push(this.large.pop());
    }
  }

  findMedian() {
    if (this.small.size() > this.large.size()) return this.small.peek();
    return (this.small.peek() + this.large.peek()) / 2;
  }
}
```

---

## Practice Problems

- [LeetCode 215 — Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/)
- [LeetCode 347 — Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
- [LeetCode 23 — Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)
- [LeetCode 295 — Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)
- [LeetCode 373 — Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/)

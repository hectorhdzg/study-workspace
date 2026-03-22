# Searching Algorithms

## Summary

| Algorithm       | Time      | Space | Notes                           |
|----------------|-----------|-------|---------------------------------|
| Linear Search  | O(n)      | O(1)  | Works on any collection         |
| Binary Search  | O(log n)  | O(1)  | Requires sorted array           |
| BFS            | O(V + E)  | O(V)  | Unweighted shortest path        |
| DFS            | O(V + E)  | O(V)  | Topological sort, cycle detect  |

---

## Binary Search

**Idea:** Compare the target with the middle element; eliminate half the search space each step.

**Precondition:** Array must be sorted.

**Time:** O(log n) | **Space:** O(1)

```javascript
function binarySearch(arr, target) {
  let left = 0, right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }

  return -1; // not found
}

// Example
const arr = [1, 3, 5, 7, 9, 11];
console.log(binarySearch(arr, 7));  // 3
console.log(binarySearch(arr, 4));  // -1
```

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target: return mid
        elif arr[mid] < target: left = mid + 1
        else: right = mid - 1
    return -1

# Or use the built-in bisect module
import bisect
def binary_search_bisect(arr, target):
    i = bisect.bisect_left(arr, target)
    return i if i < len(arr) and arr[i] == target else -1

print(binary_search([1, 3, 5, 7, 9, 11], 7))  # 3
```

```csharp
public int BinarySearch(int[] arr, int target)
{
    int left = 0, right = arr.Length - 1;
    while (left <= right)
    {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;

    // Or use built-in: Array.BinarySearch(arr, target);
}
```

### Binary Search Variants

**Find first occurrence:**

```javascript
function firstOccurrence(arr, target) {
  let left = 0, right = arr.length - 1, result = -1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) {
      result = mid;
      right = mid - 1; // keep searching left
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }

  return result;
}
```

**Search in rotated sorted array:**

```javascript
function searchRotated(arr, target) {
  let left = 0, right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;

    // Left half is sorted
    if (arr[left] <= arr[mid]) {
      if (arr[left] <= target && target < arr[mid]) right = mid - 1;
      else left = mid + 1;
    } else {
      // Right half is sorted
      if (arr[mid] < target && target <= arr[right]) left = mid + 1;
      else right = mid - 1;
    }
  }

  return -1;
}

// Example
console.log(searchRotated([4, 5, 6, 7, 0, 1, 2], 0)); // 4
```

---

## Linear Search

The simplest search: check every element one by one until you find the target or exhaust the collection. No preconditions — works on unsorted, unindexed data. Use when the data is small, unsorted, or you only need to search once.

**Time:** O(n) | **Space:** O(1)

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}
```

---

## Practice Problems

- [LeetCode 704 — Binary Search](https://leetcode.com/problems/binary-search/)
- [LeetCode 33 — Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
- [LeetCode 153 — Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
- [LeetCode 34 — Find First and Last Position of Element](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)
- [LeetCode 162 — Find Peak Element](https://leetcode.com/problems/find-peak-element/)

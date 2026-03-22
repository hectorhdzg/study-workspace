# Sorting Algorithms

## Comparison Sort Summary

| Algorithm      | Best     | Average  | Worst    | Space  | Stable |
|---------------|----------|----------|----------|--------|--------|
| Bubble Sort   | O(n)     | O(n²)    | O(n²)    | O(1)   | Yes    |
| Selection Sort| O(n²)    | O(n²)    | O(n²)    | O(1)   | No     |
| Insertion Sort| O(n)     | O(n²)    | O(n²)    | O(1)   | Yes    |
| Merge Sort    | O(n log n)| O(n log n)| O(n log n)| O(n) | Yes    |
| Quick Sort    | O(n log n)| O(n log n)| O(n²)   | O(log n)| No   |
| Heap Sort     | O(n log n)| O(n log n)| O(n log n)| O(1) | No    |
| Counting Sort | O(n+k)   | O(n+k)   | O(n+k)   | O(k)   | Yes    |
| Radix Sort    | O(nk)    | O(nk)    | O(nk)    | O(n+k) | Yes    |

---

## Merge Sort

**Idea:** Divide the array in half, recursively sort each half, then merge.

**Time:** O(n log n) | **Space:** O(n)

```javascript
function mergeSort(arr) {
  if (arr.length <= 1) return arr;

  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }

  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Example
console.log(mergeSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    return merge(left, right)

def merge(left, right):
    result, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i += 1
        else:
            result.append(right[j]); j += 1
    return result + left[i:] + right[j:]

print(merge_sort([5, 3, 8, 1, 2]))  # [1, 2, 3, 5, 8]
```

```csharp
public int[] MergeSort(int[] arr)
{
    if (arr.Length <= 1) return arr;
    int mid = arr.Length / 2;
    var left = MergeSort(arr[..mid]);
    var right = MergeSort(arr[mid..]);
    return Merge(left, right);
}

private int[] Merge(int[] left, int[] right)
{
    var result = new int[left.Length + right.Length];
    int i = 0, j = 0, k = 0;
    while (i < left.Length && j < right.Length)
        result[k++] = left[i] <= right[j] ? left[i++] : right[j++];
    while (i < left.Length) result[k++] = left[i++];
    while (j < right.Length) result[k++] = right[j++];
    return result;
}
```

---

## Quick Sort

**Idea:** Pick a pivot, partition elements smaller/larger, then recursively sort each partition.

**Time:** O(n log n) average, O(n²) worst | **Space:** O(log n)

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
  if (low < high) {
    const pivotIndex = partition(arr, low, high);
    quickSort(arr, low, pivotIndex - 1);
    quickSort(arr, pivotIndex + 1, high);
  }
  return arr;
}

function partition(arr, low, high) {
  const pivot = arr[high];
  let i = low - 1;

  for (let j = low; j < high; j++) {
    if (arr[j] <= pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }

  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1;
}

// Example
console.log(quickSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[-1]
    left = [x for x in arr[:-1] if x <= pivot]
    right = [x for x in arr[:-1] if x > pivot]
    return quick_sort(left) + [pivot] + quick_sort(right)

print(quick_sort([5, 3, 8, 1, 2]))  # [1, 2, 3, 5, 8]
```

```csharp
public void QuickSort(int[] arr, int low, int high)
{
    if (low < high)
    {
        int pivot = Partition(arr, low, high);
        QuickSort(arr, low, pivot - 1);
        QuickSort(arr, pivot + 1, high);
    }
}

private int Partition(int[] arr, int low, int high)
{
    int pivot = arr[high], i = low - 1;
    for (int j = low; j < high; j++)
    {
        if (arr[j] <= pivot)
        {
            i++;
            (arr[i], arr[j]) = (arr[j], arr[i]);
        }
    }
    (arr[i + 1], arr[high]) = (arr[high], arr[i + 1]);
    return i + 1;
}
```

---

## Insertion Sort

**Idea:** Build a sorted portion one element at a time by inserting each element in the correct position.

**Best for:** Small arrays or nearly sorted data.

**Time:** O(n²) | **Space:** O(1)

```javascript
function insertionSort(arr) {
  for (let i = 1; i < arr.length; i++) {
    const key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}

// Example
console.log(insertionSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

---

## Heap Sort

**Idea:** Build a max-heap, then repeatedly extract the maximum to get a sorted array.

**Time:** O(n log n) | **Space:** O(1)

```javascript
function heapSort(arr) {
  const n = arr.length;

  // Build max-heap
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapify(arr, n, i);
  }

  // Extract elements from heap one by one
  for (let i = n - 1; i > 0; i--) {
    [arr[0], arr[i]] = [arr[i], arr[0]];
    heapify(arr, i, 0);
  }

  return arr;
}

function heapify(arr, n, i) {
  let largest = i;
  const left = 2 * i + 1;
  const right = 2 * i + 2;

  if (left < n && arr[left] > arr[largest]) largest = left;
  if (right < n && arr[right] > arr[largest]) largest = right;

  if (largest !== i) {
    [arr[i], arr[largest]] = [arr[largest], arr[i]];
    heapify(arr, n, largest);
  }
}

// Example
console.log(heapSort([5, 3, 8, 1, 2])); // [1, 2, 3, 5, 8]
```

---

## Practice Problems

- [LeetCode 912 — Sort an Array](https://leetcode.com/problems/sort-an-array/)
- [LeetCode 147 — Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/)
- [LeetCode 315 — Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) *(uses merge sort)*
- [LeetCode 75 — Sort Colors](https://leetcode.com/problems/sort-colors/) *(Dutch National Flag / 3-way partition)*

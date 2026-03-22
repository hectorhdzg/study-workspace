# Arrays & Strings

## Key Operations & Complexity

| Operation          | Array  | Notes                     |
|-------------------|--------|---------------------------|
| Access by index   | O(1)   |                           |
| Search (unsorted) | O(n)   |                           |
| Search (sorted)   | O(log n)| Binary search             |
| Insert at end     | O(1)   | Amortized for dynamic array|
| Insert at middle  | O(n)   | Shift elements            |
| Delete at end     | O(1)   |                           |
| Delete at middle  | O(n)   | Shift elements            |

---

## Common Patterns

### Prefix Sum

A **prefix sum** (or cumulative sum) array stores the running total at each index. Once built in O(n), it answers any range-sum query in O(1) by subtracting two prefix values: `sum(l..r) = prefix[r+1] − prefix[l]`. This avoids recomputing the sum from scratch every time, which is a huge win when you have many queries over the same array.

**When to use:** Subarray sum problems, range queries, counting subarrays that sum to a target (pair with a hash map).

```javascript
// Build prefix sum array for O(1) range sum queries
function buildPrefixSum(arr) {
  const prefix = [0];
  for (const num of arr) {
    prefix.push(prefix[prefix.length - 1] + num);
  }
  return prefix;
}

function rangeSum(prefix, left, right) {
  return prefix[right + 1] - prefix[left];
}

const arr = [1, 2, 3, 4, 5];
const prefix = buildPrefixSum(arr);
console.log(rangeSum(prefix, 1, 3)); // 9 (2+3+4)
```

### Kadane's Algorithm (Maximum Subarray)

Kadane's algorithm finds the contiguous subarray with the largest sum in O(n) time. The key insight: at each position, decide whether to **extend** the current subarray or **start fresh** from the current element. If the running sum drops below the current element, it's better to restart — any prefix that makes the sum smaller is a net negative.

**When to use:** Maximum subarray sum, maximum product subarray, and variations where you track a running "best so far" vs "best ending here."

```javascript
function maxSubarray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];

  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }

  return maxSum;
}

console.log(maxSubarray([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6
```

```python
def max_subarray(nums):
    max_sum = current_sum = nums[0]

    for num in nums[1:]:
        current_sum = max(num, current_sum + num)
        max_sum = max(max_sum, current_sum)

    return max_sum

print(max_subarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]))  # 6
```

```csharp
public int MaxSubArray(int[] nums)
{
    int maxSum = nums[0], currentSum = nums[0];

    for (int i = 1; i < nums.Length; i++)
    {
        currentSum = Math.Max(nums[i], currentSum + nums[i]);
        maxSum = Math.Max(maxSum, currentSum);
    }

    return maxSum;
}
```

### String Reversal

Reversing a string (or character array) is a fundamental two-pointer technique: swap elements from both ends moving inward until the pointers meet. In-place reversal runs in O(n) time and O(1) extra space. This pattern appears as a building block in many problems — reversing words in a sentence, checking palindromes, or rotating arrays.

```javascript
function reverse(s) {
  return s.split('').reverse().join('');
}

// In-place (as char array)
function reverseInPlace(arr, left, right) {
  while (left < right) {
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }
}
```

### Anagram Check

Two strings are **anagrams** if they contain the same characters in the same frequencies. The standard approach counts character occurrences in one string, then decrements for the other — if all counts reach zero, they match. This runs in O(n) time using a hash map (or fixed-size array for lowercase letters).

**When to use:** Grouping anagrams, checking permutations, sliding-window substring problems where order doesn't matter.

```javascript
function isAnagram(s, t) {
  if (s.length !== t.length) return false;
  const count = {};
  for (const c of s) count[c] = (count[c] || 0) + 1;
  for (const c of t) {
    if (!count[c]) return false;
    count[c]--;
  }
  return true;
}
```

```python
from collections import Counter

def is_anagram(s: str, t: str) -> bool:
    return Counter(s) == Counter(t)
```

```csharp
public bool IsAnagram(string s, string t)
{
    if (s.Length != t.Length) return false;
    var count = new int[26];
    foreach (char c in s) count[c - 'a']++;
    foreach (char c in t)
    {
        count[c - 'a']--;
        if (count[c - 'a'] < 0) return false;
    }
    return true;
}
```

---

## Practice Problems

- [LeetCode 53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray/)
- [LeetCode 238 — Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)
- [LeetCode 242 — Valid Anagram](https://leetcode.com/problems/valid-anagram/)
- [LeetCode 49 — Group Anagrams](https://leetcode.com/problems/group-anagrams/)
- [LeetCode 128 — Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)
- [LeetCode 560 — Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

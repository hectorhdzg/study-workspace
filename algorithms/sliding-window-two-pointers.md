# Sliding Window & Two Pointers

These techniques reduce O(n²) brute-force solutions to O(n).

---

## Two Pointers

**Use for:** Sorted arrays, pairs with a target sum, in-place rearrangements.

### Two Sum in Sorted Array

```javascript
function twoSum(arr, target) {
  let left = 0, right = arr.length - 1;

  while (left < right) {
    const sum = arr[left] + arr[right];
    if (sum === target) return [left, right];
    if (sum < target) left++;
    else right--;
  }

  return [];
}

console.log(twoSum([1, 2, 3, 4, 6], 6)); // [1, 3]
```

### Remove Duplicates from Sorted Array

```javascript
function removeDuplicates(nums) {
  let slow = 0;

  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }

  return slow + 1;
}
```

### Three Sum

```javascript
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const result = [];

  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue; // skip duplicates

    let left = i + 1, right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }

  return result;
}
```

---

## Sliding Window

**Use for:** Subarrays/substrings with a constraint (max/min sum, length, unique chars).

### Fixed-Size Window: Max Sum Subarray of Size k

```javascript
function maxSumSubarray(arr, k) {
  let windowSum = 0;

  // Build first window
  for (let i = 0; i < k; i++) windowSum += arr[i];

  let maxSum = windowSum;

  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k]; // slide the window
    maxSum = Math.max(maxSum, windowSum);
  }

  return maxSum;
}

console.log(maxSumSubarray([2, 1, 5, 1, 3, 2], 3)); // 9 ([5,1,3])
```

### Variable-Size Window: Longest Substring Without Repeating Characters

```javascript
function lengthOfLongestSubstring(s) {
  const map = new Map();
  let left = 0, maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    if (map.has(s[right])) {
      left = Math.max(left, map.get(s[right]) + 1);
    }
    map.set(s[right], right);
    maxLen = Math.max(maxLen, right - left + 1);
  }

  return maxLen;
}

console.log(lengthOfLongestSubstring("abcabcbb")); // 3
```

### Minimum Window Substring

```javascript
function minWindow(s, t) {
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);

  let left = 0, missing = t.length;
  let start = 0, minLen = Infinity;

  for (let right = 0; right < s.length; right++) {
    if (need.get(s[right]) > 0) missing--;
    need.set(s[right], (need.get(s[right]) || 0) - 1);

    while (missing === 0) {
      if (right - left + 1 < minLen) {
        minLen = right - left + 1;
        start = left;
      }
      need.set(s[left], (need.get(s[left]) || 0) + 1);
      if (need.get(s[left]) > 0) missing++;
      left++;
    }
  }

  return minLen === Infinity ? "" : s.slice(start, start + minLen);
}

console.log(minWindow("ADOBECODEBANC", "ABC")); // "BANC"
```

---

## Practice Problems

- [LeetCode 3 — Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)
- [LeetCode 76 — Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
- [LeetCode 15 — 3Sum](https://leetcode.com/problems/3sum/)
- [LeetCode 11 — Container With Most Water](https://leetcode.com/problems/container-with-most-water/)
- [LeetCode 424 — Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)
- [LeetCode 567 — Permutation in String](https://leetcode.com/problems/permutation-in-string/)

# Hash Tables

## How Hash Tables Work

A **hash table** maps keys to values using a hash function. It provides O(1) average-case lookup, insertion, and deletion.

**Collision resolution:**
- **Chaining** — Each bucket is a linked list.
- **Open addressing** — Probe for the next empty slot.

## Complexity

| Operation | Average | Worst |
|-----------|---------|-------|
| Get       | O(1)    | O(n)  |
| Set       | O(1)    | O(n)  |
| Delete    | O(1)    | O(n)  |

---

## JavaScript Built-ins

```javascript
// Map: key-value, keys can be any type, ordered by insertion
const map = new Map();
map.set('a', 1);
map.get('a');         // 1
map.has('a');         // true
map.delete('a');
map.size;             // 0

// Set: unique values
const set = new Set([1, 2, 3, 2, 1]);
set.add(4);
set.has(2);           // true
set.delete(2);
set.size;             // 3
[...set];             // [1, 3, 4]
```

---

## Common Patterns

### Frequency Counter

```javascript
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) || 0) + 1);

  // Bucket sort by frequency
  const buckets = Array.from({ length: nums.length + 1 }, () => []);
  for (const [num, count] of freq) buckets[count].push(num);

  const result = [];
  for (let i = buckets.length - 1; i >= 0 && result.length < k; i--) {
    result.push(...buckets[i]);
  }

  return result.slice(0, k);
}

console.log(topKFrequent([1, 1, 1, 2, 2, 3], 2)); // [1, 2]
```

### Two Sum (Hash Map)

```javascript
function twoSum(nums, target) {
  const map = new Map();
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) return [map.get(complement), i];
    map.set(nums[i], i);
  }
  return [];
}

console.log(twoSum([2, 7, 11, 15], 9)); // [0, 1]
```

### Group Anagrams

```javascript
function groupAnagrams(strs) {
  const map = new Map();

  for (const str of strs) {
    const key = str.split('').sort().join('');
    if (!map.has(key)) map.set(key, []);
    map.get(key).push(str);
  }

  return [...map.values()];
}

console.log(groupAnagrams(["eat","tea","tan","ate","nat","bat"]));
// [["eat","tea","ate"],["tan","nat"],["bat"]]
```

### LRU Cache

```javascript
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.map = new Map(); // Map preserves insertion order
  }

  get(key) {
    if (!this.map.has(key)) return -1;
    const val = this.map.get(key);
    this.map.delete(key);
    this.map.set(key, val); // move to end (most recently used)
    return val;
  }

  put(key, value) {
    if (this.map.has(key)) this.map.delete(key);
    this.map.set(key, value);
    if (this.map.size > this.capacity) {
      this.map.delete(this.map.keys().next().value); // delete LRU (first)
    }
  }
}
```

---

## Practice Problems

- [LeetCode 1 — Two Sum](https://leetcode.com/problems/two-sum/)
- [LeetCode 49 — Group Anagrams](https://leetcode.com/problems/group-anagrams/)
- [LeetCode 347 — Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/)
- [LeetCode 146 — LRU Cache](https://leetcode.com/problems/lru-cache/)
- [LeetCode 128 — Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

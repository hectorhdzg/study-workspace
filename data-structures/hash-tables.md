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

The **frequency counter** pattern uses a hash map to count occurrences of each element. This transforms problems from O(n²) brute force (comparing every pair) into O(n) by counting first, then querying. A clever extension is **bucket sort by frequency**: group elements into buckets indexed by their count, then read buckets from highest to lowest.

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

The classic hash map problem: for each element, compute the complement (`target - current`) and check if it's already in the map. If yes, you've found the pair. If no, store the current element for future lookups. This is O(n) time and O(n) space — a single pass through the array.

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

```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []

print(two_sum([2, 7, 11, 15], 9))  # [0, 1]
```

```csharp
public int[] TwoSum(int[] nums, int target)
{
    var map = new Dictionary<int, int>();
    for (int i = 0; i < nums.Length; i++)
    {
        int complement = target - nums[i];
        if (map.ContainsKey(complement))
            return new[] { map[complement], i };
        map[nums[i]] = i;
    }
    return Array.Empty<int>();
}
```

### Group Anagrams

To group anagrams together, you need a way to compute the **same key** for all anagrams of a word. The simplest approach: sort each word’s characters and use the sorted string as the map key. All anagrams produce the same sorted key, so they end up in the same bucket.

**Time:** O(n · k log k) where k is the max word length | **Alternative:** Use a character count array as the key for O(n · k).

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

```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = ''.join(sorted(s))
        groups[key].append(s)
    return list(groups.values())

print(group_anagrams(["eat","tea","tan","ate","nat","bat"]))
```

```csharp
public IList<IList<string>> GroupAnagrams(string[] strs)
{
    var map = new Dictionary<string, IList<string>>();
    foreach (var s in strs)
    {
        var key = new string(s.OrderBy(c => c).ToArray());
        if (!map.ContainsKey(key)) map[key] = new List<string>();
        map[key].Add(s);
    }
    return map.Values.ToList<IList<string>>();
}
```

### LRU Cache

An **LRU (Least Recently Used) cache** evicts the oldest unused entry when it reaches capacity. It needs two operations in O(1): lookup by key (hash map) and tracking recency order (doubly linked list, or in JavaScript, `Map` which preserves insertion order).

On every `get` or `put`, the accessed entry moves to the "most recent" position. When the cache is full, the entry at the front (least recently used) is evicted.

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

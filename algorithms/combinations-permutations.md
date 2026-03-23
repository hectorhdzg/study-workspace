# Combinations & Permutations

## Core Definitions

- **Permutation** — arrangement where **order matters** → P(n, k) = n! / (n−k)!
- **Combination** — selection where **order does NOT matter** → C(n, k) = n! / (k! × (n−k)!)

| Formula           | Meaning                     | Example          |
|------------------|-----------------------------|------------------|
| n!               | All arrangements of n items  | 3! = 6           |
| P(n, k)          | k-length permutations from n | P(5, 2) = 20     |
| C(n, k)          | k-size subsets from n        | C(5, 2) = 10     |
| 2ⁿ               | All subsets of n items       | 2³ = 8           |
| C(n,k) = C(n−1,k−1) + C(n−1,k) | Pascal's triangle | Used in DP  |

---

## Generating Subsets (Power Set)

Every element is either **included or excluded** → 2ⁿ total subsets.

```javascript
function subsets(nums) {
  const result = [];

  function backtrack(start, current) {
    result.push([...current]);
    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }

  backtrack(0, []);
  return result;
}

console.log(subsets([1,2,3]));
// [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

```python
def subsets(nums):
    result = []

    def backtrack(start, current):
        result.append(current[:])
        for i in range(start, len(nums)):
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()

    backtrack(0, [])
    return result

print(subsets([1, 2, 3]))
# [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

```csharp
public IList<IList<int>> Subsets(int[] nums)
{
    var result = new List<IList<int>>();
    Backtrack(nums, 0, new List<int>(), result);
    return result;
}

void Backtrack(int[] nums, int start, List<int> current, List<IList<int>> result)
{
    result.Add(new List<int>(current));
    for (int i = start; i < nums.Length; i++)
    {
        current.Add(nums[i]);
        Backtrack(nums, i + 1, current, result);
        current.RemoveAt(current.Count - 1);
    }
}
```

---

## Generating Permutations

All orderings of elements → n! total permutations.

```javascript
function permute(nums) {
  const result = [];

  function backtrack(current, remaining) {
    if (remaining.length === 0) {
      result.push([...current]);
      return;
    }
    for (let i = 0; i < remaining.length; i++) {
      current.push(remaining[i]);
      backtrack(current, [...remaining.slice(0, i), ...remaining.slice(i + 1)]);
      current.pop();
    }
  }

  backtrack([], nums);
  return result;
}

console.log(permute([1,2,3]));
// [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

```python
def permute(nums):
    result = []

    def backtrack(current, remaining):
        if not remaining:
            result.append(current[:])
            return
        for i in range(len(remaining)):
            current.append(remaining[i])
            backtrack(current, remaining[:i] + remaining[i+1:])
            current.pop()

    backtrack([], nums)
    return result

print(permute([1, 2, 3]))
```

```csharp
public IList<IList<int>> Permute(int[] nums)
{
    var result = new List<IList<int>>();
    Backtrack(nums, new List<int>(), new bool[nums.Length], result);
    return result;
}

void Backtrack(int[] nums, List<int> current, bool[] used, List<IList<int>> result)
{
    if (current.Count == nums.Length)
    {
        result.Add(new List<int>(current));
        return;
    }
    for (int i = 0; i < nums.Length; i++)
    {
        if (used[i]) continue;
        used[i] = true;
        current.Add(nums[i]);
        Backtrack(nums, current, used, result);
        current.RemoveAt(current.Count - 1);
        used[i] = false;
    }
}
```

---

## Generating Combinations (k elements from n)

```javascript
function combine(n, k) {
  const result = [];

  function backtrack(start, current) {
    if (current.length === k) {
      result.push([...current]);
      return;
    }
    for (let i = start; i <= n; i++) {
      current.push(i);
      backtrack(i + 1, current);
      current.pop();
    }
  }

  backtrack(1, []);
  return result;
}

console.log(combine(4, 2)); // [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```

```python
def combine(n, k):
    result = []

    def backtrack(start, current):
        if len(current) == k:
            result.append(current[:])
            return
        for i in range(start, n + 1):
            current.append(i)
            backtrack(i + 1, current)
            current.pop()

    backtrack(1, [])
    return result

print(combine(4, 2))  # [[1,2],[1,3],[1,4],[2,3],[2,4],[3,4]]
```

```csharp
public IList<IList<int>> Combine(int n, int k)
{
    var result = new List<IList<int>>();
    Backtrack(n, k, 1, new List<int>(), result);
    return result;
}

void Backtrack(int n, int k, int start, List<int> current, List<IList<int>> result)
{
    if (current.Count == k)
    {
        result.Add(new List<int>(current));
        return;
    }
    for (int i = start; i <= n; i++)
    {
        current.Add(i);
        Backtrack(n, k, i + 1, current, result);
        current.RemoveAt(current.Count - 1);
    }
}
```

---

## Handling Duplicates

When the input has duplicates, **sort first** and **skip consecutive duplicates** at the same recursion level.

```javascript
function subsetsWithDup(nums) {
  nums.sort((a, b) => a - b);
  const result = [];

  function backtrack(start, current) {
    result.push([...current]);
    for (let i = start; i < nums.length; i++) {
      if (i > start && nums[i] === nums[i - 1]) continue; // skip duplicates
      current.push(nums[i]);
      backtrack(i + 1, current);
      current.pop();
    }
  }

  backtrack(0, []);
  return result;
}
```

```python
def subsets_with_dup(nums):
    nums.sort()
    result = []

    def backtrack(start, current):
        result.append(current[:])
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:
                continue   # skip duplicates
            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()

    backtrack(0, [])
    return result
```

```csharp
public IList<IList<int>> SubsetsWithDup(int[] nums)
{
    Array.Sort(nums);
    var result = new List<IList<int>>();
    Backtrack(nums, 0, new List<int>(), result);
    return result;
}

void Backtrack(int[] nums, int start, List<int> current, List<IList<int>> result)
{
    result.Add(new List<int>(current));
    for (int i = start; i < nums.Length; i++)
    {
        if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicates
        current.Add(nums[i]);
        Backtrack(nums, i + 1, current, result);
        current.RemoveAt(current.Count - 1);
    }
}
```

---

## Next Permutation

Find the next lexicographically greater arrangement in-place:

1. Find the largest `i` where `nums[i] < nums[i+1]` (the "pivot")
2. Find the largest `j > i` where `nums[j] > nums[i]`
3. Swap `nums[i]` and `nums[j]`
4. Reverse everything after index `i`

```javascript
function nextPermutation(nums) {
  let i = nums.length - 2;
  while (i >= 0 && nums[i] >= nums[i + 1]) i--;  // find pivot

  if (i >= 0) {
    let j = nums.length - 1;
    while (nums[j] <= nums[i]) j--;               // find swap target
    [nums[i], nums[j]] = [nums[j], nums[i]];      // swap
  }

  // reverse suffix
  let left = i + 1, right = nums.length - 1;
  while (left < right) {
    [nums[left], nums[right]] = [nums[right], nums[left]];
    left++; right--;
  }
}
```

```python
def next_permutation(nums):
    i = len(nums) - 2
    while i >= 0 and nums[i] >= nums[i + 1]:
        i -= 1       # find pivot

    if i >= 0:
        j = len(nums) - 1
        while nums[j] <= nums[i]:
            j -= 1   # find swap target
        nums[i], nums[j] = nums[j], nums[i]

    nums[i + 1:] = reversed(nums[i + 1:])  # reverse suffix
```

```csharp
public void NextPermutation(int[] nums)
{
    int i = nums.Length - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;

    if (i >= 0)
    {
        int j = nums.Length - 1;
        while (nums[j] <= nums[i]) j--;
        (nums[i], nums[j]) = (nums[j], nums[i]);
    }

    Array.Reverse(nums, i + 1, nums.Length - i - 1);
}
```

---

## Complexity Summary

| Problem             | Time       | Space   | Count              |
|--------------------|-----------|---------|---------------------|
| All subsets         | O(n × 2ⁿ) | O(n)    | 2ⁿ subsets          |
| All permutations    | O(n × n!) | O(n)    | n! permutations     |
| Combinations C(n,k) | O(k × C(n,k)) | O(k) | C(n,k) combinations|
| Next permutation    | O(n)       | O(1)    | —                   |

---

## LeetCode Practice

- [LeetCode 78 — Subsets](https://leetcode.com/problems/subsets/)
- [LeetCode 90 — Subsets II](https://leetcode.com/problems/subsets-ii/)
- [LeetCode 46 — Permutations](https://leetcode.com/problems/permutations/)
- [LeetCode 47 — Permutations II](https://leetcode.com/problems/permutations-ii/)
- [LeetCode 77 — Combinations](https://leetcode.com/problems/combinations/)
- [LeetCode 31 — Next Permutation](https://leetcode.com/problems/next-permutation/)
- [LeetCode 784 — Letter Case Permutation](https://leetcode.com/problems/letter-case-permutation/)

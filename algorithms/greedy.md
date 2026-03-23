# Greedy Algorithms

## Core Concept

Make the **locally optimal choice at each step**, hoping to reach a globally optimal solution. Once a choice is made, you **never reconsider** it — no backtracking.

**Greedy works when** the problem has the "greedy choice property": a local optimum leads to a global optimum. **Greedy fails when** choices interact — use DP instead.

### Greedy vs Dynamic Programming

|                        | Greedy | DP     |
|-----------------------|--------|--------|
| Reconsiders choices?   | No     | Yes    |
| Explores all options?  | No     | Yes    |
| Speed                  | Faster | Slower |
| When correct?          | Greedy choice property | Optimal substructure |

### How to Recognize a Greedy Problem

- You need to **maximize or minimize** something
- At each step, there's a clear "best" choice (largest, smallest, earliest deadline)
- Making the locally best choice never forces a suboptimal global result
- Hint words: *"minimum number of"*, *"maximum profit"*, *"earliest"*, *"at least"*

---

## Interval Scheduling

Sort intervals by **end time**, then always pick the interval that finishes earliest — this leaves room for the most future intervals.

```javascript
function eraseOverlapIntervals(intervals) {
  intervals.sort((a, b) => a[1] - b[1]); // sort by end time
  let removals = 0;
  let prevEnd = -Infinity;

  for (const [start, end] of intervals) {
    if (start >= prevEnd) {
      prevEnd = end;        // no overlap — keep this interval
    } else {
      removals++;           // overlap — remove it (greedy: keep earlier end)
    }
  }

  return removals;
}

console.log(eraseOverlapIntervals([[1,2],[2,3],[3,4],[1,3]])); // 1
```

```python
def erase_overlap_intervals(intervals):
    intervals.sort(key=lambda x: x[1])   # sort by end time
    removals = 0
    prev_end = float('-inf')

    for start, end in intervals:
        if start >= prev_end:
            prev_end = end    # no overlap — keep
        else:
            removals += 1     # overlap — remove

    return removals

print(erase_overlap_intervals([[1,2],[2,3],[3,4],[1,3]]))  # 1
```

```csharp
public int EraseOverlapIntervals(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[1].CompareTo(b[1])); // sort by end
    int removals = 0;
    int prevEnd = int.MinValue;

    foreach (var interval in intervals)
    {
        if (interval[0] >= prevEnd)
            prevEnd = interval[1];   // no overlap — keep
        else
            removals++;              // overlap — remove
    }

    return removals;
}
```

---

## Jump Game

Track the **farthest index reachable** so far. At each position, ask: "Can I still reach here?"

```javascript
function canJump(nums) {
  let farthest = 0;

  for (let i = 0; i < nums.length; i++) {
    if (i > farthest) return false;       // can't reach this index
    farthest = Math.max(farthest, i + nums[i]);
  }

  return true;
}

console.log(canJump([2,3,1,1,4])); // true
console.log(canJump([3,2,1,0,4])); // false
```

```python
def can_jump(nums):
    farthest = 0

    for i, jump in enumerate(nums):
        if i > farthest:
            return False              # can't reach this index
        farthest = max(farthest, i + jump)

    return True

print(can_jump([2,3,1,1,4]))  # True
print(can_jump([3,2,1,0,4]))  # False
```

```csharp
public bool CanJump(int[] nums)
{
    int farthest = 0;

    for (int i = 0; i < nums.Length; i++)
    {
        if (i > farthest) return false;
        farthest = Math.Max(farthest, i + nums[i]);
    }

    return true;
}
```

---

## Gas Station (Circular Route)

If total gas ≥ total cost, a solution exists. Find the start by resetting whenever the running tank goes negative.

```javascript
function canCompleteCircuit(gas, cost) {
  let totalSurplus = 0;
  let currentTank = 0;
  let start = 0;

  for (let i = 0; i < gas.length; i++) {
    const surplus = gas[i] - cost[i];
    totalSurplus += surplus;
    currentTank += surplus;

    if (currentTank < 0) {
      start = i + 1;     // can't start from any station before here
      currentTank = 0;
    }
  }

  return totalSurplus >= 0 ? start : -1;
}
```

```python
def can_complete_circuit(gas, cost):
    total_surplus = 0
    current_tank = 0
    start = 0

    for i in range(len(gas)):
        surplus = gas[i] - cost[i]
        total_surplus += surplus
        current_tank += surplus

        if current_tank < 0:
            start = i + 1     # can't start from any station before here
            current_tank = 0

    return start if total_surplus >= 0 else -1
```

```csharp
public int CanCompleteCircuit(int[] gas, int[] cost)
{
    int totalSurplus = 0, currentTank = 0, start = 0;

    for (int i = 0; i < gas.Length; i++)
    {
        int surplus = gas[i] - cost[i];
        totalSurplus += surplus;
        currentTank += surplus;

        if (currentTank < 0)
        {
            start = i + 1;
            currentTank = 0;
        }
    }

    return totalSurplus >= 0 ? start : -1;
}
```

---

## Partition Labels

Find the last occurrence of each character, then greedily extend each partition until you reach the farthest last-occurrence seen so far.

```javascript
function partitionLabels(s) {
  const last = {};
  for (let i = 0; i < s.length; i++) last[s[i]] = i;

  const result = [];
  let start = 0, end = 0;

  for (let i = 0; i < s.length; i++) {
    end = Math.max(end, last[s[i]]);
    if (i === end) {
      result.push(end - start + 1);
      start = i + 1;
    }
  }

  return result;
}

console.log(partitionLabels("ababcbacadefegdehijhklij")); // [9,7,8]
```

```python
def partition_labels(s):
    last = {c: i for i, c in enumerate(s)}
    result = []
    start = end = 0

    for i, c in enumerate(s):
        end = max(end, last[c])
        if i == end:
            result.append(end - start + 1)
            start = i + 1

    return result

print(partition_labels("ababcbacadefegdehijhklij"))  # [9, 7, 8]
```

```csharp
public IList<int> PartitionLabels(string s)
{
    var last = new int[26];
    for (int i = 0; i < s.Length; i++)
        last[s[i] - 'a'] = i;

    var result = new List<int>();
    int start = 0, end = 0;

    for (int i = 0; i < s.Length; i++)
    {
        end = Math.Max(end, last[s[i] - 'a']);
        if (i == end)
        {
            result.Add(end - start + 1);
            start = i + 1;
        }
    }

    return result;
}
```

---

## Key Greedy Patterns

| Pattern               | Strategy                              | Classic Problem               |
|----------------------|---------------------------------------|-------------------------------|
| Interval Scheduling   | Sort by end time, pick earliest end   | Non-overlapping Intervals     |
| Jump Game             | Track farthest reachable index        | Jump Game, Jump Game II       |
| Fractional Knapsack   | Sort by value/weight ratio            | Greedy knapsack               |
| Meeting Rooms         | Sort by start, track with min-heap    | Meeting Rooms II              |
| Huffman Encoding      | Merge lowest-frequency nodes first    | Huffman tree                  |
| Circular traversal    | Reset when tank goes negative         | Gas Station                   |
| Partition             | Extend window to last occurrence      | Partition Labels              |

---

## LeetCode Practice

- [LeetCode 55 — Jump Game](https://leetcode.com/problems/jump-game/)
- [LeetCode 45 — Jump Game II](https://leetcode.com/problems/jump-game-ii/)
- [LeetCode 435 — Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)
- [LeetCode 452 — Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)
- [LeetCode 134 — Gas Station](https://leetcode.com/problems/gas-station/)
- [LeetCode 763 — Partition Labels](https://leetcode.com/problems/partition-labels/)
- [LeetCode 621 — Task Scheduler](https://leetcode.com/problems/task-scheduler/)
- [LeetCode 455 — Assign Cookies](https://leetcode.com/problems/assign-cookies/)

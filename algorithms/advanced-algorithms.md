# Advanced Algorithms

## String Matching Algorithms

### KMP (Knuth-Morris-Pratt)

Finds a pattern in a text in O(n + m) by precomputing a **failure function** (longest proper prefix that's also a suffix). Never re-scans characters.

```javascript
function kmpSearch(text, pattern) {
  const lps = buildLPS(pattern);
  const result = [];
  let j = 0;

  for (let i = 0; i < text.length; i++) {
    while (j > 0 && text[i] !== pattern[j]) j = lps[j - 1];
    if (text[i] === pattern[j]) j++;
    if (j === pattern.length) {
      result.push(i - j + 1);
      j = lps[j - 1];
    }
  }

  return result;
}

function buildLPS(pattern) {
  const lps = [0];
  let len = 0, i = 1;

  while (i < pattern.length) {
    if (pattern[i] === pattern[len]) {
      lps.push(++len);
      i++;
    } else if (len > 0) {
      len = lps[len - 1];
    } else {
      lps.push(0);
      i++;
    }
  }

  return lps;
}

console.log(kmpSearch("ababcababd", "ababd")); // [5]
```

```python
def kmp_search(text, pattern):
    lps = build_lps(pattern)
    result = []
    j = 0

    for i in range(len(text)):
        while j > 0 and text[i] != pattern[j]:
            j = lps[j - 1]
        if text[i] == pattern[j]:
            j += 1
        if j == len(pattern):
            result.append(i - j + 1)
            j = lps[j - 1]

    return result

def build_lps(pattern):
    lps = [0] * len(pattern)
    length, i = 0, 1

    while i < len(pattern):
        if pattern[i] == pattern[length]:
            length += 1
            lps[i] = length
            i += 1
        elif length > 0:
            length = lps[length - 1]
        else:
            lps[i] = 0
            i += 1

    return lps

print(kmp_search("ababcababd", "ababd"))  # [5]
```

```csharp
public List<int> KmpSearch(string text, string pattern)
{
    int[] lps = BuildLPS(pattern);
    var result = new List<int>();
    int j = 0;

    for (int i = 0; i < text.Length; i++)
    {
        while (j > 0 && text[i] != pattern[j]) j = lps[j - 1];
        if (text[i] == pattern[j]) j++;
        if (j == pattern.Length)
        {
            result.Add(i - j + 1);
            j = lps[j - 1];
        }
    }

    return result;
}

int[] BuildLPS(string pattern)
{
    int[] lps = new int[pattern.Length];
    int len = 0, i = 1;

    while (i < pattern.Length)
    {
        if (pattern[i] == pattern[len]) { lps[i++] = ++len; }
        else if (len > 0) { len = lps[len - 1]; }
        else { lps[i++] = 0; }
    }

    return lps;
}
```

---

## Shortest Path Algorithms

### Dijkstra's Algorithm

Shortest path from a **single source** in a **non-negative weighted** graph. Uses a min-heap.

**Time:** O((V + E) log V) with a min-heap

```javascript
function dijkstra(graph, start) {
  const dist = new Map();
  const heap = [[0, start]]; // [distance, node]

  while (heap.length > 0) {
    heap.sort((a, b) => a[0] - b[0]); // min-heap (use a real heap in prod)
    const [d, u] = heap.shift();

    if (dist.has(u)) continue;
    dist.set(u, d);

    for (const [v, weight] of (graph.get(u) || [])) {
      if (!dist.has(v)) {
        heap.push([d + weight, v]);
      }
    }
  }

  return dist;
}
```

```python
import heapq

def dijkstra(graph, start):
    dist = {}
    heap = [(0, start)]

    while heap:
        d, u = heapq.heappop(heap)
        if u in dist:
            continue
        dist[u] = d

        for v, weight in graph.get(u, []):
            if v not in dist:
                heapq.heappush(heap, (d + weight, v))

    return dist
```

```csharp
public Dictionary<int, int> Dijkstra(Dictionary<int, List<(int node, int weight)>> graph, int start)
{
    var dist = new Dictionary<int, int>();
    var heap = new PriorityQueue<int, int>();
    heap.Enqueue(start, 0);

    while (heap.TryDequeue(out int u, out int d))
    {
        if (dist.ContainsKey(u)) continue;
        dist[u] = d;

        if (!graph.ContainsKey(u)) continue;
        foreach (var (v, w) in graph[u])
        {
            if (!dist.ContainsKey(v))
                heap.Enqueue(v, d + w);
        }
    }

    return dist;
}
```

### Bellman-Ford

Handles **negative edge weights**. Detects negative cycles. Slower than Dijkstra: O(V × E).

```javascript
function bellmanFord(edges, n, start) {
  const dist = new Array(n).fill(Infinity);
  dist[start] = 0;

  for (let i = 0; i < n - 1; i++) {
    for (const [u, v, w] of edges) {
      if (dist[u] !== Infinity && dist[u] + w < dist[v]) {
        dist[v] = dist[u] + w;
      }
    }
  }

  // Check for negative cycles
  for (const [u, v, w] of edges) {
    if (dist[u] !== Infinity && dist[u] + w < dist[v]) {
      return null; // negative cycle detected
    }
  }

  return dist;
}
```

```python
def bellman_ford(edges, n, start):
    dist = [float('inf')] * n
    dist[start] = 0

    for _ in range(n - 1):
        for u, v, w in edges:
            if dist[u] != float('inf') and dist[u] + w < dist[v]:
                dist[v] = dist[u] + w

    # Check for negative cycles
    for u, v, w in edges:
        if dist[u] != float('inf') and dist[u] + w < dist[v]:
            return None  # negative cycle

    return dist
```

```csharp
public int[] BellmanFord(int[][] edges, int n, int start)
{
    int[] dist = Enumerable.Repeat(int.MaxValue, n).ToArray();
    dist[start] = 0;

    for (int i = 0; i < n - 1; i++)
        foreach (var e in edges)
            if (dist[e[0]] != int.MaxValue && dist[e[0]] + e[2] < dist[e[1]])
                dist[e[1]] = dist[e[0]] + e[2];

    foreach (var e in edges)
        if (dist[e[0]] != int.MaxValue && dist[e[0]] + e[2] < dist[e[1]])
            return null; // negative cycle

    return dist;
}
```

---

## Union-Find (Disjoint Set)

Track connected components efficiently. Supports **union** and **find** in near O(1) amortized with path compression + union by rank.

```javascript
class UnionFind {
  constructor(n) {
    this.parent = Array.from({length: n}, (_, i) => i);
    this.rank = new Array(n).fill(0);
  }

  find(x) {
    if (this.parent[x] !== x)
      this.parent[x] = this.find(this.parent[x]); // path compression
    return this.parent[x];
  }

  union(x, y) {
    const px = this.find(x), py = this.find(y);
    if (px === py) return false;
    if (this.rank[px] < this.rank[py]) this.parent[px] = py;
    else if (this.rank[px] > this.rank[py]) this.parent[py] = px;
    else { this.parent[py] = px; this.rank[px]++; }
    return true;
  }
}
```

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # path compression
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False
        if self.rank[px] < self.rank[py]:
            self.parent[px] = py
        elif self.rank[px] > self.rank[py]:
            self.parent[py] = px
        else:
            self.parent[py] = px
            self.rank[px] += 1
        return True
```

```csharp
public class UnionFind
{
    private int[] parent, rank;

    public UnionFind(int n)
    {
        parent = Enumerable.Range(0, n).ToArray();
        rank = new int[n];
    }

    public int Find(int x)
    {
        if (parent[x] != x)
            parent[x] = Find(parent[x]); // path compression
        return parent[x];
    }

    public bool Union(int x, int y)
    {
        int px = Find(x), py = Find(y);
        if (px == py) return false;
        if (rank[px] < rank[py]) parent[px] = py;
        else if (rank[px] > rank[py]) parent[py] = px;
        else { parent[py] = px; rank[px]++; }
        return true;
    }
}
```

---

## Monotonic Stack

Maintains elements in **increasing or decreasing order**. Solves "next greater / next smaller" problems in O(n).

```javascript
function nextGreaterElement(nums) {
  const result = new Array(nums.length).fill(-1);
  const stack = []; // stores indices

  for (let i = 0; i < nums.length; i++) {
    while (stack.length > 0 && nums[stack[stack.length - 1]] < nums[i]) {
      result[stack.pop()] = nums[i];
    }
    stack.push(i);
  }

  return result;
}

console.log(nextGreaterElement([2, 1, 2, 4, 3])); // [4, 2, 4, -1, -1]
```

```python
def next_greater_element(nums):
    result = [-1] * len(nums)
    stack = []  # stores indices

    for i, num in enumerate(nums):
        while stack and nums[stack[-1]] < num:
            result[stack.pop()] = num
        stack.append(i)

    return result

print(next_greater_element([2, 1, 2, 4, 3]))  # [4, 2, 4, -1, -1]
```

```csharp
public int[] NextGreaterElement(int[] nums)
{
    int[] result = Enumerable.Repeat(-1, nums.Length).ToArray();
    var stack = new Stack<int>(); // stores indices

    for (int i = 0; i < nums.Length; i++)
    {
        while (stack.Count > 0 && nums[stack.Peek()] < nums[i])
            result[stack.Pop()] = nums[i];
        stack.Push(i);
    }

    return result;
}
```

---

## Interval Problems

### Merge Intervals

Sort by start time, then merge overlapping intervals in one pass.

```javascript
function merge(intervals) {
  intervals.sort((a, b) => a[0] - b[0]);
  const result = [intervals[0]];

  for (let i = 1; i < intervals.length; i++) {
    const last = result[result.length - 1];
    if (intervals[i][0] <= last[1]) {
      last[1] = Math.max(last[1], intervals[i][1]);
    } else {
      result.push(intervals[i]);
    }
  }

  return result;
}

console.log(merge([[1,3],[2,6],[8,10],[15,18]])); // [[1,6],[8,10],[15,18]]
```

```python
def merge(intervals):
    intervals.sort(key=lambda x: x[0])
    result = [intervals[0]]

    for start, end in intervals[1:]:
        if start <= result[-1][1]:
            result[-1][1] = max(result[-1][1], end)
        else:
            result.append([start, end])

    return result

print(merge([[1,3],[2,6],[8,10],[15,18]]))  # [[1,6],[8,10],[15,18]]
```

```csharp
public int[][] Merge(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0]));
    var result = new List<int[]> { intervals[0] };

    for (int i = 1; i < intervals.Length; i++)
    {
        var last = result[^1];
        if (intervals[i][0] <= last[1])
            last[1] = Math.Max(last[1], intervals[i][1]);
        else
            result.Add(intervals[i]);
    }

    return result.ToArray();
}
```

---

## Minimum Spanning Tree (Kruskal's)

Sort edges by weight, add each edge if it doesn't create a cycle (use Union-Find).

```javascript
function kruskal(n, edges) {
  edges.sort((a, b) => a[2] - b[2]); // sort by weight
  const uf = new UnionFind(n);
  const mst = [];
  let totalWeight = 0;

  for (const [u, v, w] of edges) {
    if (uf.union(u, v)) {
      mst.push([u, v, w]);
      totalWeight += w;
    }
  }

  return { mst, totalWeight };
}
```

```python
def kruskal(n, edges):
    edges.sort(key=lambda x: x[2])   # sort by weight
    uf = UnionFind(n)
    mst = []
    total = 0

    for u, v, w in edges:
        if uf.union(u, v):
            mst.append((u, v, w))
            total += w

    return mst, total
```

```csharp
public (List<int[]> mst, int total) Kruskal(int n, int[][] edges)
{
    Array.Sort(edges, (a, b) => a[2].CompareTo(b[2]));
    var uf = new UnionFind(n);
    var mst = new List<int[]>();
    int total = 0;

    foreach (var e in edges)
    {
        if (uf.Union(e[0], e[1]))
        {
            mst.Add(e);
            total += e[2];
        }
    }

    return (mst, total);
}
```

---

## Algorithm Comparison

| Algorithm       | Time         | Space   | Use Case                            |
|----------------|-------------|---------|-------------------------------------|
| KMP             | O(n + m)    | O(m)    | Pattern matching                    |
| Dijkstra        | O((V+E)log V)| O(V)   | Shortest path (non-negative weights)|
| Bellman-Ford    | O(V × E)   | O(V)    | Shortest path (negative weights)    |
| Union-Find      | O(α(n)) ≈ O(1) | O(n) | Connected components, cycle detection|
| Kruskal's MST   | O(E log E) | O(V)    | Minimum spanning tree               |
| Monotonic Stack  | O(n)       | O(n)    | Next greater/smaller element        |
| Merge Intervals  | O(n log n) | O(n)    | Overlapping intervals               |

---

## LeetCode Practice

- [LeetCode 743 — Network Delay Time](https://leetcode.com/problems/network-delay-time/) — Dijkstra
- [LeetCode 787 — Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) — Bellman-Ford
- [LeetCode 684 — Redundant Connection](https://leetcode.com/problems/redundant-connection/) — Union-Find
- [LeetCode 1584 — Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) — Kruskal's MST
- [LeetCode 739 — Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) — Monotonic Stack
- [LeetCode 56 — Merge Intervals](https://leetcode.com/problems/merge-intervals/) — Intervals
- [LeetCode 28 — Find the Index of the First Occurrence](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) — KMP
- [LeetCode 496 — Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) — Monotonic Stack

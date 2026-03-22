# Graphs (Data Structure)

## Representations

### Adjacency List

```javascript
// Undirected graph
const graph = new Map();
graph.set('A', ['B', 'C']);
graph.set('B', ['A', 'D']);
graph.set('C', ['A', 'D']);
graph.set('D', ['B', 'C']);

// Build from edge list
function buildGraph(edges) {
  const graph = new Map();
  for (const [u, v] of edges) {
    if (!graph.has(u)) graph.set(u, []);
    if (!graph.has(v)) graph.set(v, []);
    graph.get(u).push(v);
    graph.get(v).push(u); // remove for directed
  }
  return graph;
}
```

### Adjacency Matrix

```javascript
// n x n matrix: matrix[i][j] = 1 if edge from i to j
const n = 4;
const matrix = Array.from({ length: n }, () => new Array(n).fill(0));
// Add edge from 0 to 1:
matrix[0][1] = 1;
matrix[1][0] = 1; // for undirected
```

---

## Graph Properties

| Property        | Description                                           |
|----------------|-------------------------------------------------------|
| Directed        | Edges have direction (one-way)                        |
| Undirected      | Edges are bidirectional                               |
| Weighted        | Edges have costs/weights                              |
| Connected       | Path exists between every pair of nodes               |
| Acyclic (DAG)   | Directed graph with no cycles (used for dependencies) |
| Bipartite       | Nodes split into two groups; edges only cross groups  |

---

## Connected Components

```javascript
function countComponents(n, edges) {
  const adj = Array.from({ length: n }, () => []);
  for (const [u, v] of edges) {
    adj[u].push(v);
    adj[v].push(u);
  }

  const visited = new Set();
  let count = 0;

  function dfs(node) {
    visited.add(node);
    for (const neighbor of adj[node]) {
      if (!visited.has(neighbor)) dfs(neighbor);
    }
  }

  for (let i = 0; i < n; i++) {
    if (!visited.has(i)) {
      dfs(i);
      count++;
    }
  }

  return count;
}
```

```python
def count_components(n, edges):
    adj = [[] for _ in range(n)]
    for u, v in edges:
        adj[u].append(v)
        adj[v].append(u)

    visited = set()
    count = 0

    def dfs(node):
        visited.add(node)
        for neighbor in adj[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for i in range(n):
        if i not in visited:
            dfs(i)
            count += 1
    return count
```

```csharp
public int CountComponents(int n, int[][] edges)
{
    var adj = Enumerable.Range(0, n).Select(_ => new List<int>()).ToArray();
    foreach (var e in edges)
    {
        adj[e[0]].Add(e[1]);
        adj[e[1]].Add(e[0]);
    }

    var visited = new HashSet<int>();
    int count = 0;

    void Dfs(int node)
    {
        visited.Add(node);
        foreach (int neighbor in adj[node])
            if (!visited.Contains(neighbor)) Dfs(neighbor);
    }

    for (int i = 0; i < n; i++)
    {
        if (!visited.Contains(i)) { Dfs(i); count++; }
    }
    return count;
}
```

---

## Cycle Detection (Directed Graph)

```javascript
function hasCycleDirected(numNodes, edges) {
  const adj = Array.from({ length: numNodes }, () => []);
  for (const [u, v] of edges) adj[u].push(v);

  // 0 = unvisited, 1 = in stack, 2 = done
  const state = new Array(numNodes).fill(0);

  function dfs(node) {
    state[node] = 1;
    for (const neighbor of adj[node]) {
      if (state[neighbor] === 1) return true; // back edge = cycle
      if (state[neighbor] === 0 && dfs(neighbor)) return true;
    }
    state[node] = 2;
    return false;
  }

  for (let i = 0; i < numNodes; i++) {
    if (state[i] === 0 && dfs(i)) return true;
  }

  return false;
}
```

---

## Practice Problems

- [LeetCode 133 — Clone Graph](https://leetcode.com/problems/clone-graph/)
- [LeetCode 200 — Number of Islands](https://leetcode.com/problems/number-of-islands/)
- [LeetCode 323 — Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/)
- [LeetCode 785 — Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)
- [LeetCode 207 — Course Schedule](https://leetcode.com/problems/course-schedule/)
- [LeetCode 743 — Network Delay Time](https://leetcode.com/problems/network-delay-time/)

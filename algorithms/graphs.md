# Graph Algorithms

## Graph Representations

```javascript
// Adjacency List (most common)
const graph = {
  A: ['B', 'C'],
  B: ['A', 'D'],
  C: ['A', 'D'],
  D: ['B', 'C']
};

// Adjacency Matrix
const matrix = [
  // A  B  C  D
  [0, 1, 1, 0], // A
  [1, 0, 0, 1], // B
  [1, 0, 0, 1], // C
  [0, 1, 1, 0], // D
];
```

---

## Breadth-First Search (BFS)

**Use for:** Shortest path in unweighted graph, level-order traversal.

**Time:** O(V + E) | **Space:** O(V)

```javascript
function bfs(graph, start) {
  const visited = new Set();
  const queue = [start];
  const order = [];

  visited.add(start);

  while (queue.length > 0) {
    const node = queue.shift();
    order.push(node);

    for (const neighbor of (graph[node] || [])) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }

  return order;
}

// Shortest path (BFS)
function shortestPath(graph, start, end) {
  const visited = new Set([start]);
  const queue = [[start, [start]]];

  while (queue.length > 0) {
    const [node, path] = queue.shift();
    if (node === end) return path;

    for (const neighbor of (graph[node] || [])) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([neighbor, [...path, neighbor]]);
      }
    }
  }

  return null; // no path
}
```

---

## Depth-First Search (DFS)

**Use for:** Cycle detection, topological sort, connected components, path existence.

**Time:** O(V + E) | **Space:** O(V)

```javascript
// Iterative DFS
function dfs(graph, start) {
  const visited = new Set();
  const stack = [start];
  const order = [];

  while (stack.length > 0) {
    const node = stack.pop();
    if (visited.has(node)) continue;
    visited.add(node);
    order.push(node);

    for (const neighbor of (graph[node] || [])) {
      if (!visited.has(neighbor)) stack.push(neighbor);
    }
  }

  return order;
}

// Recursive DFS
function dfsRecursive(graph, node, visited = new Set()) {
  visited.add(node);
  for (const neighbor of (graph[node] || [])) {
    if (!visited.has(neighbor)) dfsRecursive(graph, neighbor, visited);
  }
  return visited;
}
```

---

## Topological Sort (Kahn's Algorithm — BFS)

**Use for:** Ordering tasks with dependencies (DAG only).

```javascript
function topologicalSort(numCourses, prerequisites) {
  const inDegree = new Array(numCourses).fill(0);
  const adj = Array.from({ length: numCourses }, () => []);

  for (const [course, pre] of prerequisites) {
    adj[pre].push(course);
    inDegree[course]++;
  }

  const queue = [];
  for (let i = 0; i < numCourses; i++) {
    if (inDegree[i] === 0) queue.push(i);
  }

  const order = [];
  while (queue.length > 0) {
    const node = queue.shift();
    order.push(node);
    for (const neighbor of adj[node]) {
      inDegree[neighbor]--;
      if (inDegree[neighbor] === 0) queue.push(neighbor);
    }
  }

  return order.length === numCourses ? order : []; // empty = cycle detected
}
```

---

## Dijkstra's Algorithm (Shortest Path — Weighted)

**Use for:** Shortest path in weighted graph (non-negative weights).

**Time:** O((V + E) log V)

```javascript
function dijkstra(graph, start) {
  // graph: { node: [[neighbor, weight], ...] }
  const dist = {};
  for (const node in graph) dist[node] = Infinity;
  dist[start] = 0;

  // Min-heap simulation using sorted array (use a proper heap in production)
  const pq = [[0, start]]; // [distance, node]

  while (pq.length > 0) {
    pq.sort((a, b) => a[0] - b[0]);
    const [d, u] = pq.shift();

    if (d > dist[u]) continue; // stale entry

    for (const [v, weight] of (graph[u] || [])) {
      if (dist[u] + weight < dist[v]) {
        dist[v] = dist[u] + weight;
        pq.push([dist[v], v]);
      }
    }
  }

  return dist;
}

// Example
const weightedGraph = {
  A: [['B', 1], ['C', 4]],
  B: [['C', 2], ['D', 5]],
  C: [['D', 1]],
  D: []
};
console.log(dijkstra(weightedGraph, 'A')); // { A:0, B:1, C:3, D:4 }
```

---

## Union-Find (Disjoint Set Union)

**Use for:** Connected components, cycle detection in undirected graphs, Kruskal's MST.

```javascript
class UnionFind {
  constructor(n) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.rank = new Array(n).fill(0);
  }

  find(x) {
    if (this.parent[x] !== x) {
      this.parent[x] = this.find(this.parent[x]); // path compression
    }
    return this.parent[x];
  }

  union(x, y) {
    const px = this.find(x), py = this.find(y);
    if (px === py) return false; // already connected

    // Union by rank
    if (this.rank[px] < this.rank[py]) this.parent[px] = py;
    else if (this.rank[px] > this.rank[py]) this.parent[py] = px;
    else { this.parent[py] = px; this.rank[px]++; }

    return true;
  }

  connected(x, y) {
    return this.find(x) === this.find(y);
  }
}
```

---

## Practice Problems

- [LeetCode 200 — Number of Islands](https://leetcode.com/problems/number-of-islands/)
- [LeetCode 207 — Course Schedule](https://leetcode.com/problems/course-schedule/)
- [LeetCode 210 — Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)
- [LeetCode 743 — Network Delay Time](https://leetcode.com/problems/network-delay-time/)
- [LeetCode 684 — Redundant Connection](https://leetcode.com/problems/redundant-connection/)
- [LeetCode 994 — Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
- [LeetCode 127 — Word Ladder](https://leetcode.com/problems/word-ladder/)

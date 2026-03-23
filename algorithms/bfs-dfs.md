# BFS & DFS

## At a Glance

| Feature           | BFS                              | DFS                                |
|------------------|----------------------------------|------------------------------------|
| Data structure    | Queue (FIFO)                     | Stack / recursion (LIFO)           |
| Exploration order | Level by level                   | As deep as possible, then backtrack|
| Shortest path?    | Yes (unweighted graphs)          | No                                 |
| Time complexity   | O(V + E)                         | O(V + E)                          |
| Space complexity  | O(V) — queue + visited           | O(V) — stack + visited            |
| Best for          | Shortest path, level-order       | Cycle detection, topological sort, all-paths |

---

## BFS — Breadth-First Search

Explores **level by level**. Visits all neighbors at distance 1 before distance 2. Guaranteed to find the **shortest path in unweighted graphs**.

### BFS Template (Graph)

```javascript
function bfs(graph, start) {
  const visited = new Set([start]);
  const queue = [start];

  while (queue.length > 0) {
    const node = queue.shift();
    // process node
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push(neighbor);
      }
    }
  }
}
```

```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    queue = deque([start])

    while queue:
        node = queue.popleft()
        # process node
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

```csharp
public void Bfs(Dictionary<int, List<int>> graph, int start)
{
    var visited = new HashSet<int> { start };
    var queue = new Queue<int>();
    queue.Enqueue(start);

    while (queue.Count > 0)
    {
        int node = queue.Dequeue();
        // process node
        foreach (int neighbor in graph[node])
        {
            if (visited.Add(neighbor))
                queue.Enqueue(neighbor);
        }
    }
}
```

### Level Order Traversal (Trees)

No `visited` set needed — trees have no cycles. Track levels by processing the queue in batches.

```javascript
function levelOrder(root) {
  if (!root) return [];
  const result = [];
  const queue = [root];

  while (queue.length > 0) {
    const levelSize = queue.length;
    const level = [];

    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }

    result.push(level);
  }

  return result;
}
```

```python
from collections import deque

def level_order(root):
    if not root:
        return []
    result = []
    queue = deque([root])

    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)

    return result
```

```csharp
public IList<IList<int>> LevelOrder(TreeNode root)
{
    var result = new List<IList<int>>();
    if (root == null) return result;
    var queue = new Queue<TreeNode>();
    queue.Enqueue(root);

    while (queue.Count > 0)
    {
        int size = queue.Count;
        var level = new List<int>();

        for (int i = 0; i < size; i++)
        {
            var node = queue.Dequeue();
            level.Add(node.val);
            if (node.left != null) queue.Enqueue(node.left);
            if (node.right != null) queue.Enqueue(node.right);
        }

        result.Add(level);
    }

    return result;
}
```

### Multi-Source BFS

Start BFS from **multiple sources simultaneously**. Add all starting nodes to the queue before the loop. Classic examples: Rotting Oranges, Walls and Gates.

```javascript
function orangesRotting(grid) {
  const rows = grid.length, cols = grid[0].length;
  const queue = [];
  let fresh = 0;

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 2) queue.push([r, c]);
      if (grid[r][c] === 1) fresh++;
    }
  }

  const dirs = [[1,0],[-1,0],[0,1],[0,-1]];
  let minutes = 0;

  while (queue.length > 0 && fresh > 0) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const [r, c] = queue.shift();
      for (const [dr, dc] of dirs) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] === 1) {
          grid[nr][nc] = 2;
          queue.push([nr, nc]);
          fresh--;
        }
      }
    }
    minutes++;
  }

  return fresh === 0 ? minutes : -1;
}
```

```python
from collections import deque

def oranges_rotting(grid):
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh = 0

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c))
            elif grid[r][c] == 1:
                fresh += 1

    directions = [(1,0),(-1,0),(0,1),(0,-1)]
    minutes = 0

    while queue and fresh > 0:
        for _ in range(len(queue)):
            r, c = queue.popleft()
            for dr, dc in directions:
                nr, nc = r + dr, c + dc
                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
                    grid[nr][nc] = 2
                    queue.append((nr, nc))
                    fresh -= 1
        minutes += 1

    return minutes if fresh == 0 else -1
```

```csharp
public int OrangesRotting(int[][] grid)
{
    int rows = grid.Length, cols = grid[0].Length;
    var queue = new Queue<(int r, int c)>();
    int fresh = 0;

    for (int r = 0; r < rows; r++)
        for (int c = 0; c < cols; c++)
        {
            if (grid[r][c] == 2) queue.Enqueue((r, c));
            if (grid[r][c] == 1) fresh++;
        }

    int[][] dirs = { new[]{1,0}, new[]{-1,0}, new[]{0,1}, new[]{0,-1} };
    int minutes = 0;

    while (queue.Count > 0 && fresh > 0)
    {
        int size = queue.Count;
        for (int i = 0; i < size; i++)
        {
            var (row, col) = queue.Dequeue();
            foreach (var d in dirs)
            {
                int nr = row + d[0], nc = col + d[1];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 1)
                {
                    grid[nr][nc] = 2;
                    queue.Enqueue((nr, nc));
                    fresh--;
                }
            }
        }
        minutes++;
    }

    return fresh == 0 ? minutes : -1;
}
```

---

## DFS — Depth-First Search

Explores **as deep as possible** before backtracking. Uses the call stack (recursion) or an explicit stack.

### DFS Templates

**Recursive (most common in interviews):**

```javascript
function dfs(graph, node, visited) {
  visited.add(node);
  // process node
  for (const neighbor of graph[node]) {
    if (!visited.has(neighbor)) {
      dfs(graph, neighbor, visited);
    }
  }
}
```

```python
def dfs(graph, node, visited):
    visited.add(node)
    # process node
    for neighbor in graph[node]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)
```

```csharp
public void Dfs(Dictionary<int, List<int>> graph, int node, HashSet<int> visited)
{
    visited.Add(node);
    // process node
    foreach (int neighbor in graph[node])
    {
        if (!visited.Contains(neighbor))
            Dfs(graph, neighbor, visited);
    }
}
```

**Iterative (explicit stack):**

```javascript
function dfsIterative(graph, start) {
  const visited = new Set();
  const stack = [start];

  while (stack.length > 0) {
    const node = stack.pop();
    if (visited.has(node)) continue;
    visited.add(node);
    // process node
    for (const neighbor of graph[node]) {
      if (!visited.has(neighbor)) stack.push(neighbor);
    }
  }
}
```

```python
def dfs_iterative(graph, start):
    visited = set()
    stack = [start]

    while stack:
        node = stack.pop()
        if node in visited:
            continue
        visited.add(node)
        # process node
        for neighbor in graph[node]:
            if neighbor not in visited:
                stack.append(neighbor)
```

```csharp
public void DfsIterative(Dictionary<int, List<int>> graph, int start)
{
    var visited = new HashSet<int>();
    var stack = new Stack<int>();
    stack.Push(start);

    while (stack.Count > 0)
    {
        int node = stack.Pop();
        if (!visited.Add(node)) continue;
        // process node
        foreach (int neighbor in graph[node])
        {
            if (!visited.Contains(neighbor))
                stack.Push(neighbor);
        }
    }
}
```

### Tree Traversal Orders

| Order      | Sequence                              | Use Case                     |
|-----------|---------------------------------------|------------------------------|
| Pre-order  | Node → Left → Right                  | Copy a tree, serialize       |
| In-order   | Left → Node → Right                  | BST sorted output            |
| Post-order | Left → Right → Node                  | Delete tree, evaluate expression |

### DFS Cycle Detection (Directed Graph)

Use three states: **white** (unvisited), **gray** (in progress), **black** (done). A back edge to a gray node means a cycle.

```javascript
function hasCycle(numCourses, prerequisites) {
  const graph = Array.from({length: numCourses}, () => []);
  for (const [course, prereq] of prerequisites) {
    graph[prereq].push(course);
  }

  const WHITE = 0, GRAY = 1, BLACK = 2;
  const color = new Array(numCourses).fill(WHITE);

  function dfs(node) {
    color[node] = GRAY;
    for (const neighbor of graph[node]) {
      if (color[neighbor] === GRAY) return true;    // cycle!
      if (color[neighbor] === WHITE && dfs(neighbor)) return true;
    }
    color[node] = BLACK;
    return false;
  }

  for (let i = 0; i < numCourses; i++) {
    if (color[i] === WHITE && dfs(i)) return false; // can't finish
  }
  return true; // no cycles — can finish all courses
}
```

```python
def can_finish(num_courses, prerequisites):
    graph = [[] for _ in range(num_courses)]
    for course, prereq in prerequisites:
        graph[prereq].append(course)

    WHITE, GRAY, BLACK = 0, 1, 2
    color = [WHITE] * num_courses

    def dfs(node):
        color[node] = GRAY
        for neighbor in graph[node]:
            if color[neighbor] == GRAY:
                return True     # cycle!
            if color[neighbor] == WHITE and dfs(neighbor):
                return True
        color[node] = BLACK
        return False

    return not any(color[i] == WHITE and dfs(i) for i in range(num_courses))
```

```csharp
public bool CanFinish(int numCourses, int[][] prerequisites)
{
    var graph = new List<int>[numCourses];
    for (int i = 0; i < numCourses; i++) graph[i] = new List<int>();
    foreach (var p in prerequisites) graph[p[1]].Add(p[0]);

    int[] color = new int[numCourses]; // 0=white, 1=gray, 2=black

    bool HasCycle(int node)
    {
        color[node] = 1;
        foreach (int neighbor in graph[node])
        {
            if (color[neighbor] == 1) return true;
            if (color[neighbor] == 0 && HasCycle(neighbor)) return true;
        }
        color[node] = 2;
        return false;
    }

    for (int i = 0; i < numCourses; i++)
        if (color[i] == 0 && HasCycle(i)) return false;

    return true;
}
```

---

## Key BFS Patterns

| Pattern             | Description                              | Example Problem         |
|--------------------|------------------------------------------|-------------------------|
| Level-order         | Process nodes level by level             | Binary Tree Level Order |
| Shortest path       | BFS guarantees shortest in unweighted    | Word Ladder             |
| Multi-source BFS    | Start from all sources simultaneously    | Rotting Oranges         |
| 0-1 BFS            | Deque; front for 0-weight, back for 1    | Shortest path in grid   |

## Key DFS Patterns

| Pattern              | Description                             | Example Problem          |
|---------------------|------------------------------------------|--------------------------|
| Path finding         | Explore and track current path            | All Paths Source → Target|
| Cycle detection      | White/gray/black coloring                 | Course Schedule          |
| Topological sort     | DFS post-order, reverse the result        | Course Schedule II       |
| Connected components | DFS from each unvisited node              | Number of Islands        |
| Flood fill           | DFS on grid, mark visited                 | Flood Fill               |

---

## LeetCode Practice

### BFS
- [LeetCode 102 — Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/)
- [LeetCode 200 — Number of Islands](https://leetcode.com/problems/number-of-islands/)
- [LeetCode 994 — Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)
- [LeetCode 127 — Word Ladder](https://leetcode.com/problems/word-ladder/)
- [LeetCode 1091 — Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/)
- [LeetCode 199 — Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/)

### DFS
- [LeetCode 200 — Number of Islands](https://leetcode.com/problems/number-of-islands/)
- [LeetCode 733 — Flood Fill](https://leetcode.com/problems/flood-fill/)
- [LeetCode 133 — Clone Graph](https://leetcode.com/problems/clone-graph/)
- [LeetCode 207 — Course Schedule](https://leetcode.com/problems/course-schedule/)
- [LeetCode 210 — Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)
- [LeetCode 417 — Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/)
- [LeetCode 797 — All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/)

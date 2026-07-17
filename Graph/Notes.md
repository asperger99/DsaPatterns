## Core Representations

### Adjacency List (most common in interviews)
```csharp
// Unweighted
Dictionary<int, List<int>> graph = new();

// Weighted
Dictionary<int, List<(int neighbor, int weight)>> graph = new();
// Use when: sparse graph
```

### Adjacency Matrix
```csharp
int[,] graph = new int[n, n];
// graph[u][v] = 1 means edge u→v
// Use when: dense graph, O(1) edge lookup needed
```

### Edge List
```csharp
List<(int u, int v, int weight)> edges = new();
// Use when: need to sort edges (Kruskal's MST)
```

---

## Building a Graph from Input

```csharp
// Given: int[][] edges, int n (nodes 0..n-1)
var adj = new Dictionary<int, List<int>>();
for (int i = 0; i < n; i++) adj[i] = new List<int>();

foreach (var e in edges)
{
    adj[e[0]].Add(e[1]);
    adj[e[1]].Add(e[0]); // omit for directed
}
```

---

## Traversal Patterns

### BFS — Shortest Path (unweighted), Level Order
```csharp
int[] BFS(int start, Dictionary<int, List<int>> adj)
{
    int n = adj.Count;
    int[] dist = new int[n];
    Array.Fill(dist, -1);
    dist[start] = 0;

    Queue<int> q = new();
    q.Enqueue(start);

    while (q.Count > 0)
    {
        int node = q.Dequeue();
        foreach (int nei in adj[node])
        {
            if (dist[nei] == -1)
            {
                dist[nei] = dist[node] + 1;
                q.Enqueue(nei);
            }
        }
    }
    return dist;
}
```

**When to use BFS:**
- Shortest path in unweighted graph
- Level-by-level processing
- Finding connected components
- Multi-source BFS: enqueue all sources at dist=0
- Topological search - Kahn's Algorithm

### DFS — Iterative
```csharp
void DFS(int start, Dictionary<int, List<int>> adj)
{
    bool[] visited = new bool[adj.Count];
    Stack<int> stack = new();
    stack.Push(start);

    while (stack.Count > 0)
    {
        int node = stack.Pop();
        if (visited[node]) continue;
        visited[node] = true;

        foreach (int nei in adj[node])
            if (!visited[nei]) stack.Push(nei);
    }
}
```

### DFS — Recursive
```csharp
void DFS(int node, bool[] visited, Dictionary<int, List<int>> adj)
{
    visited[node] = true;
    foreach (int nei in adj[node])
        if (!visited[nei]) DFS(nei, visited, adj);
}
```

**When to use DFS:**
- Cycle detection
- Topological sort
- Connected components
- Path existence
- Backtracking on graphs

---

## Key Algorithms & When to Use Them

| Problem | Algorithm | Time |
|---|---|---|
| Shortest path, unweighted | BFS | O(V + E) |
| Shortest path, weighted (non-neg) | Dijkstra | O((V+E) log V) |
| Shortest path, negative weights | Bellman-Ford | O(VE) |
| All-pairs shortest path | Floyd-Warshall | O(V³) |
| Minimum Spanning Tree | Prim / Kruskal | O(E log E) |
| Topological sort | Kahn's (BFS) / DFS post-order | O(V + E) |
| Cycle detection (directed) | DFS with recursion stack / Kahn's | O(V + E) |
| Cycle detection (undirected) | DFS with parent tracking / Union-Find | O(V + E) |
| Strongly connected components | Kosaraju / Tarjan | O(V + E) |
| Bipartite check | BFS/DFS 2-coloring | O(V + E) |

---

## Dijkstra's Algorithm

> LeetCode: [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/), [1514. Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/)

```csharp
int[] Dijkstra(int start, int n, Dictionary<int, List<(int nei, int w)>> adj)
{
    int[] dist = new int[n];
    Array.Fill(dist, int.MaxValue);
    dist[start] = 0;

    // MinHeap: (distance, node)
    PriorityQueue<int, int> pq = new();
    pq.Enqueue(start, 0);

    while (pq.Count > 0)
    {
        pq.TryDequeue(out int node, out int d);
        if (d > dist[node]) continue; // stale entry

        foreach (var (nei, w) in adj[node])
        {
            int newDist = dist[node] + w;
            if (newDist < dist[nei])
            {
                dist[nei] = newDist;
                pq.Enqueue(nei, newDist);
            }
        }
    }
    return dist;
}
```

### Path Reconstruction (BFS or Dijkstra)
Track a `prev[]` array to reconstruct the actual path, not just the distance.

```csharp
int[] prev = new int[n];
Array.Fill(prev, -1);

// Inside the relaxation step:
if (newDist < dist[nei])
{
    dist[nei] = newDist;
    prev[nei] = node; // record where we came from
    pq.Enqueue(nei, newDist);
}

// Reconstruct path from start → end:
List<int> path = new();
for (int cur = end; cur != -1; cur = prev[cur])
    path.Add(cur);
path.Reverse();
// path[0] == start, path[^1] == end (or empty if unreachable)
```

---

## Bellman-Ford

**When to reach for Bellman-Ford (beyond negative edges):**

**1. Negative edge weights** — Dijkstra breaks with negative weights because once a node is popped from the heap it's considered final. Bellman-Ford relaxes all edges repeatedly so it handles negative weights correctly.
> LeetCode: [743. Network Delay Time](https://leetcode.com/problems/network-delay-time/) (if weights could be negative)

**2. Negative cycle detection** — The nth relaxation pass tells you if a negative cycle exists (a cycle where total weight < 0, meaning you can loop forever and keep reducing cost).
> Useful in: currency arbitrage detection (convert exchange rates → negative log weights, then check for negative cycle)

**3. Shortest path with at most K edges** — Each outer iteration of Bellman-Ford computes shortest paths using at most `i` edges. Stop early at iteration `k` to enforce a hop limit. This is the real reason it's used for:
> LeetCode: [787. Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) — run k+1 iterations, copy `dist` each round to prevent using more than k+1 edges in one pass

**4. Dense graphs with simple implementation** — When E ≈ V², the O(VE) cost of Bellman-Ford is comparable to Dijkstra's O((V+E) log V), and the code is simpler (no priority queue needed).

### Walkthrough Example

Graph (start = A):
```
    A ---(1)---> B
    |            |
   (4)          (-3)
    |            |
    v            v
    C <--(2)--- D
```
Edges: A→B(1), A→C(4), B→D(1), D→C(-3) ... wait, let's write it simply:

```
Nodes: 0(A), 1(B), 2(C), 3(D)
Edges: 0→1 w=1, 0→2 w=4, 1→3 w=1, 3→2 w=-3

Start = 0, n = 4
Initial dist: [0, ∞, ∞, ∞]
```

**Iteration 1** (each edge relaxed once — paths using at most 1 edge):
```
Process 0→1 (w=1):  dist[0]+1=1  < dist[1]=∞  → dist[1] = 1
Process 0→2 (w=4):  dist[0]+4=4  < dist[2]=∞  → dist[2] = 4
Process 1→3 (w=1):  dist[1]=∞ (not yet updated this pass in standard B-F)
Process 3→2 (w=-3): dist[3]=∞   → skip

After iter 1: [0, 1, 4, ∞]
```

**Iteration 2** (paths using at most 2 edges):
```
Process 0→1 (w=1):  0+1=1  = dist[1]  → no change
Process 0→2 (w=4):  0+4=4  = dist[2]  → no change
Process 1→3 (w=1):  1+1=2  < dist[3]=∞ → dist[3] = 2
Process 3→2 (w=-3): dist[3]=∞ (stale in this pass)

After iter 2: [0, 1, 4, 2]
```

**Iteration 3** (paths using at most 3 edges):
```
Process 3→2 (w=-3): 2+(-3)=-1 < dist[2]=4 → dist[2] = -1

After iter 3: [0, 1, -1, 2]
```

Final: A→A=0, A→B=1, A→C=-1, A→D=2
(Path to C: A→B→D→C = 1+1-3 = -1, shorter than direct A→C=4)

**Key insight:** each iteration "unlocks" one more hop. That's why you need n-1 iterations — the longest possible shortest path in a graph with n nodes uses at most n-1 edges.
- Number of iterations needed depends on the edge order, but after n-1 iterations the shortest paths are guaranteed to have been found regardless of order.

---

```csharp
int[] BellmanFord(int start, int n, List<(int u, int v, int w)> edges)
{
    int[] dist = new int[n];
    Array.Fill(dist, int.MaxValue);
    dist[start] = 0;

    // Relax all edges n-1 times
    // After iteration i, dist[v] = shortest path using at most i+1 edges
    for (int i = 0; i < n - 1; i++)
        foreach (var (u, v, w) in edges)
            if (dist[u] != int.MaxValue && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;

    // nth relaxation: if anything still improves → negative cycle exists
    foreach (var (u, v, w) in edges)
        if (dist[u] != int.MaxValue && dist[u] + w < dist[v])
            return null;

    return dist;
}
```

### K-Stop Variant (787)
```csharp
// Run exactly k+1 iterations, copy dist each round to avoid chaining within one pass
int[] dist = ...; // init
for (int i = 0; i <= k; i++)
{
    int[] temp = (int[])dist.Clone(); // snapshot before this round
    foreach (var (u, v, w) in edges)
        if (dist[u] != int.MaxValue)
            temp[v] = Math.Min(temp[v], dist[u] + w);
    dist = temp;
}
```

Rule of thumb
- Shortest path with no edge limit: Use standard Bellman-Ford (in-place updates are fine). 
- Shortest path with an exact/maximum number of edges: Use double buffering (temp = dist.Clone()), so each iteration only builds on results from the previous iteration.

---

## Topological Sort

**What it is:** a linear ordering of nodes in a directed acyclic graph (DAG) such that every edge `u→v` has `u` before `v`.
**When to use:** scheduling, course prerequisites, build systems — anything with dependencies.

> LeetCode: [207. Course Schedule](https://leetcode.com/problems/course-schedule/), [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)

### Kahn's Algorithm (BFS)
**Idea:** nodes with no incoming edges have no prerequisites — process them first, then remove their edges and repeat.

```csharp
int[] TopologicalSort(int n, Dictionary<int, List<int>> adj)
{
    // Count how many edges point INTO each node
    int[] inDegree = new int[n];
    foreach (var (node, neighbors) in adj)
        foreach (int nei in neighbors)
            inDegree[nei]++;

    // Start with all nodes that have no prerequisites
    Queue<int> q = new();
    for (int i = 0; i < n; i++)
        if (inDegree[i] == 0) q.Enqueue(i);

    List<int> order = new();
    while (q.Count > 0)
    {
        int node = q.Dequeue();
        order.Add(node);
        // "Remove" this node's edges — reduce neighbors' in-degree
        foreach (int nei in adj[node])
            if (--inDegree[nei] == 0) q.Enqueue(nei); // nei now has no more prerequisites
    }

    // If not all nodes were processed → cycle exists (not a DAG)
    return order.Count == n ? order.ToArray() : Array.Empty<int>();
}
```

### DFS Post-Order (recursive)
**Idea:** a node belongs after all its descendants. Push to result after finishing all neighbors.

```csharp
void TopoSort(int n, Dictionary<int, List<int>> adj, out List<int> order)
{
    int[] color = new int[n]; // 0=unvisited, 1=in-progress, 2=done
    order = new List<int>();

    void DFS(int node)
    {
        color[node] = 1;
        foreach (int nei in adj[node])
        {
            if (color[nei] == 1) throw new Exception("Cycle detected");
            if (color[nei] == 0) DFS(nei);
        }
        color[node] = 2;
        order.Add(node); // add AFTER all descendants
    }

    for (int i = 0; i < n; i++)
        if (color[i] == 0) DFS(i);

    order.Reverse(); // post-order gives reverse topo order
}
```

---

## Cycle Detection

### Directed Graph — DFS with recursion stack (3-color)

Track each node with 3 states:
- `0` = never visited
- `1` = currently on the call stack (DFS is still inside this node)
- `2` = fully done (all its paths explored, no cycle found)

A cycle is detected when you reach a neighbor that is **still on the call stack** (`1`), meaning you've looped back to an ancestor.

```csharp
int[] color; // init all to 0

bool HasCycle(int node, Dictionary<int, List<int>> adj)
{
    color[node] = 1; // entering — mark in-progress

    foreach (int nei in adj[node])
    {
        if (color[nei] == 1) return true;           // nei is an ancestor → cycle!
        if (color[nei] == 0 && HasCycle(nei, adj))  // unvisited → recurse
            return true;
        // color[nei] == 2 → already fully explored, skip safely
    }

    color[node] = 2; // leaving — mark done
    return false;
}

// Call for every node (graph may be disconnected)
bool DetectCycle(int n, Dictionary<int, List<int>> adj)
{
    color = new int[n];
    for (int i = 0; i < n; i++)
        if (color[i] == 0 && HasCycle(i, adj)) return true;
    return false;
}
```

**Why not just `bool visited`?**
With only visited/unvisited you can't tell apart:
- a node still on the current path → ancestor → cycle (what you want to catch)
- a node already fully explored via a different path → fine, no cycle

Color `2` lets you skip already-finished nodes without false positives.

---

### Undirected Graph — DFS with parent tracking

In an undirected graph, every edge `A-B` appears as both `A→B` and `B→A`. When DFS visits `B` from `A`, it sees `A` as a neighbor — but that's the same edge you just came from, not a cycle.

Solution: track the `parent` (where you came from) and skip it. A cycle exists only if you reach a visited neighbor that is **not** your parent.

```csharp
bool HasCycle(int node, int parent, bool[] visited, Dictionary<int, List<int>> adj)
{
    visited[node] = true;

    foreach (int nei in adj[node])
    {
        if (!visited[nei])
            { if (HasCycle(nei, node, visited, adj)) return true; }
        else if (nei != parent)
            return true; // visited + not where we came from → cycle
    }
    return false;
}

// Call for every unvisited node
bool DetectCycle(int n, Dictionary<int, List<int>> adj)
{
    bool[] visited = new bool[n];
    for (int i = 0; i < n; i++)
        if (!visited[i] && HasCycle(i, -1, visited, adj)) return true;
    return false;
}
```

**Example:**
```
Cycle:    1 - 2 - 3
              |   |
              +---+   At node 3: neighbor 2 is visited AND 2 != parent(3) → cycle!

No cycle: 1 - 2 - 3   At node 3: neighbor 2 is visited BUT 2 == parent → skip (same edge)
```

---

### Quick Comparison

| | Directed | Undirected |
|---|---|---|
| Method | DFS 3-color (recursion stack) | DFS + parent |
| Cycle signal | reach a node with color `1` | reach visited node that isn't parent |
| Alternative | Kahn's: `order.Count != n` | Union-Find |

---

## Minimum Spanning Tree (MST)

**Spanning Tree (ST):** a subgraph that connects all `n` nodes using exactly `n-1` edges with no cycles. A graph can have many spanning trees.
**Minimum Spanning Tree (MST):** the spanning tree with the smallest possible total edge weight. Kruskal's and Prim's find this.
**When to use:** minimizing total cost to connect all nodes (network cables, roads, pipes).

> LeetCode: [1584. Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/), [1135. Connecting Cities With Minimum Cost](https://leetcode.com/problems/connecting-cities-with-minimum-cost/)

### Kruskal's Algorithm (sort edges + Union-Find)

**Idea:** greedily pick the cheapest edge that doesn't create a cycle. Repeat until n-1 edges are picked.

```
Graph:
  0 --4-- 1
  |  \    |
  2   3   1
  |    \  |
  2 --5-- 3
  
Edges sorted by weight: (0,2,2),(1,3,1),(0,3,3),(0,1,4),(2,3,5)
```

```csharp
int KruskalMST(int n, List<(int u, int v, int w)> edges)
{
    edges.Sort((a, b) => a.w.CompareTo(b.w)); // sort by weight ascending

    UnionFind uf = new(n);
    int totalCost = 0;
    int edgesPicked = 0;

    foreach (var (u, v, w) in edges)
    {
        if (uf.Union(u, v)) // returns false if already connected (would create cycle)
        {
            totalCost += w;
            edgesPicked++;
            if (edgesPicked == n - 1) break; // MST complete
        }
    }

    return edgesPicked == n - 1 ? totalCost : -1; // -1 if graph is disconnected
}
```

**Trace on example above:**
```
Pick (1,3,w=1) → no cycle → MST cost = 1,  edges = 1
Pick (0,2,w=2) → no cycle → MST cost = 3,  edges = 2
Pick (0,3,w=3) → no cycle → MST cost = 6,  edges = 3  ← n-1=3, done
Skip (0,1,w=4) → would create cycle 0-1-3-0
```

**Time:** O(E log E) for sorting + O(E·α(n)) for Union-Find ≈ O(E log E)
**Best when:** sparse graph, edges already given as a list.

---

### Prim's Algorithm (greedy + min-heap)

**Idea:** grow the MST one node at a time. Start from any node, always add the cheapest edge that connects a new node to the current MST.

```csharp
int PrimMST(int n, Dictionary<int, List<(int nei, int w)>> adj)
{
    bool[] inMST = new bool[n];
    // MinHeap: (cost, node)
    PriorityQueue<int, int> pq = new();
    pq.Enqueue(0, 0); // start from node 0 with cost 0

    int totalCost = 0;
    int nodesPicked = 0;

    while (pq.Count > 0)
    {
        pq.TryDequeue(out int node, out int cost);
        if (inMST[node]) continue; // already in MST, skip

        inMST[node] = true;
        totalCost += cost;
        nodesPicked++;

        foreach (var (nei, w) in adj[node])
            if (!inMST[nei]) pq.Enqueue(nei, w); // offer edge to MST
    }

    return nodesPicked == n ? totalCost : -1;
}
```

**Trace on same graph (start = node 0):**
```
Heap: [(0, node=0)]

Pop (cost=0, node=0) → add to MST. Push neighbors: (4,node=1),(2,node=2),(3,node=3)
Heap: [(2,2),(3,3),(4,1)]

Pop (cost=2, node=2) → add. Push: (5,node=3) already cheaper in heap
Heap: [(3,3),(4,1),(5,3)]

Pop (cost=3, node=3) → add. Push: (1,node=1) 
Heap: [(1,1),(4,1),(5,3)]

Pop (cost=1, node=1) → add. MST complete.
Total = 0+2+3+1 = 6
```

**Time:** O((V + E) log V) with a min-heap
**Best when:** dense graph, or graph given as adjacency list.

---

### Kruskal vs Prim

| | Kruskal | Prim |
|---|---|---|
| Input | Edge list | Adjacency list |
| Approach | Sort all edges, pick cheapest safe edge | Grow from one node, always expand cheapest border |
| Best for | Sparse graphs | Dense graphs |
| Data structure | Union-Find | Min-heap |
| Time | O(E log E) | O((V+E) log V) |

---

## Union-Find (Disjoint Set Union)

Use for: connectivity queries, cycle detection in undirected graphs, Kruskal's MST.

> LeetCode: [547. Number of Provinces](https://leetcode.com/problems/number-of-provinces/), [684. Redundant Connection](https://leetcode.com/problems/redundant-connection/)

```csharp
class UnionFind
{
    int[] parent, rank;

    public UnionFind(int n)
    {
        parent = Enumerable.Range(0, n).ToArray();
        rank = new int[n];
    }

    public int Find(int x) =>
        parent[x] == x ? x : parent[x] = Find(parent[x]); // path compression

    public bool Union(int x, int y)
    {
        int px = Find(x), py = Find(y);
        if (px == py) return false; // already connected → adding edge creates cycle
        if (rank[px] < rank[py]) (px, py) = (py, px);
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
        return true;
    }

    public bool Connected(int x, int y) => Find(x) == Find(y);
}
```

---

## Grid as Graph

Grids are implicit graphs. No need to build adjacency lists — just explore neighbors inline.

> LeetCode: [200. Number of Islands](https://leetcode.com/problems/number-of-islands/), [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/), [542. 01 Matrix](https://leetcode.com/problems/01-matrix/)

```csharp
// 4-directional (most common)
int[][] dirs = [[0,1],[0,-1],[1,0],[-1,0]];

// 8-directional (include diagonals)
int[][] dirs8 = [[0,1],[0,-1],[1,0],[-1,0],[1,1],[1,-1],[-1,1],[-1,-1]];

bool InBounds(int r, int c, int rows, int cols) =>
    r >= 0 && r < rows && c >= 0 && c < cols;
```

### BFS on grid
```csharp
void BFS(int[][] grid, int startR, int startC)
{
    int rows = grid.Length, cols = grid[0].Length;
    bool[,] visited = new bool[rows, cols];
    Queue<(int r, int c)> q = new();
    q.Enqueue((startR, startC));
    visited[startR, startC] = true;

    while (q.Count > 0)
    {
        var (r, c) = q.Dequeue();
        foreach (var d in dirs)
        {
            int nr = r + d[0], nc = c + d[1];
            if (InBounds(nr, nc, rows, cols) && !visited[nr, nc] && grid[nr][nc] != 0)
            {
                visited[nr, nc] = true;
                q.Enqueue((nr, nc));
            }
        }
    }
}
```

### DFS on grid
```csharp
void DFS(int[][] grid, int r, int c, bool[,] visited)
{
    visited[r, c] = true;
    foreach (var d in dirs)
    {
        int nr = r + d[0], nc = c + d[1];
        if (InBounds(nr, nc, grid.Length, grid[0].Length) && !visited[nr, nc] && grid[nr][nc] != 0)
            DFS(grid, nr, nc, visited);
    }
}
```

### Visited-via-mutation (avoid separate visited array)
Mark cells as visited by overwriting the grid value. Only works when you won't need the original grid values later.
```csharp
void DFS(int[][] grid, int r, int c)
{
    if (!InBounds(r, c, grid.Length, grid[0].Length) || grid[r][c] != 1) return;
    grid[r][c] = 0; // mark visited by changing the value
    foreach (var d in dirs) DFS(grid, r + d[0], c + d[1]);
}
```

---

## Common Problem Patterns

### Connected Components

> LeetCode: [323. Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/), [200. Number of Islands](https://leetcode.com/problems/number-of-islands/)

```csharp
int CountComponents(int n, Dictionary<int, List<int>> adj)
{
    bool[] visited = new bool[n];
    int count = 0;
    for (int i = 0; i < n; i++)
        if (!visited[i]) { DFS(i, visited, adj); count++; }
    return count;
}

void DFS(int node, bool[] visited, Dictionary<int, List<int>> adj)
{
    visited[node] = true;
    foreach (int nei in adj[node])
        if (!visited[nei]) DFS(nei, visited, adj);
}
```

### Bipartite Check (2-Coloring)

> LeetCode: [785. Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/)

```csharp
bool IsBipartite(int n, Dictionary<int, List<int>> adj)
{
    int[] color = new int[n]; // 0 = uncolored
    for (int i = 0; i < n; i++)
    {
        if (color[i] != 0) continue;
        Queue<int> q = new();
        q.Enqueue(i);
        color[i] = 1;
        while (q.Count > 0)
        {
            int node = q.Dequeue();
            foreach (int nei in adj[node])
            {
                if (color[nei] == 0) { color[nei] = -color[node]; q.Enqueue(nei); }
                else if (color[nei] == color[node]) return false;
            }
        }
    }
    return true;
}
```

### Multi-Source BFS
Useful when you have multiple starting points simultaneously.

> LeetCode: [542. 01 Matrix](https://leetcode.com/problems/01-matrix/), [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/)

```csharp
// Enqueue ALL sources first, each at distance 0
Queue<(int r, int c)> q = new();
foreach (var source in sources)
{
    q.Enqueue(source);
    dist[source.r][source.c] = 0;
}
// Then run standard BFS — all sources expand simultaneously
```

---

## Decision Checklist

```
Is it a path/connectivity problem?
  └─ Unweighted shortest path?        → BFS
  └─ Weighted shortest path?
      └─ No negative weights?         → Dijkstra
      └─ Negative weights?            → Bellman-Ford
      └─ All-pairs?                   → Floyd-Warshall
  └─ Just reachable?                  → DFS or BFS
  └─ Minimum spanning tree?           → Kruskal (sort edges + Union-Find)

Is it an ordering problem?
  └─ Dependencies / prerequisites?    → Topological sort (Kahn's)

Is it a grouping problem?
  └─ Find clusters/islands?           → DFS/BFS connected components
  └─ Dynamic connectivity?            → Union-Find

Does it involve a grid?
  └─ Treat cells as nodes, dirs as edges → BFS/DFS inline

Does it have cycles?
  └─ Directed?                        → DFS 3-color or Kahn's (order.Count != n)
  └─ Undirected?                      → DFS with parent / Union-Find

Multiple starting points?             → Multi-source BFS
```

---

## Complexity Reference

| | Time | Space |
|---|---|---|
| BFS / DFS | O(V + E) | O(V) |
| Dijkstra | O((V + E) log V) | O(V) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Union-Find (amortized) | O(α(n)) per op | O(V) |
| Topological Sort | O(V + E) | O(V) |
---
tags: [dsa, graphs, shortest-path, dijkstra, bellman-ford, floyd-warshall]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_BFS_DFS]]", "[[11_Graphs_Union_Find]]", "[[../00_Complexity_Cheatsheet]]"]
---

# Graphs -- Shortest Path Algorithms

*<- [[11_Graphs_BFS_DFS|BFS & DFS]] · [[11_Graphs_Union_Find|Union-Find ->]]*

---

## 1. Dijkstra's Algorithm (Single-Source, Non-Negative Weights)

**Why**: Find the shortest path from a source node to all other nodes in a weighted graph.
**Core Idea**: Greedily expand from the node with the current minimum distance. Uses a **Min-Priority Queue** to pick the next closest node.
- **Complexity**: $O((E + V) \log V)$ (see [[../00_Complexity_Cheatsheet#Big-O]] for heap complexity details).
- **Constraint**: Cannot handle negative weight edges (can cause infinite loops or incorrect greedy choices).

```
    Graph Structure:
       (4)
      0 ──► 1 ──► 3
      │    ▲     (3)
  (1) │ (2)│
      ▼   /
      2 ──
```

### Dijkstra Step-by-Step Execution Trace

#### Initialization:
- `dist` array: `[0, inf, inf, inf]`
- `pq` (Min-Heap): `[{0, 0}]` (stores `{distance, node}`)

#### Step 1: Pop `{0, 0}` (u = 0, dist = 0)
- Neighbor `1`: `dist[0] + weight(0->1) = 0 + 4 = 4`. Since $4 < \infty$, update `dist[1] = 4`, push `{4, 1}`.
- Neighbor `2`: `dist[0] + weight(0->2) = 0 + 1 = 1`. Since $1 < \infty$, update `dist[2] = 1`, push `{1, 2}`.
- **State**:
  - `dist` = `[0, 4, 1, inf]`
  - `pq` = `[{1, 2}, {4, 1}]`

#### Step 2: Pop `{1, 2}` (u = 2, dist = 1)
- Neighbor `1`: `dist[2] + weight(2->1) = 1 + 2 = 3`. Since $3 < 4$ (previous distance to node 1), update `dist[1] = 3`, push `{3, 1}`.
- **State**:
  - `dist` = `[0, 3, 1, inf]`
  - `pq` = `[{3, 1}, {4, 1}]` (node 1 now has two entries in heap; the stale `{4, 1}` will be ignored later)

#### Step 3: Pop `{3, 1}` (u = 1, dist = 3)
- Neighbor `3`: `dist[1] + weight(1->3) = 3 + 3 = 6`. Since $6 < \infty$, update `dist[3] = 6`, push `{6, 3}`.
- **State**:
  - `dist` = `[0, 3, 1, 6]`
  - `pq` = `[{4, 1}, {6, 3}]`

#### Step 4: Pop `{4, 1}` (u = 1, dist = 4)
- **Stale check**: `d = 4` is greater than `dist[1] = 3`. This is a stale entry! Skip processing.
- **State**:
  - `dist` = `[0, 3, 1, 6]`
  - `pq` = `[{6, 3}]`

#### Step 5: Pop `{6, 3}` (u = 3, dist = 6)
- Node 3 has no outgoing edges.
- **State**:
  - `dist` = `[0, 3, 1, 6]`
  - `pq` = `[]` (Termination)

---

```cpp
#include <vector>
#include <queue>

// Dijkstra single-source shortest path
vector<int> dijkstra(int source, int n, const vector<vector<pair<int, int>>>& adj) {
    vector<int> dist(n, 1e9); // 1e9 represents infinity
    dist[source] = 0;

    // min-heap storing: {distance, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, source});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        // Stale path check (if we already found a shorter path to u)
        if (d > dist[u]) continue;

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;

            // Relaxation Step
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

---

## 2. Bellman-Ford's Algorithm (Single-Source, Negative Weights Permitted)

**Why**: Dijkstra fails when there are negative weights. Bellman-Ford relaxes all edges $V-1$ times.
**Core Idea -- Relaxation**: A path between two nodes in a graph with $V$ vertices can contain at most $V-1$ edges. Thus, relaxing all edges $V-1$ times guarantees we find the shortest paths.
- **Negative Cycles**: Running a $V$-th relaxation round will continue decreasing distances if a negative weight cycle exists.
- **Complexity**: $O(V \times E)$.

```cpp
#include <vector>

struct Edge {
    int u, v, weight;
};

// Returns shortest path distances, or empty vector if negative cycle detected
vector<int> bellmanFord(int source, int n, const vector<Edge>& edges) {
    vector<int> dist(n, 1e9);
    dist[source] = 0;

    // Relax all edges V-1 times
    for (int i = 0; i < n - 1; i++) {
        for (auto& edge : edges) {
            if (dist[edge.u] != 1e9 && dist[edge.u] + edge.weight < dist[edge.v]) {
                dist[edge.v] = dist[edge.u] + edge.weight;
            }
        }
    }

    // V-th iteration to check for negative cycles
    for (auto& edge : edges) {
        if (dist[edge.u] != 1e9 && dist[edge.u] + edge.weight < dist[edge.v]) {
            return {}; // negative cycle detected!
        }
    }
    return dist;
}
```

---

## 3. Floyd-Warshall's Algorithm (All-Pairs Shortest Path)

**Why**: Find shortest path distances between **every pair of nodes** in the graph.
**Core Idea**: Dynamic Programming. For every pair of nodes $(i, j)$, check if we can get a shorter path by going through an intermediate node $k$.
$$\text{dist}[i][j] = \min(\text{dist}[i][j], \text{dist}[i][k] + \text{dist}[k][j])$$
- **Complexity**: $O(V^3)$ time, $O(V^2)$ space.

```cpp
#include <vector>

vector<vector<int>> floydWarshall(int n, const vector<vector<int>>& matrix) {
    // Initialize dist matrix with input adj matrix (1e9 for no edge, 0 for self)
    vector<vector<int>> dist = matrix;

    for (int k = 0; k < n; k++) { // Try every node k as helper intermediate
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (dist[i][k] != 1e9 && dist[k][j] != 1e9) {
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
                }
            }
        }
    }
    return dist;
}
```

---

## Summary Matrix

| Algorithm | Type | Time Complexity | Negative Weights? | Best For |
|---|---|---|---|---|
| **Dijkstra** | Greedy | $O((E + V) \log V)$ | No | Sparse/Large Graphs, positive weights |
| **Bellman-Ford** | Relaxation | $O(V \times E)$ | Yes | Negative weights, detecting negative cycles |
| **Floyd-Warshall** | Dynamic Programming | $O(V^3)$ | Yes (no cycles) | Small Graphs, All-Pairs query |

---

*<- [[11_Graphs_BFS_DFS|BFS & DFS]] · [[11_Graphs_Union_Find|Union-Find ->]]*

---
tags: [dsa, graphs, network-flow, max-flow, edmonds-karp, ford-fulkerson]
links: ["[[12_Advanced_Graphs_Index]]", "[[12_Advanced_Graphs_MST]]", "[[12_Advanced_Graphs_SCC]]"]
---

# Advanced Graphs -- Network Flow & Max-Flow

*<- [[12_Advanced_Graphs_MST|MST]] · [[12_Advanced_Graphs_SCC|SCC & Tarjan's ->]]*

---

## Core Network Flow Concepts

A **Flow Network** is a directed graph where each edge has a **capacity** (maximum flow it can carry) and a **flow** (current amount of traffic on the edge). We want to route the maximum possible flow from a **Source ($S$)** to a **Sink ($T$)**.

```
    Capacity Graph (edge label: capacity)
             [10]
          ┌───► 1 ───┐
          │          │ [10]
        S │      [2] │      T
          │ ──► 2 ──►│
          └───►   ───┘
             [5]     [10]
```

### The Residual Graph & Reverse Edges (CRITICAL)
When we route flow along a path, we reduce the capacity in the forward direction. Simultaneously, we add **residual capacity in the reverse direction**.
- **Why reverse edges?** They act as "undo buttons". They allow subsequent path explorations to redirect (push back) previously routed flow to find a globally optimal configuration.

```
Routing 5 units of flow along 0 -> 1:
Forward capacity (0 -> 1) goes from 10 to 5.
Reverse capacity (1 -> 0) goes from 0 to 5.
```

---

## 1. Edmonds-Karp Algorithm (BFS-based Ford-Fulkerson)

**Core Idea**: Repeatedly find the shortest augmenting path (using BFS) from $S$ to $T$ in the residual graph, find the bottleneck capacity, and update the residual capacities.
- **Complexity**: $O(V \times E^2)$ time.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

// BFS to find an augmenting path from source (s) to sink (t)
// parent array stores the path to reconstruct it later
bool bfs(int s, int t, const vector<vector<int>>& residualCapacity, vector<int>& parent) {
    int n = residualCapacity.size();
    vector<bool> visited(n, false);
    queue<int> q;

    q.push(s);
    visited[s] = true;
    parent[s] = -1;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v = 0; v < n; v++) {
            // Traverse if there is residual capacity and node is unvisited
            if (residualCapacity[u][v] > 0 && !visited[v]) {
                parent[v] = u;
                visited[v] = true;
                if (v == t) return true; // reached the sink!
                q.push(v);
            }
        }
    }
    return false;
}

int edmondsKarp(int source, int sink, vector<vector<int>>& capacity) {
    int n = capacity.size();
    // Copy capacities to build residual graph
    vector<vector<int>> residualCapacity = capacity;
    vector<int> parent(n);
    int maxFlow = 0;

    // While an augmenting path from source to sink exists
    while (bfs(source, sink, residualCapacity, parent)) {
        // Step 1: Find bottleneck capacity along the augmenting path
        int bottleneck = 1e9;
        for (int v = sink; v != source; v = parent[v]) {
            int u = parent[v];
            bottleneck = min(bottleneck, residualCapacity[u][v]);
        }

        // Step 2: Update residual capacities (subtract forward, add reverse)
        for (int v = sink; v != source; v = parent[v]) {
            int u = parent[v];
            residualCapacity[u][v] -= bottleneck; // forward path capacity reduced
            residualCapacity[v][u] += bottleneck; // reverse path capacity increased (the undo route)
        }

        maxFlow += bottleneck;
    }
    return maxFlow;
}
```

---

## 2. Max-Flow Min-Cut Theorem

**The Theorem**: The maximum flow routing from $S$ to $T$ is exactly equal to the capacity of the **minimum cut** separating $S$ from $T$.
- A **Cut** divides vertices into two sets: $A$ (containing $S$) and $B$ (containing $T$).
- The capacity of a cut is the sum of capacities of all edges going from $A \to B$.

```
Set A: {S, 1}           Set B: {2, T}
Cut Edges (A -> B): S -> 2, 1 -> 2, 1 -> T
Sum of capacities of these edges is the Cut Capacity.
```

### How to Find the Min-Cut Edges after Max-Flow
1. Run Edmonds-Karp/Ford-Fulkerson to compute max flow.
2. Find all vertices reachable from $S$ in the final residual graph using a simple BFS/DFS. Let this reachable set be $A$.
3. All other vertices belong to set $B$.
4. Any edge $(u, v)$ in the original graph where $u \in A$ and $v \in B$ is a **min-cut edge** (and is fully saturated: flow = capacity).

---

*<- [[12_Advanced_Graphs_MST|MST]] · [[12_Advanced_Graphs_SCC|SCC & Tarjan's ->]]*

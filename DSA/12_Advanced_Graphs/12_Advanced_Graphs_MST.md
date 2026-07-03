---
tags: [dsa, graphs, mst, kruskal, prim, dsu]
links: ["[[12_Advanced_Graphs_Index]]", "[[12_Advanced_Graphs_Network_Flow]]", "[[../11_Graphs/11_Graphs_Union_Find]]", "[[../00_Complexity_Cheatsheet]]"]
---

# Advanced Graphs -- Minimum Spanning Tree (MST)

*<- [[12_Advanced_Graphs_Index|Index]] · [[12_Advanced_Graphs_Network_Flow|Network Flow ->]]*

---

## What is a Minimum Spanning Tree (MST)?

A Spanning Tree is a subset of edges of a connected, undirected graph that connects all vertices without any cycles. A **Minimum Spanning Tree (MST)** is a spanning tree with the **minimum total edge weight**.
- A graph with $V$ vertices has exactly $V-1$ edges in its MST.

```
       Weighted Graph                       Minimum Spanning Tree (MST)
           (1)                                         (1)
        0 ───── 1                                   0 ───── 1
        │ \     │                                          /     │
    (3) │  \ (4)│ (2)                                     / (4)  │ (2)
        │   \   │                                        /       │
        2 ───── 3                                   2       3
           (5)                                   Total Weight = 1 + 4 + 2 = 7
```

---

## 1. Kruskal's Algorithm (Edge-Centric Greedy)

**Core Idea**: Sort-and-Union.
- Sort all edges in ascending order of their weights.
- Initialize a Disjoint Set Union (DSU) data structure (see [[../11_Graphs/11_Graphs_Union_Find|Union-Find]]).
- Iterate through the sorted edges. For each edge $(u, v)$ with weight $w$:
  - If $u$ and $v$ are in different sets, add the edge to the MST and union the sets.
  - If they are already in the same set, discard the edge (it would form a cycle).
- Stop when we have added $V-1$ edges.
- **Complexity**: $O(E \log E)$ for sorting. DSU operations take near $O(1)$ amortized time. See [[../00_Complexity_Cheatsheet#Big-O]] for DSU complexity details.

```cpp
#include <vector>
#include <algorithm>

struct Edge {
    int u, v, weight;
    bool operator<(const Edge& other) const {
        return weight < other.weight;
    }
};

// DSU helper struct inside Kruskal
struct KruskalDSU {
    vector<int> parent, size;
    KruskalDSU(int n) {
        parent.resize(n);
        for (int i = 0; i < n; i++) parent[i] = i;
        size.resize(n, 1);
    }
    int find(int i) {
        if (parent[i] == i) return i;
        return parent[i] = find(parent[i]);
    }
    bool unionBySize(int u, int v) {
        int rootU = find(u);
        int rootV = find(v);
        if (rootU == rootV) return false;
        if (size[rootU] < size[rootV]) {
            parent[rootU] = rootV;
            size[rootV] += size[rootU];
        } else {
            parent[rootV] = rootU;
            size[rootU] += size[rootV];
        }
        return true;
    }
};

int kruskalMST(int n, vector<Edge>& edges) {
    sort(edges.begin(), edges.end()); // Step 1: Sort edges by weight
    KruskalDSU dsu(n);

    int mstWeight = 0;
    int edgesCount = 0;

    for (auto& edge : edges) {
        if (dsu.unionBySize(edge.u, edge.v)) { // Step 2: Avoid cycle and merge
            mstWeight += edge.weight;
            edgesCount++;
            if (edgesCount == n - 1) break; // found V-1 edges, MST is complete
        }
    }
    return mstWeight;
}
```

---

## 2. Prim's Algorithm (Vertex-Centric Greedy)

**Core Idea**: Grow the MST outward from a start vertex.
- Maintain a `visited` array to track vertices already in the MST.
- Maintain a **Min-Priority Queue** of edges connecting the MST to unvisited vertices.
- At each step:
  - Pop the edge with the minimum weight from the queue.
  - If the target vertex is already visited, discard the edge.
  - Otherwise, mark the vertex as visited, add the edge weight to the MST, and push all edges from this new vertex to unvisited neighbors into the queue.
- **Complexity**: $O(E \log V)$ using binary heap.

```cpp
#include <vector>
#include <queue>

// Prim's algorithm for MST weight
int primMST(int n, const vector<vector<pair<int, int>>>& adj) {
    vector<bool> visited(n, false);
    // Min-heap storing: {weight, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    
    pq.push({0, 0}); // start with weight 0 at node 0
    int mstWeight = 0;
    int visitedCount = 0;

    while (!pq.empty()) {
        auto [w, u] = pq.top();
        pq.pop();

        if (visited[u]) continue; // already part of the MST

        visited[u] = true;
        mstWeight += w;
        visitedCount++;

        if (visitedCount == n) break; // all vertices added

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;
            if (!visited[v]) {
                pq.push({weight, v});
            }
        }
    }
    return mstWeight;
}
```

---

## Comparison Summary

| Algorithm | Approach | Time Complexity | Best For |
|---|---|---|---|
| **Kruskal's** | Edge-centric: sorts edges and adds them if no cycle. | $O(E \log E)$ | Sparse graphs (fewer edges). Easier to implement with DSU. |
| **Prim's** | Vertex-centric: grows tree outward from a source. | $O(E \log V)$ | Dense graphs (many edges). |

---

*<- [[12_Advanced_Graphs_Index|Index]] · [[12_Advanced_Graphs_Network_Flow|Network Flow ->]]*

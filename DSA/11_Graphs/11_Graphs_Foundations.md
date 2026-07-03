---
tags: [dsa, graphs, foundations]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_BFS_DFS]]"]
---

# Graphs -- Foundations & Representation

*<- [[11_Graphs_Index|Index]] · [[11_Graphs_BFS_DFS|BFS & DFS ->]]*

---

## What is a Graph?

A graph consists of a set of vertices (nodes) $V$ and a set of edges $E$ connecting them.
- **Undirected**: Edges are bidirectional. If there is an edge between $u$ and $v$, you can go both ways.
- **Directed (Digraph)**: Edges have arrows. Edge $u \to v$ only permits traversal from $u$ to $v$.
- **Weighted**: Edges have numerical weights (costs, distances).
- **Unweighted**: All edges are equal (effectively weight = 1).

```
    Undirected Graph                       Directed Graph
       0 ─── 1                               0 ───► 1
       │    /                                │    ▲
       │   /                                 │   /
       │  /                                  ▼  /
       2 ─── 3                               2 ───► 3
```

---

## Graph Representations

### 1. Adjacency Matrix
A 2D array of size $V \times V$ where `matrix[u][v] = 1` (or edge weight) indicates an edge between $u$ and $v$.

```
    Graph: 0-1, 0-2                       Matrix Layout:
             \ /                          Index: 0  1  2
              3                               0 [0, 1, 1, 0]
                                              1 [1, 0, 0, 1]
                                              2 [1, 0, 0, 1]
                                              3 [0, 1, 1, 0]
```

- **Pros**: $O(1)$ lookup to check if edge $(u, v)$ exists.
- **Cons**: $O(V^2)$ space even if the graph is sparse (few edges). Finding all neighbors of $u$ takes $O(V)$ time.

```cpp
#include <vector>

// Build adjacency matrix from edge list
vector<vector<int>> buildMatrix(int n, const vector<vector<int>>& edges) {
    vector<vector<int>> matrix(n, vector<int>(n, 0));
    for (auto& edge : edges) {
        int u = edge[0];
        int v = edge[1];
        matrix[u][v] = 1;
        matrix[v][u] = 1; // for undirected
    }
    return matrix;
}
```

---

### 2. Adjacency List
An array of lists/vectors of size $V$, where `adj[u]` contains all adjacent neighbors of $u$.

```
    Graph: 0-1, 0-2                       List Layout:
             \ /                          0 -> [1, 2]
              3                           1 -> [0, 3]
                                          2 -> [0, 3]
                                          3 -> [1, 2]
```

- **Pros**: Space efficient ($O(V + E)$). Iterating over all neighbors of $u$ takes $O(\text{degree}(u))$ time.
- **Cons**: Checking if edge $(u, v)$ exists takes $O(\text{degree}(u))$ time.

```cpp
#include <vector>

// Build adjacency list for unweighted graph
vector<vector<int>> buildAdjList(int n, const vector<vector<int>>& edges, bool isDirected = false) {
    vector<vector<int>> adj(n);
    for (auto& edge : edges) {
        int u = edge[0];
        int v = edge[1];
        adj[u].push_back(v);
        if (!isDirected) {
            adj[v].push_back(u);
        }
    }
    return adj;
}

// Build adjacency list for weighted graph
vector<vector<pair<int, int>>> buildWeightedAdjList(int n, const vector<vector<int>>& edges, bool isDirected = false) {
    // pair elements: {neighbor, weight}
    vector<vector<pair<int, int>>> adj(n);
    for (auto& edge : edges) {
        int u = edge[0];
        int v = edge[1];
        int w = edge[2];
        adj[u].push_back({v, w});
        if (!isDirected) {
            adj[v].push_back({u, w});
        }
    }
    return adj;
}
```

---

### 3. Edge List
A simple list of all edges in the graph, usually represented as a vector of vectors or pairs.
- Useful for algorithms that sort all edges (e.g. Kruskal's MST algorithm, Bellman-Ford).

```cpp
#include <vector>

struct Edge {
    int u;
    int v;
    int weight;
};
vector<Edge> edgeList;
```

---

*<- [[11_Graphs_Index|Index]] · [[11_Graphs_BFS_DFS|BFS & DFS ->]]*

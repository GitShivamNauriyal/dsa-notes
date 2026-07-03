---
tags: [dsa, graphs, union-find, dsu]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_Shortest_Path]]", "[[11_Graphs_Topological_Sort]]", "[[../00_Complexity_Cheatsheet]]"]
---

# Disjoint Set Union (DSU) -- Union-Find

*<- [[11_Graphs_Shortest_Path|Shortest Path]] · [[11_Graphs_Topological_Sort|Topological Sort ->]]*

---

## What is Union-Find (DSU)?

DSU is a data structure that tracks elements split into several disjoint (non-overlapping) sets. It is highly optimized to answer two queries:
1. **Find**: What set does element $i$ belong to? (Returns the representative element of the set).
2. **Union**: Merge the set containing $i$ with the set containing $j$.

### Optimization 1: Path Compression
When searching for the representative element of a node, point the node's parent pointer directly to the set representative. This flattens the tree structure and keeps subsequent lookups at $O(1)$.

```
   Before Path Compression (find(3))             After Path Compression (find(3))
               0 (root)                                      0 (root)
              /                                            / | \
             1                                            1  2  3
            /
           2
          /
         3
```

### Optimization 2: Union by Rank / Size
Always attach the smaller tree (lower rank/size) under the root of the larger tree. This prevents the tree depth from growing unnecessarily.
- **Complexity**: Combining both optimizations gives an amortized time complexity of $O(\alpha(n))$ per operation, where $\alpha(n)$ is the Inverse Ackermann function (effectively $O(1)$ for all practical values of $n$). See [[../00_Complexity_Cheatsheet#Big-O]] for amortized complexity details.

---

## DSU Class Implementation

```cpp
#include <vector>
#include <numeric>

class DisjointSet {
    vector<int> parent;
    vector<int> rank;
    vector<int> size;

public:
    DisjointSet(int n) {
        parent.resize(n);
        iota(parent.begin(), parent.end(), 0); // initially, every node is its own parent
        rank.resize(n, 0);
        size.resize(n, 1);
    }

    // Find with Path Compression
    int find(int i) {
        if (parent[i] == i) return i;
        return parent[i] = find(parent[i]); // cache root directly as parent
    }

    // Union by Rank
    void unionByRank(int u, int v) {
        int rootU = find(u);
        int rootV = find(v);
        if (rootU == rootV) return; // already in the same set

        if (rank[rootU] < rank[rootV]) {
            parent[rootU] = rootV;
        } else if (rank[rootU] > rank[rootV]) {
            parent[rootV] = rootU;
        } else {
            parent[rootV] = rootU;
            rank[rootU]++;
        }
    }

    // Union by Size
    void unionBySize(int u, int v) {
        int rootU = find(u);
        int rootV = find(v);
        if (rootU == rootV) return;

        if (size[rootU] < size[rootV]) {
            parent[rootU] = rootV;
            size[rootV] += size[rootU];
        } else {
            parent[rootV] = rootU;
            size[rootU] += size[rootV];
        }
    }

    // Check if u and v belong to the same set
    bool isConnected(int u, int v) {
        return find(u) == find(v);
    }
};
```

---

## Primary Applications

### 1. Cycle Detection in Undirected Graphs
Iterate through the edge list. For each edge $(u, v)$:
- Check if `find(u) == find(v)`.
- If true, they are already connected, and adding this edge creates a cycle!
- If false, call `union(u, v)`.

```cpp
bool detectCycleDSU(int n, const vector<pair<int, int>>& edges) {
    DisjointSet dsu(n);
    for (auto& edge : edges) {
        int u = edge.first;
        int v = edge.second;
        if (dsu.isConnected(u, v)) {
            return true; // cycle detected!
        }
        dsu.unionBySize(u, v);
    }
    return false;
}
```

### 2. Number of Connected Components
- Start with count = $n$.
- Every time a successful `union` operation occurs (merging two disconnected components), decrement count by 1.
- `count` at the end is the number of connected components.

---

*<- [[11_Graphs_Shortest_Path|Shortest Path]] · [[11_Graphs_Topological_Sort|Topological Sort ->]]*

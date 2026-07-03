---
tags: [dsa, graphs, topological-sort, kahns-algorithm, bfs, dfs]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_Union_Find]]", "[[11_Graphs_Problems_and_Exercises]]"]
---

# Graphs -- Topological Sort

*<- [[11_Graphs_Union_Find|Union-Find]] · [[11_Graphs_Problems_and_Exercises|Problems ->]]*

---

## What is Topological Sort?

Topological sort is a linear ordering of vertices in a **Directed Acyclic Graph (DAG)** such that for every directed edge $u \to v$, vertex $u$ comes before $v$ in the ordering.

```
Directed Edge: 5 ──► 0, 5 ──► 2, 4 ──► 0, 4 ──► 1, 2 ──► 3, 3 ──► 1

      5      4
     / \    / \
    ▼   ▼  ▼   ▼
    0    2     1
          \   ▲
           ▼ /
            3

One Valid Topological Sort: 5, 4, 2, 3, 1, 0
(all dependency arrows point left-to-right)
```

**Constraint**: Topological sort is **impossible if there is a cycle** in the graph. Hence, it is also a popular method for cycle detection in directed graphs.

---

## 1. Kahn's Algorithm (BFS-Based)

**Core Idea**: Indegree tracking.
- Calculate the **indegree** (number of incoming edges) for every vertex.
- Enqueue all vertices with `indegree == 0` (nodes with no dependencies).
- Pop a node, append it to the topological order, and decrement the indegree of all its neighbors.
- If a neighbor's indegree becomes 0, enqueue it.
- If the final topological order size is less than $V$, the graph contains a cycle!

```cpp
#include <vector>
#include <queue>

// Returns topological sort order of a DAG. Returns empty vector if cycle exists.
vector<int> kahnsTopologicalSort(int n, const vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++) {
        for (int v : adj[u]) {
            indegree[v]++;
        }
    }

    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) q.push(i);
    }

    vector<int> order;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        order.push_back(u);

        for (int v : adj[u]) {
            indegree[v]--;
            if (indegree[v] == 0) {
                q.push(v);
            }
        }
    }

    if ((int)order.size() != n) {
        return {}; // cycle detected, topological sort impossible!
    }
    return order;
}
// Time: O(V + E)
// Space: O(V)
```

---

## 2. DFS-Based Topological Sort

**Core Idea**: Post-order stack insertion.
- Run DFS on unvisited nodes.
- A node is pushed to a stack only **after** all of its neighbors have been completely visited (on backtracking).
- The node that is finished last will sit at the top of the stack.
- Pop from the stack to retrieve the topological order.
- To detect cycles, combine with the `pathVisited` tracking from cycle detection.

```cpp
#include <vector>
#include <stack>
#include <algorithm>

void dfs(int node, const vector<vector<int>>& adj, vector<bool>& visited, stack<int>& st) {
    visited[node] = true;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfs(neighbor, adj, visited, st);
        }
    }
    st.push(node); // post-order stack insertion
}

vector<int> dfsTopologicalSort(int n, const vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    stack<int> st;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(i, adj, visited, st);
        }
    }

    vector<int> order;
    while (!st.empty()) {
        order.push_back(st.top());
        st.pop();
    }
    return order;
}
// Time: O(V + E)
// Space: O(V)
```

---

*<- [[11_Graphs_Union_Find|Union-Find]] · [[11_Graphs_Problems_and_Exercises|Problems ->]]*

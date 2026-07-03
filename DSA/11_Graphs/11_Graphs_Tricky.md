---
tags: [dsa, graphs, tricky, hard, multi-source-bfs, bipartite, scc, bridges]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_Problems_and_Exercises]]", "[[../12_Advanced_Graphs/12_Advanced_Graphs_Index]]"]
---

# Graphs -- Tricky & Higher-Order

*<- [[11_Graphs_Problems_and_Exercises|Problems]] · [[../12_Advanced_Graphs/12_Advanced_Graphs_Index|Advanced Graphs ->]]*

---

## 1. Multi-Source BFS (LC 542)

**Why Tricky**: Standard BFS starts at **one** source. Multi-Source BFS starts by enqueuing **all source nodes simultaneously** with distance 0. This propagates distances in parallel, ensuring we find the shortest path from *any* source to all destinations in a single $O(V + E)$ pass.

```
Grid:                     Queue Init:                     Propagated Levels:
  1  1  1                   Push all '0's                   1  1  1
  1  0  1                   Queue: [(1,1)]                  1  0  1
  1  1  1                   Pop (1,1) -> push neighbors     1  1  1
                            with dist = 1
```

```cpp
#include <vector>
#include <queue>

// LeetCode 542 — 01 Matrix
vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
    int rows = mat.size(), cols = mat[0].size();
    vector<vector<int>> dist(rows, vector<int>(cols, -1));
    queue<pair<int, int>> q;

    // Initialize: enqueue all '0's and set their distance to 0
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (mat[r][c] == 0) {
                dist[r][c] = 0;
                q.push({r, c});
            }
        }
    }

    int dirs[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};
    while (!q.empty()) {
        auto [r, c] = q.front();
        q.pop();

        for (auto& d : dirs) {
            int nr = r + d[0];
            int nc = c + d[1];

            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && dist[nr][nc] == -1) {
                dist[nr][nc] = dist[r][c] + 1; // update distance
                q.push({nr, nc});
            }
        }
    }
    return dist;
}
// Time: O(M * N) — visit each cell once
// Space: O(M * N) for queue/dist matrix
```

---

## 2. Bipartite Graph Check (BFS 2-Coloring)

**Why Tricky**: A graph is bipartite if its vertices can be divided into two disjoint sets $U$ and $V$ such that every edge connects a vertex in $U$ to one in $V$. This is equivalent to checking if the graph can be **colored with 2 colors** such that no two adjacent nodes share the same color.
- A graph is bipartite if and only if it **does not contain an odd-length cycle**.

```cpp
#include <vector>
#include <queue>

// LeetCode 785 — Is Graph Bipartite?
bool checkBipartiteBFS(int start, const vector<vector<int>>& adj, vector<int>& color) {
    queue<int> q;
    q.push(start);
    color[start] = 0; // color 0

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v : adj[u]) {
            if (color[v] == -1) {
                color[v] = 1 - color[u]; // color with opposite color (1 if u is 0, 0 if u is 1)
                q.push(v);
            } else if (color[v] == color[u]) {
                // Neighbor shares same color -> Not Bipartite!
                return false;
            }
        }
    }
    return true;
}

bool isBipartite(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> color(n, -1); // -1 means uncolored

    for (int i = 0; i < n; i++) {
        if (color[i] == -1) {
            if (!checkBipartiteBFS(i, graph, color)) return false;
        }
    }
    return true;
}
```

---

## 3. Strongly Connected Components (Kosaraju's Algorithm)

**Why Tricky**: A Strongly Connected Component (SCC) is a maximal subgraph of a directed graph where every vertex is reachable from every other vertex. 
- Kosaraju's uses two DFS passes and a **Graph Transpose** (reversing all edges) to find SCCs in $O(V + E)$ time.

### Why Transpose Works
Reversing edges keeps nodes within the same SCC connected (since reachability is bidirectional within an SCC), but prevents paths from escaping to other SCCs.

```
Graph: SCC_A ──► SCC_B                  Transpose: SCC_A ◄── SCC_B
If we run DFS in finish-order on Transpose, we resolve SCCs independently without leakage.
```

```cpp
#include <vector>
#include <stack>

void dfs1(int node, const vector<vector<int>>& adj, vector<bool>& visited, stack<int>& st) {
    visited[node] = true;
    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfs1(neighbor, adj, visited, st);
        }
    }
    st.push(node); // store finish times
}

void dfs2(int node, const vector<vector<int>>& adjT, vector<bool>& visited, vector<int>& component) {
    visited[node] = true;
    component.push_back(node);
    for (int neighbor : adjT[node]) {
        if (!visited[neighbor]) {
            dfs2(neighbor, adjT, visited, component);
        }
    }
}

vector<vector<int>> getSCCs(int n, const vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    stack<int> st;

    // Step 1: DFS to get vertices ordered by finish times
    for (int i = 0; i < n; i++) {
        if (!visited[i]) dfs1(i, adj, visited, st);
    }

    // Step 2: Transpose the graph (reverse all edges)
    vector<vector<int>> adjT(n);
    for (int u = 0; u < n; u++) {
        for (int v : adj[u]) {
            adjT[v].push_back(u); // reverse v <- u
        }
    }

    // Step 3: DFS on Transpose in stack order
    fill(visited.begin(), visited.end(), false);
    vector<vector<int>> sccs;

    while (!st.empty()) {
        int u = st.top();
        st.pop();

        if (!visited[u]) {
            vector<int> component;
            dfs2(u, adjT, visited, component);
            sccs.push_back(component);
        }
    }
    return sccs;
}
// Time: O(V + E) — Two DFS runs
// Space: O(V + E) for transpose graph and stack
```

---

## 4. Tarjan's Algorithm -- Bridges and Articulation Points

**Why Tricky**: Finding bridges (edges whose removal increases the number of connected components) or articulation points (vertices whose removal increases connected components) in $O(V + E)$ requires tracking node discovery times and back-edges.

- `tin[u]`: Discovery time of node $u$.
- `low[u]`: Lowest discovery time reachable from $u$ using at most one back-edge.
- **Bridge Condition**: For edge $u \to v$, if `low[v] > tin[u]`, there is no back-edge from $v$ or its descendants to $u$ or its ancestors. Thus, $u \to v$ is a bridge!

```cpp
#include <vector>

void dfsBridge(int u, int p, int& timer, const vector<vector<int>>& adj,
               vector<int>& tin, vector<int>& low, vector<vector<int>>& bridges) {
    tin[u] = low[u] = timer++;
    
    for (int v : adj[u]) {
        if (v == p) continue; // skip parent edge

        if (tin[v] != -1) {
            // v is already visited -> back-edge found!
            low[u] = min(low[u], tin[v]);
        } else {
            // v is unvisited child
            dfsBridge(v, u, timer, adj, tin, low, bridges);
            low[u] = min(low[u], low[v]); // pull low value up from child

            // Bridge Condition check
            if (low[v] > tin[u]) {
                bridges.push_back({u, v});
            }
        }
    }
}

vector<vector<int>> findBridges(int n, const vector<vector<int>>& adj) {
    vector<int> tin(n, -1);
    vector<int> low(n, -1);
    vector<vector<int>> bridges;
    int timer = 0;

    for (int i = 0; i < n; i++) {
        if (tin[i] == -1) {
            dfsBridge(i, -1, timer, adj, tin, low, bridges);
        }
    }
    return bridges;
}
// Time: O(V + E)
// Space: O(V)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Strongly Connected Components (Kosaraju's) | Medium | [GFG](https://www.geeksforgeeks.org/strongly-connected-components/) |
| 2 | Critical Connections in a Network (Bridges) | Hard | [LC 1192](https://leetcode.com/problems/critical-connections-in-a-network/) |
| 3 | Is Graph Bipartite? | Medium | [LC 785](https://leetcode.com/problems/is-graph-bipartite/) |
| 4 | Shortest Bridge | Medium | [LC 934](https://leetcode.com/problems/shortest-bridge/) |

---

*<- [[11_Graphs_Problems_and_Exercises|Problems]] · [[../12_Advanced_Graphs/12_Advanced_Graphs_Index|Advanced Graphs ->]]*

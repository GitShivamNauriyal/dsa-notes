---
tags: [dsa, graphs, advanced-graphs, tricky, hard, dijkstra, bridge]
links: ["[[12_Advanced_Graphs_Index]]", "[[12_Advanced_Graphs_Problems_and_Exercises]]", "[[../13_DP/00_Index]]"]
---

# Advanced Graphs -- Tricky & Higher-Order

*<- [[12_Advanced_Graphs_Problems_and_Exercises|Problems]] · [[../13_DP/00_Index|Dynamic Programming ->]]*

---

## 1. Swim in Rising Water (LC 778)

**Why Tricky**: You are given a 2D grid of elevations. You want to swim from top-left to bottom-right. The time it takes to enter any cell depends on the elevation. The cost of a path is defined not as the sum of edge weights, but as the **maximum elevation encountered along the path**.
- **Solution**: We can adapt **Dijkstra's Algorithm**. Instead of updating `dist[v] = dist[u] + weight`, we update the path cost to `dist[v] = max(dist[u], grid[v])`.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

// LeetCode 778 — Swim in Rising Water
int swimInWater(vector<vector<int>>& grid) {
    int n = grid.size();
    vector<vector<int>> dist(n, vector<int>(n, 1e9));
    vector<vector<bool>> visited(n, vector<bool>(n, false));

    // Min-heap: {max_elevation_so_far, {r, c}}
    using P = pair<int, pair<int, int>>;
    priority_queue<P, vector<P>, greater<P>> pq;

    dist[0][0] = grid[0][0];
    pq.push({grid[0][0], {0, 0}});

    int dirs[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};

    while (!pq.empty()) {
        auto [d, cell] = pq.top();
        auto [r, c] = cell;
        pq.pop();

        if (r == n - 1 && c == n - 1) return d; // reached destination

        if (visited[r][c]) continue;
        visited[r][c] = true;

        for (auto& dir : dirs) {
            int nr = r + dir[0];
            int nc = c + dir[1];

            if (nr >= 0 && nr < n && nc >= 0 && nc < n && !visited[nr][nc]) {
                int maxElev = max(d, grid[nr][nc]);
                if (maxElev < dist[nr][nc]) {
                    dist[nr][nc] = maxElev;
                    pq.push({maxElev, {nr, nc}});
                }
            }
        }
    }
    return -1;
}
// Time: O(n^2 log n) — grid cells is n^2
// Space: O(n^2)
```

---

## 2. Critical Connections in a Network (LC 1192)

**Why Tricky**: Find all bridges in a large, undirected graph. Naive edge removal takes $O(E \times (V + E))$ — too slow. We must use **Tarjan's Bridge-Finding Algorithm** to solve it in a single $O(V + E)$ pass.

```
Graph DFS Walk:
Discovery times (tin): 0 -> 1 -> 2 -> 3
If 3 has an edge back to 1 (back-edge):
low[3] = min(low[3], tin[1]) = 1.
Backtrack to 2: low[2] = min(low[2], low[3]) = 1.
Backtrack to 1: low[1] = min(low[1], low[2]) = 1.
For edge 0 -> 1: low[1] (1) > tin[0] (0) -> 0 -> 1 is a Bridge!
For edge 1 -> 2: low[2] (1) == tin[1] (1) -> Not a Bridge.
```

```cpp
#include <vector>
#include <algorithm>

void dfs(int u, int p, int& timer, const vector<vector<int>>& adj, 
         vector<int>& tin, vector<int>& low, vector<vector<int>>& bridges) {
    tin[u] = low[u] = timer++;
    
    for (int v : adj[u]) {
        if (v == p) continue; // skip parent

        if (tin[v] != -1) {
            // Back-edge found
            low[u] = min(low[u], tin[v]);
        } else {
            // Forward edge (unvisited child)
            dfs(v, u, timer, adj, tin, low, bridges);
            low[u] = min(low[u], low[v]); // pull lowest reachable node up

            // Bridge detection check
            if (low[v] > tin[u]) {
                bridges.push_back({u, v});
            }
        }
    }
}

vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
    vector<vector<int>> adj(n);
    for (auto& c : connections) {
        adj[c[0]].push_back(c[1]);
        adj[c[1]].push_back(c[0]);
    }

    vector<int> tin(n, -1);
    vector<int> low(n, -1);
    vector<vector<int>> bridges;
    int timer = 0;

    dfs(0, -1, timer, adj, tin, low, bridges);
    return bridges;
}
// Time: O(V + E)
// Space: O(V + E) for adjacency list
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Critical Connections in a Network | Hard | [LC 1192](https://leetcode.com/problems/critical-connections-in-a-network/) |
| 2 | Swim in Rising Water | Hard | [LC 778](https://leetcode.com/problems/swim-in-rising-water/) |
| 3 | Eulerian Path / Circuit | Hard | [GFG](https://www.geeksforgeeks.org/eulerian-path-and-circuit/) |
| 4 | Find the City With the Smallest Number of Neighbors | Medium | [LC 1334](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) |

---

*<- [[12_Advanced_Graphs_Problems_and_Exercises|Problems]] · [[../13_DP/00_Index|Dynamic Programming ->]]*

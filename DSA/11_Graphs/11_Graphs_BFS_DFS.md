---
tags: [dsa, graphs, bfs, dfs, cycle-detection]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_Foundations]]", "[[11_Graphs_Shortest_Path]]"]
---

# Graphs -- BFS & DFS Traversals

*<- [[11_Graphs_Foundations|Foundations]] · [[11_Graphs_Shortest_Path|Shortest Path ->]]*

---

## 1. Breadth-First Search (BFS)

**Core Idea**: Visit nodes level-by-level (outward from source). BFS uses a **Queue** and is the primary tool for finding the **shortest path in an unweighted graph**.

### BFS Queue Progression Walkthrough
```
Graph:
    0 ─── 1 ─── 3
    │
    2

Queue starts with source [0]. Visited = {0}
1. Pop 0. Push neighbors 1, 2.  Queue = [1, 2],     Visited = {0, 1, 2}
2. Pop 1. Push neighbor 3.      Queue = [2, 3],     Visited = {0, 1, 2, 3}
3. Pop 2. No unvisited neighbors. Queue = [3],       Visited = {0, 1, 2, 3}
4. Pop 3. No unvisited neighbors. Queue = [],        Visited = {0, 1, 2, 3}
```

```cpp
#include <vector>
#include <queue>

// Standard BFS traversal starting from a source node
vector<int> bfs(int source, const vector<vector<int>>& adj) {
    int n = adj.size();
    vector<bool> visited(n, false);
    queue<int> q;
    vector<int> result;

    q.push(source);
    visited[source] = true;

    while (!q.empty()) {
        int node = q.front();
        q.pop();
        result.push_back(node);

        for (int neighbor : adj[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
        }
    }
    return result;
}
// Time: O(V + E) — visit every vertex and explore every edge once
// Space: O(V) for queue and visited array
```

---

## 2. Depth-First Search (DFS)

**Core Idea**: Explore as deep as possible along each branch before backtracking. DFS uses **Recursion (implicit call stack)**.

```cpp
#include <vector>

void dfsHelper(int node, const vector<vector<int>>& adj, vector<bool>& visited, vector<int>& result) {
    visited[node] = true;
    result.push_back(node);

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            dfsHelper(neighbor, adj, visited, result);
        }
    }
}

vector<int> dfs(int n, const vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    vector<int> result;
    // Iterate to handle disconnected graphs (multiple components)
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfsHelper(i, adj, visited, result);
        }
    }
    return result;
}
// Time: O(V + E)
// Space: O(V) recursion stack height in worst case (skewed graph)
```

---

## 3. Cycle Detection in Undirected Graphs (DFS)

**Key Insight**: In an undirected graph, a cycle exists if we visit an already visited node that is **not the direct parent** of the current node.

```cpp
#include <vector>

bool hasCycleUndirectedDFS(int node, int parent, const vector<vector<int>>& adj, vector<bool>& visited) {
    visited[node] = true;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            if (hasCycleUndirectedDFS(neighbor, node, adj, visited)) return true;
        } else if (neighbor != parent) {
            // Found visited node that is not parent -> Cycle detected!
            return true;
        }
    }
    return false;
}

bool detectCycleUndirected(int n, const vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            if (hasCycleUndirectedDFS(i, -1, adj, visited)) return true;
        }
    }
    return false;
}
```

---

## 4. Cycle Detection in Directed Graphs (DFS)

**Key Insight**: In a directed graph, parent checking is not enough. A cycle exists only if we hit a node that is currently in the **active recursion stack** (a back-edge). We maintain a `pathVisited` array to track the active path.

### Visualizing Back-Edge Cycle Detection
```
Path: 0 -> 1 -> 2 -> 0 (back to 0!)
Active stack: [0, 1, 2]
When at 2: neighbor 0 is in the active stack -> Cycle!

If we backtrack from 2 to 1:
Active stack becomes [0, 1] (pathVisited[2] is reset to false).
```

```cpp
#include <vector>

bool hasCycleDirectedDFS(int node, const vector<vector<int>>& adj, 
                         vector<bool>& visited, vector<bool>& pathVisited) {
    visited[node] = true;
    pathVisited[node] = true; // add to active recursion path

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            if (hasCycleDirectedDFS(neighbor, adj, visited, pathVisited)) return true;
        } else if (pathVisited[neighbor]) {
            // Neighbor is already on our active path -> Cycle!
            return true;
        }
    }

    pathVisited[node] = false; // backtrack: remove from active path
    return false;
}

bool detectCycleDirected(int n, const vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    vector<bool> pathVisited(n, false);

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            if (hasCycleDirectedDFS(i, adj, visited, pathVisited)) return true;
        }
    }
    return false;
}
// Time: O(V + E)
// Space: O(V)
```

---

*<- [[11_Graphs_Foundations|Foundations]] · [[11_Graphs_Shortest_Path|Shortest Path ->]]*

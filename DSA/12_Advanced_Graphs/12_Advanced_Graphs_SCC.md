---
tags: [dsa, graphs, scc, tarjan, euler-path, hierholzer]
links: ["[[12_Advanced_Graphs_Index]]", "[[12_Advanced_Graphs_Network_Flow]]", "[[12_Advanced_Graphs_Problems_and_Exercises]]", "[[../11_Graphs/11_Graphs_Tricky]]"]
---

# Advanced Graphs -- Tarjan's SCC & Euler Paths

*<- [[12_Advanced_Graphs_Network_Flow|Network Flow]] · [[12_Advanced_Graphs_Problems_and_Exercises|Problems ->]]*

---

## 1. Tarjan's Algorithm (Single-Pass SCC)

**Why**: Kosaraju's algorithm (covered in [[../11_Graphs/11_Graphs_Tricky|Chapter 11 Tricky]]) requires two DFS passes and a graph transpose. Tarjan's algorithm finds SCCs in a **single DFS pass** using a stack and low-link tracking.

### Core Idea
- Maintain `tin[u]` (discovery time) and `low[u]` (lowest discovery time reachable).
- Keep track of visited nodes in an active DFS stack.
- As DFS backtracks:
  - If a child node has a smaller `low` link, pull it up: `low[u] = min(low[u], low[v])`.
  - If `low[u] == tin[u]`, then $u$ is the **root of an SCC**. Pop all elements from the stack until we reach $u$; these popped nodes form one complete SCC.

```
DFS Stack: [0, 1, 2, 3]  <- active path
If low[2] == tin[2]:
Pop until 2 -> SCC elements are {3, 2}. Stack becomes [0, 1].
```

```cpp
#include <vector>
#include <stack>
#include <algorithm>

void dfsSCC(int u, int& timer, const vector<vector<int>>& adj,
            vector<int>& tin, vector<int>& low, vector<bool>& inStack,
            stack<int>& st, vector<vector<int>>& sccs) {
    tin[u] = low[u] = timer++;
    st.push(u);
    inStack[u] = true;

    for (int v : adj[u]) {
        if (tin[v] == -1) {
            // Unvisited child
            dfsSCC(v, timer, adj, tin, low, inStack, st, sccs);
            low[u] = min(low[u], low[v]);
        } else if (inStack[v]) {
            // v is already in the active DFS stack -> back-edge!
            low[u] = min(low[u], tin[v]);
        }
    }

    // If u is the root of an SCC, pop all its elements
    if (low[u] == tin[u]) {
        vector<int> currentScc;
        while (true) {
            int node = st.top();
            st.pop();
            inStack[node] = false;
            currentScc.push_back(node);
            if (node == u) break;
        }
        sccs.push_back(currentScc);
    }
}

vector<vector<int>> tarjanSCC(int n, const vector<vector<int>>& adj) {
    vector<int> tin(n, -1);
    vector<int> low(n, -1);
    vector<bool> inStack(n, false);
    stack<int> st;
    vector<vector<int>> sccs;
    int timer = 0;

    for (int i = 0; i < n; i++) {
        if (tin[i] == -1) {
            dfsSCC(i, timer, adj, tin, low, inStack, st, sccs);
        }
    }
    return sccs;
}
// Time: O(V + E) — single DFS pass
// Space: O(V) for recursion stack/arrays
```

---

## 2. Eulerian Paths & Circuits

- **Eulerian Path**: A trail in a finite graph that visits **every edge exactly once**.
- **Eulerian Circuit (Cycle)**: An Eulerian path that **starts and ends on the same vertex**.

### Existence Conditions (Directed Graphs)
- **Eulerian Circuit**: Strongly connected, and every node has `indegree == outdegree`.
- **Eulerian Path**: Strongly connected, and at most one node has `outdegree - indegree = 1` (start node), and at most one node has `indegree - outdegree = 1` (end node). All other nodes have `indegree == outdegree`.

---

## 3. Hierholzer's Algorithm (Find Eulerian Path/Circuit)

**Core Idea**: Greedy DFS cycle splicing.
- Start DFS from the starting vertex.
- Traverse unused outgoing edges recursively.
- If a vertex has no unused outgoing edges, push it to the path result (post-order).
- Reversing the path result gives the Eulerian path.

```
Graph: 0 -> 1 -> 2 -> 0, 0 -> 3
DFS(0):
Go to 1 -> Go to 2 -> Go to 0 (no outgoing left!) -> push 0.
Backtrack to 2 -> push 2.
Backtrack to 1 -> push 1.
Backtrack to 0 -> Go to 3 (no outgoing!) -> push 3.
Backtrack to 0 -> push 0.

Path result (stack order): [0, 3, 1, 2, 0]
Reversed: [0, 2, 1, 3, 0] -> Valid Eulerian Path!
```

```cpp
#include <vector>
#include <unordered_map>
#include <algorithm>

// Eulerian Path/Circuit on directed graph
// adj map stores multiset or vector of targets to handle multiple identical edges
vector<int> findEulerianPath(int startNode, unordered_map<int, vector<int>>& adj) {
    vector<int> path;
    stack<int> currPath;
    currPath.push(startNode);

    while (!currPath.empty()) {
        int u = currPath.top();
        if (!adj[u].empty()) {
            int v = adj[u].back();
            adj[u].pop_back(); // consume edge
            currPath.push(v);
        } else {
            path.push_back(u); // backtrack: save to path
            currPath.pop();
        }
    }
    reverse(path.begin(), path.end());
    return path;
}
// Time: O(V + E) — consume each edge once
// Space: O(V + E) for recursion stack
```

---

*<- [[12_Advanced_Graphs_Network_Flow|Network Flow]] · [[12_Advanced_Graphs_Problems_and_Exercises|Problems ->]]*

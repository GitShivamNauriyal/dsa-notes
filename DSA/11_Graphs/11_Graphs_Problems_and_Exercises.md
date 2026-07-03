---
tags: [dsa, graphs, problems, exercises]
links: ["[[11_Graphs_Index]]", "[[11_Graphs_Topological_Sort]]", "[[11_Graphs_Tricky]]"]
---

# Graphs -- Problems & Exercises

*<- [[11_Graphs_Topological_Sort|Topological Sort]] · [[11_Graphs_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Number of Islands | Medium | Grid connected components (BFS/DFS) | [LC 200](https://leetcode.com/problems/number-of-islands/) |
| 2 | Clone Graph | Medium | Deep copy using hashmap + DFS | [LC 133](https://leetcode.com/problems/clone-graph/) |
| 3 | Flood Fill | Easy | In-place grid replacement | [LC 733](https://leetcode.com/problems/flood-fill/) |

## Tier 2 -- Cycles & Dependencies

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Course Schedule | Medium | Directed cycle detection (Kahn's / DFS) | [LC 207](https://leetcode.com/problems/course-schedule/) |
| 5 | Course Schedule II | Medium | Topological sort order | [LC 210](https://leetcode.com/problems/course-schedule-ii/) |
| 6 | Redundant Connection | Medium | Undirected cycle detection using DSU | [LC 684](https://leetcode.com/problems/redundant-connection/) |
| 7 | Number of Provinces | Medium | Connected components (DSU/DFS) | [LC 547](https://leetcode.com/problems/number-of-provinces/) |

## Tier 3 -- Shortest Path & BFS

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Network Delay Time | Medium | Single-source shortest path (Dijkstra) | [LC 743](https://leetcode.com/problems/network-delay-time/) |
| 9 | Word Ladder | Hard | Unweighted shortest path (BFS) | [LC 127](https://leetcode.com/problems/word-ladder/) |
| 10 | Rotting Oranges | Medium | Multi-source BFS | [LC 994](https://leetcode.com/problems/rotting-oranges/) |
| 11 | Cheapest Flights Within K Stops | Medium | Modified Dijkstra / Bellman-Ford | [LC 787](https://leetcode.com/problems/cheapest-flights-within-k-stops/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 12 | Alien Dictionary | Hard | Construct dependency graph + Topo Sort | [LC 269](https://leetcode.com/problems/alien-dictionary/) |
| 13 | Shortest Path in a Grid with Obstacles Elimination | Hard | 3D state BFS `(r, c, k)` | [LC 1293](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) |
| 14 | Course Schedule III | Hard | Greedy scheduling + Max-Heap | [LC 630](https://leetcode.com/problems/course-schedule-iii/) |

---

## Worked Solution 1: LC 207 -- Course Schedule (Cycle Detection)

**Key Insight**: The problem asks if it is possible to finish all courses given prerequisite pairs. This is equivalent to checking if the directed dependency graph contains a cycle. We can solve this elegantly using **Kahn's Algorithm (BFS)**: if the count of popped elements equals the total number of courses, no cycle exists.

```cpp
#include <vector>
#include <queue>

bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
    // Build adjacency list and calculate indegrees
    vector<vector<int>> adj(numCourses);
    vector<int> indegree(numCourses, 0);
    
    for (auto& pre : prerequisites) {
        int course = pre[0];
        int prereq = pre[1];
        adj[prereq].push_back(course); // prereq -> course
        indegree[course]++;
    }

    // Push nodes with no dependencies to queue
    queue<int> q;
    for (int i = 0; i < numCourses; i++) {
        if (indegree[i] == 0) q.push(i);
    }

    int processedCount = 0;
    while (!q.empty()) {
        int u = q.front();
        q.pop();
        processedCount++;

        for (int v : adj[u]) {
            indegree[v]--;
            if (indegree[v] == 0) {
                q.push(v);
            }
        }
    }

    // If we processed all nodes, no cycle exists
    return processedCount == numCourses;
}
// Time: O(V + E) — V is numCourses, E is prerequisites size
// Space: O(V + E) for adjacency list + O(V) for queue/indegree array
```

---

## Worked Solution 2: LC 743 -- Network Delay Time (Dijkstra)

**Key Insight**: We want the minimum time it takes for all nodes to receive a signal from a source node `k`. This is equivalent to finding the shortest path from `k` to all other nodes. The answer is the **maximum of these shortest path distances**. If any node is unreachable, return `-1`. We use **Dijkstra's Algorithm**.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    // Build adjacency list: adj[u] = {neighbor, weight}
    vector<vector<pair<int, int>>> adj(n + 1);
    for (auto& t : times) {
        adj[t[0]].push_back({t[1], t[2]});
    }

    // Dijkstra initialization (1-indexed nodes)
    vector<int> dist(n + 1, 1e9);
    dist[k] = 0;

    // Min-heap: {distance, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, k});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if (d > dist[u]) continue;

        for (auto& edge : adj[u]) {
            int v = edge.first;
            int w = edge.second;
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }

    // Find the maximum distance among all valid nodes (1 to n)
    int maxTime = 0;
    for (int i = 1; i <= n; i++) {
        if (dist[i] == 1e9) return -1; // unreachable node found
        maxTime = max(maxTime, dist[i]);
    }
    return maxTime;
}
// Time: O(E log V) — V is nodes, E is edges
// Space: O(V + E) for adjacency list + O(V) for Dijkstra data structures
```

---

*<- [[11_Graphs_Topological_Sort|Topological Sort]] · [[11_Graphs_Tricky|Tricky ->]]*

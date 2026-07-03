---
tags: [dsa, graphs, advanced-graphs, problems, exercises]
links: ["[[12_Advanced_Graphs_Index]]", "[[12_Advanced_Graphs_SCC]]", "[[12_Advanced_Graphs_Tricky]]"]
---

# Advanced Graphs -- Problems & Exercises

*<- [[12_Advanced_Graphs_SCC|SCC & Tarjan's]] · [[12_Advanced_Graphs_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Min Cost to Connect All Points | Medium | Prim's or Kruskal's MST | [LC 1584](https://leetcode.com/problems/min-cost-to-connect-all-points/) |
| 2 | Find the Town Judge | Easy | Indegree / Outdegree balance | [LC 997](https://leetcode.com/problems/find-the-town-judge/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Reconstruct Itinerary | Hard | Eulerian path (Hierholzer's) | [LC 332](https://leetcode.com/problems/reconstruct-itinerary/) |
| 4 | Network Delay Time (Advanced) | Medium | Dijkstra pathing comparisons | [LC 743](https://leetcode.com/problems/network-delay-time/) |
| 5 | Strongly Connected Components | Medium | Kosaraju's/Tarjan's SCC | [GFG](https://www.geeksforgeeks.org/strongly-connected-components/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Swim in Rising Water | Hard | Dijkstra on 2D grid / DSU | [LC 778](https://leetcode.com/problems/swim-in-rising-water/) |
| 7 | Path with Maximum Probability | Medium | Modified Dijkstra (multiplicative probability) | [LC 1514](https://leetcode.com/problems/path-with-maximum-probability/) |
| 8 | Optimize Water Distribution | Hard | Virtual source trick + MST | [LC 1168](https://leetcode.com/problems/optimize-water-distribution-in-a-village/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Critical Connections in a Network | Hard | Tarjan's Bridge-Finding Algorithm | [LC 1192](https://leetcode.com/problems/critical-connections-in-a-network/) |
| 10 | Maximum Flow / Min Cut | Hard | Edmonds-Karp / Ford-Fulkerson | [GFG](https://www.geeksforgeeks.org/ford-fulkerson-algorithm-for-maximum-flow-problem/) |
| 11 | Valid Arrangement of Pairs | Hard | Eulerian path on arbitrary integers | [LC 2097](https://leetcode.com/problems/valid-arrangement-of-pairs/) |

---

## Worked Solution 1: LC 1584 -- Min Cost to Connect All Points (MST)

**Key Insight**: We are given coordinates of points. The cost of connecting two points is their Manhattan distance. We want to connect all points with minimum cost. This is exactly the **Minimum Spanning Tree (MST)** problem. 
- Since the graph is dense (every point is connected to every other point, $E = O(V^2)$), **Prim's Algorithm** is optimal.

```cpp
#include <vector>
#include <queue>
#include <cmath>

int minCostConnectPoints(vector<vector<int>>& points) {
    int n = points.size();
    vector<bool> visited(n, false);
    
    // Min-heap: {distance, node}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
    pq.push({0, 0}); // start at node 0 with weight 0
    
    int mstWeight = 0;
    int visitedCount = 0;

    while (!pq.empty()) {
        auto [dist, u] = pq.top();
        pq.pop();

        if (visited[u]) continue;

        visited[u] = true;
        mstWeight += dist;
        visitedCount++;

        if (visitedCount == n) break; // all points connected

        // Explore connections to all other unvisited points
        for (int v = 0; v < n; v++) {
            if (!visited[v]) {
                int manhattanDist = abs(points[u][0] - points[v][0]) + 
                                    abs(points[u][1] - points[v][1]);
                pq.push({manhattanDist, v});
            }
        }
    }
    return mstWeight;
}
// Time: O(V^2 log V) — each node pushes V neighbors into priority queue
// Space: O(V^2) for the priority queue
```

---

## Worked Solution 2: LC 332 -- Reconstruct Itinerary (Eulerian Path)

**Key Insight**: Reconstruct a flight itinerary from tickets. All tickets must be used exactly once. This is an **Eulerian Path** problem in a directed graph. 
- **Lexicographical Order**: If multiple itineraries exist, we must choose the one with the smallest lexical order. To achieve this, we sort the destinations of each source in descending order, so that we can pop them from the back of the vector in $O(1)$ time to access the smallest lexical destination first.
- We use **Hierholzer's Algorithm**.

```cpp
#include <vector>
#include <string>
#include <unordered_map>
#include <algorithm>
#include <stack>

vector<string> findItinerary(vector<vector<string>>& tickets) {
    // Build adjacency list. Use vector, then sort in descending order
    // so we can pop from the back (O(1)) to get lexically smallest targets first.
    unordered_map<string, vector<string>> adj;
    for (auto& t : tickets) {
        adj[t[0]].push_back(t[1]);
    }
    
    for (auto& [src, dests] : adj) {
        sort(dests.begin(), dests.end(), greater<string>());
    }

    vector<string> itinerary;
    stack<string> currPath;
    currPath.push("JFK"); // itinerary always starts at "JFK"

    while (!currPath.empty()) {
        string curr = currPath.top();
        if (adj.count(curr) && !adj[curr].empty()) {
            string nextDest = adj[curr].back();
            adj[curr].pop_back(); // consume flight ticket
            currPath.push(nextDest);
        } else {
            itinerary.push_back(curr); // backtrack: locked in position from the end
            currPath.pop();
        }
    }

    // Reconstruct the correct order
    reverse(itinerary.begin(), itinerary.end());
    return itinerary;
}
// Time: O(E log (E/V)) — sorting destinations. DFS traversal takes O(V + E)
// Space: O(V + E) for adjacency list + O(E) for stack
```

---

*<- [[12_Advanced_Graphs_SCC|SCC & Tarjan's]] · [[12_Advanced_Graphs_Tricky|Tricky ->]]*

---
tags: [dsa, dp, bitmask-dp, tsp]
links: ["[[08_Bitmask_DP_Index]]", "[[Bitmask_DP_Problems]]"]
---

# Bitmask DP -- Foundations & TSP

*<- [[08_Bitmask_DP_Index|Bitmask Index]] · [[Bitmask_DP_Problems|Problems ->]]*

---

## What is Bitmask DP?
Bitmask DP is used when we need to represent a **set of chosen items** (e.g. visited cities, assigned tasks) as part of our DP state.
- A single integer `mask` acts as a boolean array where the $i$-th bit represents whether the $i$-th item is selected (`1`) or not (`0`).
- **State Limit**: Since a mask of size $N$ requires $2^N$ states, Bitmask DP is only feasible when $N \le 20$ (since $2^{20} \approx 10^6$ states).

### Bitwise State Cheatsheet
| Operation | Expression |
|---|---|
| Check if item $i$ is selected | `(mask >> i) & 1` (evaluates to 1 or 0) |
| Set item $i$ to selected | `mask | (1 << i)` |
| Clear item $i$ to unselected | `mask & ~(1 << i)` |
| Toggle item $i$ state | `mask ^ (1 << i)` |
| Check if all $N$ items selected | `mask == (1 << N) - 1` |

---

## Travelling Salesperson Problem (TSP)

**The Problem**: Given a set of cities and distance between every pair of cities, find the shortest possible route that visits every city exactly once and returns to the starting point.

### State Formulation
Let `solve(mask, u)` be the minimum cost to visit all unvisited cities in `mask` starting from city `u` and returning to the start node `0`.
- If we are at city `u`, we can next visit any city `v` that has **not yet been visited** (i.e. `(mask >> v) & 1 == 0`).
- **Recurrence**: `solve(mask, u) = min(dist[u][v] + solve(mask | (1 << v), v))` for all unvisited `v`.
- **Base Case**: `mask == (1 << n) - 1` (all cities visited). Return distance from `u` back to starting city `0`: `dist[u][0]`.

```cpp
#include <vector>
#include <algorithm>

int solveTSP(int mask, int u, int n, const vector<vector<int>>& dist, 
             vector<vector<int>>& memo) {
    // Base Case: All cities visited
    if (mask == (1 << n) - 1) {
        return dist[u][0]; // return to start city 0
    }

    if (memo[mask][u] != -1) return memo[mask][u];

    int minCost = 1e9;
    for (int v = 0; v < n; v++) {
        // If city v is unvisited (v-th bit is 0)
        if (((mask >> v) & 1) == 0) {
            int cost = dist[u][v] + solveTSP(mask | (1 << v), v, n, dist, memo);
            minCost = min(minCost, cost);
        }
    }
    return memo[mask][u] = minCost;
}

int tsp(const vector<vector<int>>& dist) {
    int n = dist.size();
    // mask size 2^n, states n
    vector<vector<int>> memo(1 << n, vector<int>(n, -1));
    return solveTSP(1, 0, n, dist, memo); // Start at city 0 (mask = 1 << 0 = 1)
}
// Time Complexity: O(n^2 * 2^n)
// Space Complexity: O(n * 2^n)
```

---

*<- [[08_Bitmask_DP_Index|Bitmask Index]] · [[Bitmask_DP_Problems|Problems ->]]*

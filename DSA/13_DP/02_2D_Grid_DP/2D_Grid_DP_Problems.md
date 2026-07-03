---
tags: [dsa, dp, 2d-grid-dp, problems, exercises]
links: ["[[02_2D_Grid_DP_Index]]", "[[2D_Grid_DP_Grid_Obstacles]]", "[[../03_Knapsack/03_Knapsack_Index]]"]
---

# 2D Grid DP -- Problems & Exercises

*<- [[2D_Grid_DP_Grid_Obstacles|Grid Path Minimums]] · [[../03_Knapsack/03_Knapsack_Index|Knapsack ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Unique Paths | Medium | Simple right/down transition | [LC 62](https://leetcode.com/problems/unique-paths/) |
| 2 | Unique Paths II | Medium | Grid obstacles check | [LC 63](https://leetcode.com/problems/unique-paths-ii/) |
| 3 | Minimum Path Sum | Medium | Min transition step sum | [LC 64](https://leetcode.com/problems/minimum-path-sum/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Triangle | Medium | Bottom-up triangle traversal | [LC 120](https://leetcode.com/problems/triangle/) |
| 5 | Maximal Square | Medium | Size checks using `min(top, left, diag) + 1` | [LC 221](https://leetcode.com/problems/maximal-square/) |
| 6 | Minimum Path Falling Sum | Medium | 3-way falling transitions | [LC 931](https://leetcode.com/problems/minimum-falling-path-sum/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 7 | Dungeon Game | Hard | Reverse DP (min health needed) | [LC 174](https://leetcode.com/problems/dungeon-game/) |
| 8 | Out of Boundary Paths | Medium | 3D DP `(r, c, moves)` | [LC 576](https://leetcode.com/problems/out-of-boundary-paths/) |
| 9 | Knight Probability in Chessboard | Medium | Knight step probabilities | [LC 688](https://leetcode.com/problems/knight-probability-in-chessboard/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 10 | Cherry Pickup | Hard | 2 paths simultaneously ($O(n^3)$ state) | [LC 741](https://leetcode.com/problems/cherry-pickup/) |
| 11 | Cherry Pickup II | Hard | 2 robots falling simultaneously | [LC 1463](https://leetcode.com/problems/cherry-pickup-ii/) |
| 12 | K-Inverse Pairs Array | Hard | DP prefix sums optimization | [LC 629](https://leetcode.com/problems/k-inverse-pairs-array/) |

---

## Worked Solution 1: LC 221 -- Maximal Square

**Key Insight**: Find the largest square containing only 1s in a binary matrix.
- Let `dp[r][c]` be the side length of the largest square whose **bottom-right corner** is at cell `(r, c)`.
- If `matrix[r][c] == '1'`, then a square of size $S$ is only possible if its top, left, and diagonal neighbors can support squares of size $S-1$.
- **Recurrence**: `dp[r][c] = 1 + min({dp[r-1][c], dp[r][c-1], dp[r-1][c-1]})`
- **Base cases**: If $r=0$ or $c=0$, the maximum square size is simply the value of the cell itself (1 or 0).

```cpp
#include <vector>
#include <algorithm>

int maximalSquare(vector<vector<char>>& matrix) {
    int rows = matrix.size();
    if (rows == 0) return 0;
    int cols = matrix[0].size();
    
    vector<int> prevRow(cols, 0);
    vector<int> currRow(cols, 0);
    int maxSide = 0;

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (matrix[r][c] == '1') {
                if (r == 0 || c == 0) {
                    currRow[c] = 1;
                } else {
                    currRow[c] = 1 + min({prevRow[c], currRow[c - 1], prevRow[c - 1]});
                }
                maxSide = max(maxSide, currRow[c]);
            } else {
                currRow[c] = 0;
            }
        }
        prevRow = currRow; // move down a row
    }
    return maxSide * maxSide; // return Area
}
// Time Complexity: O(rows * cols)
// Space Complexity: O(cols)
```

---

## Worked Solution 2: LC 741 -- Cherry Pickup (Dual Path Simultaneous DP)

**Why Tricky**: We need to find a path from `(0,0) -> (N-1, N-1)` and back to `(0,0)` to collect maximum cherries. If we pick cherries, they turn to 0.
- **Why greedy 2-pass fails**: Collecting maximum cherries on the first pass might force the return pass to take a sub-optimal route, missing a globally optimal combination.
- **The Solution**: Imagine **two people starting at `(0,0)` and walking to `(N-1, N-1)` simultaneously**.
- At step `t`, person 1 is at `(r1, c1)` and person 2 is at `(r2, c2)`.
- Since both take 1 step per turn, `r1 + c1 == r2 + c2 == t`. Thus, we can represent the state with 3 variables: `(t, r1, r2)`. The columns are derived: `c1 = t - r1`, `c2 = t - r2`.
- If `r1 == r2` (both on the same cell), they can only collect the cherry once.

```cpp
#include <vector>
#include <algorithm>

int cherryHelper(int t, int r1, int r2, int n, const vector<vector<int>>& grid, 
                 vector<vector<vector<int>>>& memo) {
    int c1 = t - r1;
    int c2 = t - r2;

    // Out of bounds or hit an obstacle
    if (r1 >= n || r2 >= n || c1 >= n || c2 >= n || grid[r1][c1] == -1 || grid[r2][c2] == -1) {
        return -1e9;
    }

    // Base case: Reached destination
    if (t == 2 * n - 2) {
        return grid[n - 1][n - 1];
    }

    if (memo[t][r1][r2] != -1) return memo[t][r1][r2];

    // Collect cherry
    int cherries = 0;
    if (r1 == r2) {
        cherries = grid[r1][c1]; // same cell, collect once
    } else {
        cherries = grid[r1][c1] + grid[r2][c2]; // different cells, collect both
    }

    // Four possible move transitions for the two people:
    // 1. P1 down, P2 down (r1+1, r2+1)
    // 2. P1 down, P2 right (r1+1, r2)
    // 3. P1 right, P2 down (r1, r2+1)
    // 4. P1 right, P2 right (r1, r2)
    int nextMax = max({
        cherryHelper(t + 1, r1 + 1, r2 + 1, n, grid, memo),
        cherryHelper(t + 1, r1 + 1, r2, n, grid, memo),
        cherryHelper(t + 1, r1, r2 + 1, n, grid, memo),
        cherryHelper(t + 1, r1, r2, n, grid, memo)
    });

    if (nextMax < 0) return memo[t][r1][r2] = -1e9; // blocked path

    return memo[t][r1][r2] = cherries + nextMax;
}

int cherryPickup(vector<vector<int>>& grid) {
    int n = grid.size();
    // 3D Memoization table of size [2N][N][N]
    vector<vector<vector<int>>> memo(2 * n, vector<vector<int>>(n, vector<int>(n, -1)));
    int res = cherryHelper(0, 0, 0, n, grid, memo);
    return max(0, res);
}
// Time Complexity: O(N^3)
// Space Complexity: O(N^3)
```

---

*<- [[2D_Grid_DP_Grid_Obstacles|Grid Path Minimums]] · [[../03_Knapsack/03_Knapsack_Index|Knapsack ->]]*

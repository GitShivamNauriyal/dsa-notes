---
tags: [dsa, dp, 2d-grid-dp, unique-paths, obstacles]
links: ["[[02_2D_Grid_DP_Index]]", "[[2D_Grid_DP_Grid_Obstacles]]"]
---

# 2D Grid DP -- Grid Paths & Counts

*<- [[02_2D_Grid_DP_Index|2D Grid DP Index]] · [[2D_Grid_DP_Grid_Obstacles|Grid Path Minimums ->]]*

---

## 1. Unique Paths (LC 62)

**The Problem**: A robot is located at the top-left corner of a $m \times n$ grid. The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid. How many possible unique paths are there?

### State Formulation
Let `dp[r][c]` be the number of unique paths to reach cell `(r, c)`.
- Since the robot can only move down or right, any path reaching `(r, c)` must come from either `(r-1, c)` (coming down) or `(r, c-1)` (coming right).
- **Recurrence Relation**: `dp[r][c] = dp[r-1][c] + dp[r][c-1]`
- **Base cases**:
  - `dp[0][c] = 1` (only 1 way to move along the first row: keep going right)
  - `dp[r][0] = 1` (only 1 way to move along the first column: keep going down)

---

### A. Tabulation (2D Array)
- **Time Complexity**: $O(m \times n)$.
- **Space Complexity**: $O(m \times n)$ to store the 2D table.

```cpp
#include <vector>

int uniquePaths2D(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1)); // Initialize with 1s (covers base cases)

    for (int r = 1; r < m; r++) {
        for (int c = 1; c < n; c++) {
            dp[r][c] = dp[r - 1][c] + dp[r][c - 1];
        }
    }
    return dp[m - 1][n - 1];
}
```

---

### B. Space Optimization (1D Row Array)
- **Core Intuition**: To calculate the current cell `dp[r][c]`, we only need the value above it `dp[r-1][c]` (which is stored in our current row array from the previous iteration) and the value to the left `dp[r][c-1]` (which is the updated value in our row array from the current step).
- Let `temp[c]` represent the row above, and we update it in-place to represent the current row.
- **Formula**: `row[c] = row[c] (value from row above) + row[c - 1] (value to the left)`
- **Time Complexity**: $O(m \times n)$.
- **Space Complexity**: $O(n)$ where $n$ is columns.

```cpp
#include <vector>

int uniquePathsSpaceOptimized(int m, int n) {
    vector<int> row(n, 1); // Represents the previous row, initialized to 1

    for (int r = 1; r < m; r++) {
        for (int c = 1; c < n; c++) {
            row[c] = row[c] + row[c - 1];
        }
    }
    return row[n - 1];
}
```

### Dry Run Row Transition Table:
For `m = 3`, `n = 3`:
- Initial row (r=0): `[1, 1, 1]`
- **r = 1**:
  - `c = 1`: `row[1] = row[1] + row[0] = 1 + 1 = 2`
  - `c = 2`: `row[2] = row[2] + row[1] = 1 + 2 = 3`
  - Row becomes: `[1, 2, 3]`
- **r = 2**:
  - `c = 1`: `row[1] = row[1] + row[0] = 2 + 1 = 3`
  - `c = 2`: `row[2] = row[2] + row[1] = 3 + 3 = 6`
  - Row becomes: `[1, 3, 6]`
- **Result**: `6` (Correct).

---

## 2. Unique Paths II (LC 63) -- Handling Obstacles

**The Problem**: Now suppose some obstacles are added to the grids. An obstacle and space are marked as `1` and `0` respectively in the grid. How many unique paths are there?

### State Adjustment
- If `grid[r][c] == 1` (obstacle), then `dp[r][c] = 0` (no path can pass through this cell).
- Otherwise, apply standard recurrence.
- **Base Case Warning**: If `grid[0][0] == 1`, return 0. The starting cell itself is blocked.

```cpp
#include <vector>

int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
    int m = obstacleGrid.size();
    int n = obstacleGrid[0].size();
    if (obstacleGrid[0][0] == 1) return 0;

    vector<int> row(n, 0);
    row[0] = 1; // start cell

    // Initialize first row base case (can only go right until blocked)
    for (int c = 1; c < n; c++) {
        row[c] = (obstacleGrid[0][c] == 0) ? row[c - 1] : 0;
    }

    for (int r = 1; r < m; r++) {
        // Handle first column update
        row[0] = (obstacleGrid[r][0] == 0) ? row[0] : 0;
        
        for (int c = 1; c < n; c++) {
            if (obstacleGrid[r][c] == 1) {
                row[c] = 0; // blocked cell
            } else {
                row[c] = row[c] + row[c - 1];
            }
        }
    }
    return row[n - 1];
}
// Time Complexity: O(m * n)
// Space Complexity: O(n)
```

---

*<- [[02_2D_Grid_DP_Index|2D Grid DP Index]] · [[2D_Grid_DP_Grid_Obstacles|Grid Path Minimums ->]]*

---
tags: [dsa, dp, 2d-grid-dp, min-path-sum, triangle, dungeon-game]
links: ["[[02_2D_Grid_DP_Index]]", "[[2D_Grid_DP_Grid_Paths]]"]
---

# 2D Grid DP -- Grid Path Minimums

*<- [[2D_Grid_DP_Grid_Paths|Grid Paths & Counts]] · [[2D_Grid_DP_Problems|Problems ->]]*

---

## 1. Minimum Path Sum (LC 64)

**The Problem**: Given a $m \times n$ grid filled with non-negative numbers, find a path from top-left to bottom-right, which minimizes the sum of all numbers along its path.
- Note: You can only move either down or right at any point in time.

### State Formulation
Let `dp[r][c]` be the minimum path sum to reach cell `(r, c)`.
- **Recurrence**: `dp[r][c] = grid[r][c] + min(dp[r-1][c], dp[r][c-1])`
- **Base cases**:
  - `dp[0][0] = grid[0][0]`
  - First Row: `dp[0][c] = grid[0][c] + dp[0][c-1]` (can only come from the left)
  - First Column: `dp[r][0] = grid[r][0] + dp[r-1][0]` (can only come from above)

```cpp
#include <vector>
#include <algorithm>

int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size();
    int n = grid[0].size();
    
    vector<int> row(n);
    row[0] = grid[0][0];

    // Initialize first row base case
    for (int c = 1; c < n; c++) {
        row[c] = grid[0][c] + row[c - 1];
    }

    for (int r = 1; r < m; r++) {
        row[0] = grid[r][0] + row[0]; // first column of current row
        for (int c = 1; c < n; c++) {
            row[c] = grid[r][c] + min(row[c], row[c - 1]);
        }
    }
    return row[n - 1];
}
// Time Complexity: O(m * n)
// Space Complexity: O(n)
```

---

## 2. Triangle (LC 120)

**The Problem**: Given a triangle array, return the minimum path sum from top to bottom. For each step, you may move to adjacent numbers on the row below. More formally, if you are on index `i` on the current row, you may move to either index `i` or index `i+1` on the next row.

### Bottom-Up Tabulation (In-Place)
- **Why Bottom-Up works best here**: If we start from the bottom row and work our way up to the top, the answer will naturally converge at the single root node `triangle[0][0]`. This avoids edge checks at boundaries!
- **Recurrence**: `triangle[r][c] = triangle[r][c] + min(triangle[r+1][c], triangle[r+1][c+1])`
- **Time Complexity**: $O(n^2)$ where $n$ is the height of the triangle.
- **Space Complexity**: $O(1)$ if we modify the input array in-place, or $O(n)$ if we use a 1D DP array.

```cpp
#include <vector>
#include <algorithm>

int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    // Start from the second-to-last row and move upwards
    for (int r = n - 2; r >= 0; r--) {
        for (int c = 0; c <= r; c++) {
            triangle[r][c] += min(triangle[r + 1][c], triangle[r + 1][c + 1]);
        }
    }
    return triangle[0][0];
}
```

---

## 3. Dungeon Game (LC 174) -- Tricky Reverse DP

**Why Tricky**: The knight starts at top-left with some initial health and must reach bottom-right. Each cell contains a daemon (depletes health) or an orb (heals health). 
- If we try to solve it forward (from top-left to bottom-right), we need to track both the **minimum health needed so far** and the **current health**. This is because a path with high current health might have required huge initial health to survive a previous room.
- **The Solution**: Solve **backwards** (from bottom-right to top-left). Let `dp[r][c]` be the minimum health required *upon entering* cell `(r, c)` to reach the destination alive (health must stay $\ge 1$).

### Backwards Recurrence
- If we move right from `(r, c)` to `(r, c+1)`, the health needed before entering `(r, c)` is: `needed = dp[r][c+1] - grid[r][c]`.
- If we move down to `(r+1, c)`: `needed = dp[r+1][c] - grid[r][c]`.
- We take the minimum health needed of the two directions: `needed = min(needed_right, needed_down)`.
- If a cell has a large healing orb, `needed` could become $\le 0$. However, the knight's health must always be $\ge 1$. Thus: `dp[r][c] = max(1, needed)`.

```cpp
#include <vector>
#include <algorithm>

int calculateMinimumHP(vector<vector<int>>& dungeon) {
    int m = dungeon.size();
    int n = dungeon[0].size();
    
    // dp[r][c] represents min health needed entering cell (r,c)
    vector<vector<int>> dp(m, vector<int>(n, 1e9));

    // Base Case: Destination
    dp[m - 1][n - 1] = max(1, 1 - dungeon[m - 1][n - 1]);

    // Fill last column
    for (int r = m - 2; r >= 0; r--) {
        dp[r][n - 1] = max(1, dp[r + 1][n - 1] - dungeon[r][n - 1]);
    }

    // Fill last row
    for (int c = n - 2; c >= 0; c--) {
        dp[m - 1][c] = max(1, dp[m - 1][c + 1] - dungeon[m - 1][c]);
    }

    // Fill rest of the grid backwards
    for (int r = m - 2; r >= 0; r--) {
        for (int c = n - 2; c >= 0; c--) {
            int minHealthNext = min(dp[r + 1][c], dp[r][c + 1]);
            dp[r][c] = max(1, minHealthNext - dungeon[r][c]);
        }
    }
    return dp[0][0];
}
// Time Complexity: O(m * n)
// Space Complexity: O(m * n) -> can be optimized to O(n)
```

---

*<- [[2D_Grid_DP_Grid_Paths|Grid Paths & Counts]] · [[2D_Grid_DP_Problems|Problems ->]]*

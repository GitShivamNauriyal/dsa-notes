---
tags: [dsa, math, geometry, tricky, hard, matrix, line-slope]
links: ["[[17_Math_and_Geometry_Index]]", "[[17_Math_and_Geometry_Problems_and_Exercises]]", "[[../00_Home]]"]
---

# Math & Geometry -- Tricky & Higher-Order

*<- [[17_Math_and_Geometry_Problems_and_Exercises|Problems]] · [[../00_Home|Home ->]]*

---

## 1. Set Matrix Zeroes (LC 73)

**Why Tricky**: If you see a `0` and immediately set its row and column to `0`, those newly set `0`s will be processed in subsequent iterations, eventually turning the **entire matrix** to `0`. 
- **Naive solution**: Use a duplicate matrix ($O(m \times n)$ space) or two visited arrays ($O(m + n)$ space).
- **The $O(1)$ Space Solution**: Use the **first row** and **first column** of the matrix itself as the tracking arrays.
  - We use `matrix[0][c] = 0` to flag that column `c` should be zeroed.
  - We use `matrix[r][0] = 0` to flag that row `r` should be zeroed.
  - Since `matrix[0][0]` is shared by both, we use an extra boolean variable `rowZero` to track the first row, and let `matrix[0][0]` track the first column.

```cpp
#include <vector>
#include <algorithm>

void setZeroes(vector<vector<int>>& matrix) {
    int rows = matrix.size();
    int cols = matrix[0].size();
    bool rowZero = false;

    // Step 1: Scan and flag
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (matrix[r][c] == 0) {
                matrix[0][c] = 0; // flag column
                
                if (r > 0) {
                    matrix[r][0] = 0; // flag row
                } else {
                    rowZero = true; // flag first row separately
                }
            }
        }
    }

    // Step 2: Set zeroes based on flags (skip first row/col for now)
    for (int r = 1; r < rows; r++) {
        for (int c = 1; c < cols; c++) {
            if (matrix[0][c] == 0 || matrix[r][0] == 0) {
                matrix[r][c] = 0;
            }
        }
    }

    // Step 3: Zero out first column if flagged
    if (matrix[0][0] == 0) {
        for (int r = 0; r < rows; r++) {
            matrix[r][0] = 0;
        }
    }

    // Step 4: Zero out first row if flagged
    if (rowZero) {
        for (int c = 0; c < cols; c++) {
            matrix[0][c] = 0;
        }
    }
}
// Time Complexity: O(rows * cols) — two passes
// Space Complexity: O(1)
```

---

## 2. Max Points on a Line (Slope Coprimes -- LC 149)

**Why Tricky**: Find the maximum number of points that lie on the same straight line.
- The slope between points $(x_1, y_1)$ and $(x_2, y_2)$ is $m = \frac{y_2 - y_1}{x_2 - x_1} = \frac{dy}{dx}$.
- Storing `double` slopes in a hash map has precision limits and rounding errors, which can cause points on the same line to be grouped separately.
- **The Solution**: Store the slope as a reduced fraction pair of coprime integers `{dy, dx}`.
  - Calculate $\gcd(dy, dx)$ and divide both by it.
  - To handle signs consistently: if $dx < 0$, invert signs of both $dy$ and $dx$. If $dy = 0$, set $dx = 1$. If $dx = 0$, set $dy = 1$ (vertical line).

```cpp
#include <vector>
#include <map>
#include <algorithm>

using namespace std;

// Helper to compute GCD
int getGCD(int a, int b) {
    if (b == 0) return a;
    return getGCD(b, a % b);
}

int maxPoints(vector<vector<int>>& points) {
    int n = points.size();
    if (n <= 2) return n;

    int maxPointsOnLine = 1;

    for (int i = 0; i < n; i++) {
        // Map to store counts of slopes from point i: key is pair {dy, dx}
        map<pair<int, int>, int> slopeMap;
        int duplicate = 1;

        for (int j = i + 1; j < n; j++) {
            int dy = points[j][1] - points[i][1];
            int dx = points[j][0] - points[i][0];

            if (dy == 0 && dx == 0) {
                duplicate++;
                continue;
            }

            int g = getGCD(dy, dx);
            dy /= g;
            dx /= g;

            // Normalize signs
            if (dx < 0) {
                dy = -dy;
                dx = -dx;
            }

            slopeMap[{dy, dx}]++;
        }

        maxPointsOnLine = max(maxPointsOnLine, duplicate);
        for (auto& [slope, count] : slopeMap) {
            maxPointsOnLine = max(maxPointsOnLine, count + duplicate);
        }
    }
    return maxPointsOnLine;
}
// Time Complexity: O(n^2 log n) due to map operations
// Space Complexity: O(n) for the map
```

---

*<- [[17_Math_and_Geometry_Problems_and_Exercises|Problems]] · [[../00_Home|Home ->]]*

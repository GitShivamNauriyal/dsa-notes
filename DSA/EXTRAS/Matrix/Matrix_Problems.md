---
tags: [dsa, matrix, problems, diagonal-traverse]
links: ["[[00_Index]]", "[[Matrix_Patterns]]", "[[../String_Algorithms/00_Index]]"]
---

# Matrix -- Problems & Exercises

*<- [[Matrix_Patterns|Operations]] · [[../String_Algorithms/00_Index|String Algorithms ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Set Matrix Zeroes | Medium | Sentinel flags in first row/col | [LC 73](https://leetcode.com/problems/set-matrix-zeroes/) |
| 2 | Transpose Matrix | Easy | Swap `[i][j]` and `[j][i]` | [LC 867](https://leetcode.com/problems/transpose-matrix/) |

## Tier 2 -- Traversal & Search

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Spiral Matrix | Medium | 4-boundary pointers simulation | [LC 54](https://leetcode.com/problems/spiral-matrix/) |
| 4 | Search a 2D Matrix | Medium | 1D-flat binary search | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) |
| 5 | Diagonal Traverse | Medium | Parity index sum traversal | [LC 498](https://leetcode.com/problems/diagonal-traverse/) |

## Tier 3 -- Advanced Search

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Search a 2D Matrix II | Medium | Pruning search starting top-right | [LC 240](https://leetcode.com/problems/search-a-2d-matrix-ii/) |
| 7 | Rotate Image | Medium | Transpose + reverse rows | [LC 48](https://leetcode.com/problems/rotate-image/) |

---

## Worked Solution: LC 498 -- Diagonal Traverse

**Key Insight**: Traverse a matrix diagonally in a zigzag fashion.
- The cells along any anti-diagonal have the **same index sum** $s = r + c$.
- Total number of anti-diagonals is $R + C - 1$.
- For each anti-diagonal $s$ from $0$ to $R + C - 2$:
  - If $s$ is **even**, we traverse **upwards**: rows decrease, columns increase.
  - If $s$ is **odd**, we traverse **downwards**: rows increase, columns decrease.
- We can find the bounds of row $r$ for any diagonal $s$ to traverse without storing intermediate values in buckets:
  - Downward: $r \in [\max(0, s - C + 1), \min(s, R - 1)]$. Column is simply $c = s - r$.

```cpp
#include <vector>
#include <algorithm>

using namespace std;

vector<int> findDiagonalOrder(vector<vector<int>>& mat) {
    if (mat.empty()) return {};

    int R = mat.size();
    int C = mat[0].size();
    vector<int> result;

    for (int s = 0; s < R + C - 1; s++) {
        // Find bounds of row index for current sum s
        int startRow = max(0, s - C + 1);
        int endRow = min(s, R - 1);

        vector<int> diagonal;
        for (int r = startRow; r <= endRow; r++) {
            diagonal.push_back(mat[r][s - r]);
        }

        // Even sum index: traverse UP (reverse our natural row iteration order)
        if (s % 2 == 0) {
            reverse(diagonal.begin(), diagonal.end());
        }

        for (int val : diagonal) {
            result.push_back(val);
        }
    }
    return result;
}
// Time Complexity: O(R * C) — visit each cell exactly once
// Space Complexity: O(min(R, C)) for temp diagonal storage
```

---

*<- [[Matrix_Patterns|Operations]] · [[../String_Algorithms/00_Index|String Algorithms ->]]*

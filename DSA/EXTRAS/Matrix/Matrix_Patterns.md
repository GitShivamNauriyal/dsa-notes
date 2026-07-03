---
tags: [dsa, matrix, patterns, flat-mapping, diagonals]
links: ["[[00_Index]]", "[[Matrix_Problems]]"]
---

# Matrix -- Operations & Math

*<- [[00_Index|Index]] · [[Matrix_Problems|Problems ->]]*

---

## 1. Diagonal Index Properties

In an $N \times M$ matrix:
1. **Main Diagonals (Top-Left to Bottom-Right)**:
   - For all cells along any main diagonal, the difference between row and column indices is constant:
     **`r - c = constant`**
   - E.g. cells `(0,0)`, `(1,1)`, `(2,2)` all have `r - c = 0`.
   - Range of values: $-(M-1)$ to $(N-1)$. Adding offset $(M-1)$ normalizes indices to positive.
2. **Anti-Diagonals (Top-Right to Bottom-Left)**:
   - For all cells along any anti-diagonal, the sum of row and column indices is constant:
     **`r + c = constant`**
   - E.g. cells `(0,2)`, `(1,1)`, `(2,0)` all have `r + c = 2`.
   - Range of values: $0$ to $N + M - 2$.

---

## 2. Flat Array Index Mapping (2D to 1D)
When performing search operations (e.g. binary search on a sorted matrix) or storing a matrix in a contiguous 1D memory array:
- Let matrix have dimensions $R \times C$.
- **2D to 1D Mapping**:
  $$\text{flatIndex} = r \times C + c$$
- **1D to 2D Reconstruction**:
  $$r = \text{flatIndex} / C$$
  $$c = \text{flatIndex} \bmod C$$

---

## 3. Neighbor Direction Offsets (DFS/BFS)
Instead of writing 4 separate duplicate blocks of check conditions for up, down, left, and right movements, maintain direction arrays:

```cpp
// Coordinate delta offsets for 4-directional moves (Up, Down, Left, Right)
const int dr[] = {-1, 1, 0, 0};
const int dc[] = {0, 0, -1, 1};

// Iterate through neighbors
for (int i = 0; i < 4; i++) {
    int nextRow = r + dr[i];
    int nextCol = c + dc[i];
    
    // Bounds check
    if (nextRow >= 0 && nextRow < rows && nextCol >= 0 && nextCol < cols) {
        // Process neighbor (nextRow, nextCol)
    }
}
```

---

## 4. Matrix Transposition
- **Concept**: Swapping elements along the main diagonal (`matrix[i][j]` with `matrix[j][i]`). 
- Note: Only traverse elements above the main diagonal (`j > i`) to avoid swapping values back to their original positions.

```cpp
#include <vector>
#include <algorithm>

void transpose(vector<vector<int>>& matrix) {
    int n = matrix.size();
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            swap(matrix[i][j], matrix[j][i]);
        }
    }
}
```

---

*<- [[00_Index]] · [[Matrix_Problems|Problems ->]]*

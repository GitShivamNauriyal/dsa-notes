---
tags: [dsa, math, geometry, problems, exercises]
links: ["[[17_Math_and_Geometry_Index]]", "[[17_Math_and_Geometry_Patterns]]", "[[17_Math_and_Geometry_Tricky]]"]
---

# Math & Geometry -- Problems & Exercises

*<- [[17_Math_and_Geometry_Patterns|Patterns]] · [[17_Math_and_Geometry_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Count Primes | Medium | Sieve of Eratosthenes | [LC 204](https://leetcode.com/problems/count-primes/) |
| 2 | Happy Number | Easy | Floyd's cycle detection | [LC 202](https://leetcode.com/problems/happy-number/) |
| 3 | Plus One | Easy | Carry propagation | [LC 66](https://leetcode.com/problems/plus-one/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Rotate Image | Medium | Transpose + Reverse | [LC 48](https://leetcode.com/problems/rotate-image/) |
| 5 | Spiral Matrix | Medium | 4-boundary pointers simulation | [LC 54](https://leetcode.com/problems/spiral-matrix/) |
| 6 | Pow(x, n) | Medium | Binary Exponentiation | [LC 50](https://leetcode.com/problems/powx-n/) |
| 7 | Set Matrix Zeroes | Medium | Row/col sentinel flags | [LC 73](https://leetcode.com/problems/set-matrix-zeroes/) |

## Tier 3 -- Number Theory & Grids

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Multiply Strings | Medium | Grade-school multiplication logic | [LC 43](https://leetcode.com/problems/multiply-strings/) |
| 9 | Robot Bounded In Circle | Medium | State updates (direction + coords) | [LC 1041](https://leetcode.com/problems/robot-bounded-in-circle/) |
| 10 | GCD of Array | Easy | Iterative Euclidean GCD | [GFG](https://www.geeksforgeeks.org/gcd-of-two-numbers-connectivity/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 11 | Max Points on a Line | Hard | Slope hashing using reduced GCD pairs | [LC 149](https://leetcode.com/problems/max-points-on-a-line/) |
| 12 | Fraction to Recurring Decimal | Medium | Remainder hashing for recurring cycle | [LC 166](https://leetcode.com/problems/fraction-to-recurring-decimal/) |

---

## Worked Solution 1: LC 50 -- Pow(x, n) (Binary Exponentiation)

**Key Insight**: Instead of multiplying $x$ by $n$ times ($O(n)$), we can divide the exponent by 2 at each step:
- If $n$ is even: $x^n = (x^2)^{n/2}$
- If $n$ is odd: $x^n = x \times (x^2)^{(n-1)/2}$
- This reduces the operations to $O(\log n)$.
- **Negative Exponent**: $x^{-n} = (1/x)^n$. We must handle integer overflow of `-n` when $n = -2^{31}$ (cast to `long long`).

```cpp
double myPow(double x, int n) {
    long long N = n;
    if (N < 0) {
        x = 1 / x;
        N = -N;
    }

    double result = 1.0;
    double currentProduct = x;

    while (N > 0) {
        if (N % 2 == 1) {
            result *= currentProduct; // multiply odd remaining factor
        }
        currentProduct *= currentProduct; // square the base
        N /= 2;
    }
    return result;
}
// Time Complexity: O(log n)
// Space Complexity: O(1)
```

---

## Worked Solution 2: LC 54 -- Spiral Matrix

**Key Insight**: Traversal of a 2D matrix in spiral order.
- Maintain 4 boundaries: `top`, `bottom`, `left`, `right`.
- Traverse top row (left to right), increment `top`.
- Traverse right column (top to bottom), decrement `right`.
- Traverse bottom row (right to left, checking `top <= bottom`), decrement `bottom`.
- Traverse left column (bottom to top, checking `left <= right`), increment `left`.
- Repeat until boundaries overlap.

```cpp
#include <vector>

vector<int> spiralOrder(vector<vector<int>>& matrix) {
    if (matrix.empty()) return {};

    int top = 0;
    int bottom = matrix.size() - 1;
    int left = 0;
    int right = matrix[0].size() - 1;

    vector<int> result;

    while (top <= bottom && left <= right) {
        // 1. Traverse Right
        for (int c = left; c <= right; c++) result.push_back(matrix[top][c]);
        top++;

        // 2. Traverse Down
        for (int r = top; r <= bottom; r++) result.push_back(matrix[r][right]);
        right--;

        // 3. Traverse Left
        if (top <= bottom) {
            for (int c = right; c >= left; c--) result.push_back(matrix[bottom][c]);
            bottom--;
        }

        // 4. Traverse Up
        if (left <= right) {
            for (int r = bottom; r >= top; r--) result.push_back(matrix[r][left]);
            left++;
        }
    }
    return result;
}
// Time Complexity: O(m * n)
// Space Complexity: O(1) (excluding output vector)
```

---

*<- [[17_Math_and_Geometry_Patterns|Patterns]] · [[17_Math_and_Geometry_Tricky|Tricky ->]]*

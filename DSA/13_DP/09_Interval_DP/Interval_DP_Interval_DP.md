---
tags: [dsa, dp, interval-dp, stone-game, game-theory]
links: ["[[09_Interval_DP_Index]]", "[[Interval_DP_Problems]]"]
---

# Interval DP -- Foundations & Piles

*<- [[09_Interval_DP_Index|Interval Index]] · [[Interval_DP_Problems|Problems ->]]*

---

## What is Interval DP?
Interval DP (often linked with game theory or min-max optimization) solves for states defined on a contiguous interval `[i, j]`.
- State transitions typically shrink the interval from both ends: `[i, j]` transitions to either `[i+1, j]` (moving left boundary) or `[i, j-1]` (moving right boundary).

```
Interval:   [ i ..................... j ]
Transitions:    [ i+1 ............... j ] (shrink left)
             [ i ................. j-1 ] (shrink right)
```

---

## Stone Game (LC 877) / Predict the Winner (LC 486)

**The Problem**: There is an array of piles of stones `piles`. Two players take turns picking either the first or the last pile in the array. Return the maximum score margin the first player can win by.

### State Formulation
Let `solve(i, j)` be the **maximum net score difference** the current player can achieve from piles `[i..j]`.
- If the current player picks `piles[i]`, they gain `piles[i]` score. The next player plays optimally on `[i+1..j]`, so the net score margin becomes: `piles[i] - solve(i+1, j)`.
- If the current player picks `piles[j]`, the net score margin becomes: `piles[j] - solve(i, j-1)`.
- The current player wants to maximize this difference.
- **Recurrence Relation**: `solve(i, j) = max(piles[i] - solve(i+1, j), piles[j] - solve(i, j-1))`
- **Base Case**: `i == j` (only 1 pile left. Player must pick it). Return `piles[i]`.

---

## C++ Implementation (Memoization)

```cpp
#include <vector>
#include <algorithm>

int solvePiles(int i, int j, const vector<int>& piles, vector<vector<int>>& memo) {
    if (i == j) return piles[i]; // base case: 1 pile left
    if (memo[i][j] != -1) return memo[i][j];

    // Max margin by taking left or right pile
    int pickLeft  = piles[i] - solvePiles(i + 1, j, piles, memo);
    int pickRight = piles[j] - solvePiles(i, j - 1, piles, memo);

    return memo[i][j] = max(pickLeft, pickRight);
}

bool predictTheWinner(vector<int>& piles) {
    int n = piles.size();
    vector<vector<int>> memo(n, vector<int>(n, -1));
    // First player wins if net score margin >= 0
    return solvePiles(0, n - 1, piles, memo) >= 0;
}
// Time Complexity: O(n^2) — n^2 states, O(1) transition
// Space Complexity: O(n^2)
```

---

*<- [[09_Interval_DP_Index|Interval Index]] · [[Interval_DP_Problems|Problems ->]]*

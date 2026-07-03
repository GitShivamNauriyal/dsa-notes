---
tags: [dsa, dp, partition-dp, strange-printer]
links: ["[[07_Partition_DP_Index]]", "[[Partition_DP_Burst_Balloons]]", "[[Partition_DP_Problems]]"]
---

# Partition DP -- Strange Printer (LC 664)

*<- [[Partition_DP_Burst_Balloons|Burst Balloons]] · [[Partition_DP_Problems|Problems ->]]*

---

## The Problem
There is a strange printer with the following two special properties:
1. The printer can only print a sequence of **the same character** each time.
2. At each turn, the printer can print new characters starting from and ending at any place and will cover the original characters existing there.

Given a string `s`, return the minimum number of turns the printer needs to print it.

---

## State Formulation & Intuition
Let `solve(i, j)` be the minimum turns to print substring `s[i..j]`.

### Base Case
- `i == j`: 1 turn is needed (single character).

### Recurrence Transitions
1. **Naive Transition**: We print the first character `s[i]` on its own turn, and then print the rest of the string `s[i+1..j]`.
   `turns = 1 + solve(i + 1, j)`
2. **Matching Character Optimization (Partitioning)**: 
   - If there is any character `s[k]` in the range `s[i+1..j]` such that `s[k] == s[i]`:
   - We could have printed `s[i]` spanning all the way from `i` to `k` on the *very first turn* for free, and then layered the other printed characters on top of it.
   - This splits the range into two parts: `[i, k-1]` (where `s[i]` covers up to `k`) and `[k+1, j]` (the remaining suffix).
   - **Formula**: `turns = solve(i, k-1) + solve(k+1, j)` for any $k \in [i+1, j]$ where $\text{s}[k] == \text{s}[i]$.

---

## C++ Implementation

```cpp
#include <string>
#include <vector>
#include <algorithm>

int solvePrinter(int i, int j, const string& s, vector<vector<int>>& memo) {
    if (i > j) return 0;
    if (i == j) return 1; // 1 character takes 1 print turn
    if (memo[i][j] != -1) return memo[i][j];

    // Naive option: print s[i] first, then print s[i+1..j]
    int minTurns = 1 + solvePrinter(i + 1, j, s, memo);

    // Try to find matching characters s[k] == s[i] to optimize printing
    for (int k = i + 1; k <= j; k++) {
        if (s[k] == s[i]) {
            int turns = solvePrinter(i, k - 1, s, memo) + solvePrinter(k + 1, j, s, memo);
            minTurns = min(minTurns, turns);
        }
    }
    return memo[i][j] = minTurns;
}

int strangePrinter(string s) {
    if (s.empty()) return 0;
    int n = s.size();
    vector<vector<int>> memo(n, vector<int>(n, -1));
    return solvePrinter(0, n - 1, s, memo);
}
// Time Complexity: O(n^3)
// Space Complexity: O(n^2)
```

---

*<- [[Partition_DP_Burst_Balloons|Burst Balloons]] · [[Partition_DP_Problems|Problems ->]]*

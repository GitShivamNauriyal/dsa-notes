---
tags: [dsa, dp, strings, lcs]
links: ["[[04_DP_on_Strings_Index]]", "[[DP_on_Strings_LIS]]"]
---

# DP on Strings -- Longest Common Subsequence (LCS)

*<- [[04_DP_on_Strings_Index|Strings Index]] · [[DP_on_Strings_LIS|Longest Increasing Subsequence ->]]*

---

## The Problem
Given two strings `s1` and `s2`, return the length of their longest common subsequence. A subsequence is a sequence that can be derived from another string by deleting some or no characters without changing the order of the remaining characters.

---

## State Formulation
Let `dp[i][j]` be the length of the LCS of prefixes `s1[0..i-1]` and `s2[0..j-1]`.
For character `s1[i-1]` and `s2[j-1]`, we have two cases:
1. **Characters Match (`s1[i-1] == s2[j-1]`)**: They contribute 1 to the LCS length. We solve for remaining prefixes.
   `dp[i][j] = 1 + dp[i-1][j-1]`
2. **Characters Do Not Match**: We try excluding either `s1[i-1]` or `s2[j-1]`, taking the maximum.
   `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

- **Base cases**: `dp[0][j] = 0` (first string is empty), `dp[i][0] = 0` (second string is empty).

---

### A. Tabulation (2D Array)
- **Time Complexity**: $O(|s1| \times |s2|)$.
- **Space Complexity**: $O(|s1| \times |s2|)$.

```cpp
#include <string>
#include <vector>
#include <algorithm>

int lcs2D(const string& s1, const string& s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```

---

### B. Space Optimization (2 Rows)
- **Core Intuition**: `dp[i][j]` only reads from the previous row `i-1` and the current row `i`. We only need to store `prevRow` and `currRow`.
- **Time Complexity**: $O(m \times n)$.
- **Space Complexity**: $O(n)$ where $n$ is length of `s2`.

```cpp
#include <string>
#include <vector>
#include <algorithm>

int lcsSpaceOptimized(const string& s1, const string& s2) {
    int m = s1.size(), n = s2.size();
    vector<int> prev(n + 1, 0);
    vector<int> curr(n + 1, 0);

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i - 1] == s2[j - 1]) {
                curr[j] = 1 + prev[j - 1];
            } else {
                curr[j] = max(prev[j], curr[j - 1]);
            }
        }
        prev = curr; // move down a row
    }
    return prev[n];
}
```

---

## Reconstructing the LCS String (Path Backtracking)

To print the actual common subsequence characters, backtrack from `dp[m][n]` upwards and leftwards:
- If `s1[i-1] == s2[j-1]`, this character is part of LCS. Add to result, move diagonally up-left: `i--, j--`.
- Else, move in the direction of the larger neighbor: if `dp[i-1][j] > dp[i][j-1]`, move up (`i--`), else move left (`j--`).

```cpp
#include <string>
#include <vector>
#include <algorithm>

string printLCS(const string& s1, const string& s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i - 1] == s2[j - 1]) dp[i][j] = 1 + dp[i - 1][j - 1];
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }

    // Path backtracking
    string lcsStr = "";
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (s1[i - 1] == s2[j - 1]) {
            lcsStr.push_back(s1[i - 1]);
            i--; j--;
        } else if (dp[i - 1][j] > dp[i][j - 1]) {
            i--;
        } else {
            j--;
        }
    }
    reverse(lcsStr.begin(), lcsStr.end());
    return lcsStr;
}
```

### Path Reconstruction Visual
For `s1 = "abcde"`, `s2 = "ace"`:
- LCS string is `"ace"`.
- Backtrack steps:
  - `dp[5][3]` ("e" vs "e"): match! Add 'e', move to `dp[4][2]`.
  - `dp[4][2]` ("d" vs "c"): no match. `dp[3][2] > dp[4][1]` (2 > 1) -> move up to `dp[3][2]`.
  - `dp[3][2]` ("c" vs "c"): match! Add 'c', move to `dp[2][1]`.
  - `dp[2][1]` ("b" vs "a"): no match. `dp[1][1] > dp[2][0]` (1 > 0) -> move up to `dp[1][1]`.
  - `dp[1][1]` ("a" vs "a"): match! Add 'a', move to `dp[0][0]`.
- Reversed result: `"ace"`.

---

*<- [[04_DP_on_Strings_Index|Strings Index]] · [[DP_on_Strings_LIS|Longest Increasing Subsequence ->]]*

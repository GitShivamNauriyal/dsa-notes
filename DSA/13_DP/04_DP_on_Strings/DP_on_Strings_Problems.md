---
tags: [dsa, dp, strings, problems, exercises]
links: ["[[04_DP_on_Strings_Index]]", "[[DP_on_Strings_Palindrome_DP]]", "[[../05_DP_on_Stocks/05_DP_on_Stocks_Index]]"]
---

# DP on Strings -- Problems & Exercises

*<- [[DP_on_Strings_Palindrome_DP|Palindromic Substrings]] · [[../05_DP_on_Stocks/05_DP_on_Stocks_Index|DP on Stocks ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Longest Common Subsequence | Medium | Classic LCS template | [LC 1143](https://leetcode.com/problems/longest-common-subsequence/) |
| 2 | Longest Palindromic Subsequence | Medium | LCS on string and reverse | [LC 516](https://leetcode.com/problems/longest-palindromic-subsequence/) |
| 3 | Delete Operation for Two Strings | Medium | LCS difference derivation | [LC 583](https://leetcode.com/problems/delete-operation-for-two-strings/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Longest Increasing Subsequence | Medium | Card patience sort / $O(n^2)$ DP | [LC 300](https://leetcode.com/problems/longest-increasing-subsequence/) |
| 5 | Edit Distance | Hard | Insertion, deletion, replacement | [LC 72](https://leetcode.com/problems/edit-distance/) |
| 6 | Palindromic Substrings | Medium | Center expansion count | [LC 647](https://leetcode.com/problems/palindromic-substrings/) |
| 7 | Longest Palindromic Substring | Medium | Center expansion bounds check | [LC 5](https://leetcode.com/problems/longest-palindromic-substring/) |

## Tier 3 -- Matchings & Subsequences

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Distinct Subsequences | Hard | Matching matching pathways count | [LC 115](https://leetcode.com/problems/distinct-subsequences/) |
| 9 | Interleaving String | Medium | 3D grid pathways traversal | [LC 97](https://leetcode.com/problems/interleaving-string/) |
| 10 | Shortest Common Supersequence | Hard | LCS path reconstruction | [LC 1092](https://leetcode.com/problems/shortest-common-supersequence/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 11 | Wildcard Matching | Hard | `?` and `*` wildcard states | [LC 44](https://leetcode.com/problems/wildcard-matching/) |
| 12 | Regular Expression Matching | Hard | `.` and `*` (regex wildcard logic) | [LC 10](https://leetcode.com/problems/regular-expression-matching/) |
| 13 | Russian Doll Envelopes | Hard | 2D LIS (Sort width, LIS height) | [LC 354](https://leetcode.com/problems/russian-doll-envelopes/) |

---

## Worked Solution 1: LC 115 -- Distinct Subsequences

**Key Insight**: Given two strings `s` and `t`, return the number of distinct subsequences of `s` which equals `t`.
- Let `dp[i][j]` be the number of distinct subsequences in prefix `s[0..i-1]` that equal `t[0..j-1]`.
- For characters `s[i-1]` and `t[j-1]`:
  - If they do not match (`s[i-1] != t[j-1]`), we must exclude `s[i-1]`.
    `dp[i][j] = dp[i-1][j]`
  - If they match (`s[i-1] == t[j-1]`), we can either:
    1. Match them together: `dp[i-1][j-1]`
    2. Exclude `s[i-1]`: `dp[i-1][j]` (since `s` might have other copies of the same character earlier).
    - **Sum them**: `dp[i][j] = dp[i-1][j-1] + dp[i-1][j]`
- **Base Case**: `dp[i][0] = 1` (empty string `t` can always be formed by deleting all characters in `s`).

```cpp
#include <string>
#include <vector>

int numDistinct(string s, string t) {
    int m = s.size(), n = t.size();
    // Use double or unsigned long long to prevent integer overflow on large testcases
    vector<double> dp(n + 1, 0);
    dp[0] = 1; // base case

    for (int i = 1; i <= m; i++) {
        // Loop backwards for space optimization to prevent reusing the same row update
        for (int j = n; j >= 1; j--) {
            if (s[i - 1] == t[j - 1]) {
                dp[j] = dp[j] + dp[j - 1];
            }
        }
    }
    return (int)dp[n];
}
// Time Complexity: O(m * n)
// Space Complexity: O(n)
```

---

## Worked Solution 2: LC 44 -- Wildcard Matching

**Why Tricky**: Match string `s` with pattern `p` containing wildcards:
- `?` Matches any single character.
- `*` Matches any sequence of characters (including empty sequence).

**State**: Let `dp[i][j]` be a boolean indicating if prefix `s[0..i-1]` matches pattern prefix `p[0..j-1]`.
- **Recurrence**:
  - If `p[j-1] == s[i-1]` or `p[j-1] == '?'`: `dp[i][j] = dp[i-1][j-1]`
  - If `p[j-1] == '*'`: `*` can either:
    1. Match an empty sequence (meaning we ignore `*`): `dp[i][j-1]`
    2. Match one or more characters (meaning we consume `s[i-1]` and keep `*` active): `dp[i-1][j]`
    - **Formula**: `dp[i][j] = dp[i][j-1] || dp[i-1][j]`
- **Base Case**: `dp[0][0] = true`. `dp[0][j]` is true if and only if `p[0..j-1]` contains only `*` (which can match empty sequences).

```cpp
#include <string>
#include <vector>

bool isMatch(string s, string p) {
    int m = s.size(), n = p.size();
    vector<vector<bool>> dp(m + 1, vector<bool>(n + 1, false));
    dp[0][0] = true;

    // Base case: empty string matching patterns of only '*'
    for (int j = 1; j <= n; j++) {
        if (p[j - 1] == '*') {
            dp[0][j] = dp[0][j - 1];
        }
    }

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (p[j - 1] == s[i - 1] || p[j - 1] == '?') {
                dp[i][j] = dp[i - 1][j - 1];
            } else if (p[j - 1] == '*') {
                // Match empty or consume current character
                dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
            }
        }
    }
    return dp[m][n];
}
// Time Complexity: O(m * n)
// Space Complexity: O(m * n)
```

---

*<- [[DP_on_Strings_Palindrome_DP|Palindromic Substrings]] · [[../05_DP_on_Stocks/05_DP_on_Stocks_Index|DP on Stocks ->]]*

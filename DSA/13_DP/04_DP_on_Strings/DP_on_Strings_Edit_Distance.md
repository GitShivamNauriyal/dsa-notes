---
tags: [dsa, dp, strings, edit-distance]
links: ["[[04_DP_on_Strings_Index]]", "[[DP_on_Strings_LIS]]", "[[DP_on_Strings_Palindrome_DP]]"]
---

# DP on Strings -- Edit Distance

*<- [[DP_on_Strings_LIS|Longest Increasing Subsequence]] · [[DP_on_Strings_Palindrome_DP|Palindromic Substrings ->]]*

---

## The Problem
Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` to `word2`.
You have the following three operations permitted on a word:
1. Insert a character
2. Delete a character
3. Replace a character

---

## State Formulation
Let `dp[i][j]` be the minimum operations to convert prefix `word1[0..i-1]` to `word2[0..j-1]`.
For characters `word1[i-1]` and `word2[j-1]`:
- **If characters match (`word1[i-1] == word2[j-1]`)**: No operation is needed.
  `dp[i][j] = dp[i-1][j-1]`
- **If characters do not match**, we must try all three operations and take the minimum:
  1. **Insert**: Add `word2[j-1]` to `word1`. The remaining subproblem is converting `word1[0..i-1]` to `word2[0..j-2]`.
     Cost = `1 + dp[i][j-1]`
  2. **Delete**: Remove `word1[i-1]` from `word1`. The remaining subproblem is converting `word1[0..i-2]` to `word2[0..j-1]`.
     Cost = `1 + dp[i-1][j]`
  3. **Replace**: Swap `word1[i-1]` with `word2[j-1]`. The remaining subproblem is converting `word1[0..i-2]` to `word2[0..j-2]`.
     Cost = `1 + dp[i-1][j-1]`

- **Recurrence Relation**: `dp[i][j] = 1 + min({dp[i][j-1], dp[i-1][j], dp[i-1][j-1]})`
- **Base cases**:
  - `dp[0][j] = j` (empty `word1` requires `j` insertions to become `word2`)
  - `dp[i][0] = i` (converting `word1` to empty `word2` requires `i` deletions)

---

### Tabulation (2D Array)
- **Time Complexity**: $O(|word1| \times |word2|)$.
- **Space Complexity**: $O(|word1| \times |word2|)$.

```cpp
#include <string>
#include <vector>
#include <algorithm>

int minDistance(string word1, string word2) {
    int m = word1.size(), n = word2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    // Base Cases
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1[i - 1] == word2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                // min of: Insert, Delete, Replace
                dp[i][j] = 1 + min({
                    dp[i][j - 1],   // Insert
                    dp[i - 1][j],   // Delete
                    dp[i - 1][j - 1] // Replace
                });
            }
        }
    }
    return dp[m][n];
}
```

---

### Space-Optimized Implementation (2 Rows)
- Since each calculation only looks at the current row and the previous row, we can reduce space complexity to $O(n)$ where $n$ is length of `word2`.

```cpp
#include <string>
#include <vector>
#include <algorithm>

int minDistanceSpaceOptimized(string word1, string word2) {
    int m = word1.size(), n = word2.size();
    vector<int> prev(n + 1, 0);
    vector<int> curr(n + 1, 0);

    // Initialize base case row
    for (int j = 0; j <= n; j++) prev[j] = j;

    for (int i = 1; i <= m; i++) {
        curr[0] = i; // base case col for current row
        for (int j = 1; j <= n; j++) {
            if (word1[i - 1] == word2[j - 1]) {
                curr[j] = prev[j - 1];
            } else {
                curr[j] = 1 + min({
                    curr[j - 1],   // Insert
                    prev[j],       // Delete
                    prev[j - 1]    // Replace
                });
            }
        }
        prev = curr;
    }
    return prev[n];
}
// Time Complexity: O(m * n)
// Space Complexity: O(n)
```

---

*<- [[DP_on_Strings_LIS|Longest Increasing Subsequence]] · [[DP_on_Strings_Palindrome_DP|Palindromic Substrings ->]]*

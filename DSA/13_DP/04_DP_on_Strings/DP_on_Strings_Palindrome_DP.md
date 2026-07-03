---
tags: [dsa, dp, strings, palindrome, lps]
links: ["[[04_DP_on_Strings_Index]]", "[[DP_on_Strings_Edit_Distance]]", "[[DP_on_Strings_Problems]]"]
---

# DP on Strings -- Palindromic Substrings & Sequences

*<- [[DP_on_Strings_Edit_Distance|Edit Distance]] · [[DP_on_Strings_Problems|Problems ->]]*

---

## 1. Longest Palindromic Subsequence (LPS) (LC 516)

**The Problem**: Given a string `s`, find the longest palindromic subsequence's length in `s`.

### Elegant Reduction to LCS
A palindromic subsequence reads the same forwards and backwards. Therefore, the Longest Palindromic Subsequence of `s` is simply the **Longest Common Subsequence of `s` and its reversed string `reverse(s)`**!

```cpp
#include <string>
#include <vector>
#include <algorithm>

int longestPalindromeSubseq(string s) {
    string r = s;
    reverse(r.begin(), r.end());
    
    // Solve LCS on s and r
    int n = s.size();
    vector<int> prev(n + 1, 0);
    vector<int> curr(n + 1, 0);

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            if (s[i - 1] == r[j - 1]) {
                curr[j] = 1 + prev[j - 1];
            } else {
                curr[j] = max(prev[j], curr[j - 1]);
            }
        }
        prev = curr;
    }
    return prev[n];
}
// Time Complexity: O(n^2)
// Space Complexity: O(n)
```

---

## 2. Palindromic Substrings (LC 647)

**The Problem**: Given a string `s`, return the number of palindromic substrings in it. A substring is a contiguous sequence of characters within the string.

### Method A: 2D DP Table (Tabulation)
- Let `dp[i][j]` be a boolean indicating if substring `s[i..j]` is a palindrome.
- **Recurrence**: `s[i..j]` is a palindrome if:
  1. Outer characters match: `s[i] == s[j]`
  2. Inner substring is also a palindrome: `dp[i+1][j-1] == true` (only check if length > 2).
- **Formula**: `dp[i][j] = (s[i] == s[j]) && (j - i < 2 || dp[i+1][j-1])`
- We fill the table by iterating over string lengths (from 1 to $n$).

```cpp
#include <string>
#include <vector>

int countSubstringsDP(string s) {
    int n = s.size();
    vector<vector<bool>> dp(n, vector<bool>(n, false));
    int count = 0;

    // Fill table diagonally (or backwards for i, forwards for j)
    for (int i = n - 1; i >= 0; i--) {
        for (int j = i; j < n; j++) {
            if (s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1])) {
                dp[i][j] = true;
                count++;
            }
        }
    }
    return count;
}
// Time Complexity: O(n^2)
// Space Complexity: O(n^2)
```

### Method B: Expand Around Center (Optimal)
- **Intuition**: Every palindromic substring has a middle point. We can iterate through all possible middle points (there are $2n - 1$ centers: $n$ single-character centers and $n-1$ two-character centers) and expand outwards as long as characters match.
- **Time Complexity**: $O(n^2)$ (fast in practice, no array initialization).
- **Space Complexity**: $O(1)$ (massive space improvement!).

```cpp
#include <string>

int expandFromCenter(const string& s, int left, int right) {
    int count = 0;
    while (left >= 0 && right < (int)s.size() && s[left] == s[right]) {
        count++;
        left--;  // expand left
        right++; // expand right
    }
    return count;
}

int countSubstrings(string s) {
    int totalCount = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        // Odd length palindromes (center is s[i])
        totalCount += expandFromCenter(s, i, i);
        // Even length palindromes (center is between s[i] and s[i+1])
        totalCount += expandFromCenter(s, i, i + 1);
    }
    return totalCount;
}
```

---

## 3. Longest Palindromic Substring (LC 5)

Using the same **Expand Around Center** logic, we can keep track of the longest palindrome substring range.

```cpp
#include <string>

pair<int, int> expandSubstring(const string& s, int left, int right) {
    while (left >= 0 && right < (int)s.size() && s[left] == s[right]) {
        left--;
        right++;
    }
    return {left + 1, right - 1}; // return valid bounds
}

string longestPalindrome(string s) {
    if (s.empty()) return "";
    int start = 0, end = 0;

    for (int i = 0; i < (int)s.size(); i++) {
        auto [l1, r1] = expandSubstring(s, i, i);
        if (r1 - l1 > end - start) { start = l1; end = r1; }

        auto [l2, r2] = expandSubstring(s, i, i + 1);
        if (r2 - l2 > end - start) { start = l2; end = r2; }
    }
    return s.substr(start, end - start + 1);
}
// Time Complexity: O(n^2)
// Space Complexity: O(1)
```

---

*<- [[DP_on_Strings_Edit_Distance|Edit Distance]] · [[DP_on_Strings_Problems|Problems ->]]*

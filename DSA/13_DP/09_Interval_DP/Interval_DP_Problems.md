---
tags: [dsa, dp, interval-dp, problems, exercises]
links: ["[[09_Interval_DP_Index]]", "[[Interval_DP_Interval_DP]]", "[[../14_Greedy/14_Greedy_Index]]"]
---

# Interval DP -- Problems & Exercises

*<- [[Interval_DP_Interval_DP|Interval DP Foundations]] · [[../14_Greedy/14_Greedy_Index|Greedy ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Predict the Winner | Medium | Minimax net score margin | [LC 486](https://leetcode.com/problems/predict-the-winner/) |
| 2 | Stone Game | Medium | First-player win check | [LC 877](https://leetcode.com/problems/stone-game/) |

## Tier 2 -- Core Palindromes

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Longest Palindromic Subsequence | Medium | LCS reduction | [LC 516](https://leetcode.com/problems/longest-palindromic-subsequence/) |
| 4 | Minimum Insertion Steps to Make Palindrome | Hard | Length - LPS | [LC 1312](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/) |
| 5 | Valid Palindrome III | Hard | K-palindrome check | [LC 1216](https://leetcode.com/problems/valid-palindrome-iii/) |

## Tier 3 -- Game Theory & Merges

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Stone Game II | Medium | 3D state variable choices `(i, M)` | [LC 1140](https://leetcode.com/problems/stone-game-ii/) |
| 7 | Minimum Cost to Merge Stones | Hard | Range merge cost partitions | [LC 1000](https://leetcode.com/problems/minimum-cost-to-merge-stones/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Optimal BST | Hard | Min search cost range partitions | [GFG](https://www.geeksforgeeks.org/optimal-binary-search-tree-dp-24/) |
| 9 | Strange Printer II | Hard | Topological sort + grid constraints | [LC 1591](https://leetcode.com/problems/strange-printer-ii/) |

---

## Worked Solution: LC 1312 -- Minimum Insertion Steps to Make a String Palindrome

**Key Insight**: We want to find the minimum insertions to make string `s` a palindrome.
- Find the **Longest Palindromic Subsequence (LPS)** of the string.
- Any character that is *not* part of the LPS will need a matching character inserted somewhere else in the string.
- Therefore, the minimum insertions required is simply: **`s.length() - LPS(s)`**!

```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int minInsertions(string s) {
    int n = s.size();
    string r = s;
    reverse(r.begin(), r.end());

    // Solve LCS on s and r
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
    int lpsLength = prev[n];
    return n - lpsLength;
}
// Time Complexity: O(n^2)
// Space Complexity: O(n)
```

---

*<- [[Interval_DP_Interval_DP|Interval DP Foundations]] · [[../14_Greedy/14_Greedy_Index|Greedy ->]]*

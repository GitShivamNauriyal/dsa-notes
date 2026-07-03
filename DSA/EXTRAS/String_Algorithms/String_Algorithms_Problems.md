---
tags: [dsa, strings, problems, substring-first-occurrence]
links: ["[[00_Index]]", "[[String_Algorithms_Patterns]]", "[[../Number_Theory/00_Index]]"]
---

# String Algorithms -- Problems & Exercises

*<- [[String_Algorithms_Patterns|Substring Matching]] · [[../Number_Theory/00_Index|Number Theory ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Implement strStr() | Easy | First occurrence substring search | [LC 28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) |
| 2 | Repeated Substring Pattern | Easy | KMP LPS matching divisor property | [LC 459](https://leetcode.com/problems/repeated-substring-pattern/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Longest Happy Prefix | Hard | Largest LPS value lookup | [LC 1392](https://leetcode.com/problems/longest-happy-prefix/) |
| 4 | Rabin-Karp substring match | Medium | Rolling hash matching | [GFG](https://www.geeksforgeeks.org/rabin-karp-algorithm-for-pattern-searching/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 5 | Shortest Palindrome | Hard | LPS array on `s + "#" + reverse(s)` | [LC 214](https://leetcode.com/problems/shortest-palindrome/) |
| 6 | Count occurrences of pattern | Medium | KMP occurrences counter | [GFG](https://www.geeksforgeeks.org/kmp-algorithm-for-pattern-searching/) |

---

## Worked Solution: LC 28 -- First Substring Occurrence (using KMP)

**Key Insight**: Return the index of the first occurrence of `needle` in `haystack`.
- Using KMP substring search algorithm, we can retrieve the index in $O(H + N)$ time.
- If `needle` is empty, return 0.
- If we find a match `j == needle.size()`, return the starting index `i - j`.

```cpp
#include <string>
#include <vector>

using namespace std;

class Solution {
    vector<int> getLPS(const string& pat) {
        int m = pat.size();
        vector<int> lps(m, 0);
        int len = 0;
        int i = 1;

        while (i < m) {
            if (pat[i] == pat[len]) {
                len++;
                lps[i] = len;
                i++;
            } else {
                if (len != 0) {
                    len = lps[len - 1];
                } else {
                    lps[i] = 0;
                    i++;
                }
            }
        }
        return lps;
    }

public:
    int strStr(string haystack, string needle) {
        if (needle.empty()) return 0;
        int n = haystack.size();
        int m = needle.size();

        vector<int> lps = getLPS(needle);
        int i = 0; // index for haystack
        int j = 0; // index for needle

        while (i < n) {
            if (haystack[i] == needle[j]) {
                i++; j++;
            }

            if (j == m) {
                return i - j; // found first match
            } else if (i < n && haystack[i] != needle[j]) {
                if (j != 0) {
                    j = lps[j - 1];
                } else {
                    i++;
                }
            }
        }
        return -1; // no match found
    }
};
// Time Complexity: O(H + N) where H is haystack size, N is needle size
// Space Complexity: O(N) for LPS table
```

---

*<- [[String_Algorithms_Patterns|Substring Matching]] · [[../Number_Theory/00_Index|Number Theory ->]]*

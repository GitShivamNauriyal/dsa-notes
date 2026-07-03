---
tags: [dsa, two-pointers, problems, exercises]
links: ["[[02_Two_Pointers_Index]]", "[[02_Two_Pointers_Patterns]]", "[[02_Two_Pointers_Tricky]]"]
---

# Two Pointers -- Problems & Exercises

*<- [[02_Two_Pointers_Patterns|Patterns]] · [[02_Two_Pointers_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Valid Palindrome | Easy | Opposite ends, skip non-alnum | [LC 125](https://leetcode.com/problems/valid-palindrome/) |
| 2 | Two Sum II | Easy | Opposite ends on sorted | [LC 167](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) |
| 3 | Squares of Sorted Array | Easy | Opposite ends, fill back | [LC 977](https://leetcode.com/problems/squares-of-a-sorted-array/) |
| 4 | Remove Duplicates I | Easy | Same direction | [LC 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) |
| 5 | Remove Element | Easy | Same direction | [LC 27](https://leetcode.com/problems/remove-element/) |
| 6 | Move Zeroes | Easy | Same direction | [LC 283](https://leetcode.com/problems/move-zeroes/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 7 | Container With Most Water | Medium | Opposite ends, greedy move | [LC 11](https://leetcode.com/problems/container-with-most-water/) |
| 8 | 3Sum | Medium | Sort + fix + opposite ends | [LC 15](https://leetcode.com/problems/3sum/) |
| 9 | 3Sum Closest | Medium | Fix + opposite ends | [LC 16](https://leetcode.com/problems/3sum-closest/) |
| 10 | 4Sum | Medium | Fix two + opposite ends | [LC 18](https://leetcode.com/problems/4sum/) |
| 11 | Remove Duplicates II (allow 2) | Medium | Same direction, count | [LC 80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) |
| 12 | Sort Array by Parity | Easy | Same direction, partition | [LC 905](https://leetcode.com/problems/sort-array-by-parity/) |
| 13 | Kth Largest Element | Medium | QuickSelect | [LC 215](https://leetcode.com/problems/kth-largest-element-in-an-array/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 14 | Trapping Rain Water | Hard | Two pointer (max-left, max-right) | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 15 | Find the Duplicate Number | Medium | Fast-slow (Floyd cycle) | [LC 287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 16 | Boats to Save People | Medium | Greedy + opposite ends | [LC 881](https://leetcode.com/problems/boats-to-save-people/) |
| 17 | Partition Labels | Medium | Extend interval greedily | [LC 763](https://leetcode.com/problems/partition-labels/) |
| 18 | Is Subsequence | Easy | Same direction on two strings | [LC 392](https://leetcode.com/problems/is-subsequence/) |
| 19 | Longest Mountain in Array | Medium | Expand from peak | [LC 845](https://leetcode.com/problems/longest-mountain-in-array/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 20 | Minimum Size Subarray Sum | Medium | Same direction, shrink | [LC 209](https://leetcode.com/problems/minimum-size-subarray-sum/) |
| 21 | Count Pairs with Diff <= K | Medium | Sort + opposite ends | [GFG](https://www.geeksforgeeks.org/count-pairs-difference-equal-k/) |
| 22 | Maximize Distance to Closest | Medium | Two pointer scan | [LC 849](https://leetcode.com/problems/maximize-distance-to-closest-person/) |
| 23 | Count Triangles | Medium | Sort + fix hypotenuse | [GFG](https://www.geeksforgeeks.org/find-number-of-triangles-possible/) |

---

## Worked Solution: LC 881 -- Boats to Save People

> Each boat holds at most 2 people with total weight <= limit. Find minimum boats.

**Why two pointers**: Sort people by weight. Heaviest person pairs with lightest if possible, otherwise goes alone. Greedy + opposite ends.

```cpp
#include <algorithm>
int numRescueBoats(std::vector<int>& people, int limit) {
    std::sort(people.begin(), people.end());
    int left = 0, right = (int)people.size() - 1;
    int boats = 0;
    while (left <= right) {
        if (people[left] + people[right] <= limit) left++;  // pair them
        right--;   // heaviest always goes in a boat (alone or paired)
        boats++;
    }
    return boats;
}
// Time: O(n log n), Space: O(1)
```

---

## Worked Solution: LC 392 -- Is Subsequence

> Check if `s` is a subsequence of `t`.

```cpp
bool isSubsequence(std::string s, std::string t) {
    int i = 0, j = 0;
    while (i < (int)s.size() && j < (int)t.size()) {
        if (s[i] == t[j]) i++;  // matched one character of s
        j++;                     // always advance through t
    }
    return i == (int)s.size();  // did we match all of s?
}
// Time: O(|t|), Space: O(1)
```

**Follow-up** (hard): Given many query strings `s`, all to be checked against same `t`, preprocess `t` with binary search for O(|s| log |t|) per query instead of O(|t|).

```cpp
// Preprocess: for each position j in t and each character c,
// store next[j][c] = next occurrence of c at or after index j
#include <vector>
#include <string>

std::vector<std::vector<int>> preprocess(const std::string& t) {
    int n = t.size();
    // next[j][c] = smallest index >= j where character c appears, or n if none
    std::vector<std::vector<int>> nxt(n + 1, std::vector<int>(26, n));
    for (int j = n - 1; j >= 0; j--) {
        for (int c = 0; c < 26; c++) nxt[j][c] = nxt[j+1][c];
        nxt[j][t[j] - 'a'] = j;
    }
    return nxt;
}

bool isSubsequenceFast(const std::string& s, const std::vector<std::vector<int>>& nxt, int tLen) {
    int pos = 0;
    for (char c : s) {
        if (nxt[pos][c - 'a'] == tLen) return false;
        pos = nxt[pos][c - 'a'] + 1;
    }
    return true;
}
```

---

*<- [[02_Two_Pointers_Patterns|Patterns]] · [[02_Two_Pointers_Tricky|Tricky ->]]*

---
tags: [dsa, sliding-window, problems, exercises]
links: ["[[00_Index]]", "[[01_Patterns]]", "[[03_Tricky]]"]
---

# Sliding Window -- Problems & Exercises

*<- [[01_Patterns|Patterns]] · [[03_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Maximum Average Subarray I | Easy | Fixed window sum | [LC 643](https://leetcode.com/problems/maximum-average-subarray-i/) |
| 2 | Contains Duplicate II | Easy | Fixed window set | [LC 219](https://leetcode.com/problems/contains-duplicate-ii/) |
| 3 | Longest Substring Without Repeating | Medium | Variable window, last-seen map | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Minimum Size Subarray Sum | Medium | Variable window, shrink when valid | [LC 209](https://leetcode.com/problems/minimum-size-subarray-sum/) |
| 5 | Longest Repeating Character Replacement | Medium | maxFreq trick | [LC 424](https://leetcode.com/problems/longest-repeating-character-replacement/) |
| 6 | Permutation in String | Medium | Fixed window freq match | [LC 567](https://leetcode.com/problems/permutation-in-string/) |
| 7 | Sliding Window Maximum | Hard | Monotonic deque | [LC 239](https://leetcode.com/problems/sliding-window-maximum/) |
| 8 | Find All Anagrams in String | Medium | Fixed window freq match | [LC 438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Minimum Window Substring | Hard | Variable window, two maps | [LC 76](https://leetcode.com/problems/minimum-window-substring/) |
| 10 | Longest Subarray of 1s After Deleting One Element | Medium | Variable window, track zeros | [LC 1493](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) |
| 11 | Max Consecutive Ones III | Medium | Variable window, at most k zeros | [LC 1004](https://leetcode.com/problems/max-consecutive-ones-iii/) |
| 12 | Fruit Into Baskets | Medium | At most 2 distinct | [LC 904](https://leetcode.com/problems/fruit-into-baskets/) |
| 13 | Substrings of Size 3 with Distinct Chars | Easy | Fixed window distinct check | [LC 1876](https://leetcode.com/problems/substrings-of-size-three-with-distinct-characters/) |
| 14 | Longest Turbulent Subarray | Medium | Variable window, alternating sign | [LC 978](https://leetcode.com/problems/longest-turbulent-subarray/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 15 | Subarrays with K Different Integers | Hard | atMost(k) - atMost(k-1) | [LC 992](https://leetcode.com/problems/subarrays-with-k-different-integers/) |
| 16 | Longest Substring with At Most K Distinct | Medium | Variable window, freq map | [LC 340](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/) |
| 17 | Minimum Window Containing Substring with All K Distinct | Hard | Variable + distinct tracking | Custom OA variant |
| 18 | Count Number of Nice Subarrays | Medium | Prefix (odd count) / sliding | [LC 1248](https://leetcode.com/problems/count-number-of-nice-subarrays/) |

---

## Worked Solution: LC 1004 -- Max Consecutive Ones III

> Flip at most `k` zeros. Find longest subarray of ones.

```cpp
int longestOnes(std::vector<int>& nums, int k) {
    int left = 0, zeros = 0, ans = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        if (nums[right] == 0) zeros++;
        while (zeros > k) {
            if (nums[left++] == 0) zeros--;
        }
        ans = std::max(ans, right - left + 1);
    }
    return ans;
}
// Time: O(n), Space: O(1)
```

---

## Worked Solution: LC 438 -- Find All Anagrams in String

```cpp
std::vector<int> findAnagrams(std::string s, std::string p) {
    if (p.size() > s.size()) return {};
    std::vector<int> pFreq(26, 0), wFreq(26, 0), result;
    for (char c : p) pFreq[c - 'a']++;
    int k = p.size();

    for (int i = 0; i < k; i++) wFreq[s[i] - 'a']++;
    if (pFreq == wFreq) result.push_back(0);

    for (int i = k; i < (int)s.size(); i++) {
        wFreq[s[i] - 'a']++;
        wFreq[s[i - k] - 'a']--;
        if (pFreq == wFreq) result.push_back(i - k + 1);
    }
    return result;
}
```

---

*<- [[01_Patterns|Patterns]] · [[03_Tricky|Tricky ->]]*

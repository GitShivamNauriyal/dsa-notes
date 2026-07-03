---
tags: [dsa, arrays, hashing, problems, exercises, leetcode]
links: ["[[01_Arrays_and_Hashing_Index]]", "[[01_Arrays_and_Hashing_Key_Algorithms]]", "[[../02_Two_Pointers/02_Two_Pointers_Index]]"]
---

# Arrays & Hashing — Problems & Exercises

*← [[01_Arrays_and_Hashing_Key_Algorithms|Key Algorithms]] · [[../02_Two_Pointers/02_Two_Pointers_Index|Two Pointers →]]*

---

> Solve these in order. Each builds on the previous.

---

## Tier 1 — Foundations (Solve All)

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Contains Duplicate | 🟢 Easy | HashSet | [LC 217](https://leetcode.com/problems/contains-duplicate/) |
| 2 | Valid Anagram | 🟢 Easy | Freq count | [LC 242](https://leetcode.com/problems/valid-anagram/) |
| 3 | Two Sum | 🟢 Easy | HashMap | [LC 1](https://leetcode.com/problems/two-sum/) |
| 4 | Running Sum | 🟢 Easy | Prefix | [LC 1480](https://leetcode.com/problems/running-sum-of-1d-array/) |
| 5 | Majority Element | 🟢 Easy | Boyer-Moore | [LC 169](https://leetcode.com/problems/majority-element/) |
| 6 | Best Time to Buy Stock I | 🟢 Easy | Prefix min | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| 7 | Move Zeroes | 🟢 Easy | In-place write ptr | [LC 283](https://leetcode.com/problems/move-zeroes/) |
| 8 | Remove Duplicates (sorted) | 🟢 Easy | Two pointers | [LC 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) |

---

## Tier 2 — Core Patterns (Solve All)

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Group Anagrams | 🟡 Medium | Group by hash | [LC 49](https://leetcode.com/problems/group-anagrams/) |
| 10 | Top K Frequent Elements | 🟡 Medium | Freq + bucket sort | [LC 347](https://leetcode.com/problems/top-k-frequent-elements/) |
| 11 | Product of Array Except Self | 🟡 Medium | Prefix + suffix | [LC 238](https://leetcode.com/problems/product-of-array-except-self/) |
| 12 | Longest Consecutive Sequence | 🟡 Medium | HashSet | [LC 128](https://leetcode.com/problems/longest-consecutive-sequence/) |
| 13 | Maximum Subarray | 🟡 Medium | Kadane's | [LC 53](https://leetcode.com/problems/maximum-subarray/) |
| 14 | Sort Colors | 🟡 Medium | Dutch Natl Flag | [LC 75](https://leetcode.com/problems/sort-colors/) |
| 15 | Subarray Sum Equals K | 🟡 Medium | Prefix + HashMap | [LC 560](https://leetcode.com/problems/subarray-sum-equals-k/) |
| 16 | 3Sum | 🟡 Medium | Sort + two ptr | [LC 15](https://leetcode.com/problems/3sum/) |
| 17 | Range Sum Query | 🟡 Medium | Prefix sum | [LC 304](https://leetcode.com/problems/range-sum-query-2d-immutable/) |

---

## Tier 3 — Interview-Level (Do At Least 5)

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 18 | 4Sum II | 🟡 Medium | Split pairs + map | [LC 454](https://leetcode.com/problems/4sum-ii/) |
| 19 | Maximum Product Subarray | 🟡 Medium | Kadane's variant | [LC 152](https://leetcode.com/problems/maximum-product-subarray/) |
| 20 | Trapping Rain Water | 🔴 Hard | Prefix max | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 21 | Majority Element II | 🟡 Medium | Boyer-Moore x2 | [LC 229](https://leetcode.com/problems/majority-element-ii/) |
| 22 | Subarrays Div by K | 🟡 Medium | Prefix + modulo | [LC 974](https://leetcode.com/problems/subarray-sums-divisible-by-k/) |
| 23 | Contiguous Array | 🟡 Medium | Prefix (0→-1) | [LC 525](https://leetcode.com/problems/contiguous-array/) |
| 24 | Corporate Flight Bookings | 🟡 Medium | Difference array | [LC 1109](https://leetcode.com/problems/corporate-flight-bookings/) |
| 25 | First Missing Positive | 🔴 Hard | Array as hash | [LC 41](https://leetcode.com/problems/first-missing-positive/) |

---

## Tier 4 — Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 26 | Repeated DNA Sequences | 🟡 Medium | Rolling hash | [LC 187](https://leetcode.com/problems/repeated-dna-sequences/) |
| 27 | Longest Duplicate Substring | 🔴 Hard | Rolling hash + BS | [LC 1044](https://leetcode.com/problems/longest-duplicate-substring/) |
| 28 | Max Sum of 2 Non-Overlapping Subarrays | 🟡 Medium | Prefix + sliding | [LC 1031](https://leetcode.com/problems/maximum-sum-of-two-non-overlapping-subarrays/) |
| 29 | Smallest Range Covering Elements | 🔴 Hard | Priority queue + map | [LC 632](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) |
| 30 | Count Number of Nice Subarrays | 🟡 Medium | Prefix (odd count) | [LC 1248](https://leetcode.com/problems/count-number-of-nice-subarrays/) |

---

## Worked Solution: LC 238 — Product of Array Except Self

> Given array `nums`, return `output` where `output[i]` = product of all elements except `nums[i]`. No division allowed. O(n) time, O(1) extra space.

**Why it's clever**: Build prefix products going left-to-right, then multiply by suffix products going right-to-left — all in the output array itself.

```cpp
#include <vector>

std::vector<int> productExceptSelf(std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> output(n, 1);

    // Step 1: output[i] = product of all elements to the LEFT of i
    int left = 1;
    for (int i = 0; i < n; i++) {
        output[i] = left;
        left *= nums[i];
    }

    // Step 2: multiply by product of all elements to the RIGHT of i
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        output[i] *= right;
        right *= nums[i];
    }

    return output;
}
// output[i] = leftProduct[i] * rightProduct[i]  — no division, O(1) extra space
```

---

## Worked Solution: LC 41 — First Missing Positive

> Find the smallest positive integer not in the array. O(n) time, O(1) space.

**Key insight**: The answer must be in range `[1, n+1]`. Use the array itself as a hash table — place each number `x` (if `1 <= x <= n`) at index `x-1`.

```cpp
#include <vector>

int firstMissingPositive(std::vector<int>& nums) {
    int n = nums.size();

    // Step 1: Place each number at its "correct" index
    // nums[i] should be at index nums[i]-1
    for (int i = 0; i < n; i++) {
        while (nums[i] >= 1 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            std::swap(nums[i], nums[nums[i] - 1]);
        }
    }

    // Step 2: First index where nums[i] != i+1 is the missing positive
    for (int i = 0; i < n; i++) {
        if (nums[i] != i + 1) return i + 1;
    }

    return n + 1;  // all 1..n present, answer is n+1
}
// Time: O(n) — each element swapped at most once
// Space: O(1) — using the array itself as storage
```

---

*← [[01_Arrays_and_Hashing_Key_Algorithms|Key Algorithms]] · [[../02_Two_Pointers/02_Two_Pointers_Index|Next Chapter: Two Pointers →]]*

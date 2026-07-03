---
tags: [dsa, binary-search, problems, exercises]
links: ["[[05_Binary_Search_Index]]", "[[05_Binary_Search_Patterns]]", "[[05_Binary_Search_Tricky]]"]
---

# Binary Search -- Problems & Exercises

*<- [[05_Binary_Search_Patterns|Patterns]] · [[05_Binary_Search_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Binary Search | Easy | Classic exact match | [LC 704](https://leetcode.com/problems/binary-search/) |
| 2 | Search Insert Position | Easy | Lower bound | [LC 35](https://leetcode.com/problems/search-insert-position/) |
| 3 | First Bad Version | Easy | Lower bound predicate | [LC 278](https://leetcode.com/problems/first-bad-version/) |
| 4 | Sqrt(x) | Easy | Answer space (integer sqrt) | [LC 69](https://leetcode.com/problems/sqrtx/) |
| 5 | Guess Number Higher or Lower | Easy | Classic | [LC 374](https://leetcode.com/problems/guess-number-higher-or-lower/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Find First and Last Position | Medium | Lower + upper bound | [LC 34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| 7 | Search in Rotated Sorted Array | Medium | Rotated BS | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| 8 | Find Minimum in Rotated Array | Medium | Rotated min | [LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) |
| 9 | Peak Index in Mountain Array | Easy | Peak finding | [LC 852](https://leetcode.com/problems/peak-index-in-a-mountain-array/) |
| 10 | Find Peak Element | Medium | Peak finding | [LC 162](https://leetcode.com/problems/find-peak-element/) |
| 11 | Koko Eating Bananas | Medium | Answer space | [LC 875](https://leetcode.com/problems/koko-eating-bananas/) |
| 12 | Ship Packages Within D Days | Medium | Answer space | [LC 1011](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 13 | Split Array Largest Sum | Hard | Answer space (minimize max) | [LC 410](https://leetcode.com/problems/split-array-largest-sum/) |
| 14 | Minimum Limit of Balls in Bag | Medium | Answer space | [LC 1760](https://leetcode.com/problems/minimum-limit-of-balls-in-a-bag/) |
| 15 | Search in Rotated Array II (duplicates) | Medium | Rotated with dups | [LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) |
| 16 | Find Minimum in Rotated Array II | Hard | Rotated min with dups | [LC 154](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) |
| 17 | Find K Closest Elements | Medium | Binary search + two pointer | [LC 658](https://leetcode.com/problems/find-k-closest-elements/) |
| 18 | Time Based Key-Value Store | Medium | Binary search on timestamps | [LC 981](https://leetcode.com/problems/time-based-key-value-store/) |
| 19 | Median of Two Sorted Arrays | Hard | Binary search on partition | [LC 4](https://leetcode.com/problems/median-of-two-sorted-arrays/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 20 | Minimize Max Distance to Gas Station | Hard | Answer space (fractional) | [LC 774](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) |
| 21 | Smallest Divisor Given Threshold | Medium | Answer space | [LC 1283](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/) |
| 22 | Aggressive Cows (GFG) | Medium | Answer space (max of min) | [GFG](https://www.geeksforgeeks.org/aggressive-cows/) |
| 23 | Book Allocation (GFG) | Medium | Answer space (minimize max) | [GFG](https://www.geeksforgeeks.org/allocate-minimum-number-pages/) |
| 24 | Painter's Partition | Hard | Answer space | [GFG](https://www.geeksforgeeks.org/painters-partition-problem/) |
| 25 | Nth Root of M | Medium | Binary search on float | [GFG](https://www.geeksforgeeks.org/n-th-root-of-a-number/) |

---

## Worked Solution: LC 4 -- Median of Two Sorted Arrays

**Why hard**: O(log(min(m,n))) required. Binary search on partition of smaller array to find where to cut both arrays such that left half elements <= right half elements.

```cpp
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
    // Ensure nums1 is the smaller array (binary search on smaller for efficiency)
    if (nums1.size() > nums2.size()) return findMedianSortedArrays(nums2, nums1);

    int m = nums1.size(), n = nums2.size();
    int lo = 0, hi = m;

    while (lo <= hi) {
        int cut1 = lo + (hi - lo) / 2;  // partition nums1: cut1 elements on left
        int cut2 = (m + n + 1) / 2 - cut1;  // partition nums2 accordingly

        // Elements just left/right of each partition
        int l1 = (cut1 == 0) ? INT_MIN : nums1[cut1 - 1];
        int l2 = (cut2 == 0) ? INT_MIN : nums2[cut2 - 1];
        int r1 = (cut1 == m) ? INT_MAX : nums1[cut1];
        int r2 = (cut2 == n) ? INT_MAX : nums2[cut2];

        if (l1 <= r2 && l2 <= r1) {
            // Correct partition found
            if ((m + n) % 2 == 1)
                return max(l1, l2);  // odd total: median is max of left halves
            return (max(l1, l2) + min(r1, r2)) / 2.0;
        } else if (l1 > r2) {
            hi = cut1 - 1;  // cut1 too far right, move left
        } else {
            lo = cut1 + 1;  // cut1 too far left, move right
        }
    }
    return 0.0;
}
// Time: O(log(min(m,n))), Space: O(1)
```

---

## Worked Solution: Aggressive Cows (GFG) -- Classic Answer Space

> Place C cows in N stalls at given positions. Maximize minimum distance between any two cows.

```cpp
#include <algorithm>

bool canPlace(vector<int>& stalls, int cows, int minDist) {
    int placed = 1, last = stalls[0];
    for (int i = 1; i < (int)stalls.size(); i++) {
        if (stalls[i] - last >= minDist) {
            placed++;
            last = stalls[i];
            if (placed == cows) return true;
        }
    }
    return placed >= cows;
}

int aggressiveCows(vector<int>& stalls, int cows) {
    sort(stalls.begin(), stalls.end());
    int lo = 1, hi = stalls.back() - stalls.front(), ans = 0;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canPlace(stalls, cows, mid)) {
            ans = mid;
            lo = mid + 1;  // try larger minimum distance
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
// Time: O(n log n + n log(max_dist)), Space: O(1)
```

---

*<- [[05_Binary_Search_Patterns|Patterns]] · [[05_Binary_Search_Tricky|Tricky ->]]*

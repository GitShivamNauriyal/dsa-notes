---
tags: [dsa, intervals, problems, exercises]
links: ["[[15_Intervals_Index]]", "[[15_Intervals_Patterns]]", "[[15_Intervals_Tricky]]"]
---

# Intervals -- Problems & Exercises

*<- [[15_Intervals_Patterns|Patterns]] · [[15_Intervals_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Meeting Rooms | Easy | Sort & check adjacent overlaps | [LC 252](https://leetcode.com/problems/meeting-rooms/) |
| 2 | Merge Intervals | Medium | Sort by start & merge | [LC 56](https://leetcode.com/problems/merge-intervals/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | Insert Interval | Medium | 3-phase partition merge | [LC 57](https://leetcode.com/problems/insert-interval/) |
| 4 | Non-overlapping Intervals | Medium | Sort by end time, remove overlap | [LC 435](https://leetcode.com/problems/non-overlapping-intervals/) |
| 5 | Minimum Number of Arrows to Burst Balloons | Medium | Sort by end, count intersections | [LC 452](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Meeting Rooms II | Medium | Min-heap / Sweep-line (max overlap count) | [LC 253](https://leetcode.com/problems/meeting-rooms-ii/) |
| 7 | Car Pooling | Medium | Sweep-line / Prefix array capacity check | [LC 1094](https://leetcode.com/problems/car-pooling/) |
| 8 | Partition Labels (Interval mapping) | Medium | Interval merge equivalent | [LC 763](https://leetcode.com/problems/partition-labels/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Minimum Interval to Include Each Query | Hard | Sort queries, Min-Heap sweep-line | [LC 1851](https://leetcode.com/problems/minimum-interval-to-include-each-query/) |
| 10 | Employee Free Time | Hard | Merge intervals from multiple sorted lists | [LC 759](https://leetcode.com/problems/employee-free-time/) |

---

## Worked Solution: LC 452 -- Minimum Arrows to Burst Balloons

**Key Insight**: Balloons are represented as intervals `[start, end]` along the X-axis. An arrow shot vertically at coordinate $X$ bursts all balloons where `start <= X <= end`. We want to minimize the number of arrows.
- This is equivalent to finding the **minimum number of intersections** that cover all intervals.
- **Greedy choice**: Sort balloons by **end time**. Shot an arrow at the end coordinate of the first balloon (`prevEnd`). Any subsequent balloon that starts before or at `prevEnd` will be burst by this same arrow.
- If a balloon starts *after* `prevEnd`, we need a new arrow. We increment `arrows` count and update `prevEnd` to this new balloon's end coordinate.

```cpp
#include <vector>
#include <algorithm>

int findMinArrowShots(vector<vector<int>>& points) {
    if (points.empty()) return 0;

    // Step 1: Sort by end coordinate
    // Use custom lambda. Be careful with potential integer underflow/overflow if subtracting values
    sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[1] < b[1];
    });

    int arrows = 1;
    int prevEnd = points[0][1];

    for (int i = 1; i < (int)points.size(); i++) {
        if (points[i][0] > prevEnd) {
            // Balloon starts after prevEnd -> needs a new arrow
            arrows++;
            prevEnd = points[i][1];
        }
    }
    return arrows;
}
// Time Complexity: O(n log n)
// Space Complexity: O(1)
```

---

*<- [[15_Intervals_Patterns|Patterns]] · [[15_Intervals_Tricky|Tricky ->]]*

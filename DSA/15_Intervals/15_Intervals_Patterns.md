---
tags: [dsa, intervals, patterns, merge-intervals]
links: ["[[15_Intervals_Index]]", "[[15_Intervals_Problems_and_Exercises]]"]
---

# Intervals -- Patterns

*<- [[15_Intervals_Index|Index]] · [[15_Intervals_Problems_and_Exercises|Problems ->]]*

---

## Under the Hood: The Sorting Paradigm

An interval is represented as `[start, end]`. Almost all interval problems start by **sorting** the intervals.
- **Sort by Start Time**: Useful when we need to merge overlapping intervals or check for gaps sequentially.
- **Sort by End Time**: Useful when we want to make greedy scheduling choices (e.g. pick as many non-overlapping intervals as possible, since finishing earlier leaves more room for future intervals).

```
   Overlappings:
   Interval A:  [=======]
   Interval B:      [=======]   <- overlaps if B.start <= A.end
   Interval C:                  [=======] <- no overlap if C.start > A.end
```

---

## Pattern 1: Insert Interval (LC 57)

**The Problem**: Given a set of non-overlapping intervals sorted by start time, insert a new interval `newInterval` into the set (merge if necessary).

### Greedy Three-Phase Processing
Since the intervals are already sorted, we can process them in a single pass using three distinct phases:
1. **Phase 1: Before Overlap**: All intervals that end before the `newInterval` starts. Append them directly.
2. **Phase 2: During Overlap (Merging)**: All intervals that overlap with the `newInterval` (i.e. they start before or at `newInterval.end` and end after or at `newInterval.start`).
   - Merge them into `newInterval` by taking:
     `newInterval.start = min(newInterval.start, current.start)`
     `newInterval.end = max(newInterval.end, current.end)`
3. **Phase 3: After Overlap**: All intervals that start after the `newInterval` ends. Append them directly.

```cpp
#include <vector>
#include <algorithm>

vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
    vector<vector<int>> result;
    int i = 0;
    int n = intervals.size();

    // Phase 1: Add all intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) {
        result.push_back(intervals[i]);
        i++;
    }

    // Phase 2: Merge overlapping intervals
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.push_back(newInterval); // insert the merged interval

    // Phase 3: Add all remaining intervals starting after newInterval ends
    while (i < n) {
        result.push_back(intervals[i]);
        i++;
    }
    return result;
}
// Time Complexity: O(n) — single pass
// Space Complexity: O(1) (excluding output)
```

---

## Pattern 2: Merge Intervals (LC 56)

**The Problem**: Given an array of intervals, merge all overlapping intervals.

### Greedy Choice
1. Sort the intervals by start time.
2. Iterate through the sorted intervals:
   - If the `result` list is empty, or the current interval's start time is greater than the end time of the last interval in `result`, there is no overlap. Append the current interval.
   - Else, there is an overlap. Merge them by updating the end time of the last interval in `result` to be the maximum of both end times.

```cpp
#include <vector>
#include <algorithm>

vector<vector<int>> merge(vector<vector<int>>& intervals) {
    if (intervals.empty()) return {};

    // Step 1: Sort by start time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[0] < b[0];
    });

    vector<vector<int>> merged;
    merged.push_back(intervals[0]);

    for (int i = 1; i < (int)intervals.size(); i++) {
        // If current interval overlaps with the last merged interval
        if (intervals[i][0] <= merged.back()[1]) {
            merged.back()[1] = max(merged.back()[1], intervals[i][1]); // merge
        } else {
            merged.push_back(intervals[i]); // no overlap, add new
        }
    }
    return merged;
}
// Time Complexity: O(n log n) due to sorting
// Space Complexity: O(log n) for sort stack
```

---

## Pattern 3: Non-overlapping Intervals (LC 435)

**The Problem**: Given an array of intervals, find the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping.

### Greedy Choice
This is equivalent to finding the **maximum number of non-overlapping intervals** we can schedule (Interval Scheduling Maximization).
1. Sort intervals by **end time** in ascending order.
2. Maintain `prevEnd` of the last scheduled interval.
3. Iterate through intervals:
   - If the current interval starts after or at `prevEnd`, it does not overlap. We can schedule it. Update `prevEnd = current.end`.
   - Else, it overlaps. We must remove it. Increment our removal count. (We greedily keep the interval with the *smaller* end time to minimize chances of future overlaps).

```cpp
#include <vector>
#include <algorithm>

int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;

    // Step 1: Sort by end time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[1] < b[1];
    });

    int removals = 0;
    int prevEnd = intervals[0][1];

    for (int i = 1; i < (int)intervals.size(); i++) {
        if (intervals[i][0] < prevEnd) {
            removals++; // overlaps -> remove current
        } else {
            prevEnd = intervals[i][1]; // non-overlapping -> schedule it
        }
    }
    return removals;
}
// Time Complexity: O(n log n)
// Space Complexity: O(1)
```

---

*<- [[15_Intervals_Index|Index]] · [[15_Intervals_Problems_and_Exercises|Problems ->]]*

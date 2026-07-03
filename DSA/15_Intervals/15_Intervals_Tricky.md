---
tags: [dsa, intervals, tricky, hard, meeting-rooms, heap]
links: ["[[15_Intervals_Index]]", "[[15_Intervals_Problems_and_Exercises]]", "[[../16_Bit_Manipulation/16_Bit_Manipulation_Index]]"]
---

# Intervals -- Tricky & Higher-Order

*<- [[15_Intervals_Problems_and_Exercises|Problems]] · [[../16_Bit_Manipulation/16_Bit_Manipulation_Index|Bit Manipulation ->]]*

---

## 1. Meeting Rooms II (LC 253)

**Why Tricky**: Given start and end times of meetings, find the minimum number of conference rooms required. We want to find the **maximum number of overlapping meetings at any point in time**.

### Approach A: Min-Heap of End Times
- Sort meetings by start time.
- Maintain a **Min-Heap** storing the end times of meetings currently in progress. The top of the heap is the meeting that finishes earliest.
- For each meeting:
  - If its start time $\ge$ heap top (earliest finish time), the room is free! Pop the top and push this meeting's end time.
  - Else, we need a new room. Push this meeting's end time.
- The final heap size is the number of rooms needed.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

int minMeetingRoomsHeap(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;

    // Step 1: Sort by start time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[0] < b[0];
    });

    // Min-heap stores end times of active meetings
    priority_queue<int, vector<int>, greater<int>> minHeap;
    minHeap.push(intervals[0][1]);

    for (int i = 1; i < (int)intervals.size(); i++) {
        // If the room with earliest end time is free
        if (intervals[i][0] >= minHeap.top()) {
            minHeap.pop(); // reuse room
        }
        minHeap.push(intervals[i][1]); // allocate room (new or reused)
    }
    return minHeap.size();
}
```

### Approach B: Chronological Two-Pointer Sorting ($O(1)$ Extra Space)
- Separate start and end times into two sorted arrays: `starts` and `ends`.
- Maintain two pointers `s` (starts index) and `e` (ends index).
- If `starts[s] < ends[e]`, it means a meeting started before the earliest one finished. We need a new room (`rooms++`, `s++`).
- Otherwise, a meeting ended. We reuse that room (`s++`, `e++`).

```cpp
#include <vector>
#include <algorithm>

int minMeetingRooms(vector<vector<int>>& intervals) {
    int n = intervals.size();
    vector<int> starts(n), ends(n);
    for (int i = 0; i < n; i++) {
        starts[i] = intervals[i][0];
        ends[i] = intervals[i][1];
    }

    sort(starts.begin(), starts.end());
    sort(ends.begin(), ends.end());

    int rooms = 0;
    int s = 0, e = 0;

    while (s < n) {
        if (starts[s] < ends[e]) {
            rooms++; // need new room
            s++;
        } else {
            // reuse room (meaning we don't increase rooms, just move ends pointer)
            s++;
            e++;
        }
    }
    return rooms;
}
// Time Complexity: O(n log n) due to sorting
// Space Complexity: O(n) to store start/end arrays
```

---

## 2. Minimum Interval to Include Each Query (LC 1851)

**Why Tricky**: Given intervals and query points, for each query find the size of the smallest interval containing it. Doing it naively takes $O(Q \times N)$ — too slow.
- **The Solution -- Offline Queries + Min-Heap**:
  1. Sort intervals by start time.
  2. Sort queries while preserving their original indices. This allows us to process queries and intervals in a single **sweep-line** pass.
  3. Maintain a **Min-Heap** storing `{size, end_time}` of intervals.
  4. For each query `q`:
     - Push all intervals that start before or at `q` into the heap.
     - Pop all intervals from the heap that end before `q` (they cannot contain `q`).
     - The top of the heap is the smallest interval containing `q`.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

vector<int> minInterval(vector<vector<int>>& intervals, vector<int>& queries) {
    int Q = queries.size();
    vector<pair<int, int>> sortedQueries(Q);
    for (int i = 0; i < Q; i++) sortedQueries[i] = {queries[i], i};

    // Sort queries and intervals
    sort(sortedQueries.begin(), sortedQueries.end());
    sort(intervals.begin(), intervals.end());

    // Min-heap stores pair: {interval_size, interval_end}
    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> minHeap;
    vector<int> result(Q, -1);
    int idx = 0;
    int n = intervals.size();

    for (const auto& qPair : sortedQueries) {
        int qVal = qPair.first;
        int qIdx = qPair.second;

        // Step 1: Add all intervals starting <= qVal
        while (idx < n && intervals[idx][0] <= qVal) {
            int size = intervals[idx][1] - intervals[idx][0] + 1;
            minHeap.push({size, intervals[idx][1]});
            idx++;
        }

        // Step 2: Remove all intervals ending < qVal
        while (!minHeap.empty() && minHeap.top().second < qVal) {
            minHeap.pop();
        }

        // Step 3: Top of heap is the smallest interval containing qVal
        if (!minHeap.empty()) {
            result[qIdx] = minHeap.top().first;
        }
    }
    return result;
}
// Time Complexity: O(N log N + Q log Q)
// Space Complexity: O(N + Q)
```

---

*<- [[15_Intervals_Problems_and_Exercises|Problems]] · [[../16_Bit_Manipulation/16_Bit_Manipulation_Index|Bit Manipulation ->]]*

---
tags: [dsa, heap, priority-queue, problems, exercises]
links: ["[[09_Heap_Priority_Queue_Index]]", "[[09_Heap_Priority_Queue_Patterns]]", "[[09_Heap_Priority_Queue_Tricky]]"]
---

# Heap & Priority Queue -- Problems & Exercises

*<- [[09_Heap_Priority_Queue_Patterns|Patterns]] · [[09_Heap_Priority_Queue_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Kth Largest Element in a Stream | Easy | Min-heap of size K | [LC 703](https://leetcode.com/problems/kth-largest-element-in-a-stream/) |
| 2 | Last Stone Weight | Easy | Max-heap simulation | [LC 1046](https://leetcode.com/problems/last-stone-weight/) |
| 3 | Implement Stack using Queues (Heap concepts) | Easy | Priority-based simulation | [LC 225](https://leetcode.com/problems/implement-stack-using-queues/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Kth Largest Element in Array | Medium | Min-heap of size K | [LC 215](https://leetcode.com/problems/kth-largest-element-in-an-array/) |
| 5 | K Closest Points to Origin | Medium | Custom heap comparator | [LC 973](https://leetcode.com/problems/k-closest-points-to-origin/) |
| 6 | Merge K Sorted Lists | Hard | K-way merge with Heap | [LC 23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 7 | Find Median from Data Stream | Hard | Two Heaps pattern | [LC 295](https://leetcode.com/problems/find-median-from-data-stream/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Task Scheduler | Medium | Max-Heap + Cooldown Queue | [LC 621](https://leetcode.com/problems/task-scheduler/) |
| 9 | Top K Frequent Elements | Medium | Frequency map + Min-Heap | [LC 347](https://leetcode.com/problems/top-k-frequent-elements/) |
| 10 | Reorganize String | Medium | Greedy frequency placement | [LC 767](https://leetcode.com/problems/reorganize-string/) |
| 11 | Connect Ropes with Min Cost | Medium | Greedy Min-Heap merges | [GFG](https://www.geeksforgeeks.org/connect-n-ropes-minimum-cost/) |
| 12 | Smallest Range Covering K Lists | Hard | Multi-pointer / K-Way Heap | [LC 632](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 13 | IPO | Hard | Greedy choice with dual Heaps | [LC 502](https://leetcode.com/problems/ipo/) |
| 14 | Sliding Window Median | Hard | Dual Heaps + Lazy Deletions | [LC 480](https://leetcode.com/problems/sliding-window-median/) |
| 15 | Find Building Where Alice & Bob Meet | Hard | Monotonic stack + Min-Heap | [LC 2940](https://leetcode.com/problems/find-building-where-alice-and-bob-can-meet/) |

---

## Worked Solution 1: LC 973 -- K Closest Points to Origin

**Key Insight**: Instead of sorting all $n$ points by distance, maintain a **Max-Heap of size $K$** storing `{distance, {x, y}}`. If a new point has a smaller distance than the top of the heap, pop the top and push the new point. At the end, the heap contains the $K$ closest points.

```cpp
#include <vector>
#include <queue>

// Distance squared (no need for double/sqrt to compare)
int getDist(const vector<int>& p) {
    return p[0]*p[0] + p[1]*p[1];
}

vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
    // Max-heap storing pair: {distance, index_in_points}
    priority_queue<pair<int, int>> maxHeap;

    for (int i = 0; i < (int)points.size(); i++) {
        int dist = getDist(points[i]);
        maxHeap.push({dist, i});
        if ((int)maxHeap.size() > k) {
            maxHeap.pop(); // discard the point with the largest distance
        }
    }

    vector<vector<int>> result;
    while (!maxHeap.empty()) {
        result.push_back(points[maxHeap.top().second]);
        maxHeap.pop();
    }
    return result;
}
// Time: O(n log k) — much faster than O(n log n) full sort when k << n
// Space: O(k) for the heap
```

---

## Worked Solution 2: GFG -- Connect N Ropes with Minimum Cost

**Key Insight**: Grepidly merge the two shortest ropes at each step. By merging the shortest ropes first, we minimize the cost contribution of larger ropes down the line. A Min-Heap allows retrieving the two shortest ropes in $O(\log n)$ time.

```
Ropes: [4, 3, 2, 6]

Step 1: Min-Heap = [2, 3, 4, 6]
        Extract two smallest: 2 and 3.
        Merge cost = 5.
        Push back merged rope.
        Min-Heap = [4, 5, 6], Total Cost = 5.

Step 2: Extract two smallest: 4 and 5.
        Merge cost = 9.
        Push back merged rope.
        Min-Heap = [6, 9], Total Cost = 5 + 9 = 14.

Step 3: Extract two smallest: 6 and 9.
        Merge cost = 15.
        Push back merged rope.
        Min-Heap = [15], Total Cost = 14 + 15 = 29.

Final Cost = 29.
```

```cpp
#include <vector>
#include <queue>

long long minCost(vector<long long>& arr) {
    priority_queue<long long, vector<long long>, greater<long long>> minHeap(arr.begin(), arr.end());
    long long totalCost = 0;

    while (minHeap.size() > 1) {
        long long first = minHeap.top(); minHeap.pop();
        long long second = minHeap.top(); minHeap.pop();
        
        long long cost = first + second;
        totalCost += cost;
        minHeap.push(cost);
    }
    return totalCost;
}
// Time: O(n log n)
// Space: O(n)
```

---

*<- [[09_Heap_Priority_Queue_Patterns|Patterns]] · [[09_Heap_Priority_Queue_Tricky|Tricky ->]]*

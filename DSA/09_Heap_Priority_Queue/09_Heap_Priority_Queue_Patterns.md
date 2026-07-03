---
tags: [dsa, heap, priority-queue, patterns]
links: ["[[09_Heap_Priority_Queue_Index]]", "[[09_Heap_Priority_Queue_Problems_and_Exercises]]"]
---

# Heap & Priority Queue -- Patterns

*<- [[09_Heap_Priority_Queue_Index|Index]] · [[09_Heap_Priority_Queue_Problems_and_Exercises|Problems ->]]*

---

## What is a Heap?

A heap is a complete binary tree that satisfies the **heap property**:
- **Max-Heap**: Parent value $\ge$ children values. Root is the global maximum.
- **Min-Heap**: Parent value $\le$ children values. Root is the global minimum.

```
       Min-Heap Representation                    Array Layout
                 2                             Index:  0  1  2  3  4  5
                / \                                    [2, 3, 5, 8, 4, 7]
               3   5
              / \  /                           Parent of i: (i-1)/2
             8   4 7                           Left child:  2*i + 1
                                               Right child: 2*i + 2
```

### Why use a Heap?
When you need to **frequently query the minimum/maximum element while insertions are happening**. 
- Query Min/Max: $O(1)$
- Insert: $O(\log n)$
- Remove Min/Max: $O(\log n)$

---

## C++ Priority Queue Reference (No `std::`)

```cpp
#include <queue>
#include <vector>

// 1. Default Max-Heap (stores largest element at top)
priority_queue<int> maxHeap;

// 2. Min-Heap (stores smallest element at top)
priority_queue<int, vector<int>, greater<int>> minHeap;

// 3. Heap of Pairs (sorts by pair.first, then pair.second)
priority_queue<pair<int, int>> pairHeap;

// 4. Custom Comparator using Lambda (C++20 style)
auto cmp = [](const pair<int, int>& a, const pair<int, int>& b) {
    return a.first > b.first; // Min-heap behavior on first element
};
priority_queue<pair<int, int>, vector<pair<int, int>>, decltype(cmp)> customHeap(cmp);
```

---

## Pattern 1: Top K Elements (Min-Heap of Size K)

**Core Idea**: Instead of sorting the entire array of size $n$ in $O(n \log n)$, maintain a **Min-Heap of size $K$**. 
- If heap size $< K$, push the current element.
- If heap size $== K$ and the current element is larger than the top (smallest of the top $K$), pop the top and push the current element.
- The heap will always contain the $K$ largest elements seen so far.

```
Array: [3, 2, 20, 5, 10, 8], K = 3

i=0 (3)  : Heap = [3]
i=1 (2)  : Heap = [2, 3]
i=2 (20) : Heap = [2, 3, 20]
i=3 (5)  : 5 > top (2) -> Pop 2, Push 5 -> Heap = [3, 5, 20]
i=4 (10) : 10 > top (3) -> Pop 3, Push 10 -> Heap = [5, 10, 20]
i=5 (8)  : 8 > top (5) -> Pop 5, Push 8 -> Heap = [8, 10, 20]

Result: Top 3 elements are [8, 10, 20]
```

```cpp
#include <vector>
#include <queue>

// Find the k-th largest element in an unsorted array
// LeetCode 215 — Kth Largest Element in an Array
int findKthLargest(vector<int>& nums, int k) {
    // Keep smallest of the top k elements at the top
    priority_queue<int, vector<int>, greater<int>> minHeap;

    for (int x : nums) {
        minHeap.push(x);
        if (minHeap.size() > k) {
            minHeap.pop(); // discard the smallest element out of the current k+1
        }
    }
    return minHeap.top(); // top contains the k-th largest element
}
// Time: O(n log k), Space: O(k)
```

---

## Pattern 2: K-Way Merge (Multiple Sorted Input Streams)

**Core Idea**: Merge $K$ sorted arrays/lists into a single sorted list.
- Store the first element of each of the $K$ lists in a Min-Heap.
- Pop the minimum node, append it to the result, and push the next element from the same list into the heap.

```
Lists:
L1: [1, 5, 7]
L2: [2, 4, 8]
L3: [0, 9]

Initialize Heap: {value, list_idx, element_idx}
Heap: [(0, L3, 0), (1, L1, 0), (2, L2, 0)] (sorted by value)

Pop (0, L3, 0) -> Push next from L3: (9, L3, 1) -> Result: [0]
Pop (1, L1, 0) -> Push next from L1: (5, L1, 1) -> Result: [0, 1]
Pop (2, L2, 0) -> Push next from L2: (4, L2, 1) -> Result: [0, 1, 2]
...
```

```cpp
#include <vector>
#include <queue>
#include <tuple>

// Merge k sorted arrays
vector<int> mergeKSortedArrays(const vector<vector<int>>& arrays) {
    // tuple elements: {value, array_index, element_index}
    using T = tuple<int, int, int>;
    priority_queue<T, vector<T>, greater<T>> minHeap;

    // Push first element of each array
    for (int i = 0; i < (int)arrays.size(); i++) {
        if (!arrays[i].empty()) {
            minHeap.push({arrays[i][0], i, 0});
        }
    }

    vector<int> result;
    while (!minHeap.empty()) {
        auto [val, arrIdx, elemIdx] = minHeap.top();
        minHeap.pop();
        result.push_back(val);

        // Push next element from the same array if it exists
        if (elemIdx + 1 < (int)arrays[arrIdx].size()) {
            minHeap.push({arrays[arrIdx][elemIdx + 1], arrIdx, elemIdx + 1});
        }
    }
    return result;
}
// Time: O(n log k) where n is total elements, k is number of arrays
// Space: O(k) for the heap
```

---

## Pattern 3: Two Heaps (Running Median)

**Core Idea**: Maintain the median of a stream of numbers dynamically.
- Split the numbers into two halves:
  - **Left half (lower numbers)**: Stored in a **Max-Heap** (so the largest of the lower half is accessible at the top).
  - **Right half (higher numbers)**: Stored in a **Min-Heap** (so the smallest of the higher half is accessible at the top).
- Keep the heaps balanced: size difference must be $\le 1$.

```
Stream: 5, 15, 1, 3
[Left Max-Heap] (lower half)     |     [Right Min-Heap] (upper half)

Insert 5:  Left = [5],   Right = []       -> Median = 5
Insert 15: Left = [5],   Right = [15]     -> Median = (5 + 15) / 2 = 10
Insert 1:  Left = [5,1], Right = [15]
           (rebalance: Left size is 2, Right is 1 - OK) -> Median = top of Left = 5
Insert 3:  Left = [3,1], Right = [15,5]   (after pushing and rebalancing)
           -> Median = (top of Left (3) + top of Right (5)) / 2 = 4
```

```cpp
#include <queue>

// LeetCode 295 — Find Median from Data Stream
class MedianFinder {
    priority_queue<int> maxHeap;                            // lower half
    priority_queue<int, vector<int>, greater<int>> minHeap; // upper half

public:
    void addNum(int num) {
        // Step 1: Add to appropriate heap
        if (maxHeap.empty() || num <= maxHeap.top()) {
            maxHeap.push(num);
        } else {
            minHeap.push(num);
        }

        // Step 2: Balance Heaps (maxHeap can hold at most 1 more than minHeap)
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        } else if (minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }

    double findMedian() {
        if (maxHeap.size() == minHeap.size()) {
            return (maxHeap.top() + minHeap.top()) / 2.0;
        }
        return maxHeap.top(); // maxHeap has the extra middle element
    }
};
// Add: O(log n), Query Median: O(1)
// Space: O(n)
```

---

## Pattern 4: Task Scheduler / Frequency-based (Heap + Queue)

**Core Idea**: Process elements based on frequency, but with a cooldown constraint (e.g., cannot repeat the same task within $n$ units of time).
- Count frequencies and store `{frequency, task_id}` in a Max-Heap.
- Process the task with the highest frequency.
- If it still has remaining frequency, push it into a **cooldown queue** with the timestamp when it can be processed again.
- Once time passes the cooldown threshold, pop from the queue and push back into the Max-Heap.

```
Tasks: [A, A, A, B, B, B], Cooldown n = 2
Freqs: A:3, B:3. Max-Heap = [(3, A), (3, B)]
Cooldown Queue = []

Time 1: Pop A (freq becomes 2). Queue = [(2, A, time_avail=3)]. Max-Heap = [(3, B)]
Time 2: Pop B (freq becomes 2). Queue = [(2, A, time_avail=3), (2, B, time_avail=4)]
Time 3: A is available! Pop from queue, push to Max-Heap. Max-Heap = [(2, A)]
        Pop A (freq becomes 1). Queue = [(2, B, time_avail=4), (1, A, time_avail=5)]
...
```

```cpp
#include <vector>
#include <queue>
#include <unordered_map>

// LeetCode 621 — Task Scheduler
int leastInterval(vector<char>& tasks, int n) {
    unordered_map<char, int> counts;
    for (char t : tasks) counts[t]++;

    priority_queue<int> maxHeap; // only track frequencies
    for (auto& p : counts) maxHeap.push(p.second);

    // Queue stores {frequency, time_available}
    queue<pair<int, int>> cooldown;
    int time = 0;

    while (!maxHeap.empty() || !cooldown.empty()) {
        time++;

        if (!maxHeap.empty()) {
            int freq = maxHeap.top() - 1;
            maxHeap.pop();
            if (freq > 0) {
                cooldown.push({freq, time + n});
            }
        }

        // Check if any cooldown task is now available
        if (!cooldown.empty() && cooldown.front().second == time) {
            maxHeap.push(cooldown.front().first);
            cooldown.pop();
        }
    }
    return time;
}
// Time: O(tasks_count), Space: O(distinct_tasks) -> O(26) = O(1)
```

---

*<- [[09_Heap_Priority_Queue_Index|Index]] · [[09_Heap_Priority_Queue_Problems_and_Exercises|Problems ->]]*

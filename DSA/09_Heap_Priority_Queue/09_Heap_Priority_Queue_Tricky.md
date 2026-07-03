---
tags: [dsa, heap, priority-queue, tricky, hard]
links: ["[[09_Heap_Priority_Queue_Index]]", "[[09_Heap_Priority_Queue_Problems_and_Exercises]]", "[[../10_Backtracking/10_Backtracking_Index]]"]
---

# Heap & Priority Queue -- Tricky & Higher-Order

*<- [[09_Heap_Priority_Queue_Problems_and_Exercises|Problems]] · [[../10_Backtracking/10_Backtracking_Index|Backtracking ->]]*

---

## 1. Custom Comparators for Heaps

**Why Tricky**: Writing priority queue comparators behaves **opposite** to standard sorting (`sort`).
- In standard sorting: `a < b` puts `a` first (ascending order).
- In `priority_queue`: `cmp(a, b)` returning `true` means `a` has **lower priority** than `b` (so `b` is placed closer to the top). Thus, `a < b` creates a Max-Heap.

```cpp
#include <queue>
#include <vector>

// Struct-based comparator
struct Task {
    int id;
    int priority;
    int deadline;
};

// We want tasks with HIGHER priority first. 
// If priority is equal, tasks with EARLIER deadline first.
struct TaskCmp {
    bool operator()(const Task& a, const Task& b) {
        if (a.priority != b.priority) {
            return a.priority < b.priority; // Less than gives Max-Heap (highest priority on top)
        }
        return a.deadline > b.deadline; // Greater than gives Min-Heap (earliest deadline on top)
    }
};

void heapExample() {
    priority_queue<Task, vector<Task>, TaskCmp> pq;
}
```

---

## 2. Lazy Deletion (How to delete arbitrary nodes from a Heap)

**Why Tricky**: `priority_queue` does not support random access or deletion of arbitrary elements (only `push`, `pop`, `top`). 
**Solution -- Lazy Deletion**:
- Maintain a hash map/frequency map of "deleted elements awaiting pop" (`unordered_map<int, int> lazyDeletes`).
- When asked to delete `x`, increment `lazyDeletes[x]`.
- Whenever you query `top()`, check if the top element is in `lazyDeletes`. If it is, decrement its count and `pop()` it. Keep doing this until `top()` is a valid, non-deleted element.

```cpp
#include <queue>
#include <unordered_map>

class LazyMinHeap {
    priority_queue<int, vector<int>, greater<int>> pq;
    unordered_map<int, int> lazyDeletes;

    void clean() {
        while (!pq.empty() && lazyDeletes[pq.top()] > 0) {
            lazyDeletes[pq.top()]--;
            pq.pop();
        }
    }

public:
    void push(int x) { pq.push(x); }
    
    void remove(int x) { lazyDeletes[x]++; } // lazy delete
    
    int top() {
        clean();
        return pq.top();
    }
    
    void pop() {
        clean();
        pq.pop();
    }
};
// Space: O(number of lazy deleted elements)
// Time: Amortized O(log n) per operation
```

---

## 3. IPO -- Interleaved Dual Heaps (LC 502)

**Why Tricky**: You have $k$ projects. Each has a capital requirement $c_i$ and a profit $p_i$. You start with capital $W$. You can only pick projects you can afford.
**Solution**:
- Sort projects by capital requirements.
- Maintain a Max-Heap of profits of all projects you can *currently* afford.
- At each step, push all newly affordable projects into the Max-Heap, pop the most profitable project, and add its profit to your capital $W$.

```cpp
#include <vector>
#include <queue>
#include <algorithm>

int findMaximizedCapital(int k, int w, vector<int>& profits, vector<int>& capital) {
    int n = profits.size();
    vector<pair<int, int>> projects(n);
    for (int i = 0; i < n; i++) {
        projects[i] = {capital[i], profits[i]};
    }
    
    // Sort projects ascending by capital required
    sort(projects.begin(), projects.end());
    
    priority_queue<int> maxProfitHeap;
    int idx = 0;

    for (int i = 0; i < k; i++) {
        // Push all affordable projects to the heap
        while (idx < n && projects[idx].first <= w) {
            maxProfitHeap.push(projects[idx].second);
            idx++;
        }
        
        if (maxProfitHeap.empty()) break; // no affordable projects left
        
        w += maxProfitHeap.top(); // execute most profitable project
        maxProfitHeap.pop();
    }
    return w;
}
// Time: O(n log n) for sorting + O(n log n) total heap operations
// Space: O(n)
```

---

## 4. Sliding Window Median with Lazy Deletion (LC 480)

**Why Tricky**: We need the median of every sliding window of size $k$. Doing it with two heaps requires removing the element that "leaves" the window. Since we can't delete directly from C++ heaps, we must combine **Two Heaps** with **Lazy Deletion** and track the balances.

```
Window = 3. Elements: [1, 3, -1, -3, 5], k=3
Lower (Max-Heap) | Upper (Min-Heap) | Lazy Deletes

Initial state (k=3): [1, 3, -1]
Lower = [-1], Upper = [1, 3], Median = 1.0

Next step (insert -3, remove 1):
1 is in Upper. We mark 1 for lazy deletion: lazyDeletes[1] = 1.
Insert -3 into Lower.
Rebalance Lower and Upper sizes.
Discard lazy deleted elements from tops of heaps.
```

```cpp
#include <vector>
#include <queue>
#include <unordered_map>

class SlidingWindowMedian {
    priority_queue<int> maxHeap;                            // lower half
    priority_queue<int, vector<int>, greater<int>> minHeap; // upper half
    unordered_map<int, int> lazyDeletes;
    int maxHeapSize = 0;
    int minHeapSize = 0;

    template <typename T>
    void clean(T& pq) {
        while (!pq.empty() && lazyDeletes[pq.top()] > 0) {
            lazyDeletes[pq.top()]--;
            pq.pop();
        }
    }

    void balance() {
        if (maxHeapSize > minHeapSize + 1) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
            maxHeapSize--;
            minHeapSize++;
            clean(maxHeap);
        } else if (minHeapSize > maxHeapSize) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
            minHeapSize--;
            maxHeapSize++;
            clean(minHeap);
        }
    }

public:
    vector<double> medianSlidingWindow(vector<int>& nums, int k) {
        vector<double> medians;
        int n = nums.size();
        
        for (int i = 0; i < n; i++) {
            // Step 1: Insert new element
            if (maxHeap.empty() || nums[i] <= maxHeap.top()) {
                maxHeap.push(nums[i]);
                maxHeapSize++;
            } else {
                minHeap.push(nums[i]);
                minHeapSize++;
            }
            balance();

            // Step 2: Remove outgoing element
            if (i >= k) {
                int outgoing = nums[i - k];
                lazyDeletes[outgoing]++;
                
                // Track relative heap sizes
                if (outgoing <= maxHeap.top()) {
                    maxHeapSize--;
                } else {
                    minHeapSize--;
                }
                
                clean(maxHeap);
                clean(minHeap);
                balance();
            }

            // Step 3: Record median
            if (i >= k - 1) {
                clean(maxHeap);
                clean(minHeap);
                if (k % 2 == 1) {
                    medians.push_back(maxHeap.top());
                } else {
                    medians.push_back(((double)maxHeap.top() + minHeap.top()) / 2.0);
                }
            }
        }
        return medians;
    }
};
// Time: O(n log n), Space: O(n)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | IPO | Hard | [LC 502](https://leetcode.com/problems/ipo/) |
| 2 | Sliding Window Median | Hard | [LC 480](https://leetcode.com/problems/sliding-window-median/) |
| 3 | Rearrange String k Distance Apart | Hard | [LC 358](https://leetcode.com/problems/rearrange-string-k-distance-apart/) |
| 4 | Minimum Number of Refueling Stops | Hard | [LC 871](https://leetcode.com/problems/minimum-number-of-refueling-stops/) |

---

*<- [[09_Heap_Priority_Queue_Problems_and_Exercises|Problems]] · [[../10_Backtracking/10_Backtracking_Index|Backtracking ->]]*

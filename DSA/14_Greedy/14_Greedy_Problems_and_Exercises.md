---
tags: [dsa, greedy, problems, exercises]
links: ["[[14_Greedy_Index]]", "[[14_Greedy_Patterns]]", "[[14_Greedy_Tricky]]"]
---

# Greedy Algorithms -- Problems & Exercises

*<- [[14_Greedy_Patterns|Patterns]] · [[14_Greedy_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Assign Cookies | Easy | Sort children & cookies, greedily match | [LC 455](https://leetcode.com/problems/assign-cookies/) |
| 2 | Fractional Knapsack | Medium | Density ratio sorting | [GFG](https://www.geeksforgeeks.org/fractional-knapsack-problem/) |
| 3 | Lemonade Change | Easy | Save $10 and $5 bills for change | [LC 860](https://leetcode.com/problems/lemonade-change/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Jump Game | Medium | Max reachability boundary tracking | [LC 55](https://leetcode.com/problems/jump-game/) |
| 5 | Jump Game II | Medium | Greedy BFS-like window traversal | [LC 45](https://leetcode.com/problems/jump-game-ii/) |
| 6 | Gas Station | Medium | Tank deficit reset | [LC 134](https://leetcode.com/problems/gas-station/) |
| 7 | Hand of Straights | Medium | Sorted frequency count matching | [LC 846](https://leetcode.com/problems/hand-of-straights/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Partition Labels | Medium | Last occurrence range partitioning | [LC 763](https://leetcode.com/problems/partition-labels/) |
| 9 | Queue Reconstruction by Height | Medium | Sort height descending, count ascending | [LC 406](https://leetcode.com/problems/queue-reconstruction-by-height/) |
| 10 | Task Scheduler | Medium | Max frequency placement | [LC 621](https://leetcode.com/problems/task-scheduler/) |
| 11 | Non-overlapping Intervals | Medium | Sort by end time, remove overlaps | [LC 435](https://leetcode.com/problems/non-overlapping-intervals/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 12 | Candy | Hard | Two-pass local neighbor conditions | [LC 135](https://leetcode.com/problems/candy/) |
| 13 | Minimum Number of Refueling Stops | Hard | Max-heap of fuel stations passed | [LC 871](https://leetcode.com/problems/minimum-number-of-refueling-stops/) |

---

## Worked Solution 1: LC 45 -- Jump Game II (Minimum Jumps)

**Key Insight**: Instead of Dynamic Programming ($O(n^2)$), we can solve this in $O(n)$ time using a greedy approach that simulates a BFS-like window of steps.
- We maintain `currentEnd` (the boundary of our current jump step) and `farthest` (the furthest index we can reach from any index within our current jump step window).
- When we reach `currentEnd`, we must make a new jump: we increment our `jumps` count and update `currentEnd = farthest`.

```cpp
#include <vector>
#include <algorithm>

int jump(vector<int>& nums) {
    int n = nums.size();
    if (n <= 1) return 0;

    int jumps = 0;
    int currentEnd = 0;
    int farthest = 0;

    for (int i = 0; i < n - 1; i++) {
        farthest = max(farthest, i + nums[i]); // track max reach possible in this window
        
        // If we reached the boundary of the current jump
        if (i == currentEnd) {
            jumps++;
            currentEnd = farthest; // jump to the farthest candidate
            if (currentEnd >= n - 1) break; // optimized exit
        }
    }
    return jumps;
}
// Time Complexity: O(n) — single pass
// Space Complexity: O(1)
```

---

## Worked Solution 2: LC 406 -- Queue Reconstruction by Height

**Key Insight**: Reconstruct a queue where each person is represented by `[h, k]` (height `h`, and `k` people in front who are taller or equal).
- **Greedy choice**: If we sort people by height in **descending order**, and by `k` in **ascending order** if heights are equal:
  - When we insert a person `[h, k]` into our result queue, all people already in the queue are taller than or equal to `h`.
  - Therefore, we can simply insert this person exactly at index `k` of the result list! Their position is guaranteed to be correct since any subsequent smaller person inserted won't affect their `k` count.

```cpp
#include <vector>
#include <algorithm>

// Custom comparator
bool cmpQueue(const vector<int>& a, const vector<int>& b) {
    if (a[0] != b[0]) {
        return a[0] > b[0]; // sort height descending
    }
    return a[1] < b[1]; // sort k ascending
}

vector<vector<int>> reconstructQueue(vector<vector<int>>& people) {
    sort(people.begin(), people.end(), cmpQueue); // Step 1: Sort people

    vector<vector<int>> queue;
    for (const auto& p : people) {
        // Step 2: Insert person exactly at index k
        queue.insert(queue.begin() + p[1], p);
    }
    return queue;
}
// Time Complexity: O(n^2) due to vector insertions shifts
// Space Complexity: O(n)
```

### Dry Run Variable State Table:
For `people = [[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]`:
1. **Sorted**: `[[7,0], [7,1], [6,1], [5,0], [5,2], [4,4]]`
2. **Insertions**:
   - Insert `[7,0]` at index 0 -> `[[7,0]]`
   - Insert `[7,1]` at index 1 -> `[[7,0], [7,1]]`
   - Insert `[6,1]` at index 1 -> `[[7,0], [6,1], [7,1]]` (6 is smaller, doesn't affect 7's k counts)
   - Insert `[5,0]` at index 0 -> `[[5,0], [7,0], [6,1], [7,1]]`
   - Insert `[5,2]` at index 2 -> `[[5,0], [7,0], [5,2], [6,1], [7,1]]`
   - Insert `[4,4]` at index 4 -> `[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]`
- **Result**: `[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]` (Correct).

---

*<- [[14_Greedy_Patterns|Patterns]] · [[14_Greedy_Tricky|Tricky ->]]*

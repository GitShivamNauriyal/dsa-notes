---
tags: [dsa, sorting, problems, quickselect]
links: ["[[00_Index]]", "[[Sorting_Algorithms_Patterns]]", "[[../Matrix/00_Index]]"]
---

# Sorting Algorithms -- Problems & Exercises

*<- [[Sorting_Algorithms_Patterns|Templates]] · [[../Matrix/00_Index|Matrix ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Sort an Array | Medium | Implement Merge Sort / Quick Sort | [LC 912](https://leetcode.com/problems/sort-an-array/) |
| 2 | Merge Sorted Array | Easy | Two-pointer backward merging | [LC 88](https://leetcode.com/problems/merge-sorted-array/) |

## Tier 2 -- Core Variations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | K-th Largest Element | Medium | QuickSelect ($O(N)$ average time) | [LC 215](https://leetcode.com/problems/k-th-largest-element-in-an-array/) |
| 4 | Sort Colors (Dutch Flag) | Medium | 3-pointer partition | [LC 75](https://leetcode.com/problems/sort-colors/) |
| 5 | Insertion Sort List | Medium | Sort Linked List using insertion sort | [LC 147](https://leetcode.com/problems/insertion-sort-list/) |

## Tier 3 -- Advanced Sorting

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Sort List (O(n log n)) | Medium | Merge Sort on Linked List | [LC 148](https://leetcode.com/problems/sort-list/) |
| 7 | Custom Sort String | Medium | Custom hashing order comparator | [LC 791](https://leetcode.com/problems/custom-sort-string/) |

---

## Worked Solution: LC 215 -- K-th Largest Element (QuickSelect)

**Key Insight**: Instead of sorting the whole array ($O(n \log n)$), we can find the $k$-th largest element in $O(n)$ average time using **QuickSelect** (based on QuickSort partitioning).
- If we partition the array around a pivot, the pivot ends up exactly in its final sorted position.
- If the pivot's index matches the index we are looking for ($N - k$ for $k$-th largest), we are done!
- Else, if the target index is smaller, recurse only on the left partition. Otherwise, recurse on the right partition.
- This cuts the problem size in half on average: $N + N/2 + N/4 + ... = 2N \implies O(N)$ time.

```cpp
#include <vector>
#include <algorithm>
#include <cstdlib>

using namespace std;

class Solution {
    int partition(vector<int>& nums, int low, int high) {
        // Random pivot selection to prevent worst-case O(n^2) on pre-sorted arrays
        int pivotIdx = low + rand() % (high - low + 1);
        swap(nums[pivotIdx], nums[high]);
        
        int pivot = nums[high];
        int i = low - 1;

        for (int j = low; j < high; j++) {
            if (nums[j] < pivot) {
                i++;
                swap(nums[i], nums[j]);
            }
        }
        swap(nums[i + 1], nums[high]);
        return i + 1;
    }

    int quickSelect(vector<int>& nums, int low, int high, int targetIdx) {
        if (low == high) return nums[low];

        int pi = partition(nums, low, high);

        if (pi == targetIdx) {
            return nums[pi];
        } else if (pi > targetIdx) {
            return quickSelect(nums, low, pi - 1, targetIdx);
        } else {
            return quickSelect(nums, pi + 1, high, targetIdx);
        }
    }

public:
    int findKthLargest(vector<int>& nums, int k) {
        int n = nums.size();
        return quickSelect(nums, 0, n - 1, n - k); // k-th largest is at sorted index n-k
    }
};
// Time Complexity: O(n) average, O(n^2) worst case
// Space Complexity: O(1) auxiliary space (excluding recursion stack)
```

---

*<- [[Sorting_Algorithms_Patterns|Templates]] · [[../Matrix/00_Index|Matrix ->]]*

---
tags: [dsa, two-pointers, patterns]
links: ["[[02_Two_Pointers_Index]]", "[[02_Two_Pointers_Problems_and_Exercises]]"]
---

# Two Pointers -- Patterns

*<- [[02_Two_Pointers_Index|Index]] · [[02_Two_Pointers_Problems_and_Exercises|Problems ->]]*

---

## What is Two Pointers?

Instead of using a nested loop (two indices both starting from 0 and scanning everything), you maintain two indices that move intelligently -- reducing O(n²) work to O(n).

The key question to ask: **can I make progress toward the answer by moving just one pointer at a time based on a comparison?** If yes, two pointers applies.

---

## Pattern 1: Opposite Ends (Shrinking Window)

**Setup**: `left = 0`, `right = n-1`. Move them toward each other.  
**When to use**: Array is sorted. You need pairs/triplets summing to a target, or comparing from both ends.

### Sub-pattern 1a: Two Sum on Sorted Array

```
Sorted array: [1, 2, 4, 6, 8, 9], target = 10

left=0 (val=1), right=5 (val=9): sum=10 -> found!
```

```cpp
#include <vector>

// LC 167 -- Two Sum II (sorted array, 1-indexed)
vector<int> twoSum(vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;

    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) {
            return {left + 1, right + 1};  // 1-indexed
        } else if (sum < target) {
            left++;   // sum too small, increase it by moving left right
        } else {
            right--;  // sum too big, decrease it by moving right left
        }
    }
    return {};
}
// Time: O(n), Space: O(1)
```

### Sub-pattern 1b: Three Sum (Sorted + Fix One + Two Pointer)

**Why**: Fix one element, reduce to Two Sum on the rest. Sorting first enables two pointer and easy duplicate skipping.

```
Sort: [-4, -1, -1, 0, 1, 2], target sum = 0

Fix i=0 (-4): left=1, right=5 -> -4 + (-1) + 2 = -3 < 0, left++
              left=2, right=5 -> -4 + (-1) + 2 = -3 < 0, left++
              left=3, right=5 -> -4 + 0 + 2 = -2 < 0, left++
              left=4, right=5 -> -4 + 1 + 2 = -1 < 0, left++
              left=5 >= right=5, done with i=0

Fix i=1 (-1): left=2, right=5 -> -1 + (-1) + 2 = 0 -> FOUND [-1,-1,2]
              move both: left=3, right=4 -> -1 + 0 + 1 = 0 -> FOUND [-1,0,1]
              move both: left=4 >= right=4, done

Fix i=2 (-1): SKIP -- same as i=1 (duplicate)
...
```

```cpp
#include <algorithm>
#include <vector>

// LC 15 -- 3Sum
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    int n = nums.size();

    for (int i = 0; i < n - 2; i++) {
        // Skip duplicate values for the fixed element
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        // Optimization: if smallest three are already > 0, no solution possible
        if (nums[i] > 0) break;

        int left = i + 1, right = n - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.push_back({nums[i], nums[left], nums[right]});
                // Skip duplicates for left and right
                while (left < right && nums[left] == nums[left + 1]) left++;
                while (left < right && nums[right] == nums[right - 1]) right--;
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
// Time: O(n²), Space: O(1) (excluding output)
```

### Sub-pattern 1c: Four Sum

**Why**: Fix two elements, reduce to Two Sum. O(n³) total.

```cpp
vector<vector<int>> fourSum(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    int n = nums.size();

    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;
        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j-1]) continue;
            int left = j + 1, right = n - 1;
            while (left < right) {
                long long sum = (long long)nums[i] + nums[j] + nums[left] + nums[right];
                // Use long long to avoid overflow -- target can push sum past int range
                if (sum == target) {
                    result.push_back({nums[i], nums[j], nums[left], nums[right]});
                    while (left < right && nums[left] == nums[left+1]) left++;
                    while (left < right && nums[right] == nums[right-1]) right--;
                    left++; right--;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }
    }
    return result;
}
// Time: O(n³), Space: O(1) excluding output
```

---

## Pattern 2: Same Direction (Fast Read / Slow Write)

**Setup**: Both pointers start at 0, but move at different rates.  
**When to use**: In-place filtering, compacting arrays, removing elements.

```
nums = [3, 2, 2, 3], val = 3
write=0, read=0: nums[0]=3 -> skip, read++
write=0, read=1: nums[1]=2 -> write nums[write]=2, write++, read++
write=1, read=2: nums[2]=2 -> write nums[write]=2, write++, read++
write=2, read=3: nums[3]=3 -> skip, read++
Result: [2, 2, _, _], return write=2
```

```cpp
// LC 27 -- Remove Element (in-place)
int removeElement(vector<int>& nums, int val) {
    int write = 0;
    for (int read = 0; read < (int)nums.size(); read++) {
        if (nums[read] != val) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
// Time: O(n), Space: O(1)

// LC 26 -- Remove Duplicates from Sorted Array
int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    int write = 1;
    for (int read = 1; read < (int)nums.size(); read++) {
        if (nums[read] != nums[read - 1]) {
            nums[write++] = nums[read];
        }
    }
    return write;
}
```

---

## Pattern 3: Fast-Slow Pointers (Floyd's Cycle / Linked List Style)

**Setup**: Both start at 0. Slow moves +1, fast moves +2 (or jumps by value).  
**When to use**: Detect cycles, find middle of sequence, find duplicate.

```cpp
// Find middle of a sequence (conceptual -- more useful in linked list)
// Slow reaches middle when fast reaches end

// Find if array-as-linked-list has cycle (LC 287 -- find duplicate)
int findDuplicate(vector<int>& nums) {
    int slow = nums[0], fast = nums[0];
    // Phase 1: detect cycle
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    // Phase 2: find cycle entrance
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

---

## Pattern 4: Partition (Dutch National Flag / Quick Select)

**Setup**: Partition array around a pivot in O(n) using two pointers.

```cpp
// Lomuto partition (simpler, used in QuickSort/QuickSelect)
int partition(vector<int>& nums, int lo, int hi) {
    int pivot = nums[hi];   // choose last element as pivot
    int i = lo - 1;         // i = rightmost position of "less than pivot" region

    for (int j = lo; j < hi; j++) {
        if (nums[j] <= pivot) {
            i++;
            swap(nums[i], nums[j]);
        }
    }
    swap(nums[i + 1], nums[hi]);  // place pivot in correct position
    return i + 1;                       // pivot index
}

// Hoare partition (faster in practice -- fewer swaps)
int partitionHoare(vector<int>& nums, int lo, int hi) {
    int pivot = nums[lo + (hi - lo) / 2];
    int i = lo - 1, j = hi + 1;
    while (true) {
        do { i++; } while (nums[i] < pivot);
        do { j--; } while (nums[j] > pivot);
        if (i >= j) return j;
        swap(nums[i], nums[j]);
    }
}
```

### QuickSelect -- k-th Smallest in O(n) Average

```cpp
// LC 215 -- Kth Largest Element in Array
// Why: we don't need full sort. Partition to find pivot position.
// If pivot is at index k, we found it. Otherwise recurse on one side.
int quickSelect(vector<int>& nums, int lo, int hi, int k) {
    if (lo == hi) return nums[lo];
    int pivotIdx = partition(nums, lo, hi);
    if (pivotIdx == k) return nums[k];
    else if (k < pivotIdx) return quickSelect(nums, lo, pivotIdx - 1, k);
    else return quickSelect(nums, pivotIdx + 1, hi, k);
}

int findKthLargest(vector<int>& nums, int k) {
    int n = nums.size();
    // k-th largest = index (n-k) when sorted ascending
    return quickSelect(nums, 0, n - 1, n - k);
}
// Time: O(n) average, O(n²) worst (use random pivot to avoid)
// Space: O(log n) average (recursion stack)
```

---

## Pattern 5: Two Pointer on Strings

```cpp
// LC 125 -- Valid Palindrome (skip non-alphanumeric)
bool isPalindrome(string s) {
    int left = 0, right = (int)s.size() - 1;
    while (left < right) {
        while (left < right && !isalnum(s[left]))  left++;
        while (left < right && !isalnum(s[right])) right--;
        if (tolower(s[left]) != tolower(s[right])) return false;
        left++; right--;
    }
    return true;
}
```

---

## Examples

### Example 1 -- LC 11: Container With Most Water (Medium)

**Key insight**: Two pointers at ends. The area is limited by the shorter bar. Moving the shorter pointer inward is the only way to potentially find a larger container -- moving the taller one can only decrease area.

```
heights = [1,8,6,2,5,4,8,3,7]
left=0(h=1), right=8(h=7): area=1*8=8, move left (shorter)
left=1(h=8), right=8(h=7): area=7*7=49, move right (shorter)
left=1(h=8), right=7(h=3): area=3*6=18, move right
left=1(h=8), right=6(h=8): area=8*5=40, move either
...
```

```cpp
int maxArea(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1;
    int maxWater = 0;
    while (left < right) {
        int water = min(height[left], height[right]) * (right - left);
        maxWater = max(maxWater, water);
        if (height[left] < height[right]) left++;
        else right--;
    }
    return maxWater;
}
// Time: O(n), Space: O(1)
```

### Example 2 -- LC 977: Squares of a Sorted Array (Easy)

**Key insight**: Largest squares come from either end (most negative or most positive). Use two pointers filling result from back to front.

```cpp
vector<int> sortedSquares(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n);
    int left = 0, right = n - 1, pos = n - 1;
    while (left <= right) {
        int lsq = nums[left] * nums[left];
        int rsq = nums[right] * nums[right];
        if (lsq > rsq) { result[pos--] = lsq; left++; }
        else            { result[pos--] = rsq; right--; }
    }
    return result;
}
```

### Example 3 -- LC 16: 3Sum Closest (Medium)

```cpp
int threeSumClosest(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    int closest = nums[0] + nums[1] + nums[2];
    for (int i = 0; i < (int)nums.size() - 2; i++) {
        int left = i + 1, right = (int)nums.size() - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (abs(sum - target) < abs(closest - target))
                closest = sum;
            if (sum < target) left++;
            else if (sum > target) right--;
            else return sum;  // exact match
        }
    }
    return closest;
}
```

---

## Exercises

| # | Problem | Difficulty | Pattern | Link |
|---|---------|------------|---------|------|
| 1 | Valid Palindrome | Easy | Opposite ends, string | [LC 125](https://leetcode.com/problems/valid-palindrome/) |
| 2 | Two Sum II (sorted) | Easy | Opposite ends | [LC 167](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) |
| 3 | Squares of Sorted Array | Easy | Opposite ends, fill back | [LC 977](https://leetcode.com/problems/squares-of-a-sorted-array/) |
| 4 | Remove Duplicates from Sorted Array | Easy | Same direction | [LC 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) |
| 5 | Remove Element | Easy | Same direction | [LC 27](https://leetcode.com/problems/remove-element/) |
| 6 | Move Zeroes | Easy | Same direction | [LC 283](https://leetcode.com/problems/move-zeroes/) |
| 7 | Container With Most Water | Medium | Opposite ends | [LC 11](https://leetcode.com/problems/container-with-most-water/) |
| 8 | 3Sum | Medium | Fix + opposite ends | [LC 15](https://leetcode.com/problems/3sum/) |
| 9 | 3Sum Closest | Medium | Fix + opposite ends | [LC 16](https://leetcode.com/problems/3sum-closest/) |
| 10 | 4Sum | Medium | Fix two + opposite ends | [LC 18](https://leetcode.com/problems/4sum/) |
| 11 | Kth Largest Element | Medium | QuickSelect | [LC 215](https://leetcode.com/problems/kth-largest-element-in-an-array/) |
| 12 | Sort Array by Parity | Easy | Same direction | [LC 905](https://leetcode.com/problems/sort-array-by-parity/) |
| 13 | Partition Array | Medium | Partition | [GFG](https://www.geeksforgeeks.org/quick-sort/) |
| 14 | Remove Dups II (allow up to 2) | Medium | Same direction | [LC 80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) |

---

*<- [[02_Two_Pointers_Index|Index]] · [[02_Two_Pointers_Tricky|Tricky ->]]*

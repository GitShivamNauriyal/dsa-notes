---
tags: [dsa, binary-search, patterns]
links: ["[[00_Index]]", "[[02_Problems_and_Exercises]]"]
---

# Binary Search -- Patterns

*<- [[00_Index|Index]] · [[02_Problems_and_Exercises|Problems ->]]*

---

## The Core Idea

Binary search eliminates half the search space in every step. To use it, you need a **monotonic predicate**: once the predicate becomes true (or false), it stays that way. This is what makes halving valid.

If you can phrase your problem as "find the boundary where predicate P(x) flips from false to true", binary search applies.

---

## Pattern 1: Classic Target Search (Exact Match)

**When**: Find the exact index of `target` in a sorted array.  
**Loop invariant**: The target, if it exists, is always in `[left, right]`.  
**Termination**: Loop ends when `left > right` (window closed completely).

```
nums = [1, 3, 5, 7, 9], target = 5
left=0, right=4, mid=2: nums[2]=5 == target -> return 2

nums = [1, 3, 5, 7, 9], target = 4
left=0, right=4, mid=2: nums[2]=5 > 4 -> right=1
left=0, right=1, mid=0: nums[0]=1 < 4 -> left=1
left=1, right=1, mid=1: nums[1]=3 < 4 -> left=2
left=2 > right=1: loop ends -> return -1
```

```cpp
#include <vector>

int binarySearch(const std::vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;

    while (left <= right) {
        // Why left + (right-left)/2 and not (left+right)/2?
        // Because left+right can overflow int when both are large.
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;   // target is in right half
        } else {
            right = mid - 1;  // target is in left half
        }
    }
    return -1;
}
// Time: O(log n), Space: O(1)
```

---

## Pattern 2: Lower Bound -- First Position >= Target

**When**: Find the first index where `nums[index] >= target`. If all elements are smaller, returns `n` (off the end).  
**Key difference from classic**: Loop condition is `left < right` (not `<=`). Loop ends when `left == right`.  
**Why `right = mid` instead of `mid - 1`**: `mid` itself could be the answer, so don't exclude it.

```
nums = [1, 3, 3, 5, 7], target = 3
left=0, right=5 (size, not size-1!)
mid=2: nums[2]=3 >= 3 -> right=2   (mid could be the answer)
mid=1: nums[1]=3 >= 3 -> right=1
mid=0: nums[0]=1 < 3  -> left=1
left=1 == right=1 -> return 1    (first index where value >= 3)
```

```cpp
int lowerBound(const std::vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size();  // right = n, NOT n-1

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) {
            right = mid;        // mid is a candidate, but look for earlier one
        } else {
            left = mid + 1;     // nums[mid] is too small, go right
        }
    }
    return left;
    // Verify: if left < n && nums[left] == target, then target exists
}
// STL: std::lower_bound(nums.begin(), nums.end(), target) - nums.begin()
```

---

## Pattern 3: Upper Bound -- First Position > Target

**When**: Find the first index where `nums[index] > target`.  
**Use case**: Count occurrences of target = upperBound(target) - lowerBound(target).

```cpp
int upperBound(const std::vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size();

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) {
            left = mid + 1;    // move past target elements
        } else {
            right = mid;       // nums[mid] > target, could be answer
        }
    }
    return left;
}
// STL: std::upper_bound(nums.begin(), nums.end(), target) - nums.begin()

// Count occurrences of target in sorted array
int countOccurrences(const std::vector<int>& nums, int target) {
    return upperBound(nums, target) - lowerBound(nums, target);
}
```

---

## Pattern 4: Binary Search on Answer Space

**When**: The array itself may not be sorted, but the **answer** (a value you're optimizing) lies in a known range, and you have an O(n) way to check if a given answer works.

**How to recognize it**: The problem asks for "minimum X such that..." or "maximum X such that..." and you can write a `canDo(mid)` function.

**Template**:

```
Find minimum answer:
  low = smallest possible answer
  high = largest possible answer
  while low <= high:
    mid = midpoint
    if canDo(mid): result = mid; high = mid - 1  (try smaller)
    else:          low = mid + 1                 (need larger)

Find maximum answer:
  if canDo(mid): result = mid; low = mid + 1   (try larger)
  else:          high = mid - 1                (need smaller)
```

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

// LC 1011 -- Capacity to Ship Packages Within D Days
// Binary search on answer = ship capacity

bool canShip(const std::vector<int>& weights, int capacity, int days) {
    int daysUsed = 1, load = 0;
    for (int w : weights) {
        if (load + w > capacity) { daysUsed++; load = 0; }
        load += w;
    }
    return daysUsed <= days;
}

int shipWithinDays(const std::vector<int>& weights, int days) {
    // Minimum capacity: must carry the heaviest single package
    int low = *std::max_element(weights.begin(), weights.end());
    // Maximum capacity: carry everything in one day
    int high = std::accumulate(weights.begin(), weights.end(), 0);
    int result = high;

    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (canShip(weights, mid, days)) {
            result = mid;
            high = mid - 1;  // try smaller capacity
        } else {
            low = mid + 1;   // capacity too small
        }
    }
    return result;
}
// Time: O(n log(sum-max)), Space: O(1)
```

### More Examples of Answer Space

```cpp
// LC 875 -- Koko Eating Bananas
bool canEat(std::vector<int>& piles, int k, int h) {
    long long hours = 0;
    for (int p : piles) hours += (p + k - 1) / k;  // ceil(p/k)
    return hours <= h;
}
int minEatingSpeed(std::vector<int>& piles, int h) {
    int low = 1, high = *std::max_element(piles.begin(), piles.end()), ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (canEat(piles, mid, h)) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}

// LC 410 -- Split Array Largest Sum (minimize the maximum sum among k groups)
bool canSplit(std::vector<int>& nums, int maxSum, int k) {
    int parts = 1, cur = 0;
    for (int x : nums) {
        if (cur + x > maxSum) { parts++; cur = 0; }
        cur += x;
    }
    return parts <= k;
}
int splitArray(std::vector<int>& nums, int k) {
    int low = *std::max_element(nums.begin(), nums.end());
    int high = std::accumulate(nums.begin(), nums.end(), 0), ans = high;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (canSplit(nums, mid, k)) { ans = mid; high = mid - 1; }
        else low = mid + 1;
    }
    return ans;
}
```

---

## Pattern 5: Binary Search on Rotated Sorted Array

**When**: Array was sorted but then rotated at some pivot. Find target or find minimum.

**Key insight**: At any `mid`, **one of the two halves is always fully sorted**. Determine which half is sorted, then check if target falls in that half's range.

```
[4, 5, 6, 7, 0, 1, 2]  rotated at index 4
 ^           ^      ^
left=0       mid=3  right=6

nums[left]=4 <= nums[mid]=7: left half [4..7] is sorted
target=0: not in [4,7] -> search right half
```

```cpp
// LC 33 -- Search in Rotated Sorted Array
int search(std::vector<int>& nums, int target) {
    int left = 0, right = (int)nums.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;

        if (nums[left] <= nums[mid]) {
            // Left half [left..mid] is sorted
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;  // target in sorted left half
            } else {
                left = mid + 1;   // target in right half
            }
        } else {
            // Right half [mid..right] is sorted
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;   // target in sorted right half
            } else {
                right = mid - 1;  // target in left half
            }
        }
    }
    return -1;
}
// Time: O(log n), Space: O(1)
```

---

## Pattern 6: Find Minimum in Rotated Sorted Array

**Key insight**: Compare `nums[mid]` with `nums[right]` (not left).  
If `nums[mid] > nums[right]`, the minimum must be in the right half (since left half is sorted and starts higher).  
If `nums[mid] <= nums[right]`, the minimum is in the left half (including mid).

```
[4, 5, 6, 7, 0, 1, 2]
mid = nums[3]=7 > nums[6]=2 -> min is in right half: left = mid+1
mid = nums[4]=0 <= nums[6]=2 -> min is in left half: right = mid
mid = nums[4]=0 -> left == right -> return nums[left]=0
```

```cpp
// LC 153 -- Find Minimum in Rotated Sorted Array
int findMin(std::vector<int>& nums) {
    int left = 0, right = (int)nums.size() - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] > nums[right]) {
            left = mid + 1;   // min is in right half
        } else {
            right = mid;      // min is at mid or left of mid
        }
    }
    return nums[left];
}
```

---

## Pattern 7: Peak Finding

**When**: Array first increases then decreases (mountain array). Find the peak.  
**Key**: If `nums[mid] < nums[mid+1]`, peak is to the right. Otherwise left.

```cpp
// LC 162 -- Find Peak Element (any peak in array where neighbors are smaller)
int findPeakElement(std::vector<int>& nums) {
    int left = 0, right = (int)nums.size() - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < nums[mid + 1]) {
            left = mid + 1;   // ascending slope, peak is to the right
        } else {
            right = mid;      // descending slope, peak is at mid or left
        }
    }
    return left;  // left == right == peak index
}

// LC 852 -- Peak Index in a Mountain Array
int peakIndexInMountainArray(std::vector<int>& arr) {
    int left = 0, right = (int)arr.size() - 1;
    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < arr[mid + 1]) left = mid + 1;
        else right = mid;
    }
    return left;
}
```

---

## STL Binary Search Reference

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v = {1, 2, 2, 3, 4, 5};

// Check existence
bool found = std::binary_search(v.begin(), v.end(), 3);  // true

// First >= target
auto it1 = std::lower_bound(v.begin(), v.end(), 2);
int idx1 = it1 - v.begin();  // = 1 (first 2)

// First > target
auto it2 = std::upper_bound(v.begin(), v.end(), 2);
int idx2 = it2 - v.begin();  // = 3 (first element > 2)

// Count of target
int cnt = (int)(it2 - it1);  // = 2 (two 2s)

// On custom comparator (sorts by second element of pair, searches by second)
std::vector<std::pair<int,int>> vp = {{1,3},{2,5},{3,7}};
// std::lower_bound with comparator -- finds first pair where second >= 5
auto it3 = std::lower_bound(vp.begin(), vp.end(), std::make_pair(0, 5),
    [](const std::pair<int,int>& p, const std::pair<int,int>& val) {
        return p.second < val.second;
    });
```

---

*<- [[00_Index|Index]] · [[02_Problems_and_Exercises|Problems ->]]*

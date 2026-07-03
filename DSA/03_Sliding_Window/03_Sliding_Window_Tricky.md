---
tags: [dsa, sliding-window, tricky, hard, interview]
links: ["[[03_Sliding_Window_Index]]", "[[03_Sliding_Window_Problems_and_Exercises]]", "[[../04_Stack/04_Stack_Index]]"]
---

# Sliding Window -- Tricky & Higher-Order

*<- [[03_Sliding_Window_Problems_and_Exercises|Problems]] · [[../04_Stack/04_Stack_Index|Stack ->]]*

---

## 1. The "Exactly K" Meta-Pattern

**Why tricky**: Sliding window naturally handles "at most K" but not "exactly K". Use:

```
count(exactly K) = count(at most K) - count(at most K-1)
```

This converts any "exactly K" problem into two "at most K" problems.

```cpp
// Template for "at most K" subarray count
int atMost(std::vector<int>& nums, int k) {
    std::unordered_map<int,int> freq;
    int left = 0, count = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        freq[nums[right]]++;
        while ((int)freq.size() > k) {
            freq[nums[left]]--;
            if (freq[nums[left]] == 0) freq.erase(nums[left]);
            left++;
        }
        count += right - left + 1;
        // right - left + 1 = number of valid subarrays ending at 'right'
    }
    return count;
}

// LC 992 -- Subarrays with K Different Integers
int subarraysWithKDistinct(std::vector<int>& nums, int k) {
    return atMost(nums, k) - atMost(nums, k - 1);
}
```

---

## 2. Sliding Window on a Circular Array

**Why tricky**: Array wraps around. Duplicate the array (or use modular indexing) and run fixed-size window of length n on 2n array, but disallow windows > n.

```cpp
// Max sum subarray of length k in a circular array
int maxCircularSum(std::vector<int>& nums, int k) {
    int n = nums.size();
    // Duplicate conceptually: use modular index
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];
    int maxSum = windowSum;

    for (int i = k; i < n + k - 1; i++) {
        // i % n gives circular index; i - k gives element leaving window
        windowSum += nums[i % n] - nums[(i - k) % n];
        maxSum = std::max(maxSum, windowSum);
    }
    return maxSum;
}
// Constraint: window wraps but can't exceed length n
```

---

## 3. Minimum Window with All Distinct Characters

**Why tricky**: Not matching a pattern string -- the window must contain all characters present in the entire string at least once (or some target set).

```cpp
// Find minimum window containing all k distinct characters of s itself
std::string minWindowAllDistinct(std::string s) {
    std::unordered_set<char> allChars(s.begin(), s.end());
    int k = allChars.size();
    return "";  // apply minimum window template with need = allChars
    // Exercise: implement using the standard minimum window template
}
```

---

## 4. Sliding Window with Product Constraint (LC 713)

**Why tricky**: Products overflow easily. Also, when you multiply in a new element, the product changes multiplicatively -- but when you divide out left elements, you must handle zeros (division by zero).

```cpp
// LC 713 -- Number of Subarrays with Product Less Than K
// Trick: don't divide when shrinking (use cumulative product and reset on shrink)
int numSubarrayProductLessThanK(std::vector<int>& nums, int k) {
    if (k <= 1) return 0;
    int left = 0, product = 1, count = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        product *= nums[right];
        while (product >= k) product /= nums[left++];
        // All subarrays ending at right with left..right are valid
        count += right - left + 1;
    }
    return count;
}
// Time: O(n), Space: O(1)
// Why division is safe here: all nums[i] >= 1 (no zeros in this problem)
```

---

## 5. Minimum Number of Swaps to Group All 1s (LC 1151)

**Why tricky**: Find a window of size (count of 1s) that contains the maximum number of 1s. Minimum swaps = window_size - max_ones_in_window.

```cpp
int minSwaps(std::vector<int>& data) {
    int totalOnes = 0;
    for (int x : data) totalOnes += x;
    if (totalOnes == 0) return 0;

    // Fixed window of size totalOnes
    int onesInWindow = 0;
    for (int i = 0; i < totalOnes; i++) onesInWindow += data[i];
    int maxOnes = onesInWindow;

    for (int i = totalOnes; i < (int)data.size(); i++) {
        onesInWindow += data[i] - data[i - totalOnes];
        maxOnes = std::max(maxOnes, onesInWindow);
    }
    return totalOnes - maxOnes;
}
```

---

## 6. Sliding Window with Ordered Set (When You Need Sorted Window)

**Why tricky**: Sometimes you need the median, or the k-th element, of a sliding window. A deque gives max/min but not the median. Use two ordered multisets (or two heaps).

```cpp
// LC 480 -- Sliding Window Median
// Use two multisets: lo (max-heap behavior) and hi (min-heap behavior)
// lo stores the smaller half, hi stores the larger half
// Median = lo.rbegin() if odd size, or avg of lo.rbegin() and hi.begin() if even

#include <set>
class SlidingWindowMedian {
    std::multiset<int> lo, hi;  // lo = lower half, hi = upper half

    void balance() {
        // Maintain: lo.size() == hi.size() or lo.size() == hi.size() + 1
        while (lo.size() > hi.size() + 1) {
            hi.insert(*lo.rbegin());
            lo.erase(lo.find(*lo.rbegin()));
        }
        while (hi.size() > lo.size()) {
            lo.insert(*hi.begin());
            hi.erase(hi.begin());
        }
        // Also maintain: max(lo) <= min(hi)
        while (!lo.empty() && !hi.empty() && *lo.rbegin() > *hi.begin()) {
            hi.insert(*lo.rbegin());
            lo.erase(lo.find(*lo.rbegin()));
            lo.insert(*hi.begin());
            hi.erase(hi.begin());
        }
    }

public:
    std::vector<double> medianSlidingWindow(std::vector<int>& nums, int k) {
        std::vector<double> result;
        for (int i = 0; i < (int)nums.size(); i++) {
            lo.insert(nums[i]);
            balance();
            if (i >= k) {
                // Remove outgoing element
                int out = nums[i - k];
                if (lo.count(out)) lo.erase(lo.find(out));
                else hi.erase(hi.find(out));
                balance();
            }
            if (i >= k - 1) {
                double median = (k % 2 == 1)
                    ? (double)*lo.rbegin()
                    : ((double)*lo.rbegin() + *hi.begin()) / 2.0;
                result.push_back(median);
            }
        }
        return result;
    }
};
// Time: O(n log k), Space: O(k)
```

---

## 7. Binary Search on Window Size

**Why tricky**: Sometimes the question is "what is the MINIMUM window size such that some condition holds?" -- this is a binary search on window size combined with sliding window validation.

```cpp
// Generic template:
// Check if any window of size 'len' satisfies the condition
bool canAchieve(std::vector<int>& nums, int len, /* condition params */) {
    // Fixed window of size len -- check condition
    // ...
    return false;
}

int minWindowSize(std::vector<int>& nums /* condition params */) {
    int lo = 1, hi = nums.size(), ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canAchieve(nums, mid /* ... */)) {
            ans = mid;
            hi = mid - 1;  // try smaller
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}
// Total: O(n log n)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Subarrays with K Different Integers | Hard | [LC 992](https://leetcode.com/problems/subarrays-with-k-different-integers/) |
| 2 | Number of Subarrays with Product < K | Medium | [LC 713](https://leetcode.com/problems/subarray-product-less-than-k/) |
| 3 | Sliding Window Median | Hard | [LC 480](https://leetcode.com/problems/sliding-window-median/) |
| 4 | Minimum Swaps to Group 1s | Medium | [LC 1151](https://leetcode.com/problems/minimum-swaps-to-group-all-1s-together/) |
| 5 | Longest Continuous Subarray Abs Diff <= Limit | Hard | [LC 1438](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/) |
| 6 | Minimum Number of K Consecutive Bit Flips | Hard | [LC 995](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/) |
| 7 | Maximum Sum of Two Non-Overlapping Subarrays | Medium | [LC 1031](https://leetcode.com/problems/maximum-sum-of-two-non-overlapping-subarrays/) |
| 8 | Count Subarrays Where Max Element Appears at Least K Times | Medium | [LC 2962](https://leetcode.com/problems/count-subarrays-where-max-element-appears-at-least-k-times/) |

---

*<- [[03_Sliding_Window_Problems_and_Exercises|Problems]] · [[../04_Stack/04_Stack_Index|Stack ->]]*

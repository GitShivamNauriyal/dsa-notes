---
tags: [dsa, arrays, hashing, tricky, hard, interview]
links: ["[[01_Arrays_and_Hashing_Index]]", "[[01_Arrays_and_Hashing_Problems_and_Exercises]]"]
---

# Arrays & Hashing — Tricky & Higher-Order Problems

*← [[01_Arrays_and_Hashing_Problems_and_Exercises\|Problems]] · [[../02_Two_Pointers/02_Two_Pointers_Index\|Two Pointers →]]*

---

> These problems require non-obvious insight. They appear in FAANG/quant interviews and hard OAs. Each has a "why is it tricky" section before the solution.

---

## 1. Array as an Implicit Graph — Find the Duplicate (LC 287)

**Why tricky**: Given `nums` of size `n+1` with values in `[1,n]`, find the duplicate. Constraints: no modifying the array, O(1) extra space.

**Why O(1) space is hard**: You can't use a HashSet. You can't sort (modifies). The trick is treating the array as a **linked list with cycles** — index `i` points to `nums[i]`. Since there's a duplicate, there must be a cycle. Use Floyd's cycle detection.

```cpp
int findDuplicate(vector<int>& nums) {
    // Phase 1: Find intersection point inside the cycle
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);

    // Phase 2: Find cycle entrance = duplicate
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
// Time: O(n), Space: O(1)
// The insight: index-to-value mapping creates a functional graph.
// Duplicate value = two indices pointing to the same next node = cycle entrance.
```

---

## 2. First Missing Positive via Index Mapping (LC 41)

Already shown in problems, but the insight deserves emphasis:

**Why tricky**: The constraint "O(n) time, O(1) space, no extra array" seems impossible. The trick: the array itself is your hash table. Values outside `[1,n]` are useless — ignore them. Use cyclic sort to place every valid value at its correct index.

The moment we scan and find `nums[i] != i+1`, that index is the answer. No extra memory used.

---

## 3. Longest Subarray Where Max - Min <= Limit (LC 1438)

**Why tricky**: Sliding window, but you need both max and min of the current window efficiently as it slides.

**Key insight**: Use two monotonic deques — one for max (decreasing), one for min (increasing). When the window violates the condition, shrink from the left and pop from both deques if the front is the element being removed.

```cpp
#include <deque>
int longestSubarray(vector<int>& nums, int limit) {
    deque<int> maxDq, minDq;  // store indices
    int left = 0, ans = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        // Maintain decreasing deque for max
        while (!maxDq.empty() && nums[maxDq.back()] <= nums[right])
            maxDq.pop_back();
        maxDq.push_back(right);

        // Maintain increasing deque for min
        while (!minDq.empty() && nums[minDq.back()] >= nums[right])
            minDq.pop_back();
        minDq.push_back(right);

        // Shrink window if condition violated
        while (nums[maxDq.front()] - nums[minDq.front()] > limit) {
            left++;
            if (maxDq.front() < left) maxDq.pop_front();
            if (minDq.front() < left) minDq.pop_front();
        }
        ans = max(ans, right - left + 1);
    }
    return ans;
}
// Time: O(n), Space: O(n)
```

---

## 4. Count Subarrays with Exactly K Distinct (LC 992)

**Why tricky**: Sliding window naturally handles "at most K distinct" but NOT "exactly K". The trick:

```
exactly(K) = atMost(K) - atMost(K-1)
```

This is a meta-pattern that appears in many "exactly K" subarray problems.

```cpp
int atMostK(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    int left = 0, count = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        freq[nums[right]]++;
        while ((int)freq.size() > k) {
            freq[nums[left]]--;
            if (freq[nums[left]] == 0) freq.erase(nums[left]);
            left++;
        }
        count += right - left + 1;  // all subarrays ending at right with <= k distinct
    }
    return count;
}

int subarraysWithKDistinct(vector<int>& nums, int k) {
    return atMostK(nums, k) - atMostK(nums, k - 1);
}
// Time: O(n), Space: O(k)
```

---

## 5. Maximum Sum of Subarray with No Adjacent Elements after Deletion (Custom / OA variant)

**Why tricky**: Sometimes asked as "delete exactly one element, maximize subarray sum." You need Kadane's running both left-to-right and right-to-left, then combine.

```cpp
// Build:
// left[i]  = max subarray sum ending at i
// right[i] = max subarray sum starting at i
// Answer = max over all i of: left[i-1] + right[i+1]  (skip element i)

int maxSumAfterDeletion(vector<int>& nums) {
    int n = nums.size();
    vector<int> left(n), right(n);

    left[0] = nums[0];
    for (int i = 1; i < n; i++)
        left[i] = max(nums[i], left[i-1] + nums[i]);

    right[n-1] = nums[n-1];
    for (int i = n-2; i >= 0; i--)
        right[i] = max(nums[i], right[i+1] + nums[i]);

    int ans = *max_element(left.begin(), left.end()); // delete nothing
    for (int i = 1; i < n-1; i++)
        ans = max(ans, left[i-1] + right[i+1]);

    return ans;
}
```

---

## 6. Minimum Number of Operations to Make Array Continuous (LC 2009)

**Why tricky**: "Continuous" means all elements distinct and form a range of length n. You need to find the maximum number of elements you can KEEP (so the answer is n minus that).

**Key insight**: Sort + deduplicate. Then use a sliding window on the sorted unique array: for a window starting at `sorted[i]`, the valid range is `[sorted[i], sorted[i] + n - 1]`. Binary search for how many sorted unique values fall in this range. The window that fits the most elements tells you the minimum replacements.

```cpp
#include <algorithm>
int minOperations(vector<int>& nums) {
    int n = nums.size();
    sort(nums.begin(), nums.end());
    nums.erase(unique(nums.begin(), nums.end()), nums.end());
    int m = nums.size();
    int best = 0;
    for (int i = 0; i < m; i++) {
        // How many unique values fit in [nums[i], nums[i]+n-1]?
        int end = nums[i] + n - 1;
        int j = (int)(upper_bound(nums.begin(), nums.end(), end) - nums.begin());
        best = max(best, j - i);  // j-i values we can keep
    }
    return n - best;
}
// Time: O(n log n), Space: O(1) after sort
```

---

## 7. Count of Range Sum (LC 327) — Hard

**Why tricky**: Count subarrays with sum in `[lower, upper]`. Naive is O(n²). Optimal uses prefix sums + merge sort (or BIT/segment tree).

**Insight**: `prefix[j] - prefix[i]` is in `[lower, upper]` means `prefix[j] - upper <= prefix[i] <= prefix[j] - lower`. During merge sort, the left half's values are sorted, so we can use two pointers to count valid pairs in O(n) per merge.

```cpp
long long mergeCount(vector<long long>& prefix, int left, int right,
                     int lower, int upper) {
    if (right - left <= 1) return 0;
    int mid = left + (right - left) / 2;
    long long count = mergeCount(prefix, left, mid, lower, upper)
                    + mergeCount(prefix, mid, right, lower, upper);

    int j = mid, k = mid;
    for (int i = left; i < mid; i++) {
        while (j < right && prefix[j] - prefix[i] < lower) j++;
        while (k < right && prefix[k] - prefix[i] <= upper) k++;
        count += k - j;
    }

    inplace_merge(prefix.begin() + left,
                       prefix.begin() + mid,
                       prefix.begin() + right);
    return count;
}

int countRangeSum(vector<int>& nums, int lower, int upper) {
    int n = nums.size();
    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i+1] = prefix[i] + nums[i];
    return (int)mergeCount(prefix, 0, n + 1, lower, upper);
}
// Time: O(n log n), Space: O(n)
```

---

## 8. XOR-Based Tricks

**Why tricky**: XOR has magical properties — self-inverse, commutative, associative.

```cpp
// Find single non-duplicate in array where every other appears twice
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int x : nums) result ^= x;  // duplicates cancel: a^a=0
    return result;
}

// Find two non-duplicates (LC 260)
vector<int> singleNumberIII(vector<int>& nums) {
    int xorAll = 0;
    for (int x : nums) xorAll ^= x;
    // xorAll = a^b, find any set bit (they differ in this bit)
    int diffBit = xorAll & (-xorAll);  // isolate lowest set bit
    int a = 0;
    for (int x : nums)
        if (x & diffBit) a ^= x;  // one group XORs to 'a', other to 'b'
    return {a, a ^ xorAll};
}
```

---

## 9. Minimum Window with All Characters (LC 76) — Extension to General "Minimum Window" Template

**Why important**: The minimum window substring problem is the hardest sliding window problem and generalizes to many OA variants. The template:

```cpp
// General minimum window template
string minWindow(string s, string t) {
    unordered_map<char,int> need, have_map;
    for (char c : t) need[c]++;
    int have = 0, required = need.size();
    int left = 0, minLen = INT_MAX, minStart = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];
        have_map[c]++;
        if (need.count(c) && have_map[c] == need[c]) have++;

        while (have == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minStart = left;
            }
            char lc = s[left++];
            have_map[lc]--;
            if (need.count(lc) && have_map[lc] < need[lc]) have--;
        }
    }
    return minLen == INT_MAX ? "" : s.substr(minStart, minLen);
}
// Time: O(|s| + |t|), Space: O(|t|)
```

---

## 10. Anti-Hash Attack Pattern (Codeforces OA Survival)

**Why tricky**: Adversarial tests on Codeforces deliberately create inputs that cause `unordered_map` to degrade to O(n) per operation. Forgetting this in an OA costs TLE on a correct solution.

Always use custom hash (shown in [[01_Arrays_and_Hashing_Hashing\|Hashing notes]]) when using `unordered_map` or `unordered_set` in competitive settings.

```cpp
// Alternatively use policy-based tree (GNU extension) for O(log n) guaranteed:
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
// gp_hash_table<int,int> is faster than unordered_map and less hackable
gp_hash_table<int, int> safe_table;
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Find the Duplicate Number | 🟡 Medium | [LC 287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 2 | Subarrays with K Different Integers | 🔴 Hard | [LC 992](https://leetcode.com/problems/subarrays-with-k-different-integers/) |
| 3 | Longest Subarray with Max-Min <= Limit | 🔴 Hard | [LC 1438](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/) |
| 4 | Count of Range Sum | 🔴 Hard | [LC 327](https://leetcode.com/problems/count-of-range-sum/) |
| 5 | Minimum Operations to Make Array Continuous | 🔴 Hard | [LC 2009](https://leetcode.com/problems/minimum-number-of-operations-to-make-array-continuous/) |
| 6 | Minimum Window Substring | 🔴 Hard | [LC 76](https://leetcode.com/problems/minimum-window-substring/) |
| 7 | Single Number III (two missing) | 🟡 Medium | [LC 260](https://leetcode.com/problems/single-number-iii/) |
| 8 | Maximum Sum of Two Non-Overlapping Subarrays | 🟡 Medium | [LC 1031](https://leetcode.com/problems/maximum-sum-of-two-non-overlapping-subarrays/) |

---

*← [[01_Arrays_and_Hashing_Problems_and_Exercises\|Problems]] · [[../02_Two_Pointers/02_Two_Pointers_Index\|Two Pointers →]]*

---
tags: [dsa, arrays, kadane, dutch-national-flag, boyer-moore, sliding-window]
links: ["[[01_Arrays_and_Hashing_Index]]", "[[01_Arrays_and_Hashing_Hashing]]", "[[01_Arrays_and_Hashing_Problems_and_Exercises]]"]
---

# Key Array Algorithms

*← [[01_Arrays_and_Hashing_Hashing\|Hashing]] · [[01_Arrays_and_Hashing_Problems_and_Exercises\|Problems →]]*

---

## Pattern 1: Kadane's Algorithm (Maximum Subarray Sum)

**Problem**: Find the contiguous subarray with the largest sum.

**Why the naive approach fails**: O(n²) or O(n³) — check all pairs (l, r).

**Key insight**: At each position, you have a choice: extend the previous subarray, or start fresh. If the running sum becomes negative, starting fresh is always better — a negative prefix can only drag down future sums.

```
arr = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
Running: start at -2
  At 1:  max(-2+1, 1) = 1    → fresh start (dropped the -2)
  At -3: max(1-3, -3) = -2   → extend
  At 4:  max(-2+4, 4) = 4    → fresh start (dropped the -2)
  At -1: max(4-1, -1) = 3    → extend
  At 2:  max(3+2, 2) = 5     → extend
  At 1:  max(5+1, 1) = 6     → extend  ← MAX
  ...
Answer: 6 (subarray [4,-1,2,1])
```

```cpp
#include <vector>
#include <algorithm>

// Version 1: Return just the max sum
int maxSubArray(vector<int>& nums) {
    int currentSum = nums[0];
    int maxSum = nums[0];

    for (int i = 1; i < (int)nums.size(); i++) {
        // Either extend the existing subarray or start a new one from here
        currentSum = max(nums[i], currentSum + nums[i]);
        maxSum = max(maxSum, currentSum);
    }
    return maxSum;
}
// Time: O(n), Space: O(1)

// Version 2: Return the actual subarray indices (interview follow-up)
pair<int,int> maxSubArrayIndices(vector<int>& nums) {
    int currentSum = nums[0];
    int maxSum = nums[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < (int)nums.size(); i++) {
        if (nums[i] > currentSum + nums[i]) {
            currentSum = nums[i];
            tempStart = i;        // starting fresh from here
        } else {
            currentSum += nums[i];
        }
        if (currentSum > maxSum) {
            maxSum = currentSum;
            start = tempStart;
            end = i;
        }
    }
    return {start, end};  // inclusive indices
}
```

### Kadane's Variation: Maximum Product Subarray

**Why it's different**: Negative × Negative = Positive. So we must track both max and min at every position.

```cpp
int maxProduct(vector<int>& nums) {
    int curMax = nums[0], curMin = nums[0], result = nums[0];

    for (int i = 1; i < (int)nums.size(); i++) {
        // When multiplied by a negative, max becomes min and vice versa
        if (nums[i] < 0) swap(curMax, curMin);
        
        curMax = max(nums[i], curMax * nums[i]);
        curMin = min(nums[i], curMin * nums[i]);
        result = max(result, curMax);
    }
    return result;
}
// Time: O(n), Space: O(1)
```

---

## Pattern 2: Dutch National Flag (Three-Way Partition)

**Problem**: Sort an array containing only 0s, 1s, and 2s — in O(n) time and O(1) space. (LC 75 — Sort Colors)

**Why standard sort is wasteful**: std::sort is O(n log n) and uses comparisons. We know only 3 distinct values exist.

**Key insight**: Maintain three regions using two pointers:
- `[0 .. low-1]` → all 0s
- `[low .. mid-1]` → all 1s
- `[mid .. high]` → unknown (currently processing)
- `[high+1 .. n-1]` → all 2s

```
Initial: [2, 0, 2, 1, 1, 0]
          low=0, mid=0, high=5

Step 1: arr[mid]=2 → swap with high, high--
        [0, 0, 2, 1, 1, 2]  mid stays (new arr[mid] unprocessed)
         low=0, mid=0, high=4

Step 2: arr[mid]=0 → swap with low, low++, mid++
        [0, 0, 2, 1, 1, 2]  (already 0, swap is no-op)
         low=1, mid=1, high=4

Step 3: arr[mid]=0 → swap with low, low++, mid++
        [0, 0, 2, 1, 1, 2]
         low=2, mid=2, high=4

Step 4: arr[mid]=2 → swap with high, high--
        [0, 0, 1, 1, 2, 2]
         low=2, mid=2, high=3

Step 5: arr[mid]=1 → mid++
         low=2, mid=3, high=3

Step 6: arr[mid]=1 → mid++
         low=2, mid=4, high=3  → mid > high, DONE
```

```cpp
void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = (int)nums.size() - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low], nums[mid]);
            low++;
            mid++;  // element at low was already processed (it was a 1)
        } else if (nums[mid] == 1) {
            mid++;  // 1 is already in the right region
        } else {    // nums[mid] == 2
            swap(nums[mid], nums[high]);
            high--;
            // DO NOT increment mid — the swapped-in element is unprocessed
        }
    }
}
// Time: O(n), Space: O(1) — single pass!
```

---

## Pattern 3: Boyer-Moore Voting Algorithm (Majority Element)

**Problem**: Find the element that appears more than n/2 times. Guaranteed to exist.

**Why hashing is overkill**: We'd need O(n) space. Boyer-Moore does it in O(1) space.

**Key insight**: If you cancel out every pair of distinct elements, the majority element survives. Think of it as a vote — when your candidate meets an enemy, both lose one vote. The true majority always has enough votes to survive.

```
nums = [2, 2, 1, 1, 1, 2, 2]

candidate=2, votes=1
candidate=2, votes=2  (see 2, votes go up)
candidate=2, votes=1  (see 1, votes go down)
candidate=2, votes=0  (see 1, votes go down — candidate may change)
candidate=1, votes=1  (see 1, new candidate)
candidate=1, votes=0  (see 2, votes go down)
candidate=2, votes=1  (see 2, new candidate)

Final candidate = 2 ✓
```

```cpp
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], votes = 1;

    for (int i = 1; i < (int)nums.size(); i++) {
        if (votes == 0) {
            candidate = nums[i];  // start fresh with new candidate
            votes = 1;
        } else if (nums[i] == candidate) {
            votes++;
        } else {
            votes--;
        }
    }
    return candidate;
    // Note: problem guarantees majority exists. Otherwise you'd need a second pass to verify.
}
// Time: O(n), Space: O(1)
```

### Variation: Majority Element II (> n/3 times — LC 229)

**Key insight**: At most 2 elements can appear more than n/3 times. Track 2 candidates.

```cpp
vector<int> majorityElementII(vector<int>& nums) {
    int cand1 = INT_MIN, cand2 = INT_MIN, votes1 = 0, votes2 = 0;

    for (int x : nums) {
        if (x == cand1) {
            votes1++;
        } else if (x == cand2) {
            votes2++;
        } else if (votes1 == 0) {
            cand1 = x; votes1 = 1;
        } else if (votes2 == 0) {
            cand2 = x; votes2 = 1;
        } else {
            votes1--;
            votes2--;
        }
    }

    // Verification pass — candidates may not actually appear > n/3 times
    votes1 = 0; votes2 = 0;
    for (int x : nums) {
        if (x == cand1) votes1++;
        else if (x == cand2) votes2++;
    }

    vector<int> result;
    int n = nums.size();
    if (votes1 > n / 3) result.push_back(cand1);
    if (votes2 > n / 3) result.push_back(cand2);
    return result;
}
```

---

## Pattern 4: Two Pointer — Trapping Rainwater

This is an iconic problem that requires thinking about **precomputed maximums** or using two pointers.

```
heights = [0,1,0,2,1,0,1,3,2,1,2,1]
Water trapped:       1     1  2  1  1 = 6 units
```

**Why**: Water at position `i` = `min(maxLeft[i], maxRight[i]) - height[i]`.

```cpp
// Approach 1: Prefix max arrays — O(n) time, O(n) space
int trap(vector<int>& height) {
    int n = height.size();
    vector<int> maxLeft(n), maxRight(n);

    maxLeft[0] = height[0];
    for (int i = 1; i < n; i++)
        maxLeft[i] = max(maxLeft[i-1], height[i]);

    maxRight[n-1] = height[n-1];
    for (int i = n-2; i >= 0; i--)
        maxRight[i] = max(maxRight[i+1], height[i]);

    int water = 0;
    for (int i = 0; i < n; i++)
        water += min(maxLeft[i], maxRight[i]) - height[i];

    return water;
}

// Approach 2: Two pointers — O(n) time, O(1) space
int trapOptimal(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1;
    int maxL = 0, maxR = 0, water = 0;

    while (left < right) {
        if (height[left] < height[right]) {
            // The limiting factor is the left side
            if (height[left] >= maxL) maxL = height[left];
            else water += maxL - height[left];
            left++;
        } else {
            if (height[right] >= maxR) maxR = height[right];
            else water += maxR - height[right];
            right--;
        }
    }
    return water;
}
```

---

## Examples

### Example 1 — LC 53: Maximum Subarray (Medium)
Already shown in Pattern 1.

### Example 2 — LC 152: Maximum Product Subarray (Medium)
Already shown in Kadane's variation.

### Example 3 — LC 75: Sort Colors (Medium)
Dutch National Flag — already shown in Pattern 2.

### Example 4 — LC 169: Majority Element (Easy)
Boyer-Moore — already shown in Pattern 3.

### Example 5 — LC 42: Trapping Rain Water (Hard)
Already shown in Pattern 4.

---

## Exercises

| # | Problem | Platform | Difficulty | Pattern |
|---|---------|----------|------------|---------|
| 1 | [53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | LeetCode | 🟡 Medium | Kadane's |
| 2 | [152. Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | LeetCode | 🟡 Medium | Kadane's variant |
| 3 | [75. Sort Colors](https://leetcode.com/problems/sort-colors/) | LeetCode | 🟡 Medium | Dutch National Flag |
| 4 | [169. Majority Element](https://leetcode.com/problems/majority-element/) | LeetCode | 🟢 Easy | Boyer-Moore |
| 5 | [229. Majority Element II](https://leetcode.com/problems/majority-element-ii/) | LeetCode | 🟡 Medium | Boyer-Moore extended |
| 6 | [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | LeetCode | 🔴 Hard | Prefix max / Two pointer |
| 7 | [918. Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | LeetCode | 🟡 Medium | Kadane's on circular |
| 8 | [Maximum of all Subarrays of Size K](https://www.geeksforgeeks.org/sliding-window-maximum-maximum-of-all-subarrays-of-size-k/) | GFG | 🟡 Medium | Deque (monotonic) |

---

*← [[01_Arrays_and_Hashing_Hashing\|Hashing]] · [[01_Arrays_and_Hashing_Problems_and_Exercises\|Problems →]]*

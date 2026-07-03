---
tags: [dsa, dp, strings, lis, binary-search]
links: ["[[04_DP_on_Strings_Index]]", "[[DP_on_Strings_LCS]]", "[[DP_on_Strings_Edit_Distance]]", "[[../../05_Binary_Search/05_Binary_Search_Index]]"]
---

# DP on Strings -- Longest Increasing Subsequence (LIS)

*<- [[DP_on_Strings_LCS|Longest Common Subsequence]] · [[DP_on_Strings_Edit_Distance|Edit Distance ->]]*

---

## The Problem
Given an integer array `nums`, return the length of the longest strictly increasing subsequence.

---

## 1. Standard Dynamic Programming ($O(n^2)$)

### State Formulation
Let `dp[i]` be the length of the LIS ending exactly at index `i`.
- To find `dp[i]`, we look at all previous indices `j` (from `0` to `i-1`). If `nums[j] < nums[i]`, we can extend the LIS ending at `j`.
- **Recurrence Relation**: `dp[i] = 1 + max(dp[j])` for all $j < i$ where $\text{nums}[j] < \text{nums}[i]$.
- **Base case**: `dp[i] = 1` (every single element is an increasing subsequence of length 1).

```cpp
#include <vector>
#include <algorithm>

int lisNSquared(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;
    
    vector<int> dp(n, 1);
    int maxLis = 1;

    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = max(dp[i], 1 + dp[j]);
            }
        }
        maxLis = max(maxLis, dp[i]);
    }
    return maxLis;
}
// Time Complexity: O(n^2)
// Space Complexity: O(n)
```

---

## 2. Optimal Binary Search Method ($O(n \log n)$)

**Why**: An $O(n^2)$ algorithm is too slow for arrays larger than $10^4$ elements. We can optimize it to $O(n \log n)$ using a greedy technique known as **Patience Sorting**.

### Intuition: Patience Sorting (Card Pile Placement)
Imagine playing a card game where you place cards in piles from left to right:
- You must place a new card on the **leftmost pile** whose top card is **greater than or equal** to the new card.
- If no such pile exists, create a **new pile** on the right.
- The **number of piles** at the end is the length of the LIS!

```
Array: [10, 9, 2, 5, 3, 7, 101, 18]

Card 10  : Pile 1 = [10]
Card 9   : 9 <= 10 -> Place on Pile 1. Piles: [9]
Card 2   : 2 <= 9  -> Place on Pile 1. Piles: [2]
Card 5   : 5 > 2   -> Create Pile 2.   Piles: [2], [5]
Card 3   : 3 <= 5  -> Place on Pile 2. Piles: [2], [3]
Card 7   : 7 > 3   -> Create Pile 3.   Piles: [2], [3], [7]
Card 101 : 101 > 7 -> Create Pile 4.   Piles: [2], [3], [7], [101]
Card 18  : 18 <= 101-> Place on Pile 4. Piles: [2], [3], [7], [18]

Final Piles count: 4 (LIS length is 4: [2, 3, 7, 18] or [2, 5, 7, 101] etc.)
```

### Binary Search Implementation (using `lower_bound`)
Since the top cards of our piles are always sorted ascending, we can use binary search (`lower_bound` which finds first element $\ge$ target) to find the correct pile index in $O(\log n)$ time.

```cpp
#include <vector>
#include <algorithm>

int lengthOfLIS(vector<int>& nums) {
    vector<int> piles; // stores the top card of each pile

    for (int x : nums) {
        // Binary search for first element >= x
        auto it = lower_bound(piles.begin(), piles.end(), x);
        
        if (it == piles.end()) {
            piles.push_back(x); // create new pile
        } else {
            *it = x; // replace top card of existing pile
        }
    }
    return piles.size(); // number of piles equals LIS length
}
// Time Complexity: O(n log n)
// Space Complexity: O(n)
```

> [!WARNING]
> The `piles` array does NOT necessarily represent the actual elements of the LIS itself. Its elements can become scrambled during overwrites. It only guarantees that its **size** is equal to the LIS length. To reconstruct the actual LIS, you must maintain parent pointers.

---

*<- [[DP_on_Strings_LCS|Longest Common Subsequence]] · [[DP_on_Strings_Edit_Distance|Edit Distance ->]]*

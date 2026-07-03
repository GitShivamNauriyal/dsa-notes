---
tags: [dsa, two-pointers, tricky, hard, interview]
links: ["[[02_Two_Pointers_Index]]", "[[02_Two_Pointers_Problems_and_Exercises]]", "[[../03_Sliding_Window/03_Sliding_Window_Index]]"]
---

# Two Pointers -- Tricky & Higher-Order

*<- [[02_Two_Pointers_Problems_and_Exercises|Problems]] · [[../03_Sliding_Window/03_Sliding_Window_Index|Sliding Window ->]]*

---

## 1. Two Pointers on Two Separate Arrays

**Standard two pointer**: one array. The harder variant uses two sorted arrays simultaneously.

### Minimum Absolute Difference Between Two Sorted Arrays (Custom / OA)

```cpp
// Find pair (a from A, b from B) minimizing |a - b|
// Both arrays sorted. Two pointers, one per array.
int minAbsDiff(vector<int>& A, vector<int>& B) {
    int i = 0, j = 0, minDiff = INT_MAX;
    while (i < (int)A.size() && j < (int)B.size()) {
        minDiff = min(minDiff, abs(A[i] - B[j]));
        // Advance the pointer pointing to the smaller value
        // to try to close the gap
        if (A[i] < B[j]) i++;
        else j++;
    }
    return minDiff;
}
// Time: O(n + m), Space: O(1)
```

---

## 2. Palindrome After Deleting At Most One Character (LC 680)

**Why tricky**: Standard palindrome check is easy. When you hit a mismatch, you must try deleting either the left or right character and check if the rest is a palindrome. Two choices, but only one delete allowed total.

```cpp
bool checkPalindrome(const string& s, int lo, int hi) {
    while (lo < hi) {
        if (s[lo] != s[hi]) return false;
        lo++; hi--;
    }
    return true;
}

bool validPalindrome(string s) {
    int left = 0, right = (int)s.size() - 1;
    while (left < right) {
        if (s[left] != s[right]) {
            // Try skipping left OR right character
            return checkPalindrome(s, left + 1, right)
                || checkPalindrome(s, left, right - 1);
        }
        left++; right--;
    }
    return true;
}
// Time: O(n), Space: O(1)
```

---

## 3. Count Pairs in Array with Sum < Target (Variant of 2Sum)

**Why tricky**: Counting all pairs (not finding one) requires thinking about how many pairs the current pointer state implies.

```cpp
// Sorted array. Count pairs (i,j) where i<j and nums[i]+nums[j] < target.
int countPairs(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    int left = 0, right = (int)nums.size() - 1;
    int count = 0;
    while (left < right) {
        if (nums[left] + nums[right] < target) {
            // nums[left] pairs with ALL elements from left+1 to right
            // because array is sorted and all those are <= nums[right]
            count += right - left;
            left++;
        } else {
            right--;
        }
    }
    return count;
}
// Time: O(n log n), Space: O(1)
// This insight (count += right - left) is non-obvious and commonly tested.
```

---

## 4. Minimum Window Containing All Elements of Array B (Hard OA Variant)

**Why tricky**: Find shortest subarray of A that contains all elements of B (not a pattern, just element set). Classic sliding window but the "all elements" check must be efficient.

```cpp
#include <unordered_map>
int minWindowArray(vector<int>& A, vector<int>& B) {
    unordered_map<int,int> need, have;
    for (int x : B) need[x]++;
    int formed = 0, required = need.size();
    int left = 0, minLen = INT_MAX;

    for (int right = 0; right < (int)A.size(); right++) {
        have[A[right]]++;
        if (need.count(A[right]) && have[A[right]] == need[A[right]])
            formed++;
        while (formed == required) {
            minLen = min(minLen, right - left + 1);
            have[A[left]]--;
            if (need.count(A[left]) && have[A[left]] < need[A[left]])
                formed--;
            left++;
        }
    }
    return minLen == INT_MAX ? -1 : minLen;
}
```

---

## 5. Partition Array into Two Halves with Equal Sum (Quick Select variant / Hard)

**Why tricky**: Related to the partition idea. Given an array, can you partition it into two contiguous parts with equal sum? Prefix sum tells you when to cut, but finding all valid cuts requires thinking about prefix sum uniqueness.

```cpp
// Find all split indices where prefix[i] == totalSum - prefix[i]
// i.e., prefix[i] = totalSum / 2
vector<int> equalSumSplits(vector<int>& nums) {
    long long total = 0;
    for (int x : nums) total += x;
    vector<int> splits;
    long long prefix = 0;
    for (int i = 0; i < (int)nums.size() - 1; i++) {
        prefix += nums[i];
        if (prefix * 2 == total) splits.push_back(i);
    }
    return splits;
}
```

---

## 6. Trapping Rain Water -- Elevation Map in 2D (LC 407, Hard)

**Why tricky**: 3D version of trapping rainwater. Instead of two pointers, use a min-heap (priority queue) and BFS outward from the boundary. Water at each interior cell is bounded by the minimum wall seen so far on the path to the boundary.

```cpp
#include <queue>
int trapRainWater(vector<vector<int>>& heightMap) {
    if (heightMap.empty()) return 0;
    int rows = heightMap.size(), cols = heightMap[0].size();
    // min-heap: {height, row, col}
    using T = tuple<int,int,int>;
    priority_queue<T, vector<T>, greater<T>> pq;
    vector<vector<bool>> visited(rows, vector<bool>(cols, false));

    // Push all boundary cells
    for (int r = 0; r < rows; r++) {
        for (int c : {0, cols-1}) {
            pq.push({heightMap[r][c], r, c});
            visited[r][c] = true;
        }
    }
    for (int c = 0; c < cols; c++) {
        for (int r : {0, rows-1}) {
            if (!visited[r][c]) {
                pq.push({heightMap[r][c], r, c});
                visited[r][c] = true;
            }
        }
    }

    int water = 0;
    int dirs[4][2] = {{0,1},{0,-1},{1,0},{-1,0}};
    while (!pq.empty()) {
        auto [h, r, c] = pq.top(); pq.pop();
        for (auto& d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || visited[nr][nc])
                continue;
            visited[nr][nc] = true;
            // Water trapped = how much current cell is below the boundary minimum
            water += max(0, h - heightMap[nr][nc]);
            // Push with max height (the "wall" height for neighbors)
            pq.push({max(h, heightMap[nr][nc]), nr, nc});
        }
    }
    return water;
}
// Time: O(n*m*log(n*m)), Space: O(n*m)
```

---

## 7. Three Pointers -- Dutch National Flag Generalization

**Beyond 3 values**: Partition array into k groups. With k=4 you need 3 boundaries. Generalize by thinking of each boundary as a pointer.

```cpp
// Partition into: < lo | lo..hi | > hi
// Uses three pointers: left, mid, right
void threeWayPartition(vector<int>& arr, int lo, int hi) {
    int left = 0, mid = 0, right = (int)arr.size() - 1;
    while (mid <= right) {
        if (arr[mid] < lo) {
            swap(arr[left++], arr[mid++]);
        } else if (arr[mid] > hi) {
            swap(arr[mid], arr[right--]);
            // do not increment mid -- newly swapped element unprocessed
        } else {
            mid++;
        }
    }
}
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Valid Palindrome II | Medium | [LC 680](https://leetcode.com/problems/valid-palindrome-ii/) |
| 2 | 3Sum with Multiplicity | Medium | [LC 923](https://leetcode.com/problems/3sum-with-multiplicity/) |
| 3 | Trapping Rain Water II | Hard | [LC 407](https://leetcode.com/problems/trapping-rain-water-ii/) |
| 4 | Count Pairs Less Than Target | Medium | [LC 2824](https://leetcode.com/problems/count-pairs-whose-sum-is-less-than-target/) |
| 5 | Minimum Difference Between Highest and Lowest of K Scores | Easy | [LC 1984](https://leetcode.com/problems/minimum-difference-between-highest-and-lowest-of-k-scores/) |
| 6 | Number of Subsequences That Satisfy Condition | Medium | [LC 1498](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition/) |
| 7 | Shortest Unsorted Continuous Subarray | Medium | [LC 581](https://leetcode.com/problems/shortest-unsorted-continuous-subarray/) |

---

*<- [[02_Two_Pointers_Problems_and_Exercises|Problems]] · [[../03_Sliding_Window/03_Sliding_Window_Index|Sliding Window ->]]*

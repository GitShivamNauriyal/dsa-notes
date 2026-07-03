---
tags: [dsa, sliding-window, patterns]
links: ["[[03_Sliding_Window_Index]]", "[[03_Sliding_Window_Problems_and_Exercises]]"]
---

# Sliding Window -- Patterns

*<- [[03_Sliding_Window_Index|Index]] · [[03_Sliding_Window_Problems_and_Exercises|Problems ->]]*

---

## What is Sliding Window?

A sliding window is a contiguous subarray (or substring) that you move across the input. Instead of recomputing the window property from scratch each time (O(n²) total), you **add one new element on the right and remove one old element on the left** -- making each step O(1) or O(log n).

Two types:
- **Fixed-size window**: window size k is given. Slide by adding right and removing left together.
- **Variable-size window**: window grows and shrinks based on a condition. Grow by adding right; shrink by removing left until condition is satisfied.

---

## Pattern 1: Fixed-Size Window

**Setup**: window size `k` is fixed. Slide it across the array.  
**Template**:

```cpp
// Step 1: process first window [0, k-1]
// Step 2: for i from k to n-1:
//     add nums[i] to window
//     remove nums[i-k] from window
//     update answer

// Generic fixed window maximum sum
int maxSumFixedWindow(std::vector<int>& nums, int k) {
    int n = nums.size();
    if (n < k) return -1;

    // Build first window
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];
    int maxSum = windowSum;

    // Slide: add new right element, remove old left element
    for (int i = k; i < n; i++) {
        windowSum += nums[i];       // add incoming
        windowSum -= nums[i - k];   // remove outgoing
        maxSum = std::max(maxSum, windowSum);
    }
    return maxSum;
}
// Time: O(n), Space: O(1)
```

### Example: Maximum Average Subarray I (LC 643)

```cpp
double findMaxAverage(std::vector<int>& nums, int k) {
    double windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];
    double maxSum = windowSum;
    for (int i = k; i < (int)nums.size(); i++) {
        windowSum += nums[i] - nums[i - k];
        maxSum = std::max(maxSum, windowSum);
    }
    return maxSum / k;
}
```

### Example: Contains Duplicate Within Distance k (LC 219)

```cpp
// Sliding window of size k -- use a set to track elements in window
bool containsNearbyDuplicate(std::vector<int>& nums, int k) {
    std::unordered_set<int> window;
    for (int i = 0; i < (int)nums.size(); i++) {
        if (window.count(nums[i])) return true;
        window.insert(nums[i]);
        if ((int)window.size() > k) {
            window.erase(nums[i - k]);  // shrink window from left
        }
    }
    return false;
}
```

---

## Pattern 2: Variable-Size Window (Shrink When Invalid)

**Setup**: Expand right freely. When window violates a condition, shrink from left until valid again.  
**When to use**: Longest/shortest subarray with some property.

**Template**:

```cpp
int left = 0;
// window state variables (sum, count, freq map, etc.)

for (int right = 0; right < n; right++) {
    // 1. Add nums[right] to window state

    // 2. Shrink from left while window is INVALID
    while (/* window is invalid */) {
        // Remove nums[left] from window state
        left++;
    }

    // 3. Window is now valid -- update answer
    // For LONGEST: ans = max(ans, right - left + 1)
    // For SHORTEST: ans = min(ans, right - left + 1)
}
```

### Sub-pattern 2a: Longest Subarray with Sum <= Target

```cpp
// LC 209 variant -- longest subarray with sum <= target (all positive)
// Note: LC 209 asks for MINIMUM length with sum >= target
int longestSubarraySum(std::vector<int>& nums, int target) {
    int left = 0, sum = 0, ans = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        sum += nums[right];
        while (sum > target) {
            sum -= nums[left++];
        }
        ans = std::max(ans, right - left + 1);
    }
    return ans;
}
```

### Sub-pattern 2b: Minimum Length Subarray with Sum >= Target

```cpp
// LC 209 -- Minimum Size Subarray Sum
int minSubArrayLen(int target, std::vector<int>& nums) {
    int left = 0, sum = 0, ans = INT_MAX;
    for (int right = 0; right < (int)nums.size(); right++) {
        sum += nums[right];
        // Shrink as long as window is still valid (sum >= target)
        while (sum >= target) {
            ans = std::min(ans, right - left + 1);
            sum -= nums[left++];
        }
    }
    return ans == INT_MAX ? 0 : ans;
}
// Time: O(n) -- each element enters and exits window exactly once
```

---

## Pattern 3: Variable Window with Character/Element Constraint

**Setup**: Window must satisfy a constraint on distinct elements or character frequencies.

### Sub-pattern 3a: Longest Substring Without Repeating Characters

```cpp
// LC 3
int lengthOfLongestSubstring(std::string s) {
    std::unordered_map<char, int> lastSeen;  // char -> last index seen
    int left = 0, ans = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];
        if (lastSeen.count(c) && lastSeen[c] >= left) {
            // Character seen inside current window -- jump left past it
            left = lastSeen[c] + 1;
        }
        lastSeen[c] = right;
        ans = std::max(ans, right - left + 1);
    }
    return ans;
}
// Time: O(n), Space: O(min(n, alphabet_size))
```

### Sub-pattern 3b: Longest Substring with At Most K Distinct Characters

```cpp
// LC 340
int lengthOfLongestSubstringKDistinct(std::string s, int k) {
    std::unordered_map<char, int> freq;
    int left = 0, ans = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right]]++;

        // Window has more than k distinct chars -- shrink
        while ((int)freq.size() > k) {
            freq[s[left]]--;
            if (freq[s[left]] == 0) freq.erase(s[left]);
            left++;
        }
        ans = std::max(ans, right - left + 1);
    }
    return ans;
}
```

### Sub-pattern 3c: Longest Repeating Character Replacement

```cpp
// LC 424 -- Replace at most k characters to make longest uniform substring
// Key insight: window is valid if (windowLen - maxFreq) <= k
// maxFreq = count of most frequent character in window
// Only characters OTHER than the most frequent need replacing

int characterReplacement(std::string s, int k) {
    std::vector<int> freq(26, 0);
    int left = 0, maxFreq = 0, ans = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right] - 'A']++;
        maxFreq = std::max(maxFreq, freq[s[right] - 'A']);

        // Replacements needed = windowLen - maxFreq
        int windowLen = right - left + 1;
        if (windowLen - maxFreq > k) {
            // Window invalid -- shrink left by 1
            freq[s[left] - 'A']--;
            left++;
            // Note: we do NOT update maxFreq down. This is intentional.
            // We only care about windows >= current best size.
            // maxFreq can only increase going forward (optimistic tracking).
        }
        ans = std::max(ans, right - left + 1);
    }
    return ans;
}
// Time: O(n), Space: O(26) = O(1)
// Why not update maxFreq when shrinking: we want the answer to be the
// maximum window ever seen. If current window is invalid but same size
// as previous best, we just slide it (left++ and right++) maintaining size.
```

---

## Pattern 4: Two-Condition Window (Track Two Things)

### Permutation in String (LC 567)

```cpp
// Check if any permutation of pattern p exists as substring in s
bool checkInclusion(std::string p, std::string s) {
    if (p.size() > s.size()) return false;
    std::vector<int> pFreq(26, 0), wFreq(26, 0);
    for (char c : p) pFreq[c - 'a']++;

    int k = p.size();
    // Build first window
    for (int i = 0; i < k; i++) wFreq[s[i] - 'a']++;
    if (pFreq == wFreq) return true;

    // Slide
    for (int i = k; i < (int)s.size(); i++) {
        wFreq[s[i] - 'a']++;
        wFreq[s[i - k] - 'a']--;
        if (pFreq == wFreq) return true;
    }
    return false;
}
// Time: O(26 * n) = O(n), Space: O(1)

// More efficient: track how many characters are "matched"
bool checkInclusionOptimal(std::string p, std::string s) {
    if (p.size() > s.size()) return false;
    std::vector<int> count(26, 0);
    for (char c : p) count[c - 'a']++;
    int k = p.size(), have = 0, need = 0;
    for (int x : count) if (x > 0) need++;  // distinct chars we need to satisfy

    for (int right = 0; right < (int)s.size(); right++) {
        int c = s[right] - 'a';
        count[c]--;
        if (count[c] == 0) have++;  // this character is now exactly satisfied

        if (right >= k) {
            int lc = s[right - k] - 'a';
            if (count[lc] == 0) have--;  // losing a satisfied character
            count[lc]++;
        }
        if (have == need) return true;
    }
    return false;
}
```

---

## Pattern 5: Monotonic Deque Window (Sliding Window Maximum)

**Why a deque**: Maintaining max/min of a sliding window efficiently. A regular variable can't track max when the current max leaves the window. A **monotonic deque** (elements stored in decreasing order of value for max) solves this.

```
nums = [1,3,-1,-3,5,3,6,7], k=3
Deque stores indices of potential maximums in decreasing order of value.

right=0: deque=[0(val=1)]
right=1: 3>1, pop 0, push 1. deque=[1(val=3)]
right=2: -1<3, push 2. deque=[1(val=3),2(val=-1)]
window complete: max=nums[deque.front()]=3
right=3: -3<-1, push 3. deque=[1,2,3]. Remove front if out of window: 1>=0 OK. max=3
right=4: 5>-3,-1,3 -- pop all, push 4. deque=[4(val=5)]. Remove if out: 4>=2 OK. max=5
...
```

```cpp
// LC 239 -- Sliding Window Maximum
std::vector<int> maxSlidingWindow(std::vector<int>& nums, int k) {
    std::deque<int> dq;  // stores indices, front = index of current max
    std::vector<int> result;

    for (int right = 0; right < (int)nums.size(); right++) {
        // Remove elements from back that are smaller than current
        // (they can never be the max while current element is in window)
        while (!dq.empty() && nums[dq.back()] <= nums[right]) {
            dq.pop_back();
        }
        dq.push_back(right);

        // Remove front if it's outside the window
        if (dq.front() < right - k + 1) {
            dq.pop_front();
        }

        // Window is complete after first k elements
        if (right >= k - 1) {
            result.push_back(nums[dq.front()]);
        }
    }
    return result;
}
// Time: O(n) -- each element pushed and popped at most once
// Space: O(k)
```

---

## Examples

### Example 1 -- LC 643: Maximum Average Subarray I (Easy)
Shown in Pattern 1.

### Example 2 -- LC 3: Longest Substring Without Repeating (Medium)
Shown in Pattern 3a.

### Example 3 -- LC 76: Minimum Window Substring (Hard)

```cpp
// Already shown in Arrays Tricky. Template recap:
// need map: char -> count needed
// have count: how many distinct chars are fully satisfied
// Expand right, shrink left when all satisfied
std::string minWindow(std::string s, std::string t) {
    std::unordered_map<char,int> need, have;
    for (char c : t) need[c]++;
    int formed = 0, required = need.size();
    int left = 0, minLen = INT_MAX, minStart = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];
        have[c]++;
        if (need.count(c) && have[c] == need[c]) formed++;
        while (formed == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minStart = left;
            }
            char lc = s[left++];
            have[lc]--;
            if (need.count(lc) && have[lc] < need[lc]) formed--;
        }
    }
    return minLen == INT_MAX ? "" : s.substr(minStart, minLen);
}
```

---

*<- [[03_Sliding_Window_Index|Index]] · [[03_Sliding_Window_Problems_and_Exercises|Problems ->]]*

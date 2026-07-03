---
tags: [dsa, stack, tricky, hard, interview]
links: ["[[04_Stack_Index]]", "[[04_Stack_Problems_and_Exercises]]", "[[../05_Binary_Search/05_Binary_Search_Index]]"]
---

# Stack -- Tricky & Higher-Order

*<- [[04_Stack_Problems_and_Exercises\|Problems]] · [[../05_Binary_Search/05_Binary_Search_Index\|Binary Search ->]]*

---

## 1. Monotonic Stack with Contribution Technique (Sum of Subarray Maximums)

**Why tricky**: You need to sum minimums (or maximums) across ALL subarrays without iterating O(n²). The contribution technique: for each element, count subarrays where it is the min/max, then multiply.

```cpp
// Sum of subarray maximums -- symmetric to minimums but flip comparisons
long long sumSubarrayMaxs(vector<int>& arr) {
    const long long MOD = 1e9 + 7;
    int n = arr.size();
    vector<long long> left(n), right(n);
    stack<int> st;

    // previous greater (strict) to left
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] <= arr[i]) st.pop();
        left[i] = st.empty() ? i + 1 : i - st.top();
        st.push(i);
    }
    while (!st.empty()) st.pop();

    // next greater or equal to right
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] < arr[i]) st.pop();
        right[i] = st.empty() ? n - i : st.top() - i;
        st.push(i);
    }

    long long ans = 0;
    for (int i = 0; i < n; i++)
        ans = (ans + (long long)arr[i] % MOD * left[i] % MOD * right[i]) % MOD;
    return ans;
}
```

---

## 2. Basic Calculator with Full Parentheses (LC 224)

**Why tricky**: Unlike Calculator II (only +, -, *, /), Calculator I has nested parentheses and no * or /. The trick is to push the current result and sign onto the stack when entering a parenthesis, then pop and restore when exiting.

```cpp
int calculate(string s) {
    stack<int> st;  // alternates: saved result, then sign
    int result = 0, num = 0, sign = 1;

    for (char c : s) {
        if (isdigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num; num = 0; sign = 1;
        } else if (c == '-') {
            result += sign * num; num = 0; sign = -1;
        } else if (c == '(') {
            // Save current state and start fresh inside parenthesis
            st.push(result);
            st.push(sign);
            result = 0; sign = 1;
        } else if (c == ')') {
            result += sign * num; num = 0;
            result *= st.top(); st.pop();   // multiply by sign before (
            result += st.top(); st.pop();   // add to result before (
        }
    }
    return result + sign * num;
}
```

---

## 3. Lexicographically Smallest Subsequence (LC 1081 / LC 316)

**Why tricky**: Remove duplicate characters, keeping each unique character exactly once, with the result being lexicographically smallest. This uses a monotonic stack with a "last occurrence" check.

```cpp
// LC 316 / 1081 -- Remove Duplicate Letters
string removeDuplicateLetters(string s) {
    vector<int> lastIdx(26, -1);
    vector<bool> inStack(26, false);

    for (int i = 0; i < (int)s.size(); i++) lastIdx[s[i] - 'a'] = i;

    stack<char> st;
    for (int i = 0; i < (int)s.size(); i++) {
        char c = s[i];
        if (inStack[c - 'a']) continue;

        // Pop larger characters if they appear later (we can include them again later)
        while (!st.empty() && st.top() > c && lastIdx[st.top() - 'a'] > i) {
            inStack[st.top() - 'a'] = false;
            st.pop();
        }
        st.push(c);
        inStack[c - 'a'] = true;
    }

    string result = "";
    while (!st.empty()) { result += st.top(); st.pop(); }
    reverse(result.begin(), result.end());
    return result;
}
// Time: O(n), Space: O(26) = O(1)
```

---

## 4. Number of People that Can See Each Other (LC 1944)

**Why tricky**: Person i can see person j (j > i) if there's no person taller than both between them. Use a monotonic decreasing stack counting how many people each person can see using the stack's structure.

```cpp
vector<int> canSeePersonsCount(vector<int>& heights) {
    int n = heights.size();
    vector<int> ans(n, 0);
    stack<int> st;  // monotonically decreasing

    for (int i = n - 1; i >= 0; i--) {
        // Count how many shorter people to the right person i can see
        while (!st.empty() && heights[st.top()] < heights[i]) {
            ans[i]++;
            st.pop();
        }
        // Can also see the first person >= heights[i] (the one that blocks)
        if (!st.empty()) ans[i]++;
        st.push(i);
    }
    return ans;
}
```

---

## 5. Maximum Score of a Good Subarray (LC 2818 variant / LC 1793)

**Why tricky**: Find max `min(nums[i..j]) * (j - i + 1)` where `i <= k <= j`. This must include index k. Use monotonic stack to find left and right boundaries, then greedily expand from k outward choosing the larger neighbor.

```cpp
int maximumScore(vector<int>& nums, int k) {
    int n = nums.size();
    int left = k, right = k;
    int minVal = nums[k], ans = nums[k];

    while (left > 0 || right < n - 1) {
        // Expand toward the larger neighbor to keep minimum as high as possible
        if (left == 0) right++;
        else if (right == n - 1) left--;
        else if (nums[left - 1] >= nums[right + 1]) left--;
        else right++;

        minVal = min(minVal, min(
            left > 0 ? INT_MAX : nums[left],  // simplified: just track
            nums[right]
        ));
        minVal = min({minVal, nums[left], nums[right]});
        ans = max(ans, minVal * (right - left + 1));
    }
    return ans;
}
```

---

## 6. Maximum Rectangle in Histogram -- Alternative: Divide and Conquer

**Why tricky**: Besides monotonic stack, there's a divide-and-conquer approach using sparse table (Range Minimum Query) for O(n log n). Knowing multiple approaches is valued in interviews.

```cpp
// Build sparse table for RMQ, then D&C
// Left as exercise -- RMQ is covered in Advanced Graphs/Trees
// Key idea: find min in range [l,r] in O(1), then:
// max_area = max(
//   heights[min_idx] * (r - l + 1),    // min bar spans full range
//   solve(l, min_idx - 1),              // left of min
//   solve(min_idx + 1, r)               // right of min
// )
// O(n log n) average, O(n²) worst (sorted input) without random pivot
```

---

## 7. Two Stacks Simulating a Queue with O(1) Amortized Push and Pop

**Why tricky**: Implementing a queue using two stacks, but ensuring O(1) amortized is the insight -- not O(n) worst case per operation.

```cpp
class MyQueue {
    stack<int> inbox, outbox;
    // inbox: where new elements go
    // outbox: reversed inbox for FIFO order
    // Transfer only when outbox is empty (amortized O(1))
public:
    void push(int x) { inbox.push(x); }

    int pop() {
        if (outbox.empty())
            while (!inbox.empty()) { outbox.push(inbox.top()); inbox.pop(); }
        int front = outbox.top(); outbox.pop();
        return front;
    }

    int peek() {
        if (outbox.empty())
            while (!inbox.empty()) { outbox.push(inbox.top()); inbox.pop(); }
        return outbox.top();
    }

    bool empty() { return inbox.empty() && outbox.empty(); }
};
// Each element transferred at most once: O(1) amortized push and pop
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Basic Calculator I | Hard | [LC 224](https://leetcode.com/problems/basic-calculator/) |
| 2 | Remove Duplicate Letters | Medium | [LC 316](https://leetcode.com/problems/remove-duplicate-letters/) |
| 3 | Number of Visible People in Queue | Hard | [LC 1944](https://leetcode.com/problems/number-of-visible-people-in-a-queue/) |
| 4 | Maximum Width Ramp | Medium | [LC 962](https://leetcode.com/problems/maximum-width-ramp/) |
| 5 | Sum of Subarray Ranges | Medium | [LC 2104](https://leetcode.com/problems/sum-of-subarray-ranges/) |
| 6 | Maximum Score of a Good Subarray | Hard | [LC 1793](https://leetcode.com/problems/maximum-score-of-a-good-subarray/) |
| 7 | Smallest Subsequence of Distinct Chars | Medium | [LC 1081](https://leetcode.com/problems/smallest-subsequence-of-distinct-characters/) |
| 8 | Find the Most Competitive Subsequence | Medium | [LC 1673](https://leetcode.com/problems/find-the-most-competitive-subsequence/) |

---

*<- [[04_Stack_Problems_and_Exercises\|Problems]] · [[../05_Binary_Search/05_Binary_Search_Index\|Binary Search ->]]*

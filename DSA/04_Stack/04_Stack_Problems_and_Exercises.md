---
tags: [dsa, stack, problems, exercises]
links: ["[[04_Stack_Index]]", "[[04_Stack_Patterns]]", "[[04_Stack_Tricky]]"]
---

# Stack -- Problems & Exercises

*<- [[04_Stack_Patterns|Patterns]] · [[04_Stack_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Valid Parentheses | Easy | Bracket match | [LC 20](https://leetcode.com/problems/valid-parentheses/) |
| 2 | Min Stack | Medium | Aux stack | [LC 155](https://leetcode.com/problems/min-stack/) |
| 3 | Implement Queue using Stacks | Easy | Two stacks | [LC 232](https://leetcode.com/problems/implement-queue-using-stacks/) |
| 4 | Evaluate Reverse Polish Notation | Medium | Stack eval | [LC 150](https://leetcode.com/problems/evaluate-reverse-polish-notation/) |

## Tier 2 -- Monotonic Stack Core

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 5 | Daily Temperatures | Medium | NGE via monostack | [LC 739](https://leetcode.com/problems/daily-temperatures/) |
| 6 | Next Greater Element I | Easy | NGE via map | [LC 496](https://leetcode.com/problems/next-greater-element-i/) |
| 7 | Next Greater Element II | Medium | NGE circular 2x | [LC 503](https://leetcode.com/problems/next-greater-element-ii/) |
| 8 | Online Stock Span | Medium | PGE with span | [LC 901](https://leetcode.com/problems/online-stock-span/) |
| 9 | Remove K Digits | Medium | Monotonic min | [LC 402](https://leetcode.com/problems/remove-k-digits/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 10 | Largest Rectangle in Histogram | Hard | Monostack + width | [LC 84](https://leetcode.com/problems/largest-rectangle-in-histogram/) |
| 11 | Maximal Rectangle | Hard | Histogram per row | [LC 85](https://leetcode.com/problems/maximal-rectangle/) |
| 12 | Basic Calculator II | Medium | Stack + sign | [LC 227](https://leetcode.com/problems/basic-calculator-ii/) |
| 13 | Longest Valid Parentheses | Hard | Stack + base idx | [LC 32](https://leetcode.com/problems/longest-valid-parentheses/) |
| 14 | Trapping Rain Water | Hard | Stack approach | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 15 | Sum of Subarray Minimums | Medium | Contribution technique | [LC 907](https://leetcode.com/problems/sum-of-subarray-minimums/) |
| 16 | Asteroid Collision | Medium | Stack simulation | [LC 735](https://leetcode.com/problems/asteroid-collision/) |
| 17 | Decode String | Medium | Stack + count | [LC 394](https://leetcode.com/problems/decode-string/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 18 | Basic Calculator I (with parentheses) | Hard | Recursive stack | [LC 224](https://leetcode.com/problems/basic-calculator/) |
| 19 | Sum of Subarray Ranges | Medium | NGE + PGE combined | [LC 2104](https://leetcode.com/problems/sum-of-subarray-ranges/) |
| 20 | Number of Visible People in Queue | Hard | Monotonic stack count | [LC 1944](https://leetcode.com/problems/number-of-visible-people-in-a-queue/) |
| 21 | Maximum Width Ramp | Medium | Monotonic stack + reverse scan | [LC 962](https://leetcode.com/problems/maximum-width-ramp/) |

---

## Worked Solution: LC 907 -- Sum of Subarray Minimums

**Key technique -- Contribution**: Instead of iterating over all subarrays, ask: "for how many subarrays is element `A[i]` the minimum?" Multiply `A[i]` by that count. The count depends on how far left and right `A[i]` extends before hitting a smaller element -- exactly what a monotonic stack gives you.

```cpp
int sumSubarrayMins(vector<int>& arr) {
    const int MOD = 1e9 + 7;
    int n = arr.size();
    // left[i]  = distance to previous smaller element (or start)
    // right[i] = distance to next smaller or equal element (or end)
    // Why "or equal" on right: avoid double counting equal elements
    vector<int> left(n), right(n);
    stack<int> st;

    // Compute left distances (previous smaller)
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        left[i] = st.empty() ? i + 1 : i - st.top();
        st.push(i);
    }

    while (!st.empty()) st.pop();

    // Compute right distances (next smaller or equal)
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] > arr[i]) st.pop();
        right[i] = st.empty() ? n - i : st.top() - i;
        st.push(i);
    }

    long long ans = 0;
    for (int i = 0; i < n; i++) {
        // arr[i] is minimum of left[i] * right[i] subarrays
        ans = (ans + (long long)arr[i] * left[i] % MOD * right[i]) % MOD;
    }
    return (int)ans;
}
// Time: O(n), Space: O(n)
```

---

## Worked Solution: LC 394 -- Decode String

> "3[a2[c]]" -> "accaccacc"

```cpp
string decodeString(string s) {
    stack<int> counts;
    stack<string> strs;
    string cur = "";
    int num = 0;

    for (char c : s) {
        if (isdigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '[') {
            counts.push(num);   // save repeat count
            strs.push(cur);     // save string built so far
            num = 0;
            cur = "";           // start fresh inside brackets
        } else if (c == ']') {
            int k = counts.top(); counts.pop();
            string prev = strs.top(); strs.pop();
            string repeated = "";
            for (int i = 0; i < k; i++) repeated += cur;
            cur = prev + repeated;
        } else {
            cur += c;
        }
    }
    return cur;
}
// Time: O(output length), Space: O(depth * string length)
```

---

*<- [[04_Stack_Patterns|Patterns]] · [[04_Stack_Tricky|Tricky ->]]*

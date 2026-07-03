---
tags: [dsa, stack, monotonic-stack, patterns]
links: ["[[04_Stack_Index]]", "[[04_Stack_Problems_and_Exercises]]"]
---

# Stack -- Patterns

*<- [[04_Stack_Index|Index]] · [[04_Stack_Problems_and_Exercises|Problems ->]]*

---

## What is a Stack?

A stack is a LIFO (Last In, First Out) structure. The last element pushed is the first one popped.

The key insight for interview problems: **a stack lets you remember the "most recent unresolved" element**. Whenever a new element resolves an older one (e.g., closes a bracket, is larger than a previous element), you pop.

```cpp
#include <stack>

std::stack<int> st;
st.push(5);            // push
int top = st.top();    // peek at top without removing
st.pop();              // remove top
bool empty = st.empty();
int size = st.size();

// In competitive programming, vector-as-stack is often faster
// (better cache behavior, direct index access)
std::vector<int> st2;
st2.push_back(5);      // push
int t = st2.back();    // top
st2.pop_back();        // pop
```

---

## Pattern 1: Bracket / Parenthesis Matching

**Core idea**: Push opening brackets. When you see a closing bracket, check if it matches the top. If it does, pop. If it doesn't, invalid.

```cpp
// LC 20 -- Valid Parentheses
bool isValid(std::string s) {
    std::stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            st.push(c);
        } else {
            if (st.empty()) return false;
            char top = st.top(); st.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return st.empty();  // stack must be empty -- all opened brackets were closed
}
// Time: O(n), Space: O(n)
```

---

## Pattern 2: Monotonic Stack -- Next Greater Element

**Why monotonic stack**: For each element, find the next element that is greater (or smaller). Naive is O(n²). Monotonic stack does it in O(n).

**How it works**: Maintain a stack of elements waiting to find their "next greater". When a new element arrives, it resolves all stack elements smaller than it.

```
nums = [2, 1, 5, 6, 2, 3]

Process 2: stack=[2]. No element resolved.
Process 1: 1 < 2, push. stack=[2,1].
Process 5: 5 > 1 -> pop 1, NGE[1]=5.
           5 > 2 -> pop 2, NGE[2]=5.
           stack=[5].
Process 6: 6 > 5 -> pop 5, NGE[5]=6. stack=[6].
Process 2: 2 < 6, push. stack=[6,2].
Process 3: 3 > 2 -> pop 2, NGE[2]=3. 3 < 6, push. stack=[6,3].
End: NGE[6]=-1, NGE[3]=-1 (no greater element to the right).

Result: [5, 5, 6, -1, 3, -1]
```

```cpp
#include <vector>
#include <stack>

// LC 496 -- Next Greater Element I
// For each element, find next greater element to its RIGHT
std::vector<int> nextGreaterElement(std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> nge(n, -1);
    std::stack<int> st;  // stores indices

    for (int i = 0; i < n; i++) {
        // Current element resolves all smaller elements waiting in stack
        while (!st.empty() && nums[st.top()] < nums[i]) {
            nge[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return nge;  // remaining elements in stack have no NGE -> stay -1
}
// Time: O(n) -- each element pushed and popped exactly once
// Space: O(n)
```

### Variation: Next Greater Element on Circular Array (LC 503)

```cpp
// Process array twice (2*n iterations) using modular index
std::vector<int> nextGreaterElements(std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> result(n, -1);
    std::stack<int> st;

    for (int i = 0; i < 2 * n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i % n]) {
            result[st.top()] = nums[i % n];
            st.pop();
        }
        if (i < n) st.push(i);  // only push on first pass
    }
    return result;
}
```

---

## Pattern 3: Monotonic Stack -- Previous Greater / Smaller

**Why**: Many problems need the nearest greater/smaller element to the LEFT. Process right-to-left, or track what's on the stack when you push.

```cpp
// Previous Greater Element (PGE) -- nearest greater to the LEFT
std::vector<int> previousGreater(std::vector<int>& nums) {
    int n = nums.size();
    std::vector<int> pge(n, -1);
    std::stack<int> st;  // monotonically decreasing from bottom to top

    for (int i = 0; i < n; i++) {
        // Pop elements <= current (they can't be PGE for future elements either
        // because current is closer and also larger)
        while (!st.empty() && nums[st.top()] <= nums[i]) st.pop();
        // Top of stack (if any) is the nearest greater element to the left
        if (!st.empty()) pge[i] = nums[st.top()];
        st.push(i);
    }
    return pge;
}
```

---

## Pattern 4: Largest Rectangle in Histogram (LC 84)

**Why it's the canonical monotonic stack problem**: For each bar, the rectangle it can form extends as far left and right as bars taller than it. A monotonic increasing stack tracks exactly the "left boundary" of each bar.

```
heights = [2, 1, 5, 6, 2, 3]

Maintain stack of indices with increasing heights.
When we pop index i (because current height is smaller):
  - right boundary = current index
  - left boundary = new stack top (or -1 if empty)
  - width = right - left - 1
  - area = heights[i] * width
```

```cpp
// LC 84 -- Largest Rectangle in Histogram
int largestRectangleArea(std::vector<int>& heights) {
    int n = heights.size();
    std::stack<int> st;  // monotonically increasing stack of indices
    int maxArea = 0;

    for (int i = 0; i <= n; i++) {
        // Use 0 as sentinel height at the end to flush all remaining bars
        int curHeight = (i == n) ? 0 : heights[i];

        while (!st.empty() && heights[st.top()] > curHeight) {
            int h = heights[st.top()]; st.pop();
            // Width: from (new top + 1) to (i - 1)
            int width = st.empty() ? i : i - st.top() - 1;
            maxArea = std::max(maxArea, h * width);
        }
        st.push(i);
    }
    return maxArea;
}
// Time: O(n), Space: O(n)
```

### Extension: Maximal Rectangle in Binary Matrix (LC 85)

```cpp
// Reduce each row to a histogram problem.
// For each row, compute the height of consecutive 1s above (including current row).
// Then apply largestRectangleArea on that histogram.
int maximalRectangle(std::vector<std::vector<char>>& matrix) {
    if (matrix.empty()) return 0;
    int rows = matrix.size(), cols = matrix[0].size();
    std::vector<int> heights(cols, 0);
    int maxArea = 0;

    for (int r = 0; r < rows; r++) {
        // Update histogram heights
        for (int c = 0; c < cols; c++) {
            heights[c] = (matrix[r][c] == '1') ? heights[c] + 1 : 0;
        }
        maxArea = std::max(maxArea, largestRectangleArea(heights));
    }
    return maxArea;
}
// Time: O(rows * cols), Space: O(cols)
```

---

## Pattern 5: Stack for Expression Evaluation

**Why**: Parsing arithmetic expressions with correct operator precedence requires tracking operators and operands separately.

```cpp
// Evaluate: "3 + 2 * 2" = 7,  " 3/2 " = 1,  " 3+5 / 2 " = 5
// Rule: handle * and / immediately. Defer + and - (push with sign).
// At the end, sum everything in the stack.
int calculate(std::string s) {
    std::stack<int> st;
    int num = 0;
    char op = '+';  // last operator seen (start as + so first num gets pushed)

    for (int i = 0; i < (int)s.size(); i++) {
        char c = s[i];
        if (std::isdigit(c)) {
            num = num * 10 + (c - '0');
        }
        // Process when we see an operator or reach end of string
        if ((!std::isdigit(c) && c != ' ') || i == (int)s.size() - 1) {
            if (op == '+') st.push(num);
            else if (op == '-') st.push(-num);
            else if (op == '*') { int top = st.top(); st.pop(); st.push(top * num); }
            else if (op == '/') { int top = st.top(); st.pop(); st.push(top / num); }
            op = c;
            num = 0;
        }
    }

    int result = 0;
    while (!st.empty()) { result += st.top(); st.pop(); }
    return result;
}
// Time: O(n), Space: O(n)
```

---

## Pattern 6: Min Stack / Max Stack (O(1) Min with Stack Operations)

```cpp
// LC 155 -- Min Stack
// Maintain two stacks: main stack and auxiliary min stack
class MinStack {
    std::stack<int> st, minSt;
public:
    void push(int val) {
        st.push(val);
        // Push to minSt if it's the new minimum (or minSt is empty)
        if (minSt.empty() || val <= minSt.top()) minSt.push(val);
    }
    void pop() {
        if (st.top() == minSt.top()) minSt.pop();
        st.pop();
    }
    int top() { return st.top(); }
    int getMin() { return minSt.top(); }
};
// All operations: O(1), Space: O(n)
```

---

## Pattern 7: Stock Span (Monotonic Stack -- Days)

```cpp
// LC 901 -- Online Stock Span
// For each day's price, find how many consecutive days before it had price <= today
// Monotonic decreasing stack stores {price, span}
class StockSpanner {
    std::stack<std::pair<int,int>> st;  // {price, span}
public:
    int next(int price) {
        int span = 1;
        while (!st.empty() && st.top().first <= price) {
            span += st.top().second;  // absorb spans of dominated days
            st.pop();
        }
        st.push({price, span});
        return span;
    }
};
// Time: O(1) amortized per call
```

---

## Examples

### Example 1 -- LC 739: Daily Temperatures (Medium)

> For each day, how many days until a warmer day? Return -1 if none.

```cpp
std::vector<int> dailyTemperatures(std::vector<int>& temps) {
    int n = temps.size();
    std::vector<int> result(n, 0);
    std::stack<int> st;  // indices of days waiting for warmer day

    for (int i = 0; i < n; i++) {
        while (!st.empty() && temps[st.top()] < temps[i]) {
            int idx = st.top(); st.pop();
            result[idx] = i - idx;  // days until warmer
        }
        st.push(i);
    }
    return result;
}
```

### Example 2 -- LC 42: Trapping Rain Water (Stack Approach)

```cpp
// Alternative to two-pointer: use stack to track walls
int trap(std::vector<int>& height) {
    std::stack<int> st;
    int water = 0;
    for (int i = 0; i < (int)height.size(); i++) {
        while (!st.empty() && height[st.top()] < height[i]) {
            int bottom = height[st.top()]; st.pop();
            if (st.empty()) break;
            int width = i - st.top() - 1;
            int bounded = std::min(height[st.top()], height[i]) - bottom;
            water += bounded * width;
        }
        st.push(i);
    }
    return water;
}
```

### Example 3 -- LC 32: Longest Valid Parentheses (Hard)

```cpp
// Use stack storing indices. Push -1 as base.
int longestValidParentheses(std::string s) {
    std::stack<int> st;
    st.push(-1);  // base sentinel
    int maxLen = 0;

    for (int i = 0; i < (int)s.size(); i++) {
        if (s[i] == '(') {
            st.push(i);
        } else {
            st.pop();  // try to match with top
            if (st.empty()) {
                st.push(i);  // no match -- this ) becomes new base
            } else {
                maxLen = std::max(maxLen, i - st.top());
            }
        }
    }
    return maxLen;
}
```

---

## Exercises

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Valid Parentheses | Easy | Bracket match | [LC 20](https://leetcode.com/problems/valid-parentheses/) |
| 2 | Min Stack | Medium | Aux min stack | [LC 155](https://leetcode.com/problems/min-stack/) |
| 3 | Daily Temperatures | Medium | Monotonic stack NGE | [LC 739](https://leetcode.com/problems/daily-temperatures/) |
| 4 | Next Greater Element I | Easy | Monotonic stack | [LC 496](https://leetcode.com/problems/next-greater-element-i/) |
| 5 | Next Greater Element II (circular) | Medium | Monotonic, 2x pass | [LC 503](https://leetcode.com/problems/next-greater-element-ii/) |
| 6 | Largest Rectangle in Histogram | Hard | Monotonic stack | [LC 84](https://leetcode.com/problems/largest-rectangle-in-histogram/) |
| 7 | Maximal Rectangle | Hard | Histogram per row | [LC 85](https://leetcode.com/problems/maximal-rectangle/) |
| 8 | Evaluate Reverse Polish Notation | Medium | Stack eval | [LC 150](https://leetcode.com/problems/evaluate-reverse-polish-notation/) |
| 9 | Basic Calculator II | Medium | Stack + signs | [LC 227](https://leetcode.com/problems/basic-calculator-ii/) |
| 10 | Online Stock Span | Medium | Monotonic + span | [LC 901](https://leetcode.com/problems/online-stock-span/) |
| 11 | Trapping Rain Water | Hard | Stack OR two-ptr | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |
| 12 | Longest Valid Parentheses | Hard | Stack + base idx | [LC 32](https://leetcode.com/problems/longest-valid-parentheses/) |
| 13 | Remove K Digits | Medium | Monotonic stack | [LC 402](https://leetcode.com/problems/remove-k-digits/) |
| 14 | Sum of Subarray Minimums | Medium | Contribution via monostack | [LC 907](https://leetcode.com/problems/sum-of-subarray-minimums/) |

---

*<- [[04_Stack_Index|Index]] · [[04_Stack_Tricky|Tricky ->]]*

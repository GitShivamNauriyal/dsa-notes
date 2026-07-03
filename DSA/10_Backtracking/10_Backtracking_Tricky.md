---
tags: [dsa, backtracking, recursion, tricky, hard]
links: ["[[10_Backtracking_Index]]", "[[10_Backtracking_Problems_and_Exercises]]", "[[../11_Graphs/11_Graphs_Index]]"]
---

# Backtracking -- Tricky & Higher-Order

*<- [[10_Backtracking_Problems_and_Exercises|Problems]] · [[../11_Graphs/11_Graphs_Index|Graphs ->]]*

---

## 1. Backtracking Call Stack Unwinding (Visualized)

**Why Tricky**: Junior engineers often struggle to visualize how variables revert during backtracking. Below is an ASCII visualization of stack frame frames for generating permutations of `[1, 2]` using Swap-Based backtracking.

### Execution Trace & Stack State
```
1. Call permuteHelper(0)
   ├── Stack Frame [start=0, i=0]: Swap(0,0) -> nums = [1, 2]
   │     └── 2. Call permuteHelper(1)
   │            ├── Stack Frame [start=1, i=1]: Swap(1,1) -> nums = [1, 2]
   │            │     └── 3. Call permuteHelper(2) -> Base case reached! Add [1, 2]
   │            └── Backtrack: Swap(1,1) -> nums = [1, 2]
   ├── Backtrack: Swap(0,0) -> nums = [1, 2]
   │
   ├── Stack Frame [start=0, i=1]: Swap(0,1) -> nums = [2, 1]
   │     └── 4. Call permuteHelper(1)
   │            ├── Stack Frame [start=1, i=1]: Swap(1,1) -> nums = [2, 1]
   │            │     └── 5. Call permuteHelper(2) -> Base case reached! Add [2, 1]
   │            └── Backtrack: Swap(1,1) -> nums = [2, 1]
   └── Backtrack: Swap(0,1) -> nums = [1, 2] (reverted to original!)
```

---

## 2. Expression Add Operators (LC 282)

**Why Tricky**: You are given a string of digits and a target value. You must insert binary operators (`+`, `-`, or `*`) to make the expression evaluate to the target. 
- Multiplication precedence is the core challenge. For example, if evaluating `2 + 3 * 5`, when you process `* 5`, you cannot simply multiply the running sum (`5`) by `5` (which gives `25`). You must instead subtract the last added value (`3`) and add (`3 * 5`), i.e. `(5 - 3) + (3 * 5) = 17`.
- Leading zeros are invalid (e.g. `1 + 05` is invalid).

```cpp
#include <vector>
#include <string>

// LeetCode 282 — Expression Add Operators
void dfs(int idx, const string& num, int target, long long runningVal, long long prevVal, 
         string currentExpr, vector<string>& result) {
    if (idx == (int)num.size()) {
        if (runningVal == target) result.push_back(currentExpr);
        return;
    }

    for (int i = idx; i < (int)num.size(); i++) {
        // Skip leading zeros
        if (i > idx && num[idx] == '0') break;

        string partStr = num.substr(idx, i - idx + 1);
        long long val = stoll(partStr);

        if (idx == 0) {
            // First number, no operator can precede it
            dfs(i + 1, num, target, val, val, partStr, result);
        } else {
            // Addition: add val to running value, val becomes the new prevVal
            dfs(i + 1, num, target, runningVal + val, val, currentExpr + "+" + partStr, result);

            // Subtraction: subtract val from running value, -val becomes the new prevVal
            dfs(i + 1, num, target, runningVal - val, -val, currentExpr + "-" + partStr, result);

            // Multiplication: subtract prevVal from runningVal, then add (prevVal * val).
            // The new prevVal contribution becomes (prevVal * val).
            dfs(i + 1, num, target, runningVal - prevVal + (prevVal * val), prevVal * val, 
                currentExpr + "*" + partStr, result);
        }
    }
}

vector<string> addOperators(string num, int target) {
    vector<string> result;
    if (num.empty()) return result;
    dfs(0, num, target, 0, 0, "", result);
    return result;
}
// Time: O(4^n) — 4 choices per digit (+, -, *, join)
// Space: O(n) recursion stack height
```

---

## 3. Sudoku Solver (LC 37)

**Why Tricky**: Standard backtracking might repeatedly loop through valid candidate checks. Placing row, column, and $3 \times 3$ box constraint checking in $O(1)$ time using arrays or bitmasks is highly optimized.

```cpp
#include <vector>

bool isValid(const vector<vector<char>>& board, int r, int c, char ch) {
    for (int i = 0; i < 9; i++) {
        if (board[r][i] == ch) return false; // check row
        if (board[i][c] == ch) return false; // check col
        // Check 3x3 sub-grid
        if (board[3 * (r / 3) + i / 3][3 * (c / 3) + i % 3] == ch) return false;
    }
    return true;
}

bool solve(vector<vector<char>>& board) {
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (board[r][c] == '.') {
                for (char ch = '1'; ch <= '9'; ch++) {
                    if (isValid(board, r, c, ch)) {
                        board[r][c] = ch; // Choose
                        if (solve(board)) return true; // Explore
                        board[r][c] = '.'; // Backtrack
                    }
                }
                return false; // dead end
            }
        }
    }
    return true; // all cells filled
}

void solveSudoku(vector<vector<char>>& board) {
    solve(board);
}
// Time: O(9^(empty_cells)) -> O(1) in theory for fixed 9x9 board
// Space: O(empty_cells) recursion stack height
```

---

## 4. Palindrome Partitioning (LC 131)

**Why Tricky**: Partition a string such that every substring of the partition is a palindrome. Requires checking a prefix for palindromic property, then backtracking on the suffix.

```cpp
#include <vector>
#include <string>

bool isPalindrome(const string& s, int lo, int hi) {
    while (lo < hi) {
        if (s[lo++] != s[hi--]) return false;
    }
    return true;
}

void dfs(int start, const string& s, vector<string>& current, vector<vector<string>>& result) {
    if (start == (int)s.size()) {
        result.push_back(current);
        return;
    }

    for (int i = start; i < (int)s.size(); i++) {
        if (isPalindrome(s, start, i)) {
            current.push_back(s.substr(start, i - start + 1)); // Choose
            dfs(i + 1, s, current, result);                    // Explore
            current.pop_back();                               // Backtrack
        }
    }
}

vector<vector<string>> partition(string s) {
    vector<vector<string>> result;
    vector<string> current;
    dfs(0, s, current, result);
    return result;
}
// Time: O(n * 2^n)
// Space: O(n)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Sudoku Solver | Hard | [LC 37](https://leetcode.com/problems/sudoku-solver/) |
| 2 | Expression Add Operators | Hard | [LC 282](https://leetcode.com/problems/expression-add-operators/) |
| 3 | Unique Paths III | Hard | [LC 980](https://leetcode.com/problems/unique-paths-iii/) |
| 4 | Matchsticks to Square | Medium | [LC 473](https://leetcode.com/problems/matchsticks-to-square/) |

---

*<- [[10_Backtracking_Problems_and_Exercises|Problems]] · [[../11_Graphs/11_Graphs_Index|Graphs ->]]*

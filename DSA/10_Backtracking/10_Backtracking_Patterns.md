---
tags: [dsa, backtracking, recursion, patterns]
links: ["[[10_Backtracking_Index]]", "[[10_Backtracking_Problems_and_Exercises]]"]
---

# Backtracking -- Patterns

*<- [[10_Backtracking_Index|Index]] · [[10_Backtracking_Problems_and_Exercises|Problems ->]]*

---

## What is Backtracking?

Backtracking is a systematic way of searching the entire **state-space** of a problem. It behaves like DFS on a tree of choices. 

At each step, you:
1. **Choose**: Select a path/option.
2. **Explore**: Recursively search down that path.
3. **Unchoose (Backtrack)**: Undo your selection to restore the state before exploring other paths.

### Backtracking Template
```cpp
void backtrack(State& state, Result& res) {
    if (isBaseCase(state)) {
        res.add(state);
        return;
    }
    for (auto choice : getChoices(state)) {
        if (isValid(choice, state)) {
            makeChoice(choice, state); // Choose
            backtrack(state, res);     // Explore
            undoChoice(choice, state); // Unchoose / Backtrack
        }
    }
}
```

---

## Pattern 1: Subsets / Power Set (Recursive Choose/Not-Choose)

**Core Idea**: For every element in an array, you have two choices: **include it** in the current subset, or **exclude it**.

### Recursion Tree (for `nums = [1, 2]`)
```
                          Root [] (index=0)
                         /                 \
                 Include 1                 Exclude 1
                     /                         \
                State [1] (index=1)         State [] (index=1)
                /         \                 /        \
          Include 2     Exclude 2     Include 2    Exclude 2
            /             \             /             \
        State [1,2]     State [1]     State [2]       State []
        (index=2)       (index=2)     (index=2)       (index=2)
          LEAF            LEAF          LEAF            LEAF
```

```cpp
#include <vector>

// LeetCode 78 — Subsets
void subsetHelper(int idx, const vector<int>& nums, vector<int>& current, vector<vector<int>>& result) {
    if (idx == (int)nums.size()) {
        result.push_back(current); // base case: processed all elements
        return;
    }

    // Choice 1: Include nums[idx]
    current.push_back(nums[idx]);
    subsetHelper(idx + 1, nums, current, result);
    current.pop_back(); // backtrack

    // Choice 2: Exclude nums[idx]
    subsetHelper(idx + 1, nums, current, result);
}

vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    vector<int> current;
    subsetHelper(0, nums, current, result);
    return result;
}
// Time: O(n * 2^n) — 2^n subsets, copy size O(n) at leaf
// Space: O(n) recursion stack height
```

---

## Pattern 2: Combinations

**Core Idea**: Find all combinations of size $k$ from numbers $1$ to $n$. To avoid duplicate combinations like `[1, 2]` and `[2, 1]`, only look forward by using a `start` variable.

```cpp
#include <vector>

// LeetCode 77 — Combinations
void combineHelper(int start, int n, int k, vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == k) {
        result.push_back(current);
        return;
    }

    for (int i = start; i <= n; i++) {
        current.push_back(i);                      // Choose
        combineHelper(i + 1, n, k, current, result); // Explore (i+1 ensures forward-only)
        current.pop_back();                         // Backtrack
    }
}

vector<vector<int>> combine(int n, int k) {
    vector<vector<int>> result;
    vector<int> current;
    combineHelper(1, n, k, current, result);
    return result;
}
// Time: O(k * nCr)
// Space: O(k) recursion stack
```

---

## Pattern 3: Permutations (Swap-Based vs. Visited-Array)

**Core Idea**: For permutations, order matters (`[1, 2]` $\ne$ `[2, 1]`). You must try placing every unused element in the current position.

### Approach A: Swap-Based (In-Place, $O(1)$ Extra Space)
Generate permutations by swapping the current index with all subsequent indices.

```cpp
#include <vector>

// LeetCode 46 — Permutations
void permuteHelper(int start, vector<int>& nums, vector<vector<int>>& result) {
    if (start == (int)nums.size()) {
        result.push_back(nums);
        return;
    }

    for (int i = start; i < (int)nums.size(); i++) {
        swap(nums[start], nums[i]);       // Choose (swap to place element at 'start')
        permuteHelper(start + 1, nums, result); // Explore
        swap(nums[start], nums[i]);       // Backtrack (swap back)
    }
}

vector<vector<int>> permute(vector<int>& nums) {
    vector<vector<int>> result;
    permuteHelper(0, nums, result);
    return result;
}
// Time: O(n! * n)
// Space: O(n) recursion stack
```

### Approach B: Using a Visited Array (Preserves relative ordering)

```cpp
#include <vector>

void permuteVisitedHelper(const vector<int>& nums, vector<bool>& visited, 
                           vector<int>& current, vector<vector<int>>& result) {
    if (current.size() == nums.size()) {
        result.push_back(current);
        return;
    }

    for (int i = 0; i < (int)nums.size(); i++) {
        if (!visited[i]) {
            visited[i] = true;
            current.push_back(nums[i]);              // Choose
            permuteVisitedHelper(nums, visited, current, result); // Explore
            current.pop_back();                      // Backtrack
            visited[i] = false;
        }
    }
}
```

---

## Pattern 4: Grid DFS / Backtracking (Word Search)

**Core Idea**: Walk a 2D grid looking for a word. Mark visited grid cells in-place with a sentinel character (e.g. `'#'`) to avoid using an extra boolean array, then restore the original character during backtracking.

```cpp
#include <vector>
#include <string>

// LeetCode 79 — Word Search
bool dfs(int r, int c, int wordIdx, vector<vector<char>>& board, const string& word) {
    if (wordIdx == (int)word.size()) return true; // found full word

    // Bounds and matching checks
    if (r < 0 || r >= (int)board.size() || c < 0 || c >= (int)board[0].size() || board[r][c] != word[wordIdx]) {
        return false;
    }

    char temp = board[r][c];
    board[r][c] = '#'; // Choose: mark visited in-place

    // Explore: 4 directions
    int dirs[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};
    for (auto& d : dirs) {
        if (dfs(r + d[0], c + d[1], wordIdx + 1, board, word)) {
            return true;
        }
    }

    board[r][c] = temp; // Backtrack: restore original character
    return false;
}

bool exist(vector<vector<char>>& board, string word) {
    int rows = board.size(), cols = board[0].size();
    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++) {
            if (dfs(r, c, 0, board, word)) return true;
        }
    }
    return false;
}
// Time: O(M * N * 3^L) where M*N is grid size, L is word length (3 choices instead of 4 because we don't return to parent)
// Space: O(L) recursion stack depth
```

---

## Pattern 5: N-Queens (Constrained Placement)

**Core Idea**: Place $n$ queens on an $n \times n$ chessboard such that no two queens attack each other. 
- A queen can attack horizontally, vertically, and diagonally.
- Place one queen per row. Track invalid columns, positive diagonals ($r + c$), and negative diagonals ($r - c$).

```cpp
#include <vector>
#include <string>
#include <unordered_set>

// LeetCode 51 — N-Queens
void solveNQueensHelper(int row, int n, vector<string>& board,
                        unordered_set<int>& cols,
                        unordered_set<int>& posDiag, // r + c
                        unordered_set<int>& negDiag, // r - c
                        vector<vector<string>>& result) {
    if (row == n) {
        result.push_back(board);
        return;
    }

    for (int col = 0; col < n; col++) {
        if (cols.count(col) || posDiag.count(row + col) || negDiag.count(row - col)) {
            continue; // position under attack
        }

        // Choose
        board[row][col] = 'Q';
        cols.insert(col);
        posDiag.insert(row + col);
        negDiag.insert(row - col);

        // Explore
        solveNQueensHelper(row + 1, n, board, cols, posDiag, negDiag, result);

        // Backtrack
        board[row][col] = '.';
        cols.erase(col);
        posDiag.erase(row + col);
        negDiag.erase(row - col);
    }
}

vector<vector<string>> solveNQueens(int n) {
    vector<vector<string>> result;
    vector<string> board(n, string(n, '.'));
    unordered_set<int> cols, posDiag, negDiag;
    solveNQueensHelper(0, n, board, cols, posDiag, negDiag, result);
    return result;
}
// Time: O(n!) — search reduces choice count per row
// Space: O(n) for sets + recursion stack
```

---

*<- [[10_Backtracking_Index|Index]] · [[10_Backtracking_Problems_and_Exercises|Problems ->]]*

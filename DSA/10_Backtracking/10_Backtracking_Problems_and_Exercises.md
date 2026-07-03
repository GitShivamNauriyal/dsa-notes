---
tags: [dsa, backtracking, recursion, problems, exercises]
links: ["[[10_Backtracking_Index]]", "[[10_Backtracking_Patterns]]", "[[10_Backtracking_Tricky]]"]
---

# Backtracking -- Problems & Exercises

*<- [[10_Backtracking_Patterns|Patterns]] · [[10_Backtracking_Tricky|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Subsets | Medium | Include/Exclude pattern | [LC 78](https://leetcode.com/problems/subsets/) |
| 2 | Combinations | Medium | Forward-only start index | [LC 77](https://leetcode.com/problems/combinations/) |
| 3 | Letter Combinations of a Phone Number | Medium | Iterative cartesian/backtrack | [LC 17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) |

## Tier 2 -- Duplicate Handling & Constraints

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 4 | Subsets II | Medium | Sort + skip duplicates | [LC 90](https://leetcode.com/problems/subsets-ii/) |
| 5 | Permutations | Medium | Swap-based permutations | [LC 46](https://leetcode.com/problems/permutations/) |
| 6 | Permutations II | Medium | Visited array + duplicate filter | [LC 47](https://leetcode.com/problems/permutations-ii/) |
| 7 | Combination Sum | Medium | Repeating elements allowed | [LC 39](https://leetcode.com/problems/combination-sum/) |
| 8 | Combination Sum II | Medium | Unique elements, no reuse | [LC 40](https://leetcode.com/problems/combination-sum-ii/) |

## Tier 3 -- Board Games & Search

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 9 | Word Search | Medium | In-place marking + 2D DFS | [LC 79](https://leetcode.com/problems/word-search/) |
| 10 | Palindrome Partitioning | Medium | Prefix check + suffix backtrack | [LC 131](https://leetcode.com/problems/palindrome-partitioning/) |
| 11 | N-Queens | Hard | Col & diagonal checks | [LC 51](https://leetcode.com/problems/n-queens/) |
| 12 | Sudoku Solver | Hard | Row, col, and box constraints | [LC 37](https://leetcode.com/problems/sudoku-solver/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 13 | Expression Add Operators | Hard | Value tracker + multiplication carry | [LC 282](https://leetcode.com/problems/expression-add-operators/) |
| 14 | Remove Invalid Parentheses | Hard | BFS/DFS pruning minimum removals | [LC 301](https://leetcode.com/problems/remove-invalid-parentheses/) |
| 15 | Matchsticks to Square | Medium | Partition array into 4 equal groups | [LC 473](https://leetcode.com/problems/matchsticks-to-square/) |

---

## Worked Solution 1: LC 90 -- Subsets II (Handling Duplicates)

**Key Technique**: If the input has duplicate numbers, the naive choose/not-choose pattern creates duplicate subsets (e.g. `[1, 2(first)]` and `[1, 2(second)]`). 
**Solution**:
1. **Sort** the input array so duplicates are adjacent.
2. In the "exclude" branch of the recursion tree, skip over all subsequent duplicate elements.

### Decision Tree for `nums = [1, 2, 2]`
```
                          Root []
                         /       \
                 Include 1       Exclude 1
                     /               \
                 State [1]           State []
                 /       \           /      \
             Incl 2     Skip 2     Incl 2   Skip 2
              /            \         /         \
         State [1,2]     State [1] State [2]  State []
          /       \        LEAF     /     \     LEAF
       Incl 2    Skip 2          Incl 2  Skip 2
        /           \             /         \
    State [1,2,2] State [1,2] State [2,2] State [2]
      LEAF          LEAF        LEAF       LEAF
```

```cpp
#include <vector>
#include <algorithm>

void dfs(int idx, const vector<int>& nums, vector<int>& current, vector<vector<int>>& result) {
    if (idx == (int)nums.size()) {
        result.push_back(current);
        return;
    }

    // Branch 1: Include current element
    current.push_back(nums[idx]);
    dfs(idx + 1, nums, current, result);
    current.pop_back(); // backtrack

    // Branch 2: Exclude current element AND skip all its duplicates
    int nextIdx = idx + 1;
    while (nextIdx < (int)nums.size() && nums[nextIdx] == nums[idx]) {
        nextIdx++;
    }
    dfs(nextIdx, nums, current, result);
}

vector<vector<int>> subsetsWithDup(vector<int>& nums) {
    sort(nums.begin(), nums.end()); // sort to place duplicates adjacent
    vector<vector<int>> result;
    vector<int> current;
    dfs(0, nums, current, result);
    return result;
}
// Time: O(n * 2^n)
// Space: O(n)
```

---

## Worked Solution 2: LC 40 -- Combination Sum II

**Key Technique**: Find all unique combinations summing to a target, where each number in `candidates` can only be used once. Duplicates are handled by sorting the array and, at each position in the combination, only picking the first occurrence of a duplicate.

```cpp
#include <vector>
#include <algorithm>

void dfs(int start, int target, const vector<int>& candidates, 
         vector<int>& current, vector<vector<int>>& result) {
    if (target == 0) {
        result.push_back(current); // found a valid combination
        return;
    }

    for (int i = start; i < (int)candidates.size(); i++) {
        if (candidates[i] > target) break; // pruning: remaining elements too large

        // Skip duplicates in the same position
        if (i > start && candidates[i] == candidates[i - 1]) continue;

        current.push_back(candidates[i]);
        // i + 1 because we cannot reuse the same element
        dfs(i + 1, target - candidates[i], candidates, current, result);
        current.pop_back(); // backtrack
    }
}

vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end()); // sort to handle duplicates
    vector<vector<int>> result;
    vector<int> current;
    dfs(0, target, candidates, current, result);
    return result;
}
// Time: O(2^n) worst case
// Space: O(target/min_element) recursion stack depth
```

---

*<- [[10_Backtracking_Patterns|Patterns]] · [[10_Backtracking_Tricky|Tricky ->]]*

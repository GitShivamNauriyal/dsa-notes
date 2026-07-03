---
tags: [dsa, dp, trees, tree-dp, max-path-sum, house-robber]
links: ["[[06_DP_on_Trees_Index]]", "[[DP_on_Trees_Problems]]", "[[../../07_Trees/07_Trees_Index]]"]
---

# DP on Trees -- Foundations & Patterns

*<- [[06_DP_on_Trees_Index|Trees Index]] · [[DP_on_Trees_Problems|Problems ->]]*

---

## What is Tree DP?

Tree DP uses **Post-Order DFS** (bottom-up traversal) to solve subproblems on subtrees. 
Each node queries information from its left and right children, computes its own states, and returns them up to its parent.

```
                  Node (processes left & right child outputs)
                 /    \
     left child        right child
 (returns state)      (returns state)
```

---

## 1. House Robber III (LC 337)

**The Problem**: The houses are arranged as a binary tree. If two directly-linked houses are broken into on the same night, the police are contacted. Return the maximum money you can rob without alerting the police.

### State Formulation
Each node returns a pair of values: `{rob_this, skip_this}`.
1. **`rob_this`**: The maximum money if we rob the current node. This means we cannot rob its children.
   `rob_this = node->val + left.skip_this + right.skip_this`
2. **`skip_this`**: The maximum money if we do not rob the current node. We can choose to rob or skip either of its children, taking the maximum.
   `skip_this = max(left.rob_this, left.skip_this) + max(right.rob_this, right.skip_this)`

```cpp
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
};

// Helper returns pair: {rob_node, skip_node}
pair<int, int> robHelper(TreeNode* root) {
    if (!root) return {0, 0};

    auto left = robHelper(root->left);
    auto right = robHelper(root->right);

    // Option 1: Rob this node (children must be skipped)
    int robThis = root->val + left.second + right.second;

    // Option 2: Skip this node (children can be either robbed or skipped)
    int skipThis = max(left.first, left.second) + max(right.first, right.second);

    return {robThis, skipThis};
}

int rob(TreeNode* root) {
    auto res = robHelper(root);
    return max(res.first, res.second);
}
// Time Complexity: O(N) — visit each node exactly once
// Space Complexity: O(H) — recursion stack height (H is tree height)
```

---

## 2. Binary Tree Maximum Path Sum (LC 124)

**The Problem**: A path in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them. A node can only appear in the sequence at most once. Return the maximum path sum of any non-empty path.

### State Formulation
Each node must return the **maximum path sum starting at this node and extending down to at most one of its children**. This allows the parent node to extend the path.
- Let this return value be `singlePathSum = node->val + max({0, leftPath, rightPath})` (if a child returns negative path sum, we prune it by taking 0).

At the current node, we also calculate the **sum of the path that curves through this node** (using both left and right children):
- `curMaxPathSum = node->val + max(0, leftPath) + max(0, rightPath)`
- We update a global/member variable `globalMax` with `curMaxPathSum` at each node.

```cpp
#include <algorithm>

class Solution {
    int globalMax = -1e9;

    int maxPathSumHelper(TreeNode* node) {
        if (!node) return 0;

        // DFS down to children. Prune negative path sum contributions by taking max(0, val)
        int leftMax = max(0, maxPathSumHelper(node->left));
        int rightMax = max(0, maxPathSumHelper(node->right));

        // Path curving through current node
        int currentPathSum = node->val + leftMax + rightMax;
        globalMax = max(globalMax, currentPathSum); // update global max path sum

        // Return path extending to at most one child
        return node->val + max(leftMax, rightMax);
    }

public:
    int maxPathSum(TreeNode* root) {
        maxPathSumHelper(root);
        return globalMax;
    }
};
// Time Complexity: O(N)
// Space Complexity: O(H)
```

---

*<- [[06_DP_on_Trees_Index|Trees Index]] · [[DP_on_Trees_Problems|Problems ->]]*

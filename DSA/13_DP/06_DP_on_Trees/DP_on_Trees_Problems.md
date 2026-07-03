---
tags: [dsa, dp, trees, problems, exercises]
links: ["[[06_DP_on_Trees_Index]]", "[[DP_on_Trees_Tree_DP]]", "[[../07_Partition_DP/07_Partition_DP_Index]]"]
---

# DP on Trees -- Problems & Exercises

*<- [[DP_on_Trees_Tree_DP|Tree DP Foundations]] · [[../07_Partition_DP/07_Partition_DP_Index|Partition DP ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Diameter of Binary Tree | Easy | Max depth combination | [LC 543](https://leetcode.com/problems/diameter-of-binary-tree/) |
| 2 | Maximum Depth of Binary Tree | Easy | Post-order height check | [LC 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 3 | House Robber III | Medium | State pair `{rob, skip}` | [LC 337](https://leetcode.com/problems/house-robber-iii/) |
| 4 | Binary Tree Maximum Path Sum | Hard | Single path returns + global update | [LC 124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) |
| 5 | Longest Univalue Path | Medium | Match checks + post-order traversal | [LC 687](https://leetcode.com/problems/longest-univalue-path/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Sum of Distances in Tree | Hard | Re-rooting tree DP ($O(N)$ double pass) | [LC 834](https://leetcode.com/problems/sum-of-distances-in-tree/) |
| 7 | All Nodes Distance K in Binary Tree | Medium | Parent mapping + BFS | [LC 863](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/) |

## Tier 4 -- Placement-Hard / OA-Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 8 | Tree Distances I | Hard | Tree DP re-rooting maximum paths | [CSES 1130](https://cses.fi/problemset/task/1130/) |
| 9 | Maximum Sum BST in Binary Tree | Hard | Validate BST range + sum bottom-up | [LC 1373](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/) |

---

## Worked Solution: Diameter of Binary Tree (LC 543)

**Key Insight**: The diameter of a binary tree is the length of the longest path between any two nodes in a tree. This path may or may not pass through the root.
- For any node, the longest path passing through it is `leftHeight + rightHeight`.
- We run DFS. Each node returns its height `1 + max(leftHeight, rightHeight)` to its parent, while updating a global max diameter with `leftHeight + rightHeight`.

```cpp
#include <algorithm>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
};

class Solution {
    int maxDiameter = 0;

    int height(TreeNode* node) {
        if (!node) return 0;

        int leftH = height(node->left);
        int rightH = height(node->right);

        // Update maximum diameter passing through this node
        maxDiameter = max(maxDiameter, leftH + rightH);

        // Return height of current node
        return 1 + max(leftH, rightH);
    }

public:
    int diameterOfBinaryTree(TreeNode* root) {
        height(root);
        return maxDiameter;
    }
};
// Time Complexity: O(N)
// Space Complexity: O(H)
```

---

*<- [[DP_on_Trees_Tree_DP|Tree DP Foundations]] · [[../07_Partition_DP/07_Partition_DP_Index|Partition DP ->]]*

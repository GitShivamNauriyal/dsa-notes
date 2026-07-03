---
tags: [dsa, trees, problems, exercises]
links: ["[[07_Trees_Index]]", "[[07_Trees_Patterns]]", "[[07_Trees_Tricky]]"]
---

# Trees -- Problems & Exercises

*<- [[07_Trees_Patterns\|Patterns]] · [[07_Trees_Tricky\|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Invert Binary Tree | Easy | Swap left/right | [LC 226](https://leetcode.com/problems/invert-binary-tree/) |
| 2 | Maximum Depth | Easy | DFS height | [LC 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) |
| 3 | Symmetric Tree | Easy | Mirror check | [LC 101](https://leetcode.com/problems/symmetric-tree/) |
| 4 | Same Tree | Easy | Structural equality | [LC 100](https://leetcode.com/problems/same-tree/) |
| 5 | Subtree of Another Tree | Easy | Match at every node | [LC 572](https://leetcode.com/problems/subtree-of-another-tree/) |
| 6 | Level Order Traversal | Medium | BFS + level size | [LC 102](https://leetcode.com/problems/binary-tree-level-order-traversal/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 7 | Validate BST | Medium | Bounds propagation | [LC 98](https://leetcode.com/problems/validate-binary-search-tree/) |
| 8 | Kth Smallest in BST | Medium | Inorder + count | [LC 230](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) |
| 9 | LCA of BST | Medium | BST property | [LC 235](https://leetcode.com/problems/lowest-common-ancestor-of-a-bst/) |
| 10 | LCA of Binary Tree | Medium | DFS return pattern | [LC 236](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) |
| 11 | Path Sum | Easy | DFS, leaf check | [LC 112](https://leetcode.com/problems/path-sum/) |
| 12 | Path Sum II | Medium | DFS + backtrack | [LC 113](https://leetcode.com/problems/path-sum-ii/) |
| 13 | Diameter of Binary Tree | Easy | Height + global max | [LC 543](https://leetcode.com/problems/diameter-of-binary-tree/) |
| 14 | Balanced Binary Tree | Easy | Height O(n) | [LC 110](https://leetcode.com/problems/balanced-binary-tree/) |
| 15 | Right Side View | Medium | BFS last per level | [LC 199](https://leetcode.com/problems/binary-tree-right-side-view/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 16 | Construct from Preorder+Inorder | Medium | Divide by root | [LC 105](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) |
| 17 | Construct from Inorder+Postorder | Medium | Last elem = root | [LC 106](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) |
| 18 | Serialize and Deserialize | Hard | Preorder + nulls | [LC 297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) |
| 19 | Binary Tree Maximum Path Sum | Hard | DFS gain upward | [LC 124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) |
| 20 | Path Sum III | Medium | Prefix sum on path | [LC 437](https://leetcode.com/problems/path-sum-iii/) |
| 21 | Sorted Array to BST | Easy | Midpoint as root | [LC 108](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) |
| 22 | Vertical Order Traversal | Hard | BFS + col tracking | [LC 987](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/) |
| 23 | Zigzag Level Order | Medium | BFS + direction | [LC 103](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 24 | Binary Tree Cameras | Hard | Greedy DFS states | [LC 968](https://leetcode.com/problems/binary-tree-cameras/) |
| 25 | Recover BST | Hard | Inorder + two wrong nodes | [LC 99](https://leetcode.com/problems/recover-binary-search-tree/) |
| 26 | Count Complete Tree Nodes | Medium | Binary search + height | [LC 222](https://leetcode.com/problems/count-complete-tree-nodes/) |
| 27 | BST Iterator | Medium | Controlled inorder | [LC 173](https://leetcode.com/problems/binary-search-tree-iterator/) |
| 28 | All Nodes at Distance K | Medium | DFS + parent map | [LC 863](https://leetcode.com/problems/all-nodes-at-distance-k-in-binary-tree/) |

---

## Worked Solution: LC 222 -- Count Complete Tree Nodes

**Why clever**: Complete tree has all levels full except possibly last, which is filled left-to-right. Use heights of left-most and right-most paths. If equal, tree is perfect (2^h - 1 nodes). If not, recurse on both halves.

```cpp
int countNodes(TreeNode* root) {
    if (!root) return 0;

    // Check heights of leftmost and rightmost paths
    int leftH = 0, rightH = 0;
    TreeNode* l = root, *r = root;
    while (l) { leftH++;  l = l->left; }
    while (r) { rightH++; r = r->right; }

    if (leftH == rightH) {
        return (1 << leftH) - 1;  // perfect binary tree: 2^h - 1 nodes
    }
    // Not perfect: recurse on subtrees
    return 1 + countNodes(root->left) + countNodes(root->right);
}
// Time: O(log²n) -- O(log n) levels, each costs O(log n) height computation
```

---

*<- [[07_Trees_Patterns\|Patterns]] · [[07_Trees_Tricky\|Tricky ->]]*

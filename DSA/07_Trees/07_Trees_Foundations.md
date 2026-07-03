---
tags: [dsa, trees, traversal, bfs, dfs, foundations]
links: ["[[07_Trees_Index]]", "[[07_Trees_BST]]"]
---

# Trees -- Foundations & Traversals

*<- [[07_Trees_Index\|Index]] · [[07_Trees_BST\|BST ->]]*

---

## What is a Tree?

A tree is a connected, acyclic graph. In DSA, we work mostly with **rooted binary trees**: each node has at most 2 children (left and right) and exactly one parent (except root).

```
        1          <- root (depth 0)
       / \
      2   3        <- depth 1
     / \   \
    4   5   6      <- depth 2 (leaves: no children)

Height of tree = 2 (longest path from root to leaf)
Height of node = longest path from that node to a leaf
Depth of node  = distance from root to that node
```

---

## 1. Node Structure

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode() : val(0), left(nullptr), right(nullptr) {}
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
    TreeNode(int x, TreeNode* left, TreeNode* right) : val(x), left(left), right(right) {}
};
```

---

## 2. DFS Traversals (Recursive)

The three DFS traversals differ only in WHEN you process the current node relative to its children.

```
Tree:    1
        / \
       2   3
      / \
     4   5

Inorder   (L, Root, R): 4, 2, 5, 1, 3   <- for BST: gives sorted order
Preorder  (Root, L, R): 1, 2, 4, 5, 3   <- useful to reconstruct tree
Postorder (L, R, Root): 4, 5, 2, 3, 1   <- useful for deletion, evaluating subtrees
```

```cpp
// Inorder: Left -> Root -> Right
void inorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    inorder(root->left, result);
    result.push_back(root->val);   // process ROOT between children
    inorder(root->right, result);
}

// Preorder: Root -> Left -> Right
void preorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    result.push_back(root->val);   // process ROOT before children
    preorder(root->left, result);
    preorder(root->right, result);
}

// Postorder: Left -> Right -> Root
void postorder(TreeNode* root, vector<int>& result) {
    if (!root) return;
    postorder(root->left, result);
    postorder(root->right, result);
    result.push_back(root->val);   // process ROOT after children
}
// All: Time O(n), Space O(h) where h = tree height (stack frames)
```

---

## 3. DFS Traversals (Iterative)

**Why iterative**: Avoids stack overflow for very deep trees. Also asked in interviews to test understanding of the call stack.

```cpp
// Iterative Inorder (most commonly asked)
vector<int> inorderIterative(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    TreeNode* cur = root;

    while (cur || !st.empty()) {
        // Go as far left as possible, pushing to stack
        while (cur) {
            st.push(cur);
            cur = cur->left;
        }
        // Process and move right
        cur = st.top(); st.pop();
        result.push_back(cur->val);
        cur = cur->right;
    }
    return result;
}

// Iterative Preorder
vector<int> preorderIterative(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    stack<TreeNode*> st;
    st.push(root);

    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        result.push_back(node->val);      // process node
        // Push right first so left is processed first (LIFO)
        if (node->right) st.push(node->right);
        if (node->left)  st.push(node->left);
    }
    return result;
}

// Iterative Postorder (tricky -- use reverse of modified preorder)
vector<int> postorderIterative(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    stack<TreeNode*> st;
    st.push(root);

    while (!st.empty()) {
        TreeNode* node = st.top(); st.pop();
        result.push_back(node->val);
        // Push left before right (so right processed first, then reverse)
        if (node->left)  st.push(node->left);
        if (node->right) st.push(node->right);
    }
    // Reverse: Root->Right->Left becomes Left->Right->Root = postorder
    reverse(result.begin(), result.end());
    return result;
}
```

---

## 4. BFS -- Level Order Traversal

**Use when**: You need to process nodes level by level. Use a queue.

```cpp
#include <queue>

// LC 102 -- Binary Tree Level Order Traversal
vector<vector<int>> levelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    queue<TreeNode*> q;
    q.push(root);

    while (!q.empty()) {
        int levelSize = q.size();  // number of nodes at this level
        vector<int> level;

        for (int i = 0; i < levelSize; i++) {
            TreeNode* node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        result.push_back(level);
    }
    return result;
}
// Time: O(n), Space: O(w) where w = max width of tree (at most n/2 for last level)

// Variations built on level-order:
// -- Right side view: take last element of each level
// -- Left side view: take first element of each level
// -- Zigzag order: alternate direction each level (use deque or reverse alternate levels)
// -- Find level of a node: count levels in BFS
```

### Zigzag Level Order (LC 103)

```cpp
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    vector<vector<int>> result;
    if (!root) return result;
    queue<TreeNode*> q;
    q.push(root);
    bool leftToRight = true;

    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            // Fill from left or right based on direction
            int idx = leftToRight ? i : sz - 1 - i;
            level[idx] = node->val;
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        result.push_back(level);
        leftToRight = !leftToRight;
    }
    return result;
}
```

---

## 5. Height, Depth, Diameter

```cpp
// Height of tree (longest path from root to any leaf)
int height(TreeNode* root) {
    if (!root) return 0;
    return 1 + max(height(root->left), height(root->right));
}

// Check if tree is balanced (height difference of subtrees <= 1 at every node)
// Naive: O(n²) -- compute height at each node separately
// Optimal: O(n) -- compute height bottom-up, return -1 if unbalanced
int heightBalanced(TreeNode* root) {
    if (!root) return 0;
    int left = heightBalanced(root->left);
    if (left == -1) return -1;
    int right = heightBalanced(root->right);
    if (right == -1) return -1;
    if (abs(left - right) > 1) return -1;  // mark as unbalanced
    return 1 + max(left, right);
}
bool isBalanced(TreeNode* root) { return heightBalanced(root) != -1; }

// Diameter of binary tree (LC 543)
// Diameter through a node = left_height + right_height
// Try all nodes as the "bend point"
int diameterHelper(TreeNode* root, int& maxDiam) {
    if (!root) return 0;
    int left = diameterHelper(root->left, maxDiam);
    int right = diameterHelper(root->right, maxDiam);
    maxDiam = max(maxDiam, left + right);  // path through this node
    return 1 + max(left, right);           // height contributed upward
}
int diameterOfBinaryTree(TreeNode* root) {
    int maxDiam = 0;
    diameterHelper(root, maxDiam);
    return maxDiam;
}
// Time: O(n), Space: O(h)
```

---

*<- [[07_Trees_Index\|Index]] · [[07_Trees_BST\|BST ->]]*

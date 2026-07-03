---
tags: [dsa, trees, tricky, hard, morris-traversal, segment-tree]
links: ["[[07_Trees_Index]]", "[[07_Trees_Problems_and_Exercises]]", "[[../08_Tries/08_Tries_Index]]"]
---

# Trees -- Tricky & Higher-Order

*<- [[07_Trees_Problems_and_Exercises\|Problems]] · [[../08_Tries/08_Tries_Index\|Tries ->]]*

---

## 1. Morris Traversal -- O(1) Space Inorder

**Why tricky**: All recursive and iterative traversals use O(h) space (stack frames or explicit stack). Morris traversal uses the tree's own NULL pointers as temporary links (threads) to achieve O(1) space.

**How**: Before visiting a left subtree, find the inorder predecessor (rightmost node of left subtree) and make its right pointer point back to the current node (thread). After visiting left subtree, restore the pointer.

```
Tree:       1
           / \
          2   3
         / \
        4   5

Step 1: cur=1, left exists. Predecessor of 1 = rightmost of left subtree = 5.
        Set 5->right = 1 (thread). Go left: cur=2.

Step 2: cur=2, left exists. Predecessor of 2 = 4.
        Set 4->right = 2 (thread). Go left: cur=4.

Step 3: cur=4, no left. VISIT 4. Go right: cur = 4->right = 2 (thread!).

Step 4: cur=2, left exists. Predecessor = 4. 4->right already == 2 (thread exists).
        VISIT 2. Remove thread: 4->right = null. Go right: cur=5.

Step 5: cur=5, no left. VISIT 5. Go right: cur = 5->right = 1 (thread!).

Step 6: cur=1, left exists. Predecessor = 5. 5->right == 1 (thread exists).
        VISIT 1. Remove thread: 5->right = null. Go right: cur=3.

Step 7: cur=3, no left. VISIT 3. Done.
Output: 4, 2, 5, 1, 3 ✓
```

```cpp
vector<int> morrisInorder(TreeNode* root) {
    vector<int> result;
    TreeNode* cur = root;

    while (cur) {
        if (!cur->left) {
            result.push_back(cur->val);  // no left subtree: visit and go right
            cur = cur->right;
        } else {
            // Find inorder predecessor (rightmost of left subtree)
            TreeNode* pred = cur->left;
            while (pred->right && pred->right != cur) pred = pred->right;

            if (!pred->right) {
                pred->right = cur;  // create thread
                cur = cur->left;    // go left
            } else {
                pred->right = nullptr;           // remove thread (restore tree)
                result.push_back(cur->val);      // visit current
                cur = cur->right;
            }
        }
    }
    return result;
}
// Time: O(n) -- each node visited at most twice (once for threading, once for visit)
// Space: O(1) -- modifies tree temporarily but restores it
```

---

## 2. Recover BST -- Two Swapped Nodes (LC 99)

**Why tricky**: Exactly two nodes in a BST were swapped. Find and fix them without extra space (O(1) besides recursion).

**Key insight**: Inorder traversal of a BST is sorted. Find where the order is violated. There are two cases:
- Adjacent swap: one inversion `[...a, b...]` where `a > b`. First = a, second = b.
- Non-adjacent swap: two inversions `[...a, b... c, d...]`. First = a, second = d.

```cpp
TreeNode* first = nullptr, *second = nullptr, *prev = nullptr;

void inorderRecover(TreeNode* root) {
    if (!root) return;
    inorderRecover(root->left);

    if (prev && prev->val > root->val) {
        if (!first) first = prev;  // first inversion: mark prev as first wrong node
        second = root;             // always update second (handles non-adjacent case)
    }
    prev = root;
    inorderRecover(root->right);
}

void recoverTree(TreeNode* root) {
    inorderRecover(root);
    swap(first->val, second->val);
}
// Time: O(n), Space: O(h) -- use Morris traversal for true O(1)
```

---

## 3. Convert BST to Greater Tree (LC 538)

**Key insight**: Reverse inorder (Right -> Root -> Left) gives descending order for BST. Accumulate sum while doing reverse inorder.

```cpp
void reverseInorder(TreeNode* root, int& runSum) {
    if (!root) return;
    reverseInorder(root->right, runSum);
    runSum += root->val;
    root->val = runSum;
    reverseInorder(root->left, runSum);
}
TreeNode* convertBST(TreeNode* root) {
    int sum = 0;
    reverseInorder(root, sum);
    return root;
}
```

---

## 4. Binary Tree to Doubly Linked List (In-Place)

**Why tricky**: Convert BST to a sorted circular doubly linked list using inorder traversal. Left = prev, right = next. No extra space.

```cpp
// Used in: LC 426 (Premium) -- Convert BST to Sorted Circular Doubly Linked List
TreeNode* head = nullptr, *prevNode = nullptr;

void inorderToDLL(TreeNode* root) {
    if (!root) return;
    inorderToDLL(root->left);

    if (!prevNode) {
        head = root;  // first node = head
    } else {
        prevNode->right = root;  // prev->next = cur
        root->left = prevNode;   // cur->prev = prev
    }
    prevNode = root;
    inorderToDLL(root->right);
}

TreeNode* treeToDoublyList(TreeNode* root) {
    if (!root) return nullptr;
    inorderToDLL(root);
    // Make circular
    head->left = prevNode;
    prevNode->right = head;
    return head;
}
```

---

## 5. Count Nodes at Distance K from Target (LC 863)

**Why tricky**: Distance can go UP through the parent, not just down. Standard DFS can't do this directly. Build a parent map first, then BFS from target.

```cpp
int distanceK(TreeNode* root, TreeNode* target, int k) {
    // Build parent map via BFS
    unordered_map<TreeNode*, TreeNode*> parent;
    queue<TreeNode*> q;
    q.push(root);
    parent[root] = nullptr;
    while (!q.empty()) {
        TreeNode* node = q.front(); q.pop();
        if (node->left)  { parent[node->left]  = node; q.push(node->left); }
        if (node->right) { parent[node->right] = node; q.push(node->right); }
    }

    // BFS from target, distance k
    unordered_set<TreeNode*> visited;
    q.push(target);
    visited.insert(target);
    int dist = 0;

    while (!q.empty()) {
        if (dist == k) {
            vector<int> result;
            while (!q.empty()) { result.push_back(q.front()->val); q.pop(); }
            return result.size();  // return count
        }
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            for (TreeNode* neighbor : {node->left, node->right, parent[node]}) {
                if (neighbor && !visited.count(neighbor)) {
                    visited.insert(neighbor);
                    q.push(neighbor);
                }
            }
        }
        dist++;
    }
    return 0;
}
// Time: O(n), Space: O(n)
```

---

## 6. Segment Tree (Range Queries with Point Updates)

**Why critical for interviews at top companies and quant roles**: O(log n) range min/max/sum queries AND updates. Standard prefix sum only handles static arrays.

```cpp
// Segment tree for range sum queries with point updates
class SegmentTree {
    int n;
    vector<int> tree;

    void build(vector<int>& arr, int node, int lo, int hi) {
        if (lo == hi) { tree[node] = arr[lo]; return; }
        int mid = lo + (hi - lo) / 2;
        build(arr, 2*node, lo, mid);
        build(arr, 2*node+1, mid+1, hi);
        tree[node] = tree[2*node] + tree[2*node+1];
    }

    void update(int node, int lo, int hi, int idx, int val) {
        if (lo == hi) { tree[node] = val; return; }
        int mid = lo + (hi - lo) / 2;
        if (idx <= mid) update(2*node, lo, mid, idx, val);
        else            update(2*node+1, mid+1, hi, idx, val);
        tree[node] = tree[2*node] + tree[2*node+1];
    }

    int query(int node, int lo, int hi, int l, int r) {
        if (r < lo || hi < l) return 0;          // out of range
        if (l <= lo && hi <= r) return tree[node]; // fully in range
        int mid = lo + (hi - lo) / 2;
        return query(2*node, lo, mid, l, r)
             + query(2*node+1, mid+1, hi, l, r);
    }

public:
    SegmentTree(vector<int>& arr) : n(arr.size()), tree(4 * arr.size(), 0) {
        build(arr, 1, 0, n-1);
    }
    void update(int idx, int val) { update(1, 0, n-1, idx, val); }
    int query(int l, int r) { return query(1, 0, n-1, l, r); }
};
// Build: O(n), Update: O(log n), Query: O(log n), Space: O(n)
// 4*n space because tree is stored in 1-indexed array, children at 2i and 2i+1
```

---

## 7. Binary Lifting -- LCA in O(log n) After O(n log n) Preprocessing

**When**: Many LCA queries on the same tree. Each query O(n) via DFS is too slow.

```cpp
const int MAXN = 1e5 + 5, LOG = 17;
int depth[MAXN], up[MAXN][LOG];  // up[v][j] = 2^j-th ancestor of v

void dfs(int v, int par, int d, vector<vector<int>>& adj) {
    depth[v] = d;
    up[v][0] = par;
    for (int j = 1; j < LOG; j++)
        up[v][j] = up[up[v][j-1]][j-1];  // 2^j ancestor = 2^(j-1) of 2^(j-1)
    for (int u : adj[v]) if (u != par) dfs(u, v, d+1, adj);
}

int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    int diff = depth[u] - depth[v];
    // Lift u to same depth as v
    for (int j = 0; j < LOG; j++) if ((diff >> j) & 1) u = up[u][j];
    if (u == v) return u;
    // Lift both until their parents are the same
    for (int j = LOG-1; j >= 0; j--)
        if (up[u][j] != up[v][j]) { u = up[u][j]; v = up[v][j]; }
    return up[u][0];
}
// Preprocessing: O(n log n), each LCA query: O(log n)
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Recover BST | Hard | [LC 99](https://leetcode.com/problems/recover-binary-search-tree/) |
| 2 | Convert BST to Greater Tree | Medium | [LC 538](https://leetcode.com/problems/convert-bst-to-greater-tree/) |
| 3 | All Nodes at Distance K | Medium | [LC 863](https://leetcode.com/problems/all-nodes-at-distance-k-in-binary-tree/) |
| 4 | Binary Tree Cameras | Hard | [LC 968](https://leetcode.com/problems/binary-tree-cameras/) |
| 5 | Find Duplicate Subtrees | Medium | [LC 652](https://leetcode.com/problems/find-duplicate-subtrees/) |
| 6 | House Robber III | Medium | [LC 337](https://leetcode.com/problems/house-robber-iii/) |
| 7 | Range Sum Query -- Mutable | Medium | [LC 307](https://leetcode.com/problems/range-sum-query-mutable/) |
| 8 | Count of Smaller Numbers After Self | Hard | [LC 315](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) |

---

*<- [[07_Trees_Problems_and_Exercises\|Problems]] · [[../08_Tries/08_Tries_Index\|Tries ->]]*

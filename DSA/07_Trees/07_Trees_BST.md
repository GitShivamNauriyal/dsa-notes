---
tags: [dsa, trees, bst, binary-search-tree]
links: ["[[07_Trees_Index]]", "[[07_Trees_Foundations]]", "[[07_Trees_Patterns]]"]
---

# Binary Search Tree (BST)

*<- [[07_Trees_Foundations|Foundations]] · [[07_Trees_Patterns|Patterns ->]]*

---

## What Makes a Tree a BST?

**BST Property**: For every node:
- ALL nodes in left subtree have values **less than** node.val
- ALL nodes in right subtree have values **greater than** node.val

This property holds recursively for every subtree.

```
       8             Valid BST
      / \
     3   10
    / \    \
   1   6    14
      / \
     4   7
```

Key consequence: **inorder traversal of a BST gives a sorted sequence**.

---

## 1. Search in BST

```cpp
// If target < node.val, go left. If target > node.val, go right.
TreeNode* searchBST(TreeNode* root, int val) {
    if (!root || root->val == val) return root;
    if (val < root->val) return searchBST(root->left, val);
    return searchBST(root->right, val);
}
// Iterative version (O(1) space)
TreeNode* searchBSTIter(TreeNode* root, int val) {
    while (root && root->val != val) {
        root = (val < root->val) ? root->left : root->right;
    }
    return root;
}
// Time: O(h) where h = height. O(log n) for balanced, O(n) worst case (skewed)
```

---

## 2. Insert into BST

```cpp
// Find the correct leaf position and insert
TreeNode* insertIntoBST(TreeNode* root, int val) {
    if (!root) return new TreeNode(val);  // found the spot
    if (val < root->val)
        root->left = insertIntoBST(root->left, val);
    else
        root->right = insertIntoBST(root->right, val);
    return root;
}
// Time: O(h), Space: O(h) recursion stack
```

---

## 3. Delete from BST

**Three cases**:
1. Node is a **leaf** (no children): simply remove it.
2. Node has **one child**: replace node with its child.
3. Node has **two children**: replace node's value with its **inorder successor** (smallest in right subtree), then delete the inorder successor.

```
Delete 5 from:          After deletion:
       8                      8
      / \                    / \
     5   10                 6   10
    / \    \               / \    \
   3   6    14            3   7    14
        \
         7

Step 1: 5 has two children
Step 2: Inorder successor of 5 = leftmost in right subtree = 6
Step 3: Replace 5's value with 6
Step 4: Delete 6 from right subtree (6 has one child: 7, so easy)
```

```cpp
// Find leftmost (minimum) node in a subtree
TreeNode* findMin(TreeNode* root) {
    while (root->left) root = root->left;
    return root;
}

TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;

    if (key < root->val) {
        root->left = deleteNode(root->left, key);
    } else if (key > root->val) {
        root->right = deleteNode(root->right, key);
    } else {
        // Found the node to delete
        if (!root->left) {
            TreeNode* temp = root->right;
            delete root;
            return temp;   // Case 1 or 2: no left child
        } else if (!root->right) {
            TreeNode* temp = root->left;
            delete root;
            return temp;   // Case 2: no right child
        }
        // Case 3: two children
        TreeNode* successor = findMin(root->right);  // inorder successor
        root->val = successor->val;                  // overwrite value
        root->right = deleteNode(root->right, successor->val);  // delete successor
    }
    return root;
}
// Time: O(h), Space: O(h)
```

---

## 4. Validate BST (LC 98)

**Why naive check fails**: Checking `root->left->val < root->val` only checks immediate children, not the entire subtree.

```
       5
      / \
     1   4        <-- 4 < 5, but 4's right child is 6 > 5!
        / \           This violates BST property for node 5.
       3   6
```

**Correct approach**: Pass down `[min_valid, max_valid]` bounds for each subtree.

```cpp
bool validate(TreeNode* root, long long minVal, long long maxVal) {
    if (!root) return true;
    if (root->val <= minVal || root->val >= maxVal) return false;
    return validate(root->left, minVal, root->val)   // left must be < root->val
        && validate(root->right, root->val, maxVal); // right must be > root->val
}

bool isValidBST(TreeNode* root) {
    return validate(root, LLONG_MIN, LLONG_MAX);
    // Use LLONG_MIN/MAX to handle edge cases where root->val = INT_MIN or INT_MAX
}
// Time: O(n), Space: O(h)
```

---

## 5. Kth Smallest Element in BST (LC 230)

```cpp
// Inorder traversal gives sorted order -- stop at k-th element
void inorderKth(TreeNode* root, int& k, int& result) {
    if (!root) return;
    inorderKth(root->left, k, result);
    k--;
    if (k == 0) { result = root->val; return; }
    inorderKth(root->right, k, result);
}

int kthSmallest(TreeNode* root, int k) {
    int result = -1;
    inorderKth(root, k, result);
    return result;
}
// Time: O(h + k), Space: O(h)
```

---

## 6. Lowest Common Ancestor in BST (LC 235)

**Use BST property**: If both p and q are smaller than root, LCA is in left subtree. If both larger, in right. Otherwise root is LCA.

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) {
            root = root->left;   // both in left subtree
        } else if (p->val > root->val && q->val > root->val) {
            root = root->right;  // both in right subtree
        } else {
            return root;  // split point = LCA
        }
    }
    return nullptr;
}
// Time: O(h), Space: O(1)
```

---

## 7. Convert Sorted Array to Balanced BST (LC 108)

```cpp
// Binary search style: always pick the middle as root for balance
TreeNode* sortedArrayToBST(std::vector<int>& nums, int lo, int hi) {
    if (lo > hi) return nullptr;
    int mid = lo + (hi - lo) / 2;
    TreeNode* root = new TreeNode(nums[mid]);
    root->left  = sortedArrayToBST(nums, lo, mid - 1);
    root->right = sortedArrayToBST(nums, mid + 1, hi);
    return root;
}
TreeNode* sortedArrayToBST(std::vector<int>& nums) {
    return sortedArrayToBST(nums, 0, (int)nums.size() - 1);
}
```

---

## 8. Inorder Successor and Predecessor

```cpp
// Inorder successor: smallest value greater than node.val
TreeNode* inorderSuccessor(TreeNode* root, TreeNode* p) {
    TreeNode* successor = nullptr;
    while (root) {
        if (p->val < root->val) {
            successor = root;       // root is a candidate
            root = root->left;      // look for smaller successor
        } else {
            root = root->right;     // p->val >= root->val, go right
        }
    }
    return successor;
}

// Inorder predecessor: largest value less than node.val
TreeNode* inorderPredecessor(TreeNode* root, TreeNode* p) {
    TreeNode* pred = nullptr;
    while (root) {
        if (p->val > root->val) {
            pred = root;
            root = root->right;
        } else {
            root = root->left;
        }
    }
    return pred;
}
// Both: Time O(h), Space O(1)
```

---

*<- [[07_Trees_Foundations|Foundations]] · [[07_Trees_Patterns|Patterns ->]]*

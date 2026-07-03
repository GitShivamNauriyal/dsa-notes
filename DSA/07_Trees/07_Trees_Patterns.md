---
tags: [dsa, trees, patterns, lca, path-sum, construct]
links: ["[[07_Trees_Index]]", "[[07_Trees_BST]]", "[[07_Trees_Problems_and_Exercises]]"]
---

# Trees -- Patterns

*<- [[07_Trees_BST|BST]] · [[07_Trees_Problems_and_Exercises|Problems ->]]*

---

## Pattern 1: Lowest Common Ancestor -- General Binary Tree (LC 236)

**Why different from BST**: No ordering property to guide the search. Must use DFS and check subtrees.

**Key insight**: If we find p or q, return it. If both sides return non-null from a node, that node is the LCA.

```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    // Found p or q: stop and return it upward

    TreeNode* left  = lowestCommonAncestor(root->left, p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);

    if (left && right) return root;  // p in left subtree, q in right -> root is LCA
    return left ? left : right;      // one side found both (one is ancestor of other)
}
// Time: O(n), Space: O(h)
```

---

## Pattern 2: Path Sum Problems

### Path Sum I -- Does a root-to-leaf path exist with given sum? (LC 112)

```cpp
bool hasPathSum(TreeNode* root, int targetSum) {
    if (!root) return false;
    if (!root->left && !root->right) return root->val == targetSum;  // leaf
    return hasPathSum(root->left,  targetSum - root->val)
        || hasPathSum(root->right, targetSum - root->val);
}
```

### Path Sum II -- Find all root-to-leaf paths with given sum (LC 113)

```cpp
void dfs(TreeNode* node, int remain, vector<int>& path,
         vector<vector<int>>& result) {
    if (!node) return;
    path.push_back(node->val);
    if (!node->left && !node->right && remain == node->val) {
        result.push_back(path);  // valid path found
    }
    dfs(node->left,  remain - node->val, path, result);
    dfs(node->right, remain - node->val, path, result);
    path.pop_back();  // backtrack
}

vector<vector<int>> pathSum(TreeNode* root, int targetSum) {
    vector<vector<int>> result;
    vector<int> path;
    dfs(root, targetSum, path, result);
    return result;
}
```

### Path Sum III -- Count paths (any start/end) with given sum (LC 437)

**Key insight**: Use prefix sum on the path from root to current node. If `prefixSum[current] - target` was seen before, there's a valid subpath.

```cpp
int pathSumIII(TreeNode* root, long long targetSum) {
    unordered_map<long long, int> prefixCount;
    prefixCount[0] = 1;  // empty path has sum 0
    int result = 0;

    function<void(TreeNode*, long long)> dfs = [&](TreeNode* node, long long cur) {
        if (!node) return;
        cur += node->val;
        result += prefixCount[cur - targetSum];  // paths ending here with sum = target
        prefixCount[cur]++;
        dfs(node->left, cur);
        dfs(node->right, cur);
        prefixCount[cur]--;  // backtrack -- remove this path's contribution
    };

    dfs(root, 0);
    return result;
}
// Time: O(n), Space: O(n)
// function used here for concise lambda recursion -- slight overhead vs named function
```

### Maximum Path Sum (LC 124) -- Any Node to Any Node

```cpp
int maxPathHelper(TreeNode* root, int& globalMax) {
    if (!root) return 0;
    // Only include a child's contribution if it's positive
    int left  = max(0, maxPathHelper(root->left, globalMax));
    int right = max(0, maxPathHelper(root->right, globalMax));

    // Path through this node = left + root->val + right
    globalMax = max(globalMax, left + root->val + right);

    // Return max gain if we continue upward (can only pick one branch)
    return root->val + max(left, right);
}

int maxPathSum(TreeNode* root) {
    int globalMax = INT_MIN;
    maxPathHelper(root, globalMax);
    return globalMax;
}
// Time: O(n), Space: O(h)
```

---

## Pattern 3: Construct Tree from Traversals

### Construct from Preorder + Inorder (LC 105)

**Why it works**:
- Preorder[0] is always the root.
- Find root in inorder -- everything left of it is the left subtree, right is right subtree.
- Use the sizes to split preorder accordingly.

```cpp
TreeNode* build(vector<int>& pre, int preL, int preR,
                vector<int>& in,  int inL,  int inR,
                unordered_map<int,int>& inMap) {
    if (preL > preR) return nullptr;
    TreeNode* root = new TreeNode(pre[preL]);
    int mid = inMap[pre[preL]];     // root's position in inorder
    int leftSize = mid - inL;       // number of nodes in left subtree

    root->left  = build(pre, preL + 1, preL + leftSize,
                        in,  inL,      mid - 1, inMap);
    root->right = build(pre, preL + leftSize + 1, preR,
                        in,  mid + 1,  inR, inMap);
    return root;
}

TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    unordered_map<int,int> inMap;
    for (int i = 0; i < (int)inorder.size(); i++) inMap[inorder[i]] = i;
    return build(preorder, 0, preorder.size()-1, inorder, 0, inorder.size()-1, inMap);
}
// Time: O(n), Space: O(n) for map
```

### Construct from Inorder + Postorder (LC 106)

```cpp
// Postorder's last element is always the root -- mirror of above
TreeNode* buildFromPost(vector<int>& in, int inL, int inR,
                        vector<int>& post, int postL, int postR,
                        unordered_map<int,int>& inMap) {
    if (postL > postR) return nullptr;
    TreeNode* root = new TreeNode(post[postR]);  // last element = root
    int mid = inMap[post[postR]];
    int leftSize = mid - inL;

    root->left  = buildFromPost(in, inL, mid-1, post, postL, postL+leftSize-1, inMap);
    root->right = buildFromPost(in, mid+1, inR, post, postL+leftSize, postR-1, inMap);
    return root;
}
```

---

## Pattern 4: Tree Views

```cpp
// Right Side View (LC 199): last node at each level in BFS
vector<int> rightSideView(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            if (i == sz - 1) result.push_back(node->val);  // last of level
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return result;
}

// Vertical Order Traversal (LC 987): BFS with column tracking
// For each node, track (row, col). col: left child = col-1, right = col+1.
vector<vector<int>> verticalTraversal(TreeNode* root) {
    // {col -> {row -> [vals]}}
    map<int, map<int, multiset<int>>> data;
    queue<tuple<TreeNode*,int,int>> q;  // {node, row, col}
    q.push({root, 0, 0});
    while (!q.empty()) {
        auto [node, row, col] = q.front(); q.pop();
        data[col][row].insert(node->val);
        if (node->left)  q.push({node->left,  row+1, col-1});
        if (node->right) q.push({node->right, row+1, col+1});
    }
    vector<vector<int>> result;
    for (auto& [col, rows] : data) {
        vector<int> colVals;
        for (auto& [row, vals] : rows)
            for (int v : vals) colVals.push_back(v);
        result.push_back(colVals);
    }
    return result;
}
```

---

## Pattern 5: Serialize and Deserialize Binary Tree (LC 297)

```cpp
// Encode tree to string, decode back. Use preorder with null markers.
string serialize(TreeNode* root) {
    if (!root) return "null,";
    return to_string(root->val) + "," +
           serialize(root->left) + serialize(root->right);
}

TreeNode* deserializeHelper(queue<string>& tokens) {
    string token = tokens.front(); tokens.pop();
    if (token == "null") return nullptr;
    TreeNode* node = new TreeNode(stoi(token));
    node->left  = deserializeHelper(tokens);
    node->right = deserializeHelper(tokens);
    return node;
}

TreeNode* deserialize(string data) {
    queue<string> tokens;
    stringstream ss(data);
    string token;
    while (getline(ss, token, ',')) tokens.push(token);
    return deserializeHelper(tokens);
}
```

---

## Pattern 6: Symmetric / Mirror Tree

```cpp
// LC 101 -- Symmetric Tree
bool isMirror(TreeNode* l, TreeNode* r) {
    if (!l && !r) return true;
    if (!l || !r) return false;
    return (l->val == r->val)
        && isMirror(l->left, r->right)
        && isMirror(l->right, r->left);
}
bool isSymmetric(TreeNode* root) {
    return isMirror(root->left, root->right);
}

// Invert Binary Tree (LC 226)
TreeNode* invertTree(TreeNode* root) {
    if (!root) return nullptr;
    swap(root->left, root->right);
    invertTree(root->left);
    invertTree(root->right);
    return root;
}
```

---

*<- [[07_Trees_BST|BST]] · [[07_Trees_Problems_and_Exercises|Problems ->]]*

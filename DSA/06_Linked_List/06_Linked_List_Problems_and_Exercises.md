---
tags: [dsa, linked-list, problems, exercises]
links: ["[[06_Linked_List_Index]]", "[[06_Linked_List_Patterns]]", "[[06_Linked_List_Tricky]]"]
---

# Linked List -- Problems & Exercises

*<- [[06_Linked_List_Patterns\|Patterns]] · [[06_Linked_List_Tricky\|Tricky ->]]*

---

## Tier 1 -- Foundations

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 1 | Reverse Linked List | Easy | Three-pointer reversal | [LC 206](https://leetcode.com/problems/reverse-linked-list/) |
| 2 | Merge Two Sorted Lists | Easy | Dummy head, merge | [LC 21](https://leetcode.com/problems/merge-two-sorted-lists/) |
| 3 | Linked List Cycle | Easy | Fast-slow detect | [LC 141](https://leetcode.com/problems/linked-list-cycle/) |
| 4 | Middle of Linked List | Easy | Fast-slow middle | [LC 876](https://leetcode.com/problems/middle-of-the-linked-list/) |
| 5 | Delete Nth Node from End | Medium | Two-pointer gap | [LC 19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) |

## Tier 2 -- Core Patterns

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 6 | Linked List Cycle II | Medium | Fast-slow entry | [LC 142](https://leetcode.com/problems/linked-list-cycle-ii/) |
| 7 | Palindrome Linked List | Easy | Middle + reverse + compare | [LC 234](https://leetcode.com/problems/palindrome-linked-list/) |
| 8 | Intersection of Two Lists | Easy | Two-pointer redirect | [LC 160](https://leetcode.com/problems/intersection-of-two-linked-lists/) |
| 9 | Reverse Linked List II | Medium | Front-insertion in range | [LC 92](https://leetcode.com/problems/reverse-linked-list-ii/) |
| 10 | Reorder List | Medium | Middle + reverse + merge | [LC 143](https://leetcode.com/problems/reorder-list/) |
| 11 | Remove Duplicates from Sorted List | Easy | Skip same-val nodes | [LC 83](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) |
| 12 | Remove Duplicates II (leave none) | Medium | Dummy + skip all dupes | [LC 82](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/) |

## Tier 3 -- Interview Level

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 13 | Sort List | Medium | Merge sort on list | [LC 148](https://leetcode.com/problems/sort-list/) |
| 14 | Merge K Sorted Lists | Hard | Min-heap | [LC 23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 15 | Reverse Nodes in K-Group | Hard | Reverse k + recurse | [LC 25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 16 | Copy List with Random Pointer | Medium | HashMap or interleaving | [LC 138](https://leetcode.com/problems/copy-list-with-random-pointer/) |
| 17 | Add Two Numbers | Medium | Carry simulation | [LC 2](https://leetcode.com/problems/add-two-numbers/) |
| 18 | Add Two Numbers II (no reverse) | Medium | Stack + add | [LC 445](https://leetcode.com/problems/add-two-numbers-ii/) |

## Tier 4 -- Placement-Hard

| # | Problem | Difficulty | Key Concept | Link |
|---|---------|------------|-------------|------|
| 19 | LRU Cache | Medium | Doubly LL + HashMap | [LC 146](https://leetcode.com/problems/lru-cache/) |
| 20 | LFU Cache | Hard | Two HashMaps + DLL | [LC 460](https://leetcode.com/problems/lfu-cache/) |
| 21 | Flatten a Multilevel Doubly LL | Medium | DFS / stack iteration | [LC 430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |
| 22 | Rotate List | Medium | Find new tail with cycle | [LC 61](https://leetcode.com/problems/rotate-list/) |

---

## Worked Solution: LC 138 -- Copy List with Random Pointer

**Why hard**: Each node has a `random` pointer to any node or null. Deep copy requires knowing the new copies when wiring random pointers.

```cpp
struct Node {
    int val;
    Node* next;
    Node* random;
    Node(int x) : val(x), next(nullptr), random(nullptr) {}
};

// Approach 1: HashMap -- O(n) space
Node* copyRandomList(Node* head) {
    if (!head) return nullptr;
    unordered_map<Node*, Node*> map;  // old -> new

    // Pass 1: create all new nodes
    Node* cur = head;
    while (cur) {
        map[cur] = new Node(cur->val);
        cur = cur->next;
    }
    // Pass 2: wire next and random pointers
    cur = head;
    while (cur) {
        map[cur]->next   = map[cur->next];    // null-safe: map[null] = null by default
        map[cur]->random = map[cur->random];
        cur = cur->next;
    }
    return map[head];
}

// Approach 2: O(1) space -- interleave original and copy nodes
Node* copyRandomListO1(Node* head) {
    if (!head) return nullptr;
    Node* cur = head;

    // Pass 1: interleave -- insert copy of each node right after it
    // Original: 1->2->3  becomes  1->1'->2->2'->3->3'
    while (cur) {
        Node* copy = new Node(cur->val);
        copy->next = cur->next;
        cur->next = copy;
        cur = copy->next;  // advance to next original
    }

    // Pass 2: set random pointers for copies
    cur = head;
    while (cur) {
        if (cur->random) cur->next->random = cur->random->next;
        cur = cur->next->next;
    }

    // Pass 3: separate the two lists
    cur = head;
    Node* copyHead = head->next;
    while (cur) {
        Node* copy = cur->next;
        cur->next = copy->next;
        if (copy->next) copy->next = copy->next->next;
        cur = cur->next;
    }
    return copyHead;
}
```

---

## Worked Solution: LC 146 -- LRU Cache

```cpp
// Doubly linked list + HashMap
// DLL: most recently used at front, least recently used at tail
// HashMap: key -> node (O(1) lookup)
// On get/put: move node to front (O(1) with DLL)

class LRUCache {
    struct Node {
        int key, val;
        Node* prev;
        Node* next;
        Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
    };

    int capacity;
    unordered_map<int, Node*> cache;  // key -> node
    Node* head;  // dummy head (most recent end)
    Node* tail;  // dummy tail (least recent end)

    void remove(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void insertFront(Node* node) {
        node->next = head->next;
        node->prev = head;
        head->next->prev = node;
        head->next = node;
    }

public:
    LRUCache(int capacity) : capacity(capacity) {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (!cache.count(key)) return -1;
        Node* node = cache[key];
        remove(node);
        insertFront(node);  // mark as recently used
        return node->val;
    }

    void put(int key, int value) {
        if (cache.count(key)) {
            remove(cache[key]);
            delete cache[key];
        }
        Node* node = new Node(key, value);
        insertFront(node);
        cache[key] = node;

        if ((int)cache.size() > capacity) {
            Node* lru = tail->prev;  // least recently used
            remove(lru);
            cache.erase(lru->key);
            delete lru;
        }
    }
};
// All operations: O(1)
```

---

*<- [[06_Linked_List_Patterns\|Patterns]] · [[06_Linked_List_Tricky\|Tricky ->]]*

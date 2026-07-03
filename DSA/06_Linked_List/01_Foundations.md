---
tags: [dsa, linked-list, foundations]
links: ["[[00_Index]]", "[[02_Patterns]]"]
---

# Linked List -- Foundations

*<- [[00_Index|Index]] · [[02_Patterns|Patterns ->]]*

---

## Why Linked Lists?

Arrays store elements in contiguous memory. Insertion/deletion in the middle requires shifting elements -- O(n). A linked list stores elements anywhere in memory and uses **pointers** to chain them. Insertion/deletion at a known position is O(1) (just update pointers), but access by index is O(n) (must walk the chain).

```
Array:   [1][2][3][4][5]   -- contiguous, random access O(1), insert O(n)
List:    [1]->[2]->[3]->[4]->[5]->null  -- scattered, access O(n), insert O(1)
```

---

## 1. Node Structure

```cpp
// Singly Linked List Node
struct ListNode {
    int val;
    ListNode* next;

    // Default constructor
    ListNode() : val(0), next(nullptr) {}

    // Value constructor
    ListNode(int x) : val(x), next(nullptr) {}

    // Value + next constructor (used in many LeetCode problems)
    ListNode(int x, ListNode* next) : val(x), next(next) {}
};

// Doubly Linked List Node
struct DListNode {
    int val;
    DListNode* prev;
    DListNode* next;
    DListNode(int x) : val(x), prev(nullptr), next(nullptr) {}
};
```

---

## 2. Building a Linked List

```cpp
// Build list from vector: 1 -> 2 -> 3 -> 4 -> 5
ListNode* buildList(std::vector<int>& vals) {
    if (vals.empty()) return nullptr;
    ListNode* dummy = new ListNode(0);  // dummy head simplifies edge cases
    ListNode* cur = dummy;
    for (int v : vals) {
        cur->next = new ListNode(v);
        cur = cur->next;
    }
    return dummy->next;  // actual head
}

// The DUMMY HEAD pattern:
// A dummy/sentinel node before the real head removes special cases for
// operations on the first node (deletion, insertion at head).
// dummy->next always points to the real head.
// After all operations: return dummy->next.
```

---

## 3. Traversal

```cpp
// Basic traversal
void printList(ListNode* head) {
    while (head != nullptr) {
        std::cout << head->val;
        if (head->next) std::cout << " -> ";
        head = head->next;
    }
    std::cout << "\n";
}

// Count nodes
int length(ListNode* head) {
    int count = 0;
    while (head) { count++; head = head->next; }
    return count;
}

// Access k-th node (0-indexed)
ListNode* getKth(ListNode* head, int k) {
    for (int i = 0; i < k && head; i++) head = head->next;
    return head;
}
```

---

## 4. Insertion

```cpp
// Insert at beginning -- O(1)
ListNode* insertAtHead(ListNode* head, int val) {
    ListNode* node = new ListNode(val);
    node->next = head;
    return node;  // new head
}

// Insert after a given node -- O(1) given the node
void insertAfter(ListNode* node, int val) {
    ListNode* newNode = new ListNode(val);
    newNode->next = node->next;
    node->next = newNode;
}

// Insert at position k (0-indexed) using dummy head
ListNode* insertAt(ListNode* head, int k, int val) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;
    for (int i = 0; i < k && prev->next; i++) prev = prev->next;
    ListNode* node = new ListNode(val);
    node->next = prev->next;
    prev->next = node;
    return dummy->next;
}
```

---

## 5. Deletion

```cpp
// Delete a node given its PREVIOUS node -- O(1)
void deleteAfter(ListNode* prev) {
    if (!prev->next) return;
    ListNode* toDelete = prev->next;
    prev->next = toDelete->next;
    delete toDelete;  // free memory
}

// Delete a node given ONLY itself (no head, no prev) -- O(1) trick
// ONLY works when it's NOT the last node (copy next's value, delete next)
void deleteNode(ListNode* node) {
    // Overwrite current node with next node's value, then delete next
    node->val = node->next->val;
    ListNode* toDelete = node->next;
    node->next = node->next->next;
    delete toDelete;
}
// Why: we can't "delete" ourselves without the previous pointer, but we
// can impersonate the next node by copying its value.

// Delete by value using dummy head -- handles deletion of first node cleanly
ListNode* deleteVal(ListNode* head, int val) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;
    while (prev->next) {
        if (prev->next->val == val) {
            ListNode* toDelete = prev->next;
            prev->next = prev->next->next;
            delete toDelete;
            // Don't advance prev -- there could be another occurrence
        } else {
            prev = prev->next;
        }
    }
    ListNode* newHead = dummy->next;
    delete dummy;
    return newHead;
}
```

---

## 6. Memory Management Note

In competitive programming / LeetCode, you typically don't `delete` nodes -- memory leaks are acceptable. In real code and system design interviews, always free memory.

```cpp
// Free entire list
void freeList(ListNode* head) {
    while (head) {
        ListNode* next = head->next;
        delete head;
        head = next;
    }
}
```

---

*<- [[00_Index|Index]] · [[02_Patterns|Patterns ->]]*

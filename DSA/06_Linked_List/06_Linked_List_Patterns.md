---
tags: [dsa, linked-list, patterns, reversal, fast-slow, merge]
links: ["[[06_Linked_List_Index]]", "[[06_Linked_List_Foundations]]", "[[06_Linked_List_Problems_and_Exercises]]"]
---

# Linked List -- Patterns

*<- [[06_Linked_List_Foundations\|Foundations]] · [[06_Linked_List_Problems_and_Exercises\|Problems ->]]*

---

## Pattern 1: Reversal

**Why a dedicated pattern**: Reversing a list requires careful pointer manipulation. Getting the order wrong corrupts the list permanently. Memorize the three-pointer dance.

```
Original: 1 -> 2 -> 3 -> 4 -> null
After:    4 -> 3 -> 2 -> 1 -> null

Three pointers: prev, cur, next
prev = null, cur = 1

Step 1: next = cur->next (save 2)
        cur->next = prev  (1 -> null)
        prev = cur        (prev = 1)
        cur = next        (cur = 2)

Step 2: next = 2->next (save 3)
        2->next = 1   (2 -> 1)
        prev = 2, cur = 3

...until cur = null. Return prev as new head.
```

```cpp
// LC 206 -- Reverse Linked List
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;

    while (cur) {
        ListNode* next = cur->next;  // 1. save next
        cur->next = prev;            // 2. reverse arrow
        prev = cur;                  // 3. advance prev
        cur = next;                  // 4. advance cur
    }
    return prev;  // prev is now the new head
}
// Time: O(n), Space: O(1)

// Recursive version (understand but prefer iterative in interviews)
ListNode* reverseListRecursive(ListNode* head) {
    if (!head || !head->next) return head;  // base case: 0 or 1 node
    ListNode* newHead = reverseListRecursive(head->next);
    // At this point, head->next is the LAST node of the reversed sublist
    head->next->next = head;  // make head->next point back to head
    head->next = nullptr;     // head becomes the new tail
    return newHead;
}
// Time: O(n), Space: O(n) stack frames
```

### Variation: Reverse Between Positions (LC 92)

```cpp
// Reverse nodes from position left to right (1-indexed)
ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* prev = dummy;

    // Step 1: Walk prev to node just before position 'left'
    for (int i = 1; i < left; i++) prev = prev->next;

    // Step 2: Reverse (right - left) times using front-insertion
    ListNode* cur = prev->next;
    for (int i = 0; i < right - left; i++) {
        ListNode* next = cur->next;
        cur->next = next->next;          // cut next out
        next->next = prev->next;         // insert next at front of reversed section
        prev->next = next;
    }
    return dummy->next;
}
// Time: O(n), Space: O(1)
// The "front insertion" trick: we don't use three-pointer reversal here.
// Instead we repeatedly pull the node after cur and insert it after prev.
```

### Variation: Reverse in Groups of K (LC 25)

```cpp
// LC 25 -- Reverse Nodes in k-Group
// If remaining nodes < k, leave them as-is

ListNode* reverseKGroup(ListNode* head, int k) {
    // Check if there are k nodes remaining
    ListNode* check = head;
    for (int i = 0; i < k; i++) {
        if (!check) return head;  // fewer than k nodes, return as-is
        check = check->next;
    }

    // Reverse k nodes starting from head
    ListNode* prev = nullptr, *cur = head;
    for (int i = 0; i < k; i++) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    // head is now the tail of the reversed group
    // Recursively reverse the rest and attach
    head->next = reverseKGroup(cur, k);
    return prev;  // prev is the new head of this group
}
// Time: O(n), Space: O(n/k) recursion depth
```

---

## Pattern 2: Fast-Slow Pointers (Tortoise and Hare)

**Core idea**: Two pointers move at different speeds. Slow moves 1 step, fast moves 2. Used for: middle of list, cycle detection, cycle entry point.

### Find Middle of List

```cpp
// LC 876 -- Middle of the Linked List
// If even number of nodes, return SECOND middle
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
    // When fast reaches end, slow is at middle
    // Even-length list: slow lands on second middle (e.g., list of 4 -> node 3)
}
```

### Cycle Detection (LC 141)

```cpp
// Floyd's Cycle Detection
bool hasCycle(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;  // they meet inside the cycle
    }
    return false;
}
```

### Find Cycle Entry Point (LC 142)

```cpp
// Why it works:
// Let: distance from head to cycle entry = a
//      distance from cycle entry to meeting point = b
//      cycle length = c
// When they meet: slow traveled (a+b), fast traveled (a+b+c) = 2*(a+b)
// So: a+b+c = 2a+2b -> c = a+b -> a = c-b
// Resetting slow to head and moving both 1 step each:
// slow travels a more steps to reach entry.
// fast travels c-b more steps inside cycle = also reaches entry. They meet at entry.

ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head, *fast = head;

    // Phase 1: find meeting point
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) break;
    }
    if (!fast || !fast->next) return nullptr;  // no cycle

    // Phase 2: find cycle entry
    slow = head;
    while (slow != fast) {
        slow = slow->next;
        fast = fast->next;  // now both move at speed 1
    }
    return slow;
}
```

---

## Pattern 3: Merging Two Sorted Lists

```cpp
// LC 21 -- Merge Two Sorted Lists
// Use dummy head to avoid special-casing the first node

ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* cur = &dummy;  // using stack-allocated dummy (no new/delete)

    while (l1 && l2) {
        if (l1->val <= l2->val) {
            cur->next = l1;
            l1 = l1->next;
        } else {
            cur->next = l2;
            l2 = l2->next;
        }
        cur = cur->next;
    }
    cur->next = l1 ? l1 : l2;  // attach remaining
    return dummy.next;
}
// Time: O(m+n), Space: O(1)
```

### Merge K Sorted Lists (LC 23)

```cpp
#include <queue>

// Use min-heap: always extract the globally minimum node
ListNode* mergeKLists(vector<ListNode*>& lists) {
    // Min-heap comparator: compare node values
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

    for (ListNode* l : lists) if (l) pq.push(l);

    ListNode dummy(0);
    ListNode* cur = &dummy;
    while (!pq.empty()) {
        ListNode* node = pq.top(); pq.pop();
        cur->next = node;
        cur = cur->next;
        if (node->next) pq.push(node->next);
    }
    return dummy.next;
}
// Time: O(N log k) where N = total nodes, k = number of lists
```

---

## Pattern 4: N-th Node from End

```cpp
// LC 19 -- Remove Nth Node from End
// Two-pointer trick: advance fast by n+1 steps, then move both until fast = null

ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* fast = &dummy, *slow = &dummy;

    // Advance fast by n+1 steps (gap of n+1 between fast and slow)
    for (int i = 0; i <= n; i++) fast = fast->next;

    // Move both until fast reaches end
    while (fast) {
        slow = slow->next;
        fast = fast->next;
    }
    // slow->next is the node to remove
    ListNode* toDelete = slow->next;
    slow->next = slow->next->next;
    delete toDelete;
    return dummy.next;
}
// Time: O(n), Space: O(1) -- single pass
```

---

## Pattern 5: Palindrome Check

```cpp
// LC 234 -- Palindrome Linked List
// Steps: 1. Find middle (fast-slow), 2. Reverse second half, 3. Compare

bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;

    // Step 1: find middle
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: reverse second half in-place
    ListNode* prev = nullptr, *cur = slow;
    while (cur) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }

    // Step 3: compare first and reversed second half
    ListNode* left = head, *right = prev;
    while (right) {  // right (reversed) is shorter or equal
        if (left->val != right->val) return false;
        left = left->next;
        right = right->next;
    }
    return true;
}
// Time: O(n), Space: O(1)
```

---

## Pattern 6: Intersection of Two Lists (LC 160)

```cpp
// Find the node where two lists intersect (same object, not same value)
// Key insight: traverse both; when one ends, redirect to the other's head.
// Both pointers walk (A + B) total nodes. They meet at intersection or null.

ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    ListNode* a = headA, *b = headB;
    while (a != b) {
        a = a ? a->next : headB;  // when a exhausts, start at headB
        b = b ? b->next : headA;  // when b exhausts, start at headA
    }
    return a;  // either the intersection node or nullptr
}
// Time: O(m+n), Space: O(1)
// Why it works: after the switch, a travels (lenA + lenB) and so does b.
// They're synchronized and will meet at intersection if it exists.
```

---

## Pattern 7: Sort a Linked List (Merge Sort)

```cpp
// LC 148 -- Sort List in O(n log n), O(1) space
// Use merge sort: split in half (fast-slow), sort each half, merge

ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;  // base case

    // Step 1: split into two halves at middle
    ListNode* slow = head, *fast = head->next;
    // Why fast = head->next (not head)? To avoid slow landing on second half for even lists.
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode* mid = slow->next;
    slow->next = nullptr;  // cut the list in half

    // Step 2: sort both halves
    ListNode* left = sortList(head);
    ListNode* right = sortList(mid);

    // Step 3: merge
    return mergeTwoLists(left, right);
}
// Time: O(n log n), Space: O(log n) recursion stack
```

---

## Examples

### Example 1 -- LC 143: Reorder List (Medium)

> Reorder: L0->L1->...->Ln into L0->Ln->L1->Ln-1->L2->Ln-2->...

```cpp
void reorderList(ListNode* head) {
    if (!head || !head->next) return;

    // Step 1: find middle
    ListNode* slow = head, *fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: reverse second half
    ListNode* second = slow->next;
    slow->next = nullptr;
    ListNode* prev = nullptr, *cur = second;
    while (cur) {
        ListNode* next = cur->next;
        cur->next = prev; prev = cur; cur = next;
    }
    second = prev;

    // Step 3: interleave
    ListNode* first = head;
    while (second) {
        ListNode* tmp1 = first->next, *tmp2 = second->next;
        first->next = second;
        second->next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
// Time: O(n), Space: O(1) -- uses three sub-patterns: middle, reverse, merge
```

---

*<- [[06_Linked_List_Foundations\|Foundations]] · [[06_Linked_List_Problems_and_Exercises\|Problems ->]]*

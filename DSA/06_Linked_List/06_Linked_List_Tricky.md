---
tags: [dsa, linked-list, tricky, hard, interview]
links: ["[[06_Linked_List_Index]]", "[[06_Linked_List_Problems_and_Exercises]]", "[[../07_Trees/07_Trees_Index]]"]
---

# Linked List -- Tricky & Higher-Order

*<- [[06_Linked_List_Problems_and_Exercises|Problems]] · [[../07_Trees/07_Trees_Index|Trees ->]]*

---

## 1. Detect Cycle Length After Detection

**Why tricky**: After finding the meeting point in Floyd's, finding the CYCLE LENGTH (not just entry) requires one more loop.

```cpp
int cycleLength(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            // Count cycle length: move one pointer around until it returns
            int len = 1;
            ListNode* cur = slow->next;
            while (cur != slow) { cur = cur->next; len++; }
            return len;
        }
    }
    return 0;
}
```

---

## 2. Split Linked List into K Parts (LC 725)

**Why tricky**: Distribute nodes as evenly as possible. First `n % k` parts get one extra node. Must handle k > n (some parts will be null).

```cpp
std::vector<ListNode*> splitListToParts(ListNode* head, int k) {
    int n = 0;
    for (ListNode* c = head; c; c = c->next) n++;

    int base = n / k;       // minimum size of each part
    int extra = n % k;      // first 'extra' parts get one more node

    std::vector<ListNode*> result(k, nullptr);
    ListNode* cur = head;

    for (int i = 0; i < k && cur; i++) {
        result[i] = cur;
        int size = base + (i < extra ? 1 : 0);
        for (int j = 0; j < size - 1; j++) cur = cur->next;
        ListNode* next = cur->next;
        cur->next = nullptr;  // cut
        cur = next;
    }
    return result;
}
// Time: O(n + k), Space: O(k) for result
```

---

## 3. Odd-Even Grouping Without Extra Space (LC 328)

**Why tricky**: Group all odd-indexed nodes first, then even-indexed, in-place. The trick is maintaining two sub-lists simultaneously and joining at the end.

```cpp
ListNode* oddEvenList(ListNode* head) {
    if (!head) return head;
    ListNode* odd = head, *even = head->next, *evenHead = even;

    while (even && even->next) {
        odd->next = even->next;   // odd skips over even
        odd = odd->next;
        even->next = odd->next;   // even skips over odd
        even = even->next;
    }
    odd->next = evenHead;  // connect odd tail to even head
    return head;
}
// Time: O(n), Space: O(1)
```

---

## 4. Flatten a Multilevel Doubly Linked List (LC 430)

**Why tricky**: Each node can have a `child` pointer to another list. Must flatten recursively or with a stack.

```cpp
struct Node {
    int val; Node* prev; Node* next; Node* child;
};

Node* flatten(Node* head) {
    if (!head) return head;
    Node* cur = head;
    while (cur) {
        if (cur->child) {
            Node* child = cur->child;
            Node* next = cur->next;

            // Connect cur to child
            cur->next = child;
            child->prev = cur;
            cur->child = nullptr;

            // Find tail of child list
            Node* tail = child;
            while (tail->next) tail = tail->next;

            // Connect tail of child to next
            tail->next = next;
            if (next) next->prev = tail;
        }
        cur = cur->next;
    }
    return head;
}
// Time: O(n), Space: O(1) iterative
```

---

## 5. Inserting into a Sorted Circular Linked List (LC 708)

**Why tricky**: Three cases: (1) normal insertion, (2) inserting new max (goes between max and min, i.e., wraps around), (3) empty list.

```cpp
struct Node { int val; Node* next; Node(int v) : val(v), next(nullptr) {} };

Node* insert(Node* head, int insertVal) {
    Node* newNode = new Node(insertVal);
    if (!head) { newNode->next = newNode; return newNode; }

    Node* prev = head, *cur = head->next;
    bool inserted = false;

    do {
        // Case 1: normal insertion in sorted order
        if (prev->val <= insertVal && insertVal <= cur->val) {
            inserted = true;
        }
        // Case 2: at the wrap-around point (prev is max, cur is min)
        else if (prev->val > cur->val) {
            if (insertVal >= prev->val || insertVal <= cur->val) {
                inserted = true;
            }
        }
        if (inserted) {
            newNode->next = cur;
            prev->next = newNode;
            return head;
        }
        prev = cur;
        cur = cur->next;
    } while (prev != head);  // traversed full circle

    // Case 3: all values same (or single element), insert anywhere
    newNode->next = cur;
    prev->next = newNode;
    return head;
}
```

---

## 6. Skip List (Conceptual -- Quant / System Design)

**Why**: Skip lists achieve O(log n) search/insert/delete with simpler implementation than balanced BSTs. Used in Redis, Lucene.

```
Level 3: 1 -----------------------------------------> 9
Level 2: 1 ---------> 4 ---------> 7 -----------> 9
Level 1: 1 --> 2 --> 4 --> 5 --> 7 --> 8 --> 9
```

Key ideas:
- Multiple "express lanes" with fewer nodes at each higher level.
- Promotion to higher level done probabilistically (coin flip -- p=0.5).
- Search: start at highest level, drop down when overshoot.

```cpp
// Simplified skip list node
struct SkipNode {
    int val;
    std::vector<SkipNode*> forward;  // forward[i] = next node at level i
    SkipNode(int v, int levels) : val(v), forward(levels, nullptr) {}
};
// Full skip list implementation is a common system design interview question.
// Know the concept and O(log n) guarantees.
```

---

## 7. Design Twitter (LC 355) -- Linked List as Time-Ordered Feed

**Why tricky**: Each user has a feed (tweets in order). Merging k users' feeds to get top-10 recent tweets = merge k sorted lists with a heap.

```cpp
// Core data structure:
// user_tweets[userId] = linked list of {tweetId, timestamp}
// Following relationship: unordered_set<int> follows[userId]
// getNewsFeed(userId): collect lists from user + all followed, merge k lists with heap

class Twitter {
    int time = 0;
    struct Tweet { int id, ts; };
    std::unordered_map<int, std::vector<Tweet>> tweets;
    std::unordered_map<int, std::unordered_set<int>> follows;

public:
    void postTweet(int userId, int tweetId) {
        tweets[userId].push_back({tweetId, time++});
    }

    std::vector<int> getNewsFeed(int userId) {
        // min-heap: {-timestamp, tweet_id, user_id, tweet_index}
        using T = std::tuple<int,int,int,int>;
        std::priority_queue<T, std::vector<T>, std::greater<T>> pq;

        auto addLatest = [&](int uid) {
            auto& tw = tweets[uid];
            if (!tw.empty()) {
                int i = tw.size() - 1;
                pq.push({-tw[i].ts, tw[i].id, uid, i - 1});
            }
        };
        addLatest(userId);
        for (int fid : follows[userId]) addLatest(fid);

        std::vector<int> feed;
        while (!pq.empty() && (int)feed.size() < 10) {
            auto [ts, tid, uid, idx] = pq.top(); pq.pop();
            feed.push_back(tid);
            if (idx >= 0) {
                auto& tw = tweets[uid];
                pq.push({-tw[idx].ts, tw[idx].id, uid, idx - 1});
            }
        }
        return feed;
    }

    void follow(int f, int e) { follows[f].insert(e); }
    void unfollow(int f, int e) { follows[f].erase(e); }
};
```

---

## Problem List

| # | Problem | Difficulty | Link |
|---|---------|------------|------|
| 1 | Split Linked List to K Parts | Medium | [LC 725](https://leetcode.com/problems/split-linked-list-in-parts/) |
| 2 | Odd Even Linked List | Medium | [LC 328](https://leetcode.com/problems/odd-even-linked-list/) |
| 3 | Flatten Multilevel Doubly LL | Medium | [LC 430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |
| 4 | Insert into Sorted Circular LL | Medium | [LC 708](https://leetcode.com/problems/insert-into-a-sorted-circular-linked-list/) |
| 5 | Design Twitter | Medium | [LC 355](https://leetcode.com/problems/design-twitter/) |
| 6 | LFU Cache | Hard | [LC 460](https://leetcode.com/problems/lfu-cache/) |
| 7 | All O'one Data Structure | Hard | [LC 432](https://leetcode.com/problems/all-oone-data-structure/) |
| 8 | Swap Nodes in Pairs | Medium | [LC 24](https://leetcode.com/problems/swap-nodes-in-pairs/) |

---

*<- [[06_Linked_List_Problems_and_Exercises|Problems]] · [[../07_Trees/07_Trees_Index|Trees ->]]*

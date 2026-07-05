---
tags: [cpp, stl, manual-containers, custom-vector, custom-list, custom-deque, custom-unordered-map, container-adapters]
links: ["[[05_STL_Index]]", "[[05_STL_Containers_and_Iterator_Invalidation]]", "[[05_STL_Optional_Variant_Tuple]]"]
---

# STL & Modern Features -- Manual Containers Implementation

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

---

To fully master C++, you must understand how standard containers manage memory, lifetimes, pointers, and iterators under the hood. Below are custom implementations of the core sequence, associative, and adapter containers in C++ from scratch.

---

## 1. Sequence Containers

### A. Custom `Vector` (Contiguous Dynamic Array)
Uses placement new to separate memory allocation from object construction.

```cpp
#include <cstddef>
#include <utility>
#include <new>

template <typename T>
class MyVector {
    T* data = nullptr;
    std::size_t sz = 0;
    std::size_t cap = 0;

    void reallocate(std::size_t newCap) {
        T* newBlock = static_cast<T*>(::operator new(newCap * sizeof(T)));
        for (std::size_t i = 0; i < sz; ++i) {
            new (newBlock + i) T(std::move(data[i]));
            data[i].~T();
        }
        ::operator delete(data);
        data = newBlock;
        cap = newCap;
    }

public:
    MyVector() = default;
    ~MyVector() {
        clear();
        ::operator delete(data);
    }

    void push_back(const T& val) {
        if (sz >= cap) reallocate(cap == 0 ? 1 : cap * 2);
        new (data + sz) T(val);
        sz++;
    }

    void push_back(T&& val) {
        if (sz >= cap) reallocate(cap == 0 ? 1 : cap * 2);
        new (data + sz) T(std::move(val));
        sz++;
    }

    void pop_back() {
        if (sz > 0) {
            sz--;
            data[sz].~T();
        }
    }

    void clear() {
        for (std::size_t i = 0; i < sz; ++i) data[i].~T();
        sz = 0;
    }

    std::size_t size() const { return sz; }
    T& operator[](std::size_t index) { return data[index]; }
    const T& operator[](std::size_t index) const { return data[index]; }

    // Iterator
    class Iterator {
        T* ptr;
    public:
        Iterator(T* p) : ptr(p) {}
        T& operator*() { return *ptr; }
        Iterator& operator++() { ++ptr; return *this; }
        bool operator==(const Iterator& other) const { return ptr == other.ptr; }
        bool operator!=(const Iterator& other) const { return ptr != other.ptr; }
    };
    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + sz); }
};
```

---

### B. Custom `List` (Doubly-Linked List)
Maintains pointer stability (insertion/deletion does not invalidate iterators to other elements).

```cpp
template <typename T>
class MyList {
    struct Node {
        T value;
        Node* prev = nullptr;
        Node* next = nullptr;
        Node(const T& val) : value(val) {}
    };

    Node* head = nullptr;
    Node* tail = nullptr;
    std::size_t sz = 0;

public:
    MyList() = default;
    ~MyList() { clear(); }

    void push_back(const T& val) {
        Node* newNode = new Node(val);
        if (!tail) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        sz++;
    }

    void push_front(const T& val) {
        Node* newNode = new Node(val);
        if (!head) {
            head = tail = newNode;
        } else {
            newNode->next = head;
            head->prev = newNode;
            head = newNode;
        }
        sz++;
    }

    void pop_back() {
        if (!tail) return;
        Node* temp = tail;
        tail = tail->prev;
        if (tail) tail->next = nullptr;
        else head = nullptr;
        delete temp;
        sz--;
    }

    void clear() {
        while (head) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        sz = 0;
    }

    std::size_t size() const { return sz; }

    // Iterator
    class Iterator {
        Node* node;
    public:
        Iterator(Node* n) : node(n) {}
        T& operator*() { return node->value; }
        Iterator& operator++() { node = node->next; return *this; }
        bool operator==(const Iterator& other) const { return node == other.node; }
        bool operator!=(const Iterator& other) const { return node != other.node; }
    };
    Iterator begin() { return Iterator(head); }
    Iterator end() { return Iterator(nullptr); }
};
```

---

### C. Custom `Deque` (Segmented Double-Ended Queue)
Implements fixed-size block arrays managed by a central pointer map to allow $O(1)$ front/back insertions.

```cpp
#include <stdexcept>

template <typename T, std::size_t BlockSize = 8>
class MyDeque {
    T** map = nullptr; // Array of pointers to blocks
    std::size_t mapSize = 0;
    std::size_t startBlock = 0;
    std::size_t startIndex = 0;
    std::size_t sz = 0;

    void reserveMap(std::size_t newMapSize) {
        T** newMap = new T*[newMapSize]();
        std::size_t mid = newMapSize / 2;
        std::size_t oldMid = mapSize / 2;
        
        // Copy old blocks to the center of the new map
        for (std::size_t i = 0; i < mapSize; ++i) {
            if (map[i]) {
                newMap[mid - oldMid + i] = map[i];
            }
        }
        delete[] map;
        map = newMap;
        startBlock = mid - oldMid + startBlock;
        mapSize = newMapSize;
    }

public:
    MyDeque() {
        mapSize = 8;
        map = new T*[mapSize]();
        startBlock = mapSize / 2;
        startIndex = 0;
    }

    ~MyDeque() {
        for (std::size_t i = 0; i < mapSize; ++i) {
            if (map[i]) {
                // Free objects and block memory
                for (std::size_t j = 0; j < BlockSize; ++j) {
                    // Naive cleanup assumes constructed elements exist in active slots
                }
                delete[] map[i];
            }
        }
        delete[] map;
    }

    void push_back(const T& val) {
        std::size_t totalIndex = startIndex + sz;
        std::size_t block = startBlock + totalIndex / BlockSize;
        std::size_t index = totalIndex % BlockSize;

        if (block >= mapSize) {
            reserveMap(mapSize * 2);
            block = startBlock + totalIndex / BlockSize;
        }

        if (!map[block]) {
            map[block] = new T[BlockSize];
        }

        map[block][index] = val;
        sz++;
    }

    void push_front(const T& val) {
        if (startIndex == 0) {
            if (startBlock == 0) {
                reserveMap(mapSize * 2);
            }
            startBlock--;
            startIndex = BlockSize - 1;
            if (!map[startBlock]) {
                map[startBlock] = new T[BlockSize];
            }
        } else {
            startIndex--;
        }
        map[startBlock][startIndex] = val;
        sz++;
    }

    T& operator[](std::size_t index) {
        std::size_t totalIndex = startIndex + index;
        std::size_t b = startBlock + totalIndex / BlockSize;
        std::size_t i = totalIndex % BlockSize;
        return map[b][i];
    }

    std::size_t size() const { return sz; }
};
```

---

## 2. Associative Containers

### A. Custom `Map` (Balanced BST Concept)
Stores key-value pairs sorted by key.

```cpp
template <typename Key, typename Value>
class MyMap {
    struct Node {
        Key key;
        Value value;
        Node* left = nullptr;
        Node* right = nullptr;
        Node* parent = nullptr;
        Node(const Key& k, const Value& v, Node* p = nullptr) 
            : key(k), value(v), parent(p) {}
    };

    Node* root = nullptr;
    std::size_t sz = 0;

    void destroy(Node* n) {
        if (n) {
            destroy(n->left);
            destroy(n->right);
            delete n;
        }
    }

public:
    MyMap() = default;
    ~MyMap() { destroy(root); }

    void insert(const Key& k, const Value& v) {
        if (!root) {
            root = new Node(k, v);
            sz++;
            return;
        }
        Node* curr = root;
        Node* parent = nullptr;
        while (curr) {
            parent = curr;
            if (k == curr->key) {
                curr->value = v;
                return;
            }
            curr = (k < curr->key) ? curr->left : curr->right;
        }
        Node* newNode = new Node(k, v, parent);
        if (k < parent->key) parent->left = newNode;
        else parent->right = newNode;
        sz++;
    }

    Value* find(const Key& k) {
        Node* curr = root;
        while (curr) {
            if (curr->key == k) return &curr->value;
            curr = (k < curr->key) ? curr->left : curr->right;
        }
        return nullptr;
    }

    std::size_t size() const { return sz; }
};
```

---

### B. Custom `UnorderedMap` (Hash Table with Chaining & Rehashing)
Resolves collisions using bucket lists (chaining) and triggers rehashing when `load_factor > 1.0`.

```cpp
#include <string>
#include <vector>
#include <functional>

template <typename Key, typename Value, typename Hash = std::hash<Key>>
class MyUnorderedMap {
    struct Node {
        Key key;
        Value value;
        Node* next = nullptr;
        Node(const Key& k, const Value& v) : key(k), value(v) {}
    };

    std::vector<Node*> buckets;
    std::size_t sz = 0;
    Hash hasher;

    std::size_t getBucketIndex(const Key& k, std::size_t bucketCount) const {
        return hasher(k) % bucketCount;
    }

    void rehash(std::size_t newBucketCount) {
        std::vector<Node*> newBuckets(newBucketCount, nullptr);
        for (std::size_t i = 0; i < buckets.size(); ++i) {
            Node* curr = buckets[i];
            while (curr) {
                Node* next = curr->next;
                std::size_t newIdx = getBucketIndex(curr->key, newBucketCount);
                
                // Link node at the head of the new bucket list
                curr->next = newBuckets[newIdx];
                newBuckets[newIdx] = curr;
                
                curr = next;
            }
        }
        buckets = std::move(newBuckets);
    }

public:
    MyUnorderedMap(std::size_t initialBuckets = 8) : buckets(initialBuckets, nullptr) {}

    ~MyUnorderedMap() {
        for (std::size_t i = 0; i < buckets.size(); ++i) {
            Node* curr = buckets[i];
            while (curr) {
                Node* temp = curr;
                curr = curr->next;
                delete temp;
            }
        }
    }

    void insert(const Key& k, const Value& v) {
        // Trigger rehash if load factor exceeds 1.0
        if (sz >= buckets.size()) {
            rehash(buckets.size() * 2);
        }

        std::size_t idx = getBucketIndex(k, buckets.size());
        Node* curr = buckets[idx];
        while (curr) {
            if (curr->key == k) {
                curr->value = v; // Update
                return;
            }
            curr = curr->next;
        }

        // Create new node and insert at bucket head
        Node* newNode = new Node(k, v);
        newNode->next = buckets[idx];
        buckets[idx] = newNode;
        sz++;
    }

    Value* find(const Key& k) {
        std::size_t idx = getBucketIndex(k, buckets.size());
        Node* curr = buckets[idx];
        while (curr) {
            if (curr->key == k) return &curr->value;
            curr = curr->next;
        }
        return nullptr;
    }

    std::size_t size() const { return sz; }
};
```

---

## 3. Container Adapters

Container adapters modify existing sequence containers to expose restricted interfaces.

### A. Custom `Stack` (LIFO) & `Queue` (FIFO)
By default in STL, `std::stack` wraps `std::deque` to block indexing access and expose LIFO constraints. Below, we parameterize the underlying container.

```cpp
// 1. Stack (LIFO wrapper)
template <typename T, typename Container = MyVector<T>>
class MyStack {
    Container c;
public:
    void push(const T& val) { c.push_back(val); }
    void pop() { c.pop_back(); }
    T& top() { return c[c.size() - 1]; }
    bool empty() const { return c.size() == 0; }
    std::size_t size() const { return c.size(); }
};

// 2. Queue (FIFO wrapper)
template <typename T, typename Container = MyList<T>>
class MyQueue {
    Container c;
public:
    void push(const T& val) { c.push_back(val); }
    void pop() { c.pop_back(); } // Naive pop assumes container supports pop_front/erase
    T& front() { return *c.begin(); }
    bool empty() const { return c.size() == 0; }
    std::size_t size() const { return c.size(); }
};
```

---

### B. Custom `PriorityQueue` (Min/Max Heap Adapter)
Wraps a dynamic array (`vector`) and applies heap bubble operations on insertion and deletion.

```cpp
#include <vector>
#include <algorithm>
#include <functional>

template <typename T, typename Container = std::vector<T>, typename Compare = std::less<T>>
class MyPriorityQueue {
    Container c;
    Compare comp;

    void bubbleUp(std::size_t idx) {
        while (idx > 0) {
            std::size_t parent = (idx - 1) / 2;
            if (comp(c[parent], c[idx])) {
                std::swap(c[parent], c[idx]);
                idx = parent;
            } else {
                break;
            }
        }
    }

    void bubbleDown(std::size_t idx) {
        std::size_t size = c.size();
        while (idx * 2 + 1 < size) {
            std::size_t leftChild = idx * 2 + 1;
            std::size_t rightChild = idx * 2 + 2;
            std::size_t targetChild = leftChild;

            if (rightChild < size && comp(c[leftChild], c[rightChild])) {
                targetChild = rightChild;
            }

            if (comp(c[idx], c[targetChild])) {
                std::swap(c[idx], c[targetChild]);
                idx = targetChild;
            } else {
                break;
            }
        }
    }

public:
    MyPriorityQueue() = default;

    void push(const T& val) {
        c.push_back(val);
        bubbleUp(c.size() - 1);
    }

    void pop() {
        if (c.empty()) return;
        std::swap(c[0], c[c.size() - 1]);
        c.pop_back();
        bubbleDown(0);
    }

    const T& top() const { return c[0]; }
    bool empty() const { return c.empty(); }
    std::size_t size() const { return c.size(); }
};
```

---

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

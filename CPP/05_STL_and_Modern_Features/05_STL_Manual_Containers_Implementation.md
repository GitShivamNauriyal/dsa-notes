---
tags: [cpp, stl, manual-containers, custom-vector, custom-map, memory-management]
links: ["[[05_STL_Index]]", "[[05_STL_Containers_and_Iterator_Invalidation]]", "[[05_STL_Optional_Variant_Tuple]]"]
---

# STL & Modern Features -- Manual Containers Implementation

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

---

To write production-grade containers in C++, you must understand the distinction between **allocating raw memory** (reserving space on the heap) and **constructing objects** (initializing active variables in that space).

---

## 1. Custom `Vector` Implementation

A naive vector implementation (like using `new T[capacity]`) is incorrect for two reasons:
1. It forces the default-construction of all `capacity` elements immediately, which is slow.
2. It fails for types that do not have a default constructor.

### The Correct C++ Strategy
- **Allocation**: Allocate raw uninitialized byte memory using `operator new` or `std::allocator`.
- **Construction**: Initialize objects on-demand in the allocated space using **placement new**: `new (address) T(value)`.
- **Destruction**: Call the destructors of active objects manually: `ptr->~T()`.
- **Deallocation**: Release the raw memory block using `operator delete`.

```cpp
#include <cstddef>
#include <utility>
#include <new>
#include <algorithm>
#include <iostream>

template <typename T>
class MyVector {
    T* data = nullptr;
    std::size_t sz = 0;
    std::size_t cap = 0;

    void reallocate(std::size_t newCap) {
        // 1. Allocate raw byte memory
        T* newBlock = static_cast<T*>(::operator new(newCap * sizeof(T)));

        // 2. Move existing elements to new memory using placement new
        for (std::size_t i = 0; i < sz; ++i) {
            new (newBlock + i) T(std::move(data[i]));
            data[i].~T(); // Call destructor on old object
        }

        // 3. Free old memory block
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
        if (sz >= cap) {
            reallocate(cap == 0 ? 1 : cap * 2);
        }
        new (data + sz) T(val); // placement new (copy)
        sz++;
    }

    void push_back(T&& val) {
        if (sz >= cap) {
            reallocate(cap == 0 ? 1 : cap * 2);
        }
        new (data + sz) T(std::move(val)); // placement new (move)
        sz++;
    }

    void pop_back() {
        if (sz > 0) {
            sz--;
            data[sz].~T(); // Call destructor on popped element
        }
    }

    void clear() {
        for (std::size_t i = 0; i < sz; ++i) {
            data[i].~T();
        }
        sz = 0;
    }

    std::size_t size() const { return sz; }
    std::size_t capacity() const { return cap; }

    T& operator[](std::size_t index) { return data[index]; }
    const T& operator[](std::size_t index) const { return data[index]; }

    // --- Iterator Implementation ---
    class Iterator {
        T* ptr;
    public:
        Iterator(T* p) : ptr(p) {}
        T& operator*() { return *ptr; }
        Iterator& operator++() { ++ptr; return *this; }
        Iterator operator++(int) { Iterator temp = *this; ++ptr; return temp; }
        bool operator==(const Iterator& other) const { return ptr == other.ptr; }
        bool operator!=(const Iterator& other) const { return ptr != other.ptr; }
    };

    Iterator begin() { return Iterator(data); }
    Iterator end() { return Iterator(data + sz); }
};
```

---

## 2. Custom `Map` (Binary Search Tree) Implementation

A standard `std::map` is implemented using a Red-Black Tree. Below is a custom Binary Search Tree (BST) map implementation to demonstrate node-linking, recursion, and key-value sorting mechanics.

```cpp
#include <utility>
#include <iostream>

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

    void destroyTree(Node* node) {
        if (node) {
            destroyTree(node->left);
            destroyTree(node->right);
            delete node;
        }
    }

    Node* findNode(Node* node, const Key& k) const {
        if (!node || node->key == k) return node;
        if (k < node->key) return findNode(node->left, k);
        return findNode(node->right, k);
    }

public:
    MyMap() = default;
    ~MyMap() {
        destroyTree(root);
    }

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
                curr->value = v; // Update existing key
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
        Node* node = findNode(root, k);
        return node ? &(node->value) : nullptr;
    }

    std::size_t size() const { return sz; }

    // --- BST Iterator: In-order traversal ---
    class Iterator {
        Node* curr;

        // Helper to find the successor node (next in sorted order)
        Node* successor(Node* node) {
            if (node->right) {
                node = node->right;
                while (node->left) node = node->left;
                return node;
            }
            Node* p = node->parent;
            while (p && node == p->right) {
                node = p;
                p = p->parent;
            }
            return p;
        }

    public:
        Iterator(Node* n) : curr(n) {}

        std::pair<const Key&, Value&> operator*() {
            return {curr->key, curr->value};
        }

        Iterator& operator++() {
            curr = successor(curr);
            return *this;
        }

        bool operator==(const Iterator& other) const { return curr == other.curr; }
        bool operator!=(const Iterator& other) const { return curr != other.curr; }
    };

    Iterator begin() {
        Node* node = root;
        if (node) {
            while (node->left) node = node->left; // Leftmost node has minimum key
        }
        return Iterator(node);
    }

    Iterator end() {
        return Iterator(nullptr);
    }
};
```

---

*<- [[05_STL_Containers_and_Iterator_Invalidation|Containers & Invalidation]] · [[05_STL_Optional_Variant_Tuple|Optional, Variant & Tuple ->]]*

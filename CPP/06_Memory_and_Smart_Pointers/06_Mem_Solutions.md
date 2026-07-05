---
tags: [cpp, memory, solutions]
links: ["[[06_Mem_Index]]", "[[06_Mem_Problems]]", "[[06_Mem_Tricky]]"]
---

# Memory & Smart Pointers -- Solutions

*<- [[06_Mem_Problems|Problems]] · [[06_Mem_Tricky|Tricky Memory Gotchas ->]]*

---

## Tier 1 -- Concept Checks

### Solution 1.1: Size of Smart Pointers
On a 64-bit system, raw pointers are 8 bytes:
1. `sizeof(Widget*)` = **8 bytes**.
2. `sizeof(std::unique_ptr<Widget>)` = **8 bytes** (zero-overhead wrapper around raw pointer, assuming empty deleter).
3. `sizeof(std::shared_ptr<Widget>)` = **16 bytes** (contains two pointers: raw pointer to object + pointer to control block).
4. `sizeof(std::weak_ptr<Widget>)` = **16 bytes** (contains the same two pointers to track the control block's weak count).

---

### Solution 1.2: Reference Count Tracking
- (1) `auto p1 = std::make_shared<int>(10);` -> **1**
- (2) `auto p2 = p1;`                       -> **2** (p1 and p2 share ownership)
- (3) `std::weak_ptr<int> wp = p2;`          -> **2** (weak_ptr does not increase strong count)
- (4) `p1.reset();`                          -> **1** (p1 released, p2 still owns it)
- (5) `auto p3 = wp.lock();`                 -> **2** (wp successfully locked, p3 shares ownership)
- (6) `p2.reset();`                          -> **1** (p2 released, p3 still owns it)
- (7) `p3.reset();`                          -> **0** (last owner released, object destroyed)

---

## Tier 2 -- Implementation

### Solution 2.1: Cyclic References Memory Leak & Fix

#### The Leak (Buggy):
```cpp
#include <memory>
#include <iostream>

struct Child;

struct Parent {
    std::shared_ptr<Child> child;
    ~Parent() { std::cout << "~Parent()\n"; }
};

struct Child {
    std::shared_ptr<Parent> parent; // Cycle created here!
    ~Child() { std::cout << "~Child()\n"; }
};

void leakDemo() {
    auto p = std::make_shared<Parent>();
    auto c = std::make_shared<Child>();
    p->child = c;
    c->parent = p;
} // Exiting function: p and c are destroyed, but reference counts stay at 1. No destructors called!
```

#### The Fix (Correct):
Change `Child::parent` to `std::weak_ptr<Parent>`.

```cpp
#include <memory>
#include <iostream>

struct Child;

struct Parent {
    std::shared_ptr<Child> child;
    ~Parent() { std::cout << "~Parent()\n"; }
};

struct Child {
    std::weak_ptr<Parent> parent; // Safe: weak_ptr breaks cycle!
    ~Child() { std::cout << "~Child()\n"; }
};
```

---

## Tier 3 -- Interview-Level

### Solution 3.1: Thread-Safe Object Cache using `weak_ptr`

```cpp
#include <string>
#include <unordered_map>
#include <memory>
#include <mutex>

class Texture {
public:
    std::string name;
    Texture(const std::string& n) : name(n) {}
};

class TextureCache {
    std::unordered_map<std::string, std::weak_ptr<Texture>> cache;
    std::mutex mtx;
public:
    std::shared_ptr<Texture> getTexture(const std::string& name) {
        std::lock_guard<std::mutex> lock(mtx);
        
        // Check if texture is in cache and still alive
        auto it = cache.find(name);
        if (it != cache.end()) {
            if (auto sharedTex = it->second.lock()) {
                return sharedTex; // Return existing instance
            }
        }

        // Not in cache, or has been deleted -> create new
        auto newTex = std::shared_ptr<Texture>(new Texture(name), [this, name](Texture* ptr) {
            // Custom deleter: when the texture is destroyed, clean it up from the cache map
            std::lock_guard<std::mutex> cacheLock(this->mtx);
            this->cache.erase(name);
            delete ptr;
        });

        cache[name] = newTex; // Add weak pointer to cache
        return newTex;
    }
};
```

---

## Tier 4 -- Systems / Placement-Hard

### Solution 4.1: Custom `unique_ptr` Implementation

```cpp
#include <utility>

template <typename T>
struct default_delete {
    void operator()(T* ptr) const {
        delete ptr;
    }
};

template <typename T, typename Deleter = default_delete<T>>
class my_unique_ptr {
    T* rawPtr;
    Deleter deleter;
public:
    // Constructors
    explicit my_unique_ptr(T* ptr = nullptr) : rawPtr(ptr) {}
    my_unique_ptr(T* ptr, Deleter d) : rawPtr(ptr), deleter(d) {}

    // Destructor
    ~my_unique_ptr() {
        if (rawPtr) {
            deleter(rawPtr);
        }
    }

    // Disable copy operations
    my_unique_ptr(const my_unique_ptr&) = delete;
    my_unique_ptr& operator=(const my_unique_ptr&) = delete;

    // Move constructor (transfer ownership)
    my_unique_ptr(my_unique_ptr&& other) noexcept : rawPtr(other.release()), deleter(std::move(other.deleter)) {}

    // Move assignment operator
    my_unique_ptr& operator=(my_unique_ptr&& other) noexcept {
        if (this != &other) {
            reset(other.release());
            deleter = std::move(other.deleter);
        }
        return *this;
    }

    // Pointer Operations
    T* get() const noexcept { return rawPtr; }

    T* release() noexcept {
        T* temp = rawPtr;
        rawPtr = nullptr;
        return temp;
    }

    void reset(T* ptr = nullptr) noexcept {
        T* oldPtr = rawPtr;
        rawPtr = ptr;
        if (oldPtr) {
            deleter(oldPtr);
        }
    }

    // Dereference operators
    T& operator*() const { return *rawPtr; }
    T* operator->() const noexcept { return rawPtr; }

    explicit operator bool() const noexcept { return rawPtr != nullptr; }
};
```

---

*<- [[06_Mem_Problems|Problems]] · [[06_Mem_Tricky|Tricky Memory Gotchas ->]]*

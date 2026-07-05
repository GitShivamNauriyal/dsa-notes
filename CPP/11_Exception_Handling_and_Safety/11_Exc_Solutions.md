---
tags: [cpp, exceptions, solutions]
links: ["[[11_Exc_Index]]", "[[11_Exc_Problems]]", "[[11_Exc_Tricky]]"]
---

# Exception Handling & Safety -- Solutions

*<- [[11_Exc_Problems|Problems]] · [[11_Exc_Tricky|Tricky Exception Gotchas ->]]*

---

### Solution 1: Transactional Assignment (Strong Guarantee)

```cpp
#include <utility>
#include <cstddef>
#include <algorithm>

class TransactionalBuffer {
    int* buffer = nullptr;
    std::size_t size = 0;

public:
    TransactionalBuffer() = default;
    
    // Copy Constructor
    TransactionalBuffer(const TransactionalBuffer& other) : size(other.size) {
        if (size > 0) {
            buffer = new int[size];
            std::copy(other.buffer, other.buffer + size, buffer);
        }
    }

    // Destructor (No-throw)
    ~TransactionalBuffer() noexcept {
        delete[] buffer;
    }

    // No-throw Swap
    void swap(TransactionalBuffer& other) noexcept {
        using std::swap;
        swap(buffer, other.buffer);
        swap(size, other.size);
    }

    // Assignment implementing the Strong Exception Guarantee
    TransactionalBuffer& operator=(TransactionalBuffer other) { // Copied by value
        swap(other); // Swapped atomically
        return *this;
    } // Old buffer is deleted in other's destructor when function returns
};
```

---

### Solution 2: Audit `noexcept` Move Reallocations

To fix the allocation fallback, mark the move constructor explicitly **`noexcept`**.

```cpp
#include <type_traits>

struct Connection {
    int socketId;
    
    // Fixed: added noexcept specifier!
    Connection(Connection&& other) noexcept {
        socketId = other.socketId;
        other.socketId = -1;
    }
};

// Static assertion to audit optimization at compile time
static_assert(std::is_nothrow_move_constructible_v<Connection>, 
              "Connection must be nothrow move constructible to prevent copy fallback!");
```

---

### Solution 3: Double Resource Allocation Safety

We use nested `try-catch` blocks or conditional tracking to roll back the first allocation if the second fails.

```cpp
#include <new>

struct ResourceA {};
struct ResourceB {};

void processData() {
    ResourceA* resA = nullptr;
    ResourceB* resB = nullptr;

    try {
        resA = new ResourceA(); // Might throw bad_alloc
        
        try {
            resB = new ResourceB(); // Might throw bad_alloc
        } catch (...) {
            // Rollback: delete resA to prevent memory leak!
            delete resA; 
            throw; // Re-throw exception
        }
    } catch (...) {
        // Exception propagates out
        throw;
    }

    // Clean up if successful
    delete resA;
    delete resB;
}
```

---

*<- [[11_Exc_Problems|Problems]] · [[11_Exc_Tricky|Tricky Exception Gotchas ->]]*

---
tags: [cpp, exceptions, stack-unwinding, raii, destructors]
links: ["[[11_Exc_Index]]", "[[11_Exc_Exception_Safety_Guarantees]]", "[[11_Exc_Problems]]"]
---

# Exception Handling & Safety -- Stack Unwinding & RAII

*<- [[11_Exc_Exception_Safety_Guarantees|Exception Safety Guarantees]] · [[11_Exc_Problems|Problems ->]]*

---

C++ manages resource lifetimes during exceptions through **Stack Unwinding** paired with **RAII**.

---

## 1. Stack Unwinding Mechanics

When an exception is thrown, the C++ runtime stops execution of the current function and begins traversing back up the active call stack (frame by frame) until it reaches a matching `catch` block.
- **The Cleanup**: For each stack frame it exits, the runtime executes the **destructors of all local objects** constructed in that frame in the reverse order of their construction.
- If no matching catch block is found, the runtime calls `std::terminate()`.

---

## 2. RAII (Resource Acquisition Is Initialization)

If resources (like raw heap pointers, database connections, or mutex locks) are managed manually, throwing an exception skips the subsequent cleanup lines, leaking resources.
**RAII** binds resource lifetimes to the stack-allocated object's lifetime:
1. Acquire resource in the constructor.
2. Release resource in the destructor.
- During stack unwinding, the runtime is guaranteed to execute the destructor, freeing the resource safely.

```cpp
#include <iostream>
#include <string>
#include <stdexcept>
#include <cstdio>

// RAII Wrapper for FILE*
class RAIIFile {
    FILE* fp = nullptr;
public:
    explicit RAIIFile(const char* path, const char* mode) {
        fp = std::fopen(path, mode);
        if (!fp) throw std::runtime_error("Failed to open file");
    }

    ~RAIIFile() noexcept {
        if (fp) {
            std::fclose(fp);
            std::cout << "File closed safely via RAII destructor.\n";
        }
    }
};

void riskyProcess() {
    RAIIFile file("doc.txt", "r"); // Resource acquired
    
    // Some logic throws an exception
    throw std::runtime_error("Network error"); 
    
    // Manual file closing code here would have been skipped!
}

void demoRAII() {
    try {
        riskyProcess();
    } catch (const std::exception& e) {
        // Exception caught. File is already closed safely!
        std::cout << "Exception caught: " << e.what() << "\n";
    }
}
```

---

*<- [[11_Exc_Exception_Safety_Guarantees|Exception Safety Guarantees]] · [[11_Exc_Problems|Problems ->]]*

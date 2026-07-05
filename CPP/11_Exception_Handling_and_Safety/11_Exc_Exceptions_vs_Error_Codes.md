---
tags: [cpp, exceptions, error-codes, std-expected, Cpp23, performance]
links: ["[[11_Exc_Index]]", "[[11_Exc_Noexcept_and_Specifications]]"]
---

# Exception Handling & Safety -- Exceptions vs Error Codes

*<- [[11_Exc_Index|Index]] · [[11_Exc_Noexcept_and_Specifications|Noexcept & Specifications ->]]*

---

Modern C++ offers two primary error-handling mechanisms: **Exceptions** and **Error Codes**. C++23 introduced **`std::expected`** to bridge the gap.

---

## 1. Tradeoffs Comparison Matrix

| Aspect | Exceptions | Error Codes | `std::expected` (C++23) |
|---|---|---|---|
| **Happy Path Performance** | **Zero Overhead** (No checks performed) | Minor Overhead (Branches on every return) | Minor Overhead (Check active state) |
| **Error Path Performance** | **High Cost** (Unwinding tables: $\sim 10\mu\text{s}$) | Low Cost (Single return jump) | Low Cost (Single return jump) |
| **Code Readability** | Clean (Happy path isolated from errors) | Cluttered (Pervasive `if (err)` checks) | Clean (Supports monadic chaining) |
| **Safety** | Implicit (Uncaught errors abort program) | Dangerous (Easy to silently ignore return value) | Safe (Must inspect or unpack value) |

---

## 2. Asymmetric Performance: Zero-Cost Exceptions
Modern compilers (like GCC/Clang) implement the **Zero-Cost Exception Model**:
- If no exception is thrown, the happy path runs with **zero runtime overhead** (no branch instructions are executed).
- When an exception is thrown, the runtime must search compiled **metadata tables** (side tables) to locate matching `catch` blocks, unwind stack frames, and invoke destructors. This is extremely slow, making exceptions unsuitable for control-flow logic.

---

## 3. C++23 Monadic Error Handling: `std::expected`

`std::expected<T, E>` holds either a success value of type `T` or an error of type `E`. It resides entirely on the stack and requires no dynamic allocation or stack unwinding.

```cpp
#include <expected>
#include <string>
#include <iostream>

enum class FileError {
    NotFound,
    AccessDenied
};

// Returns either a string or a FileError code
std::expected<std::string, FileError> readFile(const std::string& path) {
    if (path == "secret.txt") {
        return std::unexpected(FileError::AccessDenied); // Return error state
    }
    if (path != "doc.txt") {
        return std::unexpected(FileError::NotFound);
    }
    return "File contents here"; // Return success value
}

void demoExpected() {
    auto result = readFile("secret.txt");
    
    if (result.has_value()) {
        std::cout << "Success: " << result.value() << "\n";
    } else {
        FileError err = result.error();
        if (err == FileError::AccessDenied) {
            std::cout << "Error: Access Denied!\n"; // Prints
        }
    }
}
```

---

*<- [[11_Exc_Index|Index]] · [[11_Exc_Noexcept_and_Specifications|Noexcept & Specifications ->]]*

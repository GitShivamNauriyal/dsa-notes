---
tags: [cpp, oop, constructors, destructors, raii, initialization-order]
links: ["[[03_OOP_Index]]", "[[03_OOP_Classes_and_Access_Control]]", "[[03_OOP_Rule_of_Zero_Three_Five]]"]
---

# OOP in C++ -- Constructors, Destructors, & RAII

*<- [[03_OOP_Classes_and_Access_Control|Classes & Access Control]] · [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five ->]]*

---

## 1. Resource Acquisition Is Initialization (RAII)

**RAII** is the core management pattern of modern C++. 
- **Concept**: Bind the lifecycle of a resource (heap memory, file handles, sockets, database locks) to the lifecycle of a **stack-allocated local object**.
- **Constructor**: Acquires the resource.
- **Destructor**: Automatically releases the resource when the object goes out of scope.
- **Exception Safety**: Because destructors are guaranteed to be called during stack unwinding, RAII ensures no resource leaks occur even if an exception is thrown.

```cpp
#include <fstream>
#include <string>

class FileWrapper {
    std::ofstream fileStream;
public:
    // Resource acquired in constructor
    FileWrapper(const std::string& filename) : fileStream(filename) {
        if (!fileStream.is_open()) {
            throw std::runtime_error("Failed to open file!");
        }
    }

    void writeData(const std::string& data) {
        fileStream << data;
    }

    // Resource automatically released in destructor
    ~FileWrapper() {
        if (fileStream.is_open()) {
            fileStream.close();
        }
    }
};
```

---

## 2. Member Initialization Lists vs Assignment

Using member initialization lists inside constructors is preferred over assigning values in the constructor body.

### Why it Matters for Performance:
- **Assignment in Body**: The compiler first calls the default constructor of the member variable, and then calls its assignment operator inside the constructor body (double initialization).
- **Initialization List**: The compiler calls the copy/parameterized constructor directly once.

```cpp
#include <string>

class Widget {
    std::string name;
public:
    // Suboptimal: default construct name -> assign "Bob"
    Widget(const std::string& n) {
        name = n; 
    }

    // Optimal: directly construct name with n
    Widget(const std::string& n) : name(n) {}
};
```

---

## 3. The Member Initialization Order Trap

> [!IMPORTANT]
> Class members are always initialized in the **order of their declaration in the class definition**, NOT the order they appear in the member initialization list!

```cpp
#include <iostream>

class OrderDemo {
    int x;
    int y; // x is declared before y
public:
    // Trap: y appears first in the initializer list, but x is initialized first!
    // Since x is initialized using y (which is uninitialized garbage), x gets garbage values!
    OrderDemo(int val) : y(val), x(y + 5) {
        std::cout << "x: " << x << ", y: " << y << "\n";
    }
};
```

---

## 4. Delegating Constructors

Allows a constructor to call another constructor of the same class to reduce duplicate code.

```cpp
class Server {
    std::string host;
    int port;
public:
    // Core constructor
    Server(std::string h, int p) : host(h), port(p) {}

    // Delegating constructor
    Server(int p) : Server("localhost", p) {} 
};
```

---

*<- [[03_OOP_Classes_and_Access_Control|Classes & Access Control]] · [[03_OOP_Rule_of_Zero_Three_Five|Rule of Zero/Three/Five ->]]*

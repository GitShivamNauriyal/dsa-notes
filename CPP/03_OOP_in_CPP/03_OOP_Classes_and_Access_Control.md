---
tags: [cpp, oop, class, struct, friend, access-control]
links: ["[[03_OOP_Index]]", "[[03_OOP_Constructors_Destructors_and_RAII]]"]
---

# OOP in C++ -- Classes & Access Control

*<- [[03_OOP_Index|Index]] · [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII ->]]*

---

## 1. Class vs Struct: Deep Dive

In C++, `class` and `struct` are compile-time constructs that resolve to identical object layouts in memory. The distinction is purely syntactic and defaults-based.

### Structural and Semantic Differences
1. **Default Member Access**:
   - `class` defaults to `private`.
   - `struct` defaults to `public`.
2. **Default Inheritance Access**:
   - `class` defaults to `private` inheritance: `class Derived : Base` is treated as `class Derived : private Base`.
   - `struct` defaults to `public` inheritance: `struct Derived : Base` is treated as `struct Derived : public Base`.
3. **Template Parameter Defaults**:
   - In template type parameters, you can use `class` (e.g., `template <class T>`) but not `struct` (e.g., `template <struct T>` is invalid syntax).

### Memory and Alignment Layout
There is **zero difference** in memory footprint, alignment, or padding between a `class` and a `struct` with identical members.
- Members are laid out sequentially in memory in the order of their declaration.
- Alignment constraints apply identically to both.

```cpp
#include <iostream>

struct StructPoint {
    char c;   // 1 byte + 3 bytes padding
    int x;    // 4 bytes
    double y; // 8 bytes
};

class ClassPoint {
    char c;
    int x;
public:
    double y;
};

void printSizes() {
    // Both structures occupy exactly 16 bytes due to alignment padding (8-byte boundary)
    std::cout << "Struct size: " << sizeof(StructPoint) << "\n"; // Output: 16
    std::cout << "Class size: " << sizeof(ClassPoint) << "\n";   // Output: 16
}
```

---

## 2. Access Specifiers & Inheritance Modifiers

Access control is checked **strictly at compile-time**. Once compilation finishes, access specifiers are stripped. At runtime, a member variable is just an offset in memory, meaning private variables are accessed just as fast as public variables with no size or speed overhead.

### The Access Matrix for Members
- **`public`**: Accessible by any code that has visibility of the class.
- **`protected`**: Accessible by member functions, friends, and derived classes.
- **`private`**: Accessible only by member functions and friends of the class.

---

### Inheritance Modifiers (Access Filtering)
When inheriting from a base class, you can specify `public`, `protected`, or `private` inheritance. This acts as a **filter** that restricts the maximum accessibility of base members in the derived class.

```
       Base Members                  Inheritance Type                  Derived Visibility
     ┌──────────────┐                ┌──────────────┐                ┌──────────────────┐
     │  public      ├───────────────►│    public    ├───────────────►│  public          │
     │  protected   │                └──────────────┘                │  protected       │
     └──────────────┘                                                └──────────────────┘
     
     ┌──────────────┐                ┌──────────────┐                ┌──────────────────┐
     │  public      ├───────────────►│  protected   ├───────────────►│  protected       │
     │  protected   │                └──────────────┘                │  protected       │
     └──────────────┘                                                └──────────────────┘
     
     ┌──────────────┐                ┌──────────────┐                ┌──────────────────┐
     │  public      ├───────────────►│   private    ├───────────────►│  private         │
     │  protected   │                └──────────────┘                │  private         │
     └──────────────┘                                                └──────────────────┘
```

#### Detailed Inheritance Visibility Matrix:
| Base Member Access | Public Inheritance | Protected Inheritance | Private Inheritance |
|---|---|---|---|
| **`public`** | Remains `public` | Becomes `protected` | Becomes `private` |
| **`protected`** | Remains `protected` | Becomes `protected` | Becomes `private` |
| **`private`** | **Inaccessible** | **Inaccessible** | **Inaccessible** |

---

### Compiler Casting Restriction (Upcasting)
A critical interview question is how inheritance types affect **upcasting** (casting a derived pointer/reference to a base pointer/reference):
- **Public Inheritance**: Anyone can cast `Derived*` to `Base*` (behaves as a true IS-A relationship).
- **Protected Inheritance**: Only member functions and friends of the derived class (and subclasses of the derived class) can cast `Derived*` to `Base*`.
- **Private Inheritance**: Only member functions and friends of the derived class itself can cast `Derived*` to `Base*`. Outsiders cannot compile this cast.

```cpp
class Base {};
class PrivateDerived : private Base {
    void internalCast() {
        Base* ptr = this; // OK: permitted inside the private derived class
    }
};

void outsiderCast() {
    PrivateDerived pd;
    // Base* ptr = &pd; // COMPILER ERROR: 'Base' is an inaccessible base of 'PrivateDerived'!
}
```

---

## 3. Friend Mechanics & Security

The `friend` keyword allows external classes or functions to read and write private/protected members.

### Properties of Friendship:
- **Friendship is granted, not taken**: Class A must declare class B as a friend inside its definition. Class B cannot declare itself a friend of A.
- **Not Symmetric**: If A is a friend of B, B is not automatically a friend of A.
- **Not Transitive**: If A is a friend of B, and B is a friend of C, A is not a friend of C.
- **Not Inherited**: A friend of a base class is not a friend of its derived classes. Similarly, if a base class has friends, those friends do not automatically have access to members in subclasses.

### Code Implementation (Friend Namespace Resolution)

```cpp
#include <iostream>
#include <string>

class BankAccount {
    std::string owner;
    double balance;

    // Friend function declaration
    friend void printBalance(const BankAccount& acct);
    
    // Friend class declaration
    friend class AuditService;

public:
    BankAccount(std::string name, double bal) : owner(name), balance(bal) {}
};

// Friend function can access private members of BankAccount directly
void printBalance(const BankAccount& acct) {
    std::cout << "Owner: " << acct.owner << ", Balance: $" << acct.balance << "\n";
}

class AuditService {
public:
    void audit(BankAccount& acct) {
        if (acct.balance > 1000000.0) {
            // Permitted access to private balance
            std::cout << "Auditing " << acct.owner << ": Large Balance detected!\n";
        }
    }
};
```

### Performance & Security Tradeoffs
- **Performance**: Friendship has **zero** runtime cost. It is purely a compile-time authorization check.
- **Encapsulation Security**: While friendship is useful for tight coupling (e.g. Iterator accessing Node class internals, or operator overloading), overuse of friends breaks encapsulation. If a class has too many friends, changing its private data members requires updating and re-compiling all friend functions and classes.

---

*<- [[03_OOP_Index|Index]] · [[03_OOP_Constructors_Destructors_and_RAII|Constructors & RAII ->]]*

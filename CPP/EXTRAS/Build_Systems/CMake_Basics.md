---
tags: [cpp, extras, build-systems, cmake, target-linking, compile-commands]
links: ["[[../../00_Home]]"]
---

# Extras -- CMake Basics

*<- [[../../00_Home|Home]]*

---

CMake is a cross-platform build system generator. It reads a `CMakeLists.txt` file and generates native build files (like Makefiles, Ninja build scripts, or Visual Studio solutions).

---

## 1. Minimal `CMakeLists.txt` Template

Here is a standard, modern target-based template for configuring a C++ project:

```cmake
cmake_minimum_required(VERSION 3.15)
project(TradingEngine VERSION 1.0 LANGUAGES CXX)

# Force C++20 standard compile requirements
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Export compile_commands.json for clangd/IDE code completion
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# 1. Define a library target
add_library(core_lib STATIC
    src/math_helper.cpp
    src/logger.cpp
)

# Specify public header include directories for the library
target_include_directories(core_lib PUBLIC include)

# 2. Define the main executable target
add_executable(trading_app src/main.cpp)

# Link the library target to the executable target
target_link_libraries(trading_app PRIVATE core_lib)
```

---

## 2. Dynamic Target Linking and Dependencies

Modern CMake promotes **Target-Based Configurations** (using `PRIVATE`, `INTERFACE`, or `PUBLIC` transitives) instead of global variables.

- **`PRIVATE`**: Dependency is only needed to build the target itself, not by anything linking to it.
- **`PUBLIC`**: Dependency is needed to build the target and by any consumer linking to this target.
- **`INTERFACE`**: Dependency is not needed to build this target, but is required by any consumer linking to it (common for header-only libraries).

### Finding External Packages (`find_package`)
To locate libraries installed on the system (like POSIX threads or Boost):

```cmake
# Locate POSIX threads package
find_package(Threads REQUIRED)

# Link Threads library target to our application
target_link_libraries(trading_app PRIVATE Threads::Threads)
```

---

## 3. Standard Out-of-Source Build Process

Never run cmake inside the source folder. Always use out-of-source builds to prevent cluttering directories with temporary object files.

```bash
mkdir build
cd build
cmake ..      # Generate build files
cmake --build . # Compile project (compiler independent command)
```

---

*<- [[../../00_Home|Home]]*

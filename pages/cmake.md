# CMake
### 1. [Basic CMakeLists.txt](#basic-cmakeliststxt)
```cmake
cmake_minimum_required(VERSION 3.22)

project(name_here)

add_executable(${PROJECT_NAME} main.cpp)
```

### 2. [Setting the CXX Standard](#setting-standard)
```cmake
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

### 3. [Force enable color output for Ninja](#enable-color-for-ninja)
There's a problem with ```Ninja``` generator which makes it that no colors show on compile. This fixes it.
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
```

### 4. [Building with CMake](#building-with-cmake)
A script file can be made executable and the commands bellow incorporated:
```bash
set -xe

# Create make files
cmake -B build -GNinja src/

# Build files
cmake --build build/
```

Use [Change Permissions](/pages/linux.md#change-file-permissions) to make the file executable.

### 5. [Ninja extension for CMake](#ninja-for-cmake)
A faster alternative to the default CMake generator. <br/>
Can be installed via:
```bash
$ sudo apt install ninja-build
```

## References
- https://github.com/ninja-build/ninja/issues/1984
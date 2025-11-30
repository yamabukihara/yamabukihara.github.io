# CMake

## CMake核心知识
### Ⅰ、基础结构与项目定义
```cmake
# 声明所需的最低 CMake 版本。建议使用 3.10+，以支持现代特性。
cmake_minimum_required(VERSION 3.10)
# 定义项目名称、版本和所使用的语言.
project(MyApp VERSION 1.0 LANGUAGES C CXX)

# C++标准
# 明确指定 C++ 标准，这是现代实践的基石。REQUIRED 确保旧编译器无法编译时报错。
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 模块包含
# 在顶层 CMakeLists.txt 中引入子目录。
add_subdirectory(lib)
add_subdirectory(src)

# 编译器输出
# 推荐配置，生成 compile_commands.json，供 VS Code, Clangd 等工具链使用。
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
```

### Ⅱ、核心：目标管理
在现代 CMake 中，一切都围绕着 目标（Targets） 展开。
```cmake
# 可执行文件
# 创建最终运行的程序。
add_executable(MyApp main.cc)

# 库
# 创建库。STATIC（静态库）或 SHARED（动态库）。
add_library(MyLib STATIC calculator.cc)

# 目标属性
# 使用 target_sources() 管理目标所需的源文件列表。
target_sources(MyLib PRIVATE ${HEADERS} ${SOURCES})

# 设置通用的编译器警告旗标（GCC/Clang）。
target_compile_options(MyApp PRIVATE -Wall -Wextra)
```

### Ⅲ、依赖管理和传播
通过声明 依赖关系 来管理路径和链接。
```cmake
# 设置头文件路径。 告诉目标去哪里找头文件。
target_include_directories(MyLib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

# 链接依赖。 告诉目标 A 需要链接目标 B。这是连接模块的核心。
target_link_libraries(MyApp PRIVATE MyLib)

# 设置编译宏定义（-D 选项）。
target_compile_definitions(MyApp PRIVATE DEBUG_MODE)
```

### Ⅳ、外部依赖和查找包
```cmake
# 查找系统或 Vcpkg/Conan 安装的第三方库。
find_package(fmt CONFIG REQUIRED)

# 引入一个包含自己 CMakeLists.txt 的子项目
add_subdirectory(${FIREBASE_SDK_PATH})

# 链接到 find_package 或 add_subdirectory 找到的库目标。
target_link_libraries(MyApp PRIVATE fmt::fmt)
```

### Ⅴ、实用程序和自动化
```cmake
# 启用 CTest 并将可执行文件注册为测试。
enable_testing()
add_executable(test_runner test_calculator.cc)
add_test(NAME MyTest COMMAND test_runner)

# TestB 只在 TestA 成功后运行
add_test(NAME TestA COMMAND test_setup)
add_test(NAME TestB COMMAND test_final_result DEPENDS TestA)

# 根据操作系统、配置类型或变量值执行不同操作。
if (WIN32)
    target_link_libraries(MyApp PRIVATE ws2_32)
endif()

# 设置 CMake 变量
set(CMAKE_BUILD_TYPE Debug)
```

## Example
创建工程cxx_demo, 项目结构如下
```plain
cxx_demo/
|----CMakeLists.txt
|----lib
| |----CMakeLists.txt
| |----calculator.hpp
| |----calculator.cc
|----src
| |----CMakeLists.txt
| |----main.cc
|----tests
| |----CMakeLists.txt
| |----test_calculator.cc
|----vcpkg.json
|----vcpkg-configuration.json
```

`cxx_demo/CMakeLists.txt`
```cmake
cmake_minimum_required(VERSION 3.10)
project(CXX_Demo VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(CTest)
enable_testing()

add_subdirectory(lib)
add_subdirectory(src)
add_subdirectory(tests)
```

`cxx_demo/lib/CMakeLists.txt`
```cmake
add_library(calculator STATIC 
    calculator.cc 
    calculator.hpp
)

target_include_directories(
    calculator
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
)   
```

`cxx_demo/lib/calculator.hpp`
```cxx
#ifndef CALCULATOR_H
#define CALCULATOR_H

int add(int x, int y);

#endif
```
`cxx_demo/lib/calculator.cc`
```cxx
#include "calculator.hpp"

int add(int x, int y)
{
    return x + y;
}
```

`cxx_demo/src/CMakeLists.txt`
```cmake
find_package(fmt CONFIG REQUIRED)

add_executable(cxx_demo main.cc)

target_link_libraries(cxx_demo PRIVATE 
    calculator
    fmt::fmt
)
```

`cxx_demo/src/main.cc`
```cxx
#include "calculator.hpp"
#include <fmt/core.h>

int main(int argc, char *argv[])
{
    int result = add(10, 5);
    fmt::println("print by fmt: ");
    fmt::println("{} + {} = {}", 10, 5, result);

    if (result == 15) {
        return 0;
    } else {
        return 1;
    }
}
```

`cxx_demo/tests/CMakeLists.txt`
```cmake
enable_testing()

add_executable(test_runner test_calculator.cc)

target_link_libraries(test_runner calculator)

add_test(NAME AddTest COMMAND test_runner)
```

`cxx_demo/tests/test_calculator.cc`
```cxx
#include <iostream>
#include <cstdlib>
#include "calculator.hpp"

int main()
{
    int expected = 15;
    int actual = add(7, 8);

    if (actual == expected)
    {
        std::cout << "Test passed: 7 + 8 = " << actual << std::endl;
        return EXIT_SUCCESS;
    }
    else {
        std::cerr << "Test FAILED! Expected: " << expected << ", Actual: " << actual << std::endl;
        return EXIT_FAILURE;
    }
}
```

vcpkg.json文件通过在cxx_demo目录执行命令`vcpkg new --name=cxx-demo --version=1.0.0`自动生成  

`cxx_demo/vcpkg.json`
```json
{
  "name": "cxx-demo",
  "version": "1.0.0",
  "dependencies": [
    {
      "name": "fmt"
    }
  ]
}
```

通过以下命令运行示例工程
```sh
# 构建项目，通过配置DCMAKE_TOOLCHAIN_FILE集成vcpkg，将自动根据vcpkg.json下载依赖
mkdir build && cd build
cmake .. -D-DCMAKE_TOOLCHAIN_FILE=$VCPKG_CMAKE_TOOLCHAIN_FILE

# 编译项目
cmake --build .

# 运行主程序
./src/cxx_demo

# 执行测试
ctest
```

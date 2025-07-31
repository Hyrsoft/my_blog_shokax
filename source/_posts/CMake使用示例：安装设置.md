---
title: CMake使用示例：安装设置
date: 2025-07-31 22:08:33
tags: CMake
categories: Programing
---
# 说明

发布项目时需要指定项目文件的安装路径
这里的路径是相对于`CMAKE_INSTALL_PREFIX`的路径，比如可执行文件就会在`${CMAKE_INSTALL_PREFIX}/bin`下
需要指定`CMAKE_INSTALL_PREFIX`的路径，如果不指定，默认是`/usr/local`

**安装可执行文件和库：**

```cmake
install(TARGETS test_account
    RUNTIME DESTINATION bin # 可执行文件
    LIBRARY DESTINATION lib # 动态库
    ARCHIVE DESTINATION lib # 静态库
)
```

**安装头文件：**

1. 单独安装

    ```cmake
    install(DIRECTORY 
        include/ DESTINATION include
    )
    ```

2. 和其他文件一起安装

    ```cmake
    install(TARGETS test_account
        RUNTIME DESTINATION bin # 可执行文件
        LIBRARY DESTINATION lib # 动态库
        ARCHIVE DESTINATION lib # 静态库
        PUBLIC_HEADER DESTINATION include # 公共头文件
    )
    ```



# 使用示例

项目结构

```bash
.
├── CMakeLists.txt
├── include
│   ├── dlib.h
│   └── slib.h
├── main.cpp # 入口文件
└── src
    ├── dlib.cpp
    └── slib.cpp
```

`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)

project(install_demo)
set(CMAKE_CXX_STANDARD 11)

# 添加公共头文件路径
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

# 添加静态库
add_library(slib STATIC
    src/slib.cpp
    include/slib.h
)

# 添加动态库
add_library(dlib SHARED
    src/dlib.cpp
    include/dlib.h
)

# 设置安装路径
set(CMAKE_INSTALL_PREFIX "/home/hao/installed/test")
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib") 

# 添加可执行文件
add_executable(${PROJECT_NAME}
    main.cpp
)

# 添加链接库
target_link_libraries(${PROJECT_NAME} 
    slib
    dlib
)

# 打印安装路径默认值
message(STATUS "CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")

install(DIRECTORY 
    include/ DESTINATION include
)


# 安装
install(TARGETS ${PROJECT_NAME} slib dlib
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)
```

## 一、安装流程


```bash
# 生成cmake配置文件
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=./installed
cmake -S . -B build # 如果不指定安装目录，使用cmake中已经写好的或者默认值

# 编译
cmake --build build

# 安装
cmake --install build
```

安装结果：

`\home\hao\installed\test`

```bash
.
├── bin
│   └── install_demo
├── include
│   ├── dlib.h
│   └── slib.h
└── lib
    ├── libdlib.so
    └── libslib.a
```

## 二、设置运行时库搜索路径(RPATH)

设置可执行文件运行时搜索动态库的路径

```cmake
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib") 
```

默认情况下，CMake在构建时会使用构建目录下的库路径（如 `build/lib/`），但安装后会切换到 `${CMAKE_INSTALL_PREFIX}/lib`。此设置强制让构建阶段和安装阶段的 RPATH 一致，避免运行时因路径切换找不到库。

**注意：**需要放在`add_executable`前面

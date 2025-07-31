---
title: CMake使用示例：寻找依赖和自定义库
date: 2025-07-31 22:09:44
tags: CMake
categories: Programing
---
## 一、添加第三方依赖示例

```cmake
cmake_minimum_required(VERSION 3.10)

project(find_demo)

# 添加可执行文件
add_executable(${PROJECT_NAME}
    main.cpp
)

# 寻找gflags库，REQUIRED表示必须找到，否则报错
find_package(gflags REQUIRED)

if(gflags_FOUND)
    message(STATUS "gflags found: ${gflags_INCLUDE_DIRS}")
    message(STATUS "gflags library: ${gflags_LIBRARIES}")

    # 为可执行文件添加头文件目录和库目录
    target_include_directories(${PROJECT_NAME} PRIVATE ${gflags_INCLUDE_DIRS})
    target_link_libraries(${PROJECT_NAME} PRIVATE ${gflags_LIBRARIES})
else()
    message(FATAL_ERROR "gflags not found")
endif()
```



|        变量名         |                             作用                             |
| :-------------------: | :----------------------------------------------------------: |
| `gflags_INCLUDE_DIRS` |        GFlags 头文件目录（如 `/usr/include/gflags/`）        |
|  `gflags_LIBRARIES`   | GFlags 库文件路径（如 `/usr/lib/libgflags.so` 或 `libgflags.a`） |
|    `gflags_FOUND`     |                 布尔值，标记是否找到 GFlags                  |

这些变量是由`find_package(gflags)` 命令自动生成的

## 二、自己编写依赖库

两种情况：

1. 非cmake安装的库：

   需要提供`Find<lib name>.cmake`，如`Finddlib.cmake`

2. 使用cmake安装的库：

   使用`install`安装，生成`<lib name>Config.cmake`文件，适用于导入自己开发的cmake项目

### 1、非cmake安装的库

项目结构：

```bash
.
├── CMakeLists.txt
├── cmake
│   └── Finddlib.cmake
└── main.cpp
```

`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)

project(find_demo)

# 设置CMAKE_MODULE_PATH，添加自定义模块的路径，find_package会从这里查找Findlib.cmake文件
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)

# 使用find_package查找自定义模块，会从CMAKE_MODULE_PATH中查找
find_package(dlib REQUIRED)

if(dlib_FOUND)
    message(STATUS "dlib found: ${dlib_INCLUDE_DIR}")
    message(STATUS "dlib library: ${dlib_LIBRARY}")
    message(STATUS "dlib version: ${dlib_VERSION}")
    message(STATUS "dlib author: ${dlib_AUTHOR}")
    message(STATUS "dlib lib dir: ${dlib_LIBRARY_DIR}")
else()
    message(FATAL_ERROR "dlib not found")
endif()

# 设置RPATH
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH "${dlib_LIBRARY_DIR}")

# 添加可执行文件
add_executable(${PROJECT_NAME}
    main.cpp
)

# 添加头文件
target_include_directories(${PROJECT_NAME} PRIVATE
    ${dlib_INCLUDE_DIR}
)

# 链接动态库
target_link_libraries(${PROJECT_NAME} PRIVATE
    ${dlib_LIBRARY}
)
```

`./cmake/Finddlib.cmake`，顶层`CMakeLists.txt`通过这个文件来查找依赖库

因此，需要自己编写，设置对应的库路径和头文件路径

```cmake
# 找 dlib.h
find_path(dlib_INCLUDE_DIR
    NAMES dlib.h
    PATHS /home/hao/installed/test/include
)

# 找 libdlib.so
find_library(dlib_LIBRARY
    NAMES dlib
    PATHS /home/hao/installed/test/lib
)

# 如果dlib.h和libdlib.so都找到了，就设置dlib_FOUND为true
if(dlib_INCLUDE_DIR AND dlib_LIBRARY)
    set(dlib_FOUND TRUE)

    # 设置dlib的版本和作者信息
    set(dlib_VERSION "1.0.0")
    set(dlib_AUTHOR "hao")

    # lib文件所在的目录
    get_filename_component(dlib_LIBRARY_DIR ${dlib_LIBRARY} DIRECTORY)
else()
    set(dlib_FOUND FALSE)
endif()
```

输出结果：

```bash
❯ cmake -S . -B build
-- The C compiler identification is GNU 11.4.0
-- The CXX compiler identification is GNU 11.4.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- dlib found: /home/hao/installed/test/include
-- dlib library: /home/hao/installed/test/lib/libdlib.so
-- dlib version: 1.0.0
-- dlib author: hao
-- dlib lib dir: /home/hao/installed/test/lib
-- Configuring done
-- Generating done
-- Build files have been written to: /home/hao/cpp-cmake_learn/cmake_projects/custom_find/find_demo/build
```

### 2、cmake安装的库

项目结构：

```bash
.
├── 1.custom_mod
│   ├── CMakeLists.txt
│   ├── cmake
│   │   └── dlibConfig.cmake.in
│   ├── include
│   │   └── dlib.h
│   └── src
│       └── dlib.cpp
└── 2.find_test
    ├── CMakeLists.txt
    └── main.cpp
```

**自己的cmake编写和安装的库：**

`CMakeLists.txt`中的

```cmake
set(CMAKE_INSTALL_PREFIX "/home/hao/installed/test")

# 路径被两次引用，1、编译dlib；2、install export写入config时，所以会报错
# 添加头文件目录
target_include_directories(dlib PUBLIC include) 

# # 使用生成器表达式，解决路径被两次引用的问题
# target_include_directories(dlib PUBLIC
#         $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>  # 生成时的头文件目录
#         $<INSTALL_INTERFACE:include>                    # 安装时的头文件目录
# )

# 设置公共头文件
set_target_properties(dlib PROPERTIES PUBLIC_HEADER include/dlib.h) 

# 添加安装
install(TARGETS dlib
    EXPORT dlib      # 导出目标
    RUNTIME DESTINATION bin # 可执行文件
    LIBRARY DESTINATION lib # 动态库
    ARCHIVE DESTINATION lib # 静态库
    PUBLIC_HEADER DESTINATION include # 公共头文件，只有在设置了 PUBLIC_HEADER 时才会安装，或者使用下面一句
    )


# 通过模板生成 <Package>Config.cmake，用于 find_package() 寻找包，可以包装一些信息，比如版本号等
set(TARGET_NAME dlib)

install(EXPORT dlib
    FILE ${TARGET_NAME}Target.cmake
    DESTINATION config
)

include(CMakePackageConfigHelpers)
# configure_package_config_file 参数：模板文件，生成文件，安装路径
configure_package_config_file(
    ${CMAKE_SOURCE_DIR}/cmake/${TARGET_NAME}Config.cmake.in
    ${TARGET_NAME}Config.cmake
    INSTALL_DESTINATION config
)
# 将生成的 <Package>Config.cmake 安装到 ${CMAKE_INSTALL_PREFIX}/config 目录下
# CMAKE_CURRENT_BINARY_DIR 为当前编译目录
install( FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${TARGET_NAME}Config.cmake
    DESTINATION config
)
```

安装路径为`/home/hao/installed/test`

`dlibConfig.cmake.in`

```cmake
@PACKAGE_INIT@

include(CMakeFindDependencyMacro)

include("${CMAKE_CURRENT_LIST_DIR}/dlibTarget.cmake")
check_required_components("@TARGET_NAME@")

get_target_property(@TARGET_NAME@_INCLUDE_DIR @TARGET_NAME@ INTERFACE_INCLUDE_DIRECTORIES)
set(@TARGET_NAME@_LIBRARIES @TARGET_NAME@)
set(@TARGET_NAME@_AUTHOR ENPEI)
```

**引用库的项目**

顶层`CMakeLists.txt`：

```cmake
# 首先会在CMAKE_MODULE_PATH中查找Finddlib.cmake文件
# 如果找不到，会在CMAKE_PREFIX_PATH中查找dlibConfig.cmake配置文件
# set(CMAKE_PREFIX_PATH  "~/Documents/course_lib/config")

find_package(dlib REQUIRED)

if (dlib_FOUND)
    message(STATUS "dlib_FOUND: ${dlib_FOUND}")
    message(STATUS "dlib_INCLUDE_DIR: ${dlib_INCLUDE_DIR}")
    message(STATUS "dlib_LIBRARIES: ${dlib_LIBRARIES}")
    message(STATUS "dlib_AUTHOR: ${dlib_AUTHOR}")

    # 获取${dlib_INCLUDE_DIR}的父目录
    get_filename_component(dlib_INCLUDE_DIR_PARENT ${dlib_INCLUDE_DIR} DIRECTORY)
    message(STATUS "dlib_INCLUDE_DIR_PARENT: ${dlib_INCLUDE_DIR_PARENT}")

    
else()
    message(FATAL_ERROR "dlib not found")
endif()



# 设置RPATH，否则install后，运行时找不到动态库
SET(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE) 
SET(CMAKE_INSTALL_RPATH "${dlib_INCLUDE_DIR_PARENT}/lib")
```


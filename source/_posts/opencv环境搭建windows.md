---
title: opencv环境搭建windows
date: 2025-07-28 21:39:04
tags: 
categories: OpenCV
---
# 说明

Windows平台，使用CMake构建，Clion开发。官网下载的OpenCV是使用MSVC编译的，所以工具链需要配置为MSVC，否则会有ABI不兼容的问题。如果要用MinGW的话需要从源码编译一遍OpenCV。

# 流程

## 一、设置环境变量

#### 添加系统变量`OpenCV_DIR`

![QQ20250617-151450](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506171515987.png)

#### 添加环境变量

![QQ20250617-151514](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506171515202.png)

**配置好后需要重启电脑**

## 二、设置Clion工具链和CMake配置

#### 工具链配置

![QQ20250617-151714](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506171517203.png)

#### CMake配置

![QQ20250617-151739](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506171518178.png)

## 三、CMakeLists.txt编写

参考

```cmake
# 1. 设置CMake最低版本要求和项目信息
cmake_minimum_required(VERSION 3.10)

# 定义项目名称和版本
project(MyOpenCVProject VERSION 1.0)

# 2. 设置C++标准
# 推荐使用C++11或更高版本
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 3. 查找OpenCV库
# REQUIRED 参数表示如果找不到OpenCV，CMake会报错并停止构建
# COMPONENTS 参数指定你需要使用的OpenCV模块，可以减少最终程序的体积
# 常见的模块有：core, highgui, imgcodecs, imgproc, videoio, objdetect等
find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc)

# 检查是否成功找到OpenCV
if(OpenCV_FOUND)
    message(STATUS "OpenCV found: ${OpenCV_LIBS}")
else()
    message(FATAL_ERROR "OpenCV not found")
endif()

# 4. 包含OpenCV的头文件目录
# 这使得你可以在代码中使用 #include <opencv2/core.hpp> 等
include_directories(${OpenCV_INCLUDE_DIRS})

# 5. 添加源文件并生成可执行文件
# 将你的C++源文件（例如 main.cpp）添加到这里
# ${PROJECT_NAME} 会被替换为上面定义的 "MyOpenCVProject"
add_executable(${PROJECT_NAME} main.cpp)

# 6. 链接OpenCV库
# 将你的可执行文件与找到的OpenCV库链接起来
# ${OpenCV_LIBS} 是一个由find_package创建的变量，包含了所有需要的库文件路径
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})

# 7. (可选) 为Windows平台设置输出目录
# 这段代码可以让你生成的.exe文件和.dll文件都在同一个目录下，方便运行
if(WIN32)
    set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
    set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

    # 在Debug模式下，输出调试信息
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        message(STATUS "Build type: Debug")
    endif()
endif()
```


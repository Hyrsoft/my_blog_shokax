---
title: 使用开源工具链开发STM32
date: 2025-07-28 21:34:59
tags: STM32
categories: MCU/单片机/微控制器
---
## 使用的工具

- vscode 			      编辑器
- cmake  	 	            工程构建工具
- ninja     		             工程编译工具
- openOCD  			烧录工具
- arm-gnu-toolchain          交叉编译器
- clangd                               代码提示

# 一、安装需要的工具链

## 1、下载



## 2、添加到系统环境变量

系统环境变量path

- D:\toochain\ninja
- D:\toochain\arm-gnu-toolchain-14.2\bin
- D:\toochain\clang+llvm-19.1.7\bin
- D:\toochain\OpenOCD-20240916\bin

新建系统环境变量OPENOCD_SCRIPTS

- D:\toochain\OpenOCD-20240916\share\openocd\scripts

# 二、构建工程

## 1、直接用命令行

在工程根目录

```bash
cmake -G Ninja -B build
```

指定Ninja为构建工具，生成到build文件夹中



编译

```bash
cmake --build build --target all
```

或者进入到build目录下，执行

```bash
ninja
```

## 2、设置task

创建`.vscode/tasks.json`文件

创建三个任务：

- CMake configure：用cmake生成ninja、compile_commands.json等文件
- Build：编译
- Clean：删除build和.cache文件夹

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "CMake configure",
            "type": "shell",
            "command": "cmake",
            "args": [
                "-G", "Ninja",
                "-B", "build"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"],
            "detail": "Generated task for CMake configure"
        },
        {
            "label": "Build",
            "type": "shell",
            "command": "cmake",
            "args": [
                "--build", "build",
                "--target", "all"
            ],
            "group": {
                "kind": "build",
                "isDefault": false
            },
            "problemMatcher": ["$gcc"],
            "detail": "Generated task for building the project"
        },
        {
            "label": "Clean",
            "type": "shell",
            "command": "cmd",
            "args": [
                "/c",
                "rmdir /s /q build && rmdir /s /q .cache"
            ],
            "group": {
                "kind": "build",
                "isDefault": false
            },
            "problemMatcher": [],
            "detail": "Generated task for cleaning the build and cache directories"
        }
    ]
}
```

如果是Linux系统，Clean任务需要修改为：

```json
		{
            "label": "Clean",
            "type": "shell",
            "command": "sh",
            "args": [
                "-c",
                "rm -rf build .cache"
            ],
            "group": {
                "kind": "build",
                "isDefault": false
            },
            "problemMatcher": [],
            "detail": "Generated task for cleaning the build and cache directories"
        }
```



## 3、设置快捷按钮

使用vscode插件**Task Buttons**

在`.vscode/settings.json`中自定义按钮

对应tasks.json中定义的三个任务

```json
{
    "clangd.arguments": [
        "--compile-commands-dir=${workspaceFolder}/build"
    ],

    "VsCodeTaskButtons.showCounter": true,
    "VsCodeTaskButtons.tasks": [
        {
            "label": "$(wrench) Configure",
            "task": "CMake configure",
            "tooltip": "Configure the project with CMake"
        },
        {
            "label": "$(tools) Build",
            "task": "Build",
            "tooltip": "Build the project"
        },
        {
            "label": "$(trash) Clean",
            "task": "Clean",
            "tooltip": "Build the project"
        }
    ]
}
```

# 三、修改芯片ram、flash、堆栈

在工程根目录中的`STM32H750XBHx_FLASH.ld`链接脚本中进行修改

### 1、ram、flash大小修改

   ```cmake
   /* Specify the memory areas */
   MEMORY
   {
   DTCMRAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 128K
   RAM (xrw)      : ORIGIN = 0x24000000, LENGTH = 512K
   RAM_D2 (xrw)      : ORIGIN = 0x30000000, LENGTH = 288K
   RAM_D3 (xrw)      : ORIGIN = 0x38000000, LENGTH = 64K
   ITCMRAM (xrw)      : ORIGIN = 0x00000000, LENGTH = 64K
   FLASH (rx)      : ORIGIN = 0x8000000, LENGTH = 128K
   }
   ```

   比如我要用外部存储w25q256，修改flash的地址和长度

   根据图中的配置来修改：

![flash_1](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202505092247882.png)

   ```cmake
   /* Specify the memory areas */
   MEMORY
   {
   DTCMRAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 128K
   RAM (xrw)      : ORIGIN = 0x24000000, LENGTH = 512K
   RAM_D2 (xrw)      : ORIGIN = 0x30000000, LENGTH = 288K
   RAM_D3 (xrw)      : ORIGIN = 0x38000000, LENGTH = 64K
   ITCMRAM (xrw)      : ORIGIN = 0x00000000, LENGTH = 64K
   FLASH (rx)      : ORIGIN = 0x90000000, LENGTH = 32M
   }
   ```

### 2、修改堆栈大小

```c
/* Generate a link error if heap and stack don't fit into RAM */
_Min_Heap_Size = 0x200;      	/* required amount of heap  */
_Min_Stack_Size = 0x400; 		/* required amount of stack */
```



# 四、烧录

## 1、使用STM32CubeProgrammer烧录

参考网上的使用指南，比如[安富莱外部下载算法](https://www.armbbs.cn/forum.php?mod=viewthread&tid=100658)

## 2、使用openOCD进行烧录

参考[用openocd进行烧录和调试](../../环境配置&工具使用/用openocd进行烧录和调试.md)

# 五、其他配置

### 1、添加ninja输出高亮

   在顶层CMakeLists中添加

   ```cmake
   # 启用 Ninja 颜色支持
    add_compile_options(-fdiagnostics-color=always)
   ```

### 2、设置编译出的文件为hex文件

   在顶层cmakelists文件末尾添加：

   ```cmake
   # 添加生成 hex 文件的步骤
   add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
       COMMAND ${CMAKE_OBJCOPY} -O ihex ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.elf ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.hex
       COMMENT "Generating hex file from elf"
   )
   ```

### 3、**设置为从外部flash中启动**

   修改中断向量表偏移地址

   在`./Core/Src/system_stm32h7xx.c`的SystemInit函数末尾添加

   ```c
   void SystemInit (void)
   {
   ......
   #endif /* USER_VECT_TAB_ADDRESS */
   
   #endif /*DUAL_CORE && CORE_CM4*/
     /*偏移中断向量表*/
     SCB->VTOR = 0x90000000;
   }
   ```

### 4、**设置格式化工具**

   在工程根目录中创建`.clang-format`文件

   将格式化代码的形式设置为微软的格式：

   ```yaml
   BasedOnStyle: Microsoft
   AccessModifierOffset: -4
   AlignConsecutiveMacros: true
   AlignTrailingComments: true
   AllowShortFunctionsOnASingleLine: Inline
   AllowShortIfStatementsOnASingleLine: false
   BreakBeforeBraces: Allman
   ColumnLimit: 0
   ```

### 5、**clangd无法识别标准c库**

  

### 6、**启用微库microLIB**

   比如重定向`printf()`标准输出函数到串口

   在顶层CMakeLists中添加

   ```cmake
   # 启用microLIB
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -specs=nosys.specs")
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -specs=nosys.specs")
   ```

### 7、**重定向printf函数**

   前提：启用微库microLIB

   在usart.c中添加

   ```c
   ......
       
   /* 包含头文件 */
   #include <stdarg.h>
   #include "stdio.h"
       
       
   /* 用户函数 */
   /* USER CODE BEGIN 1 */
   __IO int _write(int file, char *ptr, int len)
   {
       (void)file;
   
       HAL_UART_Transmit(&huart1,(uint8_t *)ptr,len, 200);        //串口发送字符数组（字符串）
   
       return len;
   }
   /* USER CODE END 1 */
   
   ......
   ```

   将这两个头文件放在`main.h`中，这样所有文件都可以用`printf()`函数

   **注意**：不同交叉编译器使用的microLIB不一样，重定向printf的方法不一样，上述方法适用于使用`arm-none-eabi-gcc`的工程，比如cmake或者stm32cubeide

### 8、**添加自定义的源文件和头文件路径**

示例：

用户源文件在`./Drivers/user/Src`中，遍历该路径下所有源文件

用户头文件在`./Drivers/user/Inc`中，添加该头文件路径

```cmake
......
# Link directories setup
target_link_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined library search paths
)

# 从./Drivers/user/Src遍历的添加源文件，添加到USER_SOURCES变量中
file(GLOB USER_SOURCES ./Drivers/user/Src/*.c)

# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here
    ${USER_SOURCES}
)

# 添加头文件路径
# Add include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
    ./Drivers/user/Inc
)
......
```

1. 先用`file`命令遍历式的把指定文件夹中的所有源文件都添加到USER_SOURCE变量中，再用已有的`target_sources`添加到可执行文件

2. 使用已有的`target_include_directories`添加头文件路径


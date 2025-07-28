---
title: 设置vim编码格式为utf-8
date: 2025-07-28 21:47:38
tags: VIM
categories: 单板计算机/Linux开发板
---
#### 配置 Vim 实现永久解决 

为了让 Vim 以后新建或打开文件时都默认使用 UTF-8，需要配置 Vim 的启动文件 `.vimrc`。

1. **回到您的终端** (如果还在Vim里，先输入 `:q` 退出)。

2. **编辑 Vim 配置文件**。这个文件位于您的用户主目录下，名为 `.vimrc`。如果不存在，这个命令会自动创建它。

   Bash

   ```
   vim ~/.vimrc
   ```

3. **在文件中加入以下配置**：

   Vim Script

   ```
   " 设置Vim内部使用的编码为UTF-8
   set encoding=utf-8
   
   " 设置Vim保存文件时使用的编码为UTF-8
   set fileencoding=utf-8
   
   " 设置Vim与终端通信时使用的编码为UTF-8
   set termencoding=utf-8
   ```

   - **提示**：在 Vim 配置文件中，`"` 开头的行是注释。

4. **保存并退出**。

   - 在 `nano` 中，按 `Ctrl+X`，然后按 `Y`，再按 `Enter`。


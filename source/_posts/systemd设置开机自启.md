---
title: systemd设置开机自启
date: 2025-07-28 21:43:52
tags: 系统配置
categories: 单板计算机/Linux开发板
---
### **Ubuntu/Debian 使用 `systemd` 设置开机自启服务快速指南**

本文档提供了在基于 `systemd` 的现代 Linux 系统（如 Ubuntu, Debian 等）上创建和管理开机自启服务的标准流程。

#### **第1步：准备您的脚本**

确保您的脚本可以被系统直接执行。

1. 指定解释器 (Shebang)

   在脚本文件的第一行，指明用于执行它的程序。

   ```bash
   #!/usr/bin/env python3 
   # 或者 #!/bin/bash
   ```

2. 放置在标准位置

   将脚本移动到系统的可执行路径下，推荐使用 /usr/local/bin。

   ```bash
   sudo mv your_script.py /usr/local/bin/
   ```

3. 赋予执行权限

   使脚本文件变为可执行。

   ```bash
   sudo chmod +x /usr/local/bin/your_script.py
   ```

------

#### **第2步：创建 `systemd` 服务文件**

创建一个服务单元文件来告诉 `systemd` 如何管理您的脚本。

1. 创建 .service 文件

   在 /etc/systemd/system/ 目录下创建一个以 .service 结尾的新文件。文件名将作为您的服务名。

   ```bash
   sudo nano /etc/systemd/system/myservice.service
   ```

2. 填入模板内容

   将以下模板粘贴到文件中，并根据您的需求修改 Description 和 ExecStart 字段。

   ```ini
   [Unit]
   # 对服务的简短描述
   Description=My Custom Startup Service
   
   # 指定服务在网络准备好之后启动
   After=network.target
   
   [Service]
   # 指定要执行的脚本的完整路径
   ExecStart=/usr/local/bin/your_script.py
   
   # 定义服务失败后总是自动重启
   Restart=always
   
   [Install]
   # 指定服务在多用户模式下启用
   WantedBy=multi-user.target
   ```

------

#### **第3步：管理服务**

使用 `systemctl` 命令来控制您的新服务。

1. 重新加载配置

   在创建或修改服务文件后，必须运行此命令来让 systemd 读取新的配置。

   ```bash
   sudo systemctl daemon-reload
   ```

2. 启用服务

   将服务设置为开机自启。

   ```bash
   sudo systemctl enable myservice.service
   ```

3. 立即启动服务

   无需重启，立即运行您的服务以进行测试。

   ```bash
   sudo systemctl start myservice.service
   ```

4. 检查服务状态

   查看服务是否正在运行，以及查看最新的日志输出，这是调试的关键。

   ```bash
   sudo systemctl status myservice.service
   ```

------

### **常用命令备忘**

- **启用开机自启**： `sudo systemctl enable myservice.service`
- **禁止开机自启**： `sudo systemctl disable myservice.service`
- **启动服务**： `sudo systemctl start myservice.service`
- **停止服务**： `sudo systemctl stop myservice.service`
- **重启服务**： `sudo systemctl restart myservice.service`
- **查看状态和日志**：`sudo systemctl status myservice.service`
- **查看完整日志**： `sudo journalctl -u myservice.service`
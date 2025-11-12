---
title: ADB工具基础用法
date: 2025-11-12 15:30:32
tags: ADB
categories: 环境配置&工具使用
---
### 设备管理 (Device Management)

| 命令 (Command) | 功能说明 (Description) | 示例 (Example) |
| :--- | :--- | :--- |
| `adb devices` | 列出所有已连接的设备及其状态。 | `adb devices -l` |
| `adb root` | 以 root 权限重启 `adbd` 服务，获取完全访问权限。 | `adb root` |
| `adb reboot` | 重启开发板。 | `adb reboot` |
| `adb reboot bootloader` | 重启到 Bootloader 模式。 | `adb reboot bootloader` |
| `adb get-serialno` | 获取设备的序列号。 | `adb get-serialno` |
| `adb get-state` | 获取设备状态 (`device`, `offline`, `bootloader`)。 | `adb get-state` |

### 文件传输 (File Transfer)

| 命令 (Command) | 功能说明 (Description) | 示例 (Example) |
| :--- | :--- | :--- |
| `adb push <本地> <远程>` | 从电脑向开发板推送文件或文件夹。 | `adb push main /data/local/tmp/` |
| `adb pull <远程> <本地>` | 从开发板向电脑拉取文件或文件夹。 | `adb pull /data/anr/traces.txt .` |

### 应用管理 (Application Management - 主要用于Android)

| 命令 (Command) | 功能说明 (Description) | 示例 (Example) |
| :--- | :--- | :--- |
| `adb install <apk路径>` | 安装一个 APK 应用。 | `adb install my_app.apk` |
| `adb uninstall <包名>` | 卸载一个应用。 | `adb uninstall com.example.myapp` |
| `adb shell pm list packages` | 列出设备上所有应用的包名。 | `adb shell pm list packages \| grep myapp` |

### 远程 Shell 与调试 (Remote Shell & Debugging)

| 命令 (Command) | 功能说明 (Description) | 示例 (Example) |
| :--- | :--- | :--- |
| `adb shell` | 进入开发板的交互式 Shell 环境。 | `adb shell` |
| `adb shell <命令>` | 在开发板上执行单条 Shell 命令。 | `adb shell "ps -ef \| grep main"` |
| `adb logcat` | 查看 Android 系统的实时日志。 | `adb logcat` |
| `adb bugreport` | 生成一份包含所有系统信息的完整 bug 报告。 | `adb bugreport ./report.zip` |
| `adb forward` | 端口转发，将主机的端口映射到开发板的端口。 | `adb forward tcp:8080 tcp:80` |
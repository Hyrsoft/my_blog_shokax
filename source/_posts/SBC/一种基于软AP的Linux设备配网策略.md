---
title: 一种基于软AP的Linux设备配网策略
date: 2025-11-12 15:03:52
tags: 网络
categories: SBC
---
# 一种基于软AP的Linux设备配网策略

#### 背景

对于某些设备，比如物联网设备、机器人，本身不带屏幕键鼠，不方便进shell里操作，也没有桌面或者其他GUI可供配网，就可以用软AP策略来进行配网。完整的软AP策略可以让手机链接热点后进入一个web界面，本文章提供一种较为原始但简单的方案。

当设备启动后检测不到网络链接，尝试链接已保存的Wi-Fi也不成功后，它会进入ap模式，提供一个热点供其他设备链接。其他设备链接后，就可以ssh进本设备，在shell里执行配网操作。

本文章的具体思路基于systemd和networkmanager，其他情景思路大致相同。

### 一、创建AP（热点）连接配置文件

我们将创建一个名为 `N100-Setup` 的AP。最关键的设置是 `autoconnect no`，因为我们不希望它在正常情况下（例如在家时）自动启动，我们只希望在脚本调用它时才启动。

```bash
# 请将 'wlan0' 替换为实际的Wi-Fi设备名 (用 `nmcli dev` 查看)
sudo nmcli connection add \
    type wifi \
    ifname wlan0 \
    con-name "N100-Setup" \
    autoconnect no \
    ssid "N100-Setup" \
    802-11-wireless.mode ap \
    802-11-wireless.band bg \
    ipv4.method shared \
    ipv4.addresses 192.168.42.1/24 \
    wifi-sec.key-mgmt wpa-psk \
    wifi-sec.psk "YourPassword"
```

- `con-name "N100-Setup"`: 连接的名称。
- `autoconnect no`: 我们只在需要时手动（通过脚本）启动它，所以设置为不允许自动链接。
- `802-11-wireless.mode ap`: 将此接口设为AP（热点）模式。
- `ipv4.method shared`: NM会自动为此连接启动一个DHCP服务器（在 `192.168.42.0/24` 网段）并处理NAT（如果N100未来有其他方式联网）。
- `ipv4.addresses 192.168.42.1/24`: 定义AP的网关IP。其他设备连上后，会获取一个 `192.168.42.x` 的IP，而N100本身就是 `192.168.42.1`。
- `wifi-sec.*`: 设置WPA2密码。

------

### 二、创建回落检查脚本

我们将创建一个Shell脚本，由 `systemd` 定时调用。

**创建脚本文件：**

```bash
sudo mkdir -p /usr/local/bin
sudo vim /usr/local/bin/check-network-and-fallback.sh
```

存放以下内容

```bash
#!/bin/bash

# 脚本的目的是：检查NM是否处于"完全连接"状态。
# 如果不是，就启动AP回落热点。

# NM状态码: 70 = 已连接 (全局)
# 详情: nmcli general status
#       STATE          CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN
#       connected      full          enabled  enabled  enabled  enabled

# `nmcli -t -f STATE general` 的输出会是 "connected", "connecting", "disconnected"

if nmcli -t -f STATE general | grep -q "^connected"; then
    # 状态是 "connected"，意味着我们已经连上了某个网络（有线或无线）。
    # 任务完成，什么也不做。
    exit 0
else
    # 状态不是 "connected" (可能是 'disconnected' 或 'connecting')
    # 我们认为这是“在新环境”，需要启动AP。
    
    # 使用 systemd-cat 将日志发送到 systemd-journal，方便排查
    systemd-cat -t ap-fallback echo "No network detected. Activating AP fallback 'N100-Setup'."
    
    # 启动我们的AP连接
    # (如果它已启动，此命令也无害)
    nmcli connection up "N100-Setup"
fi
```

**使其可执行：**

```bash
sudo chmod +x /usr/local/bin/check-network-and-fallback.sh
```

------

### 三、创建 `systemd` 服务和定时器

我们需要两个 `systemd` 单元文件：一个 `.service` 文件来运行脚本，一个 `.timer` 文件来定时触发它。

#### 1. 创建 `.service` 文件

```bash
sudo nano /etc/systemd/system/ap-fallback.service
```

粘贴以下内容：

```toml
[Unit]
Description=Activate AP mode fallback if no network is available
# 确保在NetworkManager准备就绪后才运行
After=NetworkManager-wait-online.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/check-network-and-fallback.sh
```

#### 2. 创建 `.timer` 文件

```bash
sudo vim /etc/systemd/system/ap-fallback.timer
```

粘贴以下内容：

```bash
[Unit]
Description=Run AP fallback check 2 minutes after boot

[Timer]
# 在开机1分钟后执行
OnBootSec=1min
# 保持此设置，以便在系统休眠唤醒后也能正确触发
Persistent=true

[Install]
WantedBy=timers.target
```

**参数说明：**

- `OnBootSec=2min`: 我们给N100和NetworkManager **1分钟**的时间去尝试连接所有它已知的Wi-Fi（比如你家里的）。如果1分钟后（由 `ap-fallback.service` 检查）它仍然没连上，就启动AP。

------

### 四、启用并启动服务

1. **重新加载 systemd daemon：**

   ```bash
   sudo systemctl daemon-reload
   ```
   
2. **启用并立即启动定时器：**

   ```bash
   sudo systemctl enable --now ap-fallback.timer
   ```
   
3. **检查定时器状态：**

   ```bash
   systemctl list-timers | grep ap-fallback
   ```
   
   应该能看到 `ap-fallback.timer` 和 `ap-fallback.service` 的条目。

------

### 五、在新环境中的工作流程

1. 给设备通电启动。

2. 设备启动后，`ap-fallback.timer` 开始计时。NetworkManager会尝试连接它所有已知的、`autoconnect yes` 的Wi-Fi网络。

3. 在1分钟的等待时间内，所有尝试都失败了（因为在新环境）。

4. 1分钟后，`ap-fallback.timer` 触发 `ap-fallback.service`。

5. `check-network-and-fallback.sh` 脚本运行。它通过 `nmcli general status` 发现状态是 `disconnected`。

6. 脚本执行 `nmcli connection up "N100-Setup"`。

7. 设备开始广播一个名为 **"N100-Setup"** 的Wi-Fi热点。

8. 其他设备连接到这个 "N100-Setup" Wi-Fi。

9. 连接成功后，终端，SSH登入设备的固定IP：

   ```bash
   ssh hao@192.168.42.1
   ```
   
10. 在设备的Shell中，你运行 `nmcli` 来扫描并连接到*新环境*的Wi-Fi：

    ```bash
    # 1. 添加一个新连接。这个命令*不会*尝试立即连接，它只是创建配置文件。
    sudo nmcli connection add \
        type wifi \
        con-name "Kwrt_5G" \
        ifname wlan0 \
        ssid "Kwrt_5G"
    
    # 2. 为这个新连接设置密码
    sudo nmcli connection modify "Kwrt_5G" \
        wifi-sec.key-mgmt wpa-psk \
        wifi-sec.psk "20542054"
    
    # 3. 确保这个新连接会自动连接
    sudo nmcli connection modify "Kwrt_5G" connection.autoconnect yes
    
    # 4. 主动断开AP
    # 这个命令会立即关闭 N100-Setup AP
    sudo nmcli connection down "N100-Setup"
    ```

12. 当设备成功连接到新环境的Wi-Fi后，`nmcli general status` 就会变为 `connected`。

13. 下次启动时，`check-network-and-fallback.sh` 脚本会在2分钟后运行，但它会检测到状态是 `connected`（因为连上了新环境的Wi-Fi），因此它**不会**再启动AP热点。


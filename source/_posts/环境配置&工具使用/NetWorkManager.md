---
title: NetWorkManager
date: 2025-11-12 15:30:16
tags: 网络
categories: 环境配置&工具使用
---
# 使用 `nmcli` 管理网络连接

`nmcli` 是 NetworkManager 的命令行客户端，是在现代 Linux 发行版（如 Debian、Ubuntu、CentOS）中管理网络连接的强大工具。

## 一、基础查看命令

### 1\. 查看网络设备

使用 `nmcli device` (或 `nmcli device status`) 查看系统中所有网络设备及其状态。

```bash
❯ nmcli device
DEVICE           TYPE      STATE         CONNECTION      
wlp2s0           wifi      已连接        虚空终端        
tailscale0       tun       连接（外部）  tailscale0      
docker0          bridge    连接（外部）  docker0         
lo               loopback  连接（外部）  lo              
br-2dd69cafb18b  bridge    连接（外部）  br-2dd69cafb18b 
...
enp6s0f3u2u4     ethernet  已断开        --              
p2p-dev-wlp2s0   wifi-p2p  已断开        --   
```

### 2\. 查看所有连接配置

使用 `nmcli connection show` 列出所有已保存的网络连接配置。

```bash
[hao@archlinux ~]$ nmcli connection show
NAME        UUID                                  TYPE      DEVICE     
有线连接 1  605f8e47-9a35-3183-80af-447a18307c8b  ethernet  enp2s0    
Kwrt_5G     536344bf-a2df-41cf-9dfe-f71ca37fe4d7  wifi      wlan0     
tailscale0  a8ebcfa3-e44d-472c-9ead-65fc154d2bb7  tun       tailscale0 
...      
N100-Setup  a655b45d-1c87-468f-8a14-914534aa6c63  wifi      --         
```

### 3\. 查看连接的自动连接状态

可以-f指定字段，查看更详细的自启配置。

```bash
[hao@archlinux ~]$ nmcli -f NAME,TYPE,AUTOCONNECT,AUTOCONNECT-PRIORITY connection show
NAME           TYPE      AUTOCONNECT  AUTOCONNECT-PRIORITY 
有线连接 1     ethernet  是           -999                 
Kwrt_5G        wifi      是           0                    
tailscale0     tun       是           0                    
lo             loopback  是           0                    
Hotspot        wifi      否           0                    
N100-Setup     wifi      否           0                    
```

## 二、WiFi 配置

### 1\. 扫描 WiFi 网络

使用以下命令扫描并列出周围可用的 WiFi 网络。

```bash
[hao@archlinux ~]$ nmcli dev wifi list
IN-USE  BSSID              SSID                MODE   CHAN  RATE         SIGNAL  BARS  SECURITY  
        80:AF:CA:C9:50:B3  Kwrt_2.4G           Infra  1     130 Mbit/s   100     ▂▄▆█  WPA2 WPA3 
* 82:AF:CA:C9:50:B4  Kwrt_5G             Infra  36    1170 Mbit/s  99      ▂▄▆█  WPA2 WPA3 
        4C:C6:4C:BE:77:10  Xiaomi_770F         Infra  11    270 Mbit/s   79      ▂▄▆_  WPA1 WPA2 
...
```

### 2\. 连接、断开与删除 WiFi

#### 连接到 WiFi

使用 `connect` 命令连接到指定的 WiFi 网络 (SSID)。连接成功后，NetworkManager 会自动创建一个同名的连接配置文件。

```bash
# 格式: sudo nmcli device wifi connect <SSID> password <密码>
sudo nmcli device wifi connect Kwrt_5G password 20542054
sudo nmcli device wifi connect 503 password 503503503
sudo nmcli device wifi connect 虚空终端 password 503503503
```

#### 设置为开机自动连接

默认情况下，新连接会自动设为自动连接。如果需要手动修改：

```bash
# 格式: sudo nmcli connection modify <连接名称> connection.autoconnect yes
sudo nmcli connection modify Kwrt_5G connection.autoconnect yes
sudo nmcli connection modify 503 connection.autoconnect yes
```

#### 断开连接

可以断开指定网卡（`device disconnect`）或断开某个连接配置（`connection down`）。

```bash
# 方式一：断开指定连接配置
sudo nmcli connection down "Kwrt_5G"

# 方式二：断开指定网卡
nmcli device disconnect wlan0
```

#### 删除配置

永久删除（遗忘）指定名称的连接配置文件。

```bash
# 格式: sudo nmcli connection delete <连接名称>
sudo nmcli connection delete "Kwrt_5G"
sudo nmcli connection delete 虚空终端
```

## 三、有线连接

对于有线连接，如果网卡已插入网线，可以直接 `connect` 设备名。

```bash
# 激活名为 enp6s0f3u2u4 的有线网卡
sudo nmcli device connect enp6s0f3u2u4
```

## 四、设置静态 IP 地址

在某些场景下（如将设备用作服务器或通过旁路由上网），需要将动态 IP (DHCP) 修改为静态 IP。

### 1\. 确定要修改的连接

首先，使用 `nmcli connection show` 找到需要修改的连接配置的名称 (`NAME`)。

```bash
nmcli connection show
# NAME                UUID                                  TYPE      DEVICE 
# 503                 82cbca27-7035-420a-96c0-f54aef5c5f83  wifi      wlan0  
# 有线连接 1          xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  ethernet  enp2s0
```

### 2\. 修改连接为静态 IP

使用 `nmcli connection modify` 命令修改指定连接的网络参数。

```bash
# 语法示例
sudo nmcli connection modify "<连接名称>" \
    ipv4.method manual \
    ipv4.addresses <IP地址>/<子网掩码前缀> \
    ipv4.gateway <网关地址> \
    ipv4.dns "<DNS服务器1>,<DNS服务器2>"

# 示例：将名为 "503" 的 WiFi 连接修改为静态IP
sudo nmcli connection modify "503" \
    ipv4.method manual \
    ipv4.addresses 192.168.31.152/24 \
    ipv4.gateway 192.168.31.141 \
    ipv4.dns "223.5.5.5,114.114.114.114"
    
# 示例：将名为 "有线连接 1" 的有线连接修改为静态IP
sudo nmcli connection modify "有线连接 1" \
    ipv4.method manual \
    ipv4.addresses 192.168.31.68/24 \
    ipv4.gateway 192.168.31.141 \
    ipv4.dns "223.5.5.5,114.114.114.114"
```

| **参数** | **说明** |
| :--- | :--- |
| `ipv4.method` | `manual` 表示静态 IP，`auto` 表示 DHCP。 |
| `ipv4.addresses` | 设置静态 IP 地址和子网掩码（使用 CIDR 表示法）。 |
| `ipv4.gateway` | 设置默认网关地址。 |
| `ipv4.dns` | 设置 DNS 服务器，多个服务器用逗号分隔。 |

### 3\. 应用更改

修改配置后，需要重新激活连接以使设置生效。

```bash
sudo nmcli connection down "503" && sudo nmcli connection up "503"

sudo nmcli connection down "有线连接 1" && sudo nmcli connection up "有线连接 1"
```

### 4\. 恢复为 DHCP

如果需要从静态 IP 切换回动态 IP (DHCP)，只需将 `ipv4.method` 修改为 `auto` 并清空静态配置即可。

```bash
sudo nmcli connection modify "503" ipv4.method auto
# 清空静态配置（可选但推荐）
sudo nmcli connection modify "503" ipv4.addresses "" ipv4.gateway "" ipv4.dns ""

# 重启连接
sudo nmcli connection up "503"
```

## 五、网络测试

配置完成后，可以使用以下工具测试网络连通性。

| **命令** | **用途** |
| :--- | :--- |
| `ping` | 测试与目标主机（IP或域名）的基本连通性和延迟。 `ping 8.8.8.8` |
| `ip addr` | 查看网络接口的 IP 地址、MAC 地址等详细信息。 |
| `iwconfig` | （旧工具）查看无线网卡的连接状态，如 SSID、信号强度 (Link Quality)、比特率 (Bit Rate) 等。 |
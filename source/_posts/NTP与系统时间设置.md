---
title: NTP与系统时间设置
date: 2025-07-28 21:45:22
tags: 系统配置
categories: 单板计算机/Linux开发板
---
## 一、安装基础服务

```bash
sudo apt install systemd-timesyncd
```

#### 如果因为系统时间问题导致apt源不可用

手动设置时间

```bash
sudo date -s "2025-06-26 00:26:00" # 设置为当前时间
```

## 二、修改NTP服务器地址

```bash
sudo vim /etc/systemd/timesyncd.conf
```

添加NTP服务器

```ini
[Time]
NTP=ntp.aliyun.com ntp1.aliyun.com ntp2.aliyun.com
#FallbackNTP=ntp.ubuntu.com
```

重启服务

```bash
sudo systemctl restart systemd-timesyncd
```

检查状态

```bash
timedatectl status

# 理想输出
tspi@localhost:~$ timedatectl status
               Local time: Thu 2025-06-26 00:29:05 CST
           Universal time: Wed 2025-06-25 16:29:05 UTC
                 RTC time: Wed 2025-06-25 16:29:06
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
tspi@localhost:~$ sudo apt update
```


---
title: 调试USB摄像头
date: 2025-08-24 20:21:57
tags: 音视频
categories: SBC
---
# USB摄像头（带麦克风和扬声器）调试
带麦克风咪头和扬声器的USB摄像头
![qq_pic_merged_1756038364912](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/20250824202654485.jpg)

## 摄像头探测

首先，需要确认摄像头设备是否被系统正确识别。

1. 列出USB设备

   使用lsusb命令可以查看所有连接到系统的USB设备。

   ```bash
   hao@lckfb-taishanpi:~$ lsusb
   Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
   Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
   Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
   Bus 004 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
   Bus 004 Device 002: ID 349c:3307 Generic HD video  
   Bus 005 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
   Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
   ```

   其中`HD video`就是我们的USB摄像头。

2. 查看视频设备节点

   系统会为视频设备创建相应的设备文件。

   ```bash
   hao@lckfb-taishanpi:~$ ls /dev/video*
   /dev/video0  /dev/video1  /dev/video2  /dev/video5  /dev/video-dec1  /dev/video-dec2
   ```

3. 使用v4l-utils工具

   v4l-utils是一个强大的视频设备调试工具集，可以提供更详细的设备信息。如果系统未安装，请先安装。

   ```bash
   sudo apt-get install v4l-utils
   ```

   使用`v4l2-ctl`列出所有视频设备及其关联的设备节点。

   ```bash
   hao@lckfb-taishanpi:~$ v4l2-ctl --list-devices
   rockchip,rk3328-vpu-dec (platform:fdea0000.video-codec):
           /dev/video2
           /dev/media1
   
   rockchip,rk3568-vepu-enc (platform:fdee0000.video-codec):
           /dev/video5
           /dev/media3
   
   rockchip-rga (platform:rga):
           /dev/video0
   
   rkvdec2 (platform:rkvdec2):
           /dev/video1
           /dev/media0
   
   HD video  : HD video   (usb-fd840000.usb-1):
           /dev/video3
           /dev/video4
           /dev/media2
   ```

   从输出中可以看到，名为`HD video`的USB摄像头关联了`/dev/video3`和`/dev/video4`两个设备节点。

## 查看摄像头参数

接下来，我们需要确定哪个设备节点是有效的，并查看其支持的视频格式、分辨率和帧率。

```bash
hao@lckfb-taishanpi:~$ v4l2-ctl -d /dev/video4 --list-formats-ext
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

hao@lckfb-taishanpi:~$ v4l2-ctl -d /dev/video3 --list-formats-ext
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'MJPG' (Motion-JPEG, compressed)
                Size: Discrete 1280x720
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.100s (10.000 fps)
                Size: Discrete 800x480
                        Interval: Discrete 0.050s (20.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 640x480
                        Interval: Discrete 0.040s (25.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 480x320
                        Interval: Discrete 0.040s (25.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                Size: Discrete 480x854
                        Interval: Discrete 0.040s (25.000 fps)
                        Interval: Discrete 0.050s (20.000 fps)
```

结果表明，`/dev/video4`不支持视频捕获格式，而`/dev/video3`支持`MJPG`格式，并列出了多种分辨率和帧率的组合。因此，我们应该使用`/dev/video3`作为视频输入设备。

## 使用摄像头

### 拍照

使用`ffmpeg`从摄像头捕获一帧图像并保存为图片。

```bash
ffmpeg -f v4l2 -i /dev/video3 -ss 0.5 -vframes 1 -update 1 <您的文件名.jpg>
```

### 录制视频（无声）

```bash
ffmpeg -f v4l2 -input_format mjpeg -s 1280x720 -r 15 -i /dev/video3 -c:v copy output.mkv
```

在终端里按`Ctrl + C`停止录制。

**参数说明:**

| 参数                  | 说明                                            |
| --------------------- | ----------------------------------------------- |
| `-f v4l2`             | 强制使用`v4l2`驱动。                            |
| `-input_format mjpeg` | 指定输入流是摄像头支持的`mjpeg`格式。           |
| `-s 1280x720`         | 设置视频分辨率。                                |
| `-r 15`               | 设置视频帧率。                                  |
| `-i /dev/video3`      | 指定输入设备。                                  |
| `-c:v copy`           | 直接复制视频流，不进行重新编码，以降低CPU消耗。 |
| `output.mkv`          | 输出的视频文件名。                              |

**注意**: 分辨率和帧率需要选择摄像头支持的组合。根据探测结果，`1280x720`分辨率下支持`15.000 fps`。

## 音频设备探测

使用`aplay`和`arecord`命令探测播放和录音设备。

```bash
hao@lckfb-taishanpi:~$ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: video [HD video], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: RK809 [Analog RK809], device 0: fe410000.i2s-rk817-hifi rk817-hifi-0 [fe410000.i2s-rk817-hifi rk817-hifi-0]
  Subdevices: 0/1
  Subdevice #0: subdevice #0

hao@lckfb-taishanpi:~$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: video [HD video], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: RK809 [Analog RK809], device 0: fe410000.i2s-rk817-hifi rk817-hifi-0 [fe410000.i2s-rk817-hifi rk817-hifi-0]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
```

**设备列表:**

| 设备名称         | 类型        | 声卡号 (Card) | 设备号 (Device) | ALSA标识 |
| ---------------- | ----------- | ------------- | --------------- | -------- |
| **HD video**     | 播放 & 录制 | 0             | 0               | `hw:0,0` |
| **Analog RK809** | 播放 & 录制 | 1             | 0               | `hw:1,0` |

我们的USB摄像头集成了麦克风和扬声器，被识别为声卡0。

## 使用音频设备

### 录制有声视频

结合视频和音频输入，录制带有声音的视频。

```bash
ffmpeg -f v4l2 -input_format mjpeg -i /dev/video3 -f alsa -channels 1 -i hw:0,0 -c:v copy -c:a aac video_with_audio.mp4
```

**新增音频参数说明:**

| 参数          | 说明                                                 |
| ------------- | ---------------------------------------------------- |
| `-f alsa`     | 强制使用`alsa`作为音频驱动。                         |
| `-channels 1` | 设置音频通道为单声道。某些设备可能不支持双声道录制。 |
| `-i hw:0,0`   | 指定音频输入设备（声卡0，设备0）。                   |
| `-c:a aac`    | 指定音频编码器为`aac`。                              |

### 播放音频

- **使用`aplay`播放`.wav`格式文件**

  ```bash
  aplay -D hw:0,0 test_sound.wav
  ```

  `-D hw:0,0`明确指定使用声卡0，设备0进行播放。

- 使用ffplay播放多种格式音频

  ffplay可以播放mp3等多种格式，并且能自动选择默认声卡。

  ```bash
  ffplay music.mp3
  ```
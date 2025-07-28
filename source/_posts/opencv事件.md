---
title: opencv事件
date: 2025-07-28 21:38:01
tags: 
categories: OpenCV
---
## 一、鼠标事件的响应

当鼠标在窗口上进行操作（如点击、移动、滚轮滚动）时，会触发一个鼠标事件。OpenCV允许我们通过设置一个回调函数来响应这些事件。

### 1. 常见鼠标事件

| 事件常量 (event) | 描述 |
| :--- | :--- |
| `EVENT_MOUSEMOVE` | 鼠标移动 |
| `EVENT_LBUTTONDOWN` | 左键按下 |
| `EVENT_RBUTTONDOWN` | 右键按下 |
| `EVENT_MBUTTONDOWN` | 中键按下 |
| `EVENT_LBUTTONUP` | 左键松开 |
| `EVENT_RBUTTONUP` | 右键松开 |
| `EVENT_MBUTTONUP` | 中键松开 |
| `EVENT_LBUTTONDBLCLK` | 左键双击 |
| `EVENT_RBUTTONDBLCLK` | 右键双击 |
| `EVENT_MBUTTONDBLCLK` | 中键双击 |
| `EVENT_MOUSEWHEEL` | 滚轮向前滚动 |
| `EVENT_MOUSEHWHEEL` | 滚轮向后滚动 |

这些事件通常与标志位（flags）结合使用，以判断组合键状态。

| 标志常量 (flags) | 描述 |
| :--- | :--- |
| `EVENT_FLAG_LBUTTON` | 左键被按下 |
| `EVENT_FLAG_RBUTTON` | 右键被按下 |
| `EVENT_FLAG_MBUTTON` | 中键被按下 |
| `EVENT_FLAG_CTRLKEY` | `Ctrl` 键被按下 |
| `EVENT_FLAG_SHIFTKEY` | `Shift` 键被按下 |
| `EVENT_FLAG_ALTKEY` | `Alt` 键被按下 |

### 2. 设置回调函数

通过 `cv::setMouseCallback` 函数为指定窗口注册一个回调函数。当该窗口发生鼠标事件时，注册的函数将被自动调用。

```cpp
// 注册鼠标事件回调函数
// 第一个参数是窗口名称
// 第二个参数是回调函数的指针
// 第三个参数是传递给回调函数的用户自定义数据指针(void*)，默认为0
cv::setMouseCallback(windows_name, on_mouse, &image);
```

### 3. 回调函数实现

下面是一个回调函数的示例，它实现在图像上响应鼠标拖拽并画出红色的线条。

```cpp
/**
 * @brief 鼠标事件回调函数，event和flags共同决定了鼠标具体是什么操作。
 * @param event 鼠标事件类型 (如 EVENT_LBUTTONDOWN)
 * @param x 鼠标在窗口中的x坐标
 * @param y 鼠标在窗口中的y坐标
 * @param flags 鼠标事件标志 (如 EVENT_FLAG_CTRLKEY)
 * @param userdata 用户自定义数据指针
 */
void on_mouse(int event, int x, int y, int flags, void *userdata) {
    using namespace cv;
    static Point per_point;
    static Point cur_point;

    // 将void*类型的userdata转换为cv::Mat类型的引用
    Mat& img = *static_cast<Mat*>(userdata);

    if (event == EVENT_LBUTTONDOWN) { // 左键被按下
        per_point = {x, y}; // 记录按下时的坐标
    } else if (event == EVENT_MOUSEMOVE && flags == EVENT_FLAG_LBUTTON) { // 鼠标移动且左键被按下（拖拽）
        cur_point = {x, y};

        // 在图像上画线
        // Scalar(0, 0, 255) 代表 BGR 红色; thickness 表示线的粗细
        line(img, per_point, cur_point, Scalar(0, 0, 255), 4);

        // 刷新显示
        imshow(windows_name, img);

        // 更新上一个点的位置为当前点
        per_point = cur_point;
    }
}
```

## 二、键盘事件的响应

键盘事件主要通过 `cv::waitKey()` 函数捕获。该函数在指定的毫秒数内等待按键事件。

* **`cv::waitKey(delay)`**:
    * `delay > 0`: 等待 `delay` 毫秒。如果在等待期间有按键按下，函数返回该按键的ASCII码，否则返回 -1。
    * `delay <= 0`: 无限期等待，直到有按键按下，然后返回按键的ASCII码。

下面的代码实现了播放视频的功能，当按下 `ESC` 键时视频暂停（循环结束），再按任意键则程序终止。

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

// 假设视频路径已定义
const std::string video_path = "your_video_file.mp4"; 

int main() {
    cv::VideoCapture cap(video_path);
    if (!cap.isOpened()) {
        std::cerr << "Error: Could not open video file." << std::endl;
        return -1;
    }

    cv::namedWindow("show video", cv::WINDOW_NORMAL);
    cv::resizeWindow("show video", 800, 600);
    
    cv::Mat frame; // 视频帧
    while (true) {
        cap >> frame;
        if (frame.empty()) { // 如果视频播放完毕，退出循环
            break;
        }
        imshow("show video", frame);

        // 等待16ms，如果期间有按键，则捕获其ASCII码
        // 27 是 ESC 键对应的ASCII码
        if (cv::waitKey(16) == 27) { 
            break; // 如果按下ESC键，退出循环，视频暂停
        }
    }

    // 循环结束后，无限期等待下一次按键，然后程序结束
    cv::waitKey(0);

    return 0;
}
```

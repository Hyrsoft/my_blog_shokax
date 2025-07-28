---
title: opencv图像处理模块
date: 2025-07-28 21:38:28
tags: 
categories: OpenCV
---
## 一、颜色空间变换

在数字图像处理中，颜色空间（Color Space）是描述和表示颜色的数学模型。OpenCV默认使用的颜色空间是BGR（蓝-绿-红），但在特定任务中，转换到其他颜色空间会更加有效。

### 1\. `cv::cvtColor` 函数

OpenCV使用 `cv::cvtColor` 函数来进行颜色空间变换。

```cpp
// 原型
void cv::cvtColor(
    cv::InputArray src,     // 输入图像
    cv::OutputArray dst,    // 输出图像
    int code,               // 颜色空间转换代码
    int dstCn = 0           // 目标图像的通道数，0表示根据输入和代码自动确定
);
```

### 2\. 常用颜色空间及其转换

不同的颜色空间有不同的应用场景。

| 颜色空间 | 转换代码 | 描述与用途 |
| :--- | :--- | :--- |
| **Grayscale** (灰度) | `cv::COLOR_BGR2GRAY` | 将彩色图像转换为单通道的灰度图像，只包含亮度信息。常用于简化图像，减少计算量，是许多算法（如边缘检测）的预处理步骤。 |
| **HSV** (色相, 饱和度, 明度) | `cv::COLOR_BGR2HSV` | 一种更符合人类视觉感知的颜色模型。在HSV空间中，颜色（Hue）信息与光照强度（Value）分离，非常适合用于基于颜色的对象分割和跟踪。 |
| **YUV** | `cv::COLOR_BGR2YUV` | 将亮度（Y）与色度（U, V）分量分离。广泛应用于视频编码和传输，因为它允许在不显著影响视觉质量的情况下降低色度信息的分辨率。 |
| **Lab** | `cv::COLOR_BGR2Lab` | 一种感知上均匀的颜色空间，即在空间中两个颜色之间的欧氏距离能更好地对应人眼感知的颜色差异。常用于颜色测量和比较。 |

### 3\. 示例：颜色分割

利用HSV颜色空间可以方便地筛选出特定颜色的区域。`cv::inRange` 函数可以检查图像中的像素值是否在指定的上下限范围内，并生成一个二值掩码（mask）。

  * **`cv::inRange(src, lowerb, upperb, dst)`**:
      * `src`: 输入的HSV图像。
      * `lowerb`: 颜色的下限值（如`cv::Scalar(h_min, s_min, v_min)`）。
      * `upperb`: 颜色的上限值（如`cv::Scalar(h_max, s_max, v_max)`）。
      * `dst`: 输出的单通道二值掩码。在范围内的像素点设置为255（白色），否则设置为0（黑色）。

下面的代码演示了将BGR图像转换为灰度图和HSV图，并使用`cv::inRange`在HSV空间中提取红色部分。

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

// 假设 image_path 已定义
// const std::string image_path = "your_image.jpg";

int main() {
    // 读取原始BGR图像
    const cv::Mat srcImage = cv::imread(image_path);
    if (srcImage.empty()) {
        std::cerr << "Could not open or find the image" << std::endl;
        return -1;
    }

    // 1. 转换为灰度图
    cv::Mat grayImage;
    cv::cvtColor(srcImage, grayImage, cv::COLOR_BGR2GRAY);

    // 2. 转换为HSV图
    cv::Mat hsvImage;
    cv::cvtColor(srcImage, hsvImage, cv::COLOR_BGR2HSV);

    // 3. 在HSV空间中提取红色区域
    // 定义红色的HSV范围，注意OpenCV中H范围为[0, 180]
    const cv::Scalar red_low(156, 43, 46);   // H(色相), S(饱和度), V(明度) 下限
    const cv::Scalar red_up(180, 255, 255);  // H, S, V 上限
    
    cv::Mat red_mask; // 用于存放红色区域的掩码
    // inRange函数会遍历hsvImage中的每个像素，
    // 如果像素值在red_low和red_up之间，则在red_mask对应位置设为255，否则设为0
    cv::inRange(hsvImage, red_low, red_up, red_mask);

    // 显示结果
    cv::imshow("Original BGR", srcImage);
    cv::imshow("Grayscale", grayImage);
    cv::imshow("HSV", hsvImage);
    cv::imshow("Red Part Mask", red_mask);

    cv::waitKey(0);

    return 0;
}
```

#### 实现效果

下图展示了原始图像、灰度图、HSV图以及提取出的红色区域掩码。


![QQ20250619-232100](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506192322418.png)

## 二、Scalar：像素值容器

`cv::Scalar` 是一个可以存储1到4个数值的模板类，常用于表示和传递像素值。它本质上是一个包含四个元素的vector，可以灵活地表示单通道（灰度）、三通道（BGR）或四通道（BGRA）的像素。

| 通道数 | 用途 | 初始化示例 | 说明 |
| :--- | :--- | :--- | :--- |
| **1** | 灰度图像 | `cv::Scalar(128)` | 第一个通道设为128，其余通道默认为0。 |
| **3** | BGR彩色图像 | `cv::Scalar(0, 255, 255)` | 按B, G, R顺序分别赋值。 |
| **4** | BGRA带透明度图像 | `cv::Scalar(0, 255, 255, 255)` | 按B, G, R, Alpha顺序分别赋值。 |

### 在`cv::Mat`构造函数中的应用

在使用`cv::Mat`的构造函数创建图像时，`cv::Scalar`可以用来初始化图像的所有像素。

```cpp
cv::Mat mat(rows, cols, type, scalar);
```

#### `type`参数详解

`type`参数定义了矩阵中每个元素的数据类型和通道数，其格式为 `CV_<bit_depth><S/U/F>C<channels>`。

| 组成部分 | 说明 |
| :--- | :--- |
| **bit\_depth** | 每个通道的位数，如 `8`, `16`, `32`, `64`。 |
| **S/U/F** | 数据类型：`S` (signed-有符号整型), `U` (unsigned-无符号整型), `F` (float-浮点型)。 |
| **C\<channels\>** | 通道数，如 `C1`, `C2`, `C3`, `C4`。 |

**示例解析:**

  * `CV_8UC1`: 8位无符号单通道图像。像素值范围为 [0, 255]。
  * `CV_32FC3`: 32位浮点型三通道图像。像素值范围通常为 [0.0, 1.0]。

**注意**: 当进行频繁的图像处理（如滤波、几何变换）时，使用 `CV_8U` 可能会因其有限的精度（0-255的整数）导致累积误差和图像失真。在这种情况下，推荐使用 `CV_32F` 或 `CV_64F` 等浮点数类型以保留更高的精度。

### 代码示例

```cpp
#include <opencv2/opencv.hpp>

int main() {
    // 示例1: 创建一个灰度图像
    // cv::Scalar(128) -> 单通道，第一个值有效，代表灰度值。0为黑，255为白。
    cv::Scalar scalar1(128);
    cv::Mat mat1(120, 240, CV_8UC1, scalar1);
    cv::imshow("Grayscale Image", mat1);

    // 示例2: 创建一个彩色图像
    // cv::Scalar(0, 255, 255) -> BGR三通道。B=0, G=255, R=255。
    // 使用32F类型，imshow会自动将[0.0, 1.0]范围映射到[0, 255]进行显示。
    cv::Scalar scalar2(0, 255, 255);
    cv::Mat mat2(120, 240, CV_8UC3, scalar2);
    cv::imshow("Color Image", mat2);

    cv::waitKey(0);
    return 0;
}
```

## 三、给图片加边框

```cpp
int main() {
    cv::Mat srcImg = cv::imread(image_path);

    cv::Mat dstImg;

    // 产生随机数的类
    cv::RNG rng(time(NULL)); // 使用当前时间作为种子

    // 特殊类，cv::RNG,用于产生随机数
    while (true) {
        cv::Scalar value(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255)); // 黑色边界
        // 参数：输入图像，输出图像，上下左右边距，边界类型，边界值
        cv::copyMakeBorder(srcImg, dstImg, 50, 50, 50, 50, cv::BORDER_CONSTANT, value);
        // 常用的边界类型：

        cv::namedWindow(windows_name, cv::WINDOW_NORMAL);
        cv::imshow(windows_name, dstImg);
        cv::resizeWindow(windows_name, 800, 480);

        if (cv::waitKey(100) == 27) {
            break;
        }
    }
    return 0;
}
```

示例：

![QQ20250624-225005](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506242254834.png)

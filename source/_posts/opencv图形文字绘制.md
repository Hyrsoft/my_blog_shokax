---
title: opencv图形文字绘制
date: 2025-07-28 21:38:38
tags: 
categories: OpenCV
---
## 一、绘制矩形

使用 `cv::rectangle` 函数在图像上绘制矩形。该函数有两种常见的重载形式：一种使用 `cv::Rect` 对象，另一种使用两个 `cv::Point` 对象（左上角和右下角顶点）。

| 参数 | 说明 |
| :--- | :--- |
| `img` | 目标图像。 |
| `rec` | `cv::Rect` 类型的矩形区域。 |
| `pt1` | 矩形的左上角顶点。 |
| `pt2` | 矩形的右下角顶点。 |
| `color` | 矩形边框的颜色，使用 `cv::Scalar` 表示。 |
| `thickness` | 边框线的粗细。如果为负值（如-1），则表示填充整个矩形。 |
| `lineType` | 线的类型，默认为 `cv::LINE_8`。 |
| `shift` | 坐标点的小数位数。 |

```cpp
#include <opencv2/opencv.hpp>

const std::string image_path = "path/to/your/image.jpg";
const std::string windows_name = "Drawing Shapes";

int main() {
    cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        printf("could not load image...\n");
        return -1;
    }

    // 1. 使用 cv::Rect 定义矩形
    const cv::Rect rec(10, 30, 600, 200);

    // 2. 绘制空心矩形，线条粗细为 5
    cv::rectangle(srcImg, rec, cv::Scalar(0, 0, 255), 5);

    // 3. 绘制实心矩形 (thickness = -1)
    // cv::rectangle(srcImg, rec, cv::Scalar(255, 0, 0), -1);
    
    // 4. 使用两个 cv::Point 定义矩形并绘制
    cv::rectangle(srcImg, cv::Point(10, 30), cv::Point(610, 230), cv::Scalar(0, 255, 255), 2);

    cv::imshow(windows_name, srcImg);
    cv::waitKey(0);
    return 0;
}
```

---

## 二、绘制圆形

使用 `cv::circle` 函数在图像上绘制圆形。

| 参数 | 说明 |
| :--- | :--- |
| `img` | 目标图像。 |
| `center` | 圆心的坐标 `cv::Point`。 |
| `radius` | 圆的半径（以像素为单位）。 |
| `color` | 圆边框的颜色，使用 `cv::Scalar` 表示。 |
| `thickness` | 边框线的粗细。如果为负值（如-1），则表示填充整个圆形。 |
| `lineType` | 线的类型，默认为 `cv::LINE_8`。 |
| `shift` | 坐标点的小数位数。 |

```cpp
#include <opencv2/opencv.hpp>

const std::string image_path = "path/to/your/image.jpg";
const std::string windows_name = "Drawing Shapes";

int main() {
    cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        printf("could not load image...\n");
        return -1;
    }

    // 在图像上绘制一个半径为100，线条粗细为2的绿色圆形
    cv::circle(srcImg, cv::Point(300, 300), 100, cv::Scalar(0, 255, 0), 2);

    cv::imshow(windows_name, srcImg);
    cv::waitKey(0);
    return 0;
}
```

---

## 三、绘制椭圆形

使用 `cv::ellipse` 函数在图像上绘制椭圆或椭圆弧。

| 参数 | 说明 |
| :--- | :--- |
| `img` | 目标图像。 |
| `center` | 椭圆中心的坐标 `cv::Point`。 |
| `axes` | 椭圆长轴和短轴的一半长度，用 `cv::Size` 表示。 |
| `angle` | 椭圆的旋转角度（顺时针方向）。 |
| `startAngle` | 椭圆弧的起始角度。 |
| `endAngle` | 椭圆弧的结束角度。 |
| `color` | 椭圆的颜色，使用 `cv::Scalar` 表示。 |
| `thickness` | 边框线的粗细。如果为负值（如-1），则表示填充整个椭圆。 |

**注意**:
* 通过设置不同的 `startAngle` 和 `endAngle` 可以绘制椭圆弧。
* 将长轴和短轴设置为相等的值，即可绘制出一个（可能旋转了的）圆形或圆弧。

```cpp
#include <opencv2/opencv.hpp>

const std::string image_path = "path/to/your/image.jpg";
const std::string windows_name = "Drawing Shapes";

int main() {
    cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        printf("could not load image...\n");
        return -1;
    }

    // 绘制一个完整的绿色椭圆
    cv::ellipse(srcImg,
                cv::Point(srcImg.cols / 2, srcImg.rows / 2), // 中心点
                cv::Size(200, 100),  // 长轴和短轴长度的一半
                0,                  // 旋转角度
                0,                  // 起始角度
                360,                // 结束角度
                cv::Scalar(0, 255, 0), // 颜色
                2);                 // 粗细

    // 绘制一个红色椭圆弧
    cv::ellipse(srcImg,
                cv::Point(srcImg.cols / 2, srcImg.rows / 2),
                cv::Size(200, 100),
                30,                 // 旋转30度
                0,
                180,                // 绘制上半部分的弧线
                cv::Scalar(0, 0, 255),
                2);

    cv::namedWindow(windows_name, cv::WINDOW_NORMAL);
    cv::imshow(windows_name, srcImg);
    cv::resizeWindow(windows_name, 800, 480);
    cv::waitKey(0);
    return 0;
}
```

---

## 四、绘制线段与多边形

### 1. 绘制线段
使用 `cv::line` 函数绘制一条直线段。

| 参数 | 说明 |
| :--- | :--- |
| `img` | 目标图像。 |
| `pt1` | 线段的起点 `cv::Point`。 |
| `pt2` | 线段的终点 `cv::Point`。 |
| `color` | 线的颜色，使用 `cv::Scalar` 表示。 |
| `thickness` | 线的粗细。 |

### 2. 绘制多边形
* **`cv::polylines`**: 绘制多边形的轮廓。
* **`cv::fillPoly`**: 绘制并填充一个或多个多边形。

| 函数 | 参数 | 说明 |
| :--- | :--- | :--- |
| `cv::polylines` | `img` | 目标图像。 |
| | `pts` | `cv::Point` 类型的数组或向量的数组。 |
| | `isClosed` | 布尔值，指示是否闭合多边形。 |
| | `color` | 颜色。 |
| | `thickness` | 线宽。 |
| `cv::fillPoly` | `img` | 目标图像。 |
| | `pts` | `cv::Point` 类型的数组或向量的数组。 |
| | `color` | 颜色。 |


```cpp
#include <opencv2/opencv.hpp>
#include <vector>

const std::string image_path = "path/to/your/image.jpg";
const std::string windows_name = "Drawing Shapes";

int main() {
    cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        printf("could not load image...\n");
        return -1;
    }

    // 1. 绘制线段 (主对角线)
    cv::line(srcImg, cv::Point(0, 0), cv::Point(srcImg.cols, srcImg.rows), cv::Scalar(0, 255, 0), 2);

    // 2. 准备多边形的顶点
    const std::vector<cv::Point> polygon_points = {
        cv::Point(100, 100),
        cv::Point(250, 150),
        cv::Point(300, 300),
        cv::Point(150, 250)
    };
    
    // 3. 绘制多边形轮廓 (不填充)
    // cv::polylines(srcImg, polygon_points, true, cv::Scalar(0, 255, 255), 3);

    // 4. 绘制填充多边形
    cv::fillPoly(srcImg, polygon_points, cv::Scalar(255, 0, 0));

    cv::namedWindow(windows_name, cv::WINDOW_NORMAL);
    cv::imshow(windows_name, srcImg);
    cv::resizeWindow(windows_name, 800, 480);
    cv::waitKey(0);
    return 0;
}
```

## 五、绘制文字

在OpenCV中，使用 `cv::putText` 函数可以在图像上绘制文字。要精确控制文字的位置（例如居中），则需要借助 `cv::getTextSize` 来预先获取文本的尺寸。

-----

**`cv::putText` 函数**

该函数用于在图像的指定位置绘制文本字符串。

| 参数 | 说明 |
| :--- | :--- |
| `img` | 目标图像。 |
| `text` | 要绘制的文本内容，为 `std::string` 类型。 |
| `org` | 文本框的左下角坐标，为 `cv::Point` 类型。 |
| `fontFace` | 字体类型，如 `cv::FONT_HERSHEY_SIMPLEX`。 |
| `fontScale` | 字体大小的缩放比例。 |
| `color` | 文本颜色，使用 `cv::Scalar` 表示。 |
| `thickness` | 构成文本的线条粗细。 |
| `lineType` | 线条类型，默认为 `cv::LINE_8`。 |

**基本用法:**

```cpp
// 在图像大致中心的位置绘制黑色文字 "Mahiro"
cv::putText(srcImg, "Mahiro", cv::Point(150, 250), cv::FONT_HERSHEY_SIMPLEX, 4.5,
            cv::Scalar(0, 0, 0), 2);
```

> **注意**：`org` 参数指定的是文本基线的左端点，而不是文本框的中心点或左上角。

-----

**将文字居中绘制**

为了将文字的**中心**放置在图像的特定坐标 `(cx, cy)` 上，需要以下步骤：

1.  使用 `cv::getTextSize` 函数获取文本在特定字体、大小和粗细下的像素宽度和高度。
2.  根据获取到的尺寸计算出文本框的中心点。
3.  通过 `(cx, cy)` 和文本尺寸，反向计算出 `cv::putText` 所需的左下角坐标 `(x, y)`。
      * `x = cx - (文本宽度 / 2)`
      * `y = cy + (文本高度 / 2)`

**`cv::getTextSize` 函数**

该函数计算并返回刚好能包裹指定文本的矩形框的大小。

| 参数 | 说明 |
| :--- | :--- |
| `text` | 要测量的文本字符串。 |
| `fontFace`| 字体类型，必须与 `cv::putText` 中使用的一致。 |
| `fontScale`| 字体大小，必须与 `cv::putText` 中使用的一致。 |
| `thickness`| 线条粗细，必须与 `cv::putText` 中使用的一致。 |
| `baseline`| 一个整型指针，函数会通过它返回基线相对于文本框底部的y坐标。|

**完整示例代码**

下面的代码创建一个类，实现计算并在图像中心绘制文本。

```cpp
#include <opencv2/opencv.hpp>
#include <string>

// --- TextDrawer 类 ---
class TextDrawer {
public:
    // 构造函数，用于设置默认文本样式
    TextDrawer(int fontFace = cv::FONT_HERSHEY_SIMPLEX, double fontScale = 1.0, 
               const cv::Scalar& color = cv::Scalar(255, 255, 255), int thickness = 1)
        : m_fontFace(fontFace), m_fontScale(fontScale), m_color(color), m_thickness(thickness) {}

    // 核心方法：在指定中心点绘制居中文字
    void drawCentered(cv::Mat& img, const std::string& text, const cv::Point& centerPoint) const {
        int baseline = 0;
        cv::Size textSize = cv::getTextSize(text, m_fontFace, m_fontScale, m_thickness, &baseline);
        cv::Point textOrigin(centerPoint.x - textSize.width / 2,
                             centerPoint.y + textSize.height / 2);
        
        cv::putText(img, text, textOrigin, m_fontFace, m_fontScale, m_color, m_thickness);
    }
    
    // 也可提供一个绘制非居中文字的方法
    void draw(cv::Mat& img, const std::string& text, const cv::Point& origin) const {
        cv::putText(img, text, origin, m_fontFace, m_fontScale, m_color, m_thickness);
    }

    // 可以提供修改样式的接口
    void setStyle(double newScale, const cv::Scalar& newColor) {
        m_fontScale = newScale;
        m_color = newColor;
    }

private:
    // 成员变量，保存文本样式
    int m_fontFace;
    double m_fontScale;
    cv::Scalar m_color;
    int m_thickness;
};

int main() {
    // ... 加载图像同上 ...
    std::string image_path = "your_image.jpg";
    std::string windows_name = "Class-based Text Drawing";
    cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        srcImg = cv::Mat::zeros(600, 1000, CV_8UC3);
        srcImg.setTo(cv::Scalar(20, 20, 20));
    }
    
    // --- 使用 TextDrawer ---

    // 1. 创建一个用于绘制大标题的 "Drawer"
    TextDrawer titleDrawer(cv::FONT_HERSHEY_SIMPLEX, 4.5, cv::Scalar(255, 255, 255), 3);
    
    // 2. 创建一个用于绘制副标题的 "Drawer"
    TextDrawer subtitleDrawer(cv::FONT_HERSHEY_DUPLEX, 2.0, cv::Scalar(100, 255, 100), 2);

    // 3. 使用它们进行绘制
    cv::Point imageCenter(srcImg.cols / 2, srcImg.rows / 2);
    titleDrawer.drawCentered(srcImg, "Mahiro", imageCenter);
    subtitleDrawer.drawCentered(srcImg, "Class-based Style!", cv::Point(imageCenter.x, imageCenter.y + 200));

    // ... 显示图像 ...
    cv::namedWindow(windows_name, cv::WINDOW_NORMAL);
    cv::imshow(windows_name, srcImg);
    cv::resizeWindow(windows_name, 800, 480);
    cv::waitKey(0);
    return 0;
}
```



## 六、应用示例：提取多边形ROI

在图像处理中，ROI (Region of Interest) 是指图像中我们感兴趣的特定区域。一个常见的任务是根据一个不规则的多边形来定义和提取这个区域。其核心思想是利用掩码（Mask）来标定ROI，然后将掩码应用到原始图像上，从而分离出所需部分。

---

### 实现步骤

1.  **定义多边形顶点**: 首先，使用一个点的向量 `std::vector<cv::Point>` 来定义构成ROI边界的多边形顶点坐标。
2.  **创建与绘制掩码 (Mask)**:
    * 创建一个与原图像尺寸相同、内容全黑的 `cv::Mat` 对象作为掩码。
    * 使用 `cv::fillPoly` 函数，在全黑的掩码上将上一步定义的多边形区域填充为白色（像素值为255）。这样，掩码上白色区域就精确地对应了我们感兴趣的多边形区域。
3.  **应用掩码提取ROI**:
    * 调用 `cv::Mat::copyTo` 方法，并传入上一步生成的掩码。
    * 该方法会遍历掩码，仅将原图像中与掩码白色区域（非零像素）对应的像素复制到目标图像中。掩码黑色区域（零像素）对应的部分则不被复制。

### 代码示例

```cpp
#include <opencv2/opencv.hpp>
#include <vector>

const std::string image_path = "path/to/your/image.jpg";
const std::string windows_name = "Polygon ROI";

int main() {
    const cv::Mat srcImg = cv::imread(image_path);
    if (srcImg.empty()) {
        printf("could not load image...\n");
        return -1;
    }

    // 1. 定义多边形的顶点
    std::vector<cv::Point> points {
        cv::Point(0, 0),
        cv::Point(srcImg.cols / 2, 0),
        cv::Point(srcImg.cols, srcImg.rows / 2),
        cv::Point(0, srcImg.rows)
    };
    
    // 2. 创建一个全黑的掩码
    cv::Mat mask = cv::Mat::zeros(srcImg.size(), srcImg.type());

    // 3. 在掩码上将多边形区域填充为白色
    cv::fillPoly(mask, points, cv::Scalar(255, 255, 255));

    // 4. 使用 copyTo 和掩码来提取ROI
    cv::Mat dstImg;
    srcImg.copyTo(dstImg, mask); // 该操作只复制掩码中像素值不为0的区域

    cv::namedWindow(windows_name, cv::WINDOW_NORMAL);
    cv::imshow(windows_name, dstImg);
    cv::resizeWindow(windows_name, 800, 480);
    cv::waitKey(0);

    return 0;
}
```

### 实现效果

![QQ20250620-231524](https://markdownforyuanhao.oss-cn-hangzhou.aliyuncs.com/img1/202506202320527.png)

* 最终显示的目标图像 `dstImg` 将会是一张背景为黑色的图片。
* 图像中只有被多边形框选的区域会显示原始图像的内容，其他部分（即多边形外部的区域）则为黑色，从而实现了对特定不规则区域的精确提取。

### 应用场景
* **车道线检测**: 在自动驾驶中，可以提取出图像中的车道区域，以减少后续处理的计算量和干扰。
* **物体分割**: 在目标识别任务中，可以用于分割出感兴趣的物体，以便进行后续的特征分析或识别。


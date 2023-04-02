---
title: "YUV基础知识"
date: 2023-03-31T20:58:56+08:00
draft: true
tags: ["流媒体"]
categories: ["音视频"]
---

## RAW数据

RAW数据是指sensor采集到的原始数据。\
每个感光点只能对一种颜色的光进行采样和量化（每个pixel上加一个香像素级的滤光片），RAW8、RAW10、RAW12表示每个像素点有8bit，10bit和12bit。\
RAW数据格式一般采用的是Bayer排列方式，鉴于人眼对绿色波段颜色敏感，所有绿色分量比重最大，按照1:2:1。以2*2像素矩阵，一般Bayer格式有GBRG、GRBG、BGGR、RGGB四种模式，下图为GRBG格式。

![音视频_RAW数据](音视频_RAW数据.bmp)

在实际处理中，每个像素的RGB信号由像素本身输出的某一种颜色信号和其相邻像素输出的其他颜色信号构成，这种采样方式在基本不降低图像质量的同时，可以降低采样率。

由于Bayer模式看起来像一个个马赛克，所以通过ISP-Demosaic（插值运算）为每个pixel恢复完整的RGB数据，成为去马赛克。

ISP常用算法：
1. 亮度调节
2. 对比度调节
3. 饱和度调节
4. gamma校正
5. 去噪
6. 锐化
7. 自动白平衡（AWB）

## RGB

RGB使用红、绿、蓝三原色表示颜色。\
RGB格式是RAW数据通过ISP模块插值计算得来，每个像素均包含RGB三种颜色信息。常见的有RGB565、RGB555、RGB888，如RGB888表示一个像素包含R(8bit)、G(8bit)、B(8bit)信息，共3Byte。

![音视频_RGB格式](音视频_RGB格式.png)

![音视频_RGB形成图像](音视频_RGB形成图像.png)

## YUV

YUV使用亮度Y、色度UV来表示颜色。\
将亮度与色彩信息分离，可以很好的解决彩色电视与黑白电视的兼容问题。 \
通常视频采集芯片输出的都是YUV格式的数据，在此基础上进行编码。

- Y：亮度分量，黑白图片
- U：色度分量（蓝色投影），照片蓝色部分去掉亮度Y
- V：色度分量（红色投影），照片红色部分去掉亮度Y

![音视频_YUV形成图像](音视频_YUV形成图像.png)

### YUV采样方式

相比RGB24格式，利用人眼对Y分量敏感，UV分量不敏感，视频可降低UV分量的采样数据，达到降低数据量、降低带宽压力的目的。

![音视频_YUV采样](音视频_YUV采样.png)

- YUV444：对每个像素点的的YUV分量都进行采样，这样的三个分量信息量完整。
- YUV422：按一行4个像素点，Y全部采样，UV水平间隔采样，所以Y:U:V=4:2:2，相当于每个像素点的采样值由3变为2，可节省1/3存储空间和1/3的数据传输量。
- YUV420（最常用）：并不是指只采样U分量而不采样V分量。而是指，在每一行扫描时，只扫描一种色度分量（U或者V），和Y分量按照2 : 1的方式采样。比如，第一行扫描时，YU 按照 2 : 1的方式采样，那么第二行扫描时，YV分量按照 2:1的方式采样，所以单行来看，Y为4，U/V其中一个为2，另一个为0。对于每个色度分量来说，它的水平方向和竖直方向的采样和Y分量相比都是2:1 。可节省1/2存储空间和1/2的数据传输量。

采样及内存占用：

采样格式 | 采样方式 | 单像素内存占用(byte)
---|---|---
YUV444 | YUV三个分量全采样 | 3
YUV422 | Y全采样，U/V水平间隔采样 | 2
YUV420 | Y全采样，U/V水平/垂直间隔采样 |1.5

### YUV存储格式

YUV存储信息有两种格式：
- packed（打包格式）：YUV分量被连续交替存储在同一个数组中
- planar（平面格式，后缀P）：用三个数据分开存储三个分量
- Semi-planar（半平面格式，后缀SP）：介于上面中间的一种格式，Y单独存储，UV交叉存储

常见的YUV422和YUV420采样的格式有：
 /| YUV422 | YUV420 
---|---|---
packed | YUYV、UYUV | /
P类型 | YUV422P |YV12、YU12
SP类型 | / |NV12、NV21

### YUV422：YUYV/UYVY/YUV422P

```
// YUYV格式：Y0和Y1公用U0 V0分量，Y2和Y3公用U2 V2分量
// (Y0,U0,V0), (Y1,U0,V0), (Y2,U2,V2), (Y3,U2,V2)
Y0 UO Y1 V0  Y2 U2 Y3 V2

// UYVY格式：先采用U分量再采样Y分量
U0 Y0 V0 Y1  U2 Y2 V2 Y3

// YUV422P：3个平面
Y0  Y1  Y2  Y3  Y4  Y5  Y6  Y7 --> Y
Y8  Y9 Y10 Y11 Y12 Y13 Y14 Y15
U0  U2  U4  U6  U8 U10 U12 U14 --> U
V0  V2  V4  V6  V8 V10 V12 V14 --> V
```

### YUV420：YUV420P/YUV420SP
以8*2像素图像为例：
![音视频_YUV420P](音视频_YUV420P.png)

- YU12：先U后V
- YV12：先V后U

![音视频_YUV420SP](音视频_YUV420SP.png)

- NV12：UV交错
- NV21：VU交错

## RGB与YUV转换

这个公式里面所使用的转换矩阵，不同的ColorSpace是不相同的，目前比较常见的colorSpace有BT601、BT709和BT2020。

- RGB转YUV
```
Y = 0.299R + 0.587G + 0.114B
U= -0.147R - 0.289G + 0.436B
V = 0.615R - 0.515G - 0.100B
```

- YUV转RGB
```
R = Y + 1.14V
G = Y - 0.39U - 0.58V
B = Y + 2.03U
```

## 参考
1. [拜尔模板 bayer pattern](https://blog.csdn.net/u010783226/article/details/120516602)
2. [RAW, YUV, RGB, JPEG之间关系](https://blog.csdn.net/qq_39791130/article/details/109093393)
3. [传感器原始图像格式:Bayer RGB 和RGB RAW](https://www.pianshen.com/article/42761065662/)
4. [一文读懂rawRGB、RGB和YUV数据格式与转换](一文读懂rawRGB、RGB和YUV数据格式与转换)
5. [RAW、RGB和YUV格式](https://blog.csdn.net/weixin_42463482/article/details/127702176)
6. [YUV 420、YCbCr 422、RGB 444以及色度二次采样](https://blog.csdn.net/skynullcode/article/details/122190729)
7. [图文理解YUV](https://blog.csdn.net/m0_60259116/article/details/124458889?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168026760216800182189601%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168026760216800182189601&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-124458889-null-null.blog_rank_default&utm_term=YUV&spm=1018.2226.3001.4450)
8. [YUV内存里的存放顺序](https://blog.csdn.net/m0_60259116/article/details/125434762?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168026760216800182189601%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168026760216800182189601&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-4-125434762-null-null.blog_rank_default&utm_term=YUV&spm=1018.2226.3001.4450)
9. [YUV采样方式与存储格式](https://blog.csdn.net/m0_60259116/article/details/126728385?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168026760216800182189601%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=168026760216800182189601&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-3-126728385-null-null.blog_rank_default&utm_term=YUV&spm=1018.2226.3001.4450)
10. [MATLAB：RGB转BT601、BT709协议中各种YUV格式的转换函数](https://blog.csdn.net/llljjlj/article/details/115052159?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522167619676016782427473261%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=167619676016782427473261&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-7-115052159-null-null.142%5Ev73%5Einsert_down3,201%5Ev4%5Eadd_ask,239%5Ev1%5Econtrol&utm_term=colorspaceconversion&spm=1018.2226.3001.4187)
---
title: "数据流:PS和TS"
date: 2023-03-25T18:51:47+08:00
draft: true
tags: ["流媒体"]
categories: ["音视频"]
---

MPEG(moving picture expert group)动态图像专家组开发的标准成为MPEG标准，主要用来阐明视像、音频的编码和解码过程，规定编码后组成位流的语法等。

MPEG-1的编码系统由两部分组成：视像编码和声音编码、系统层上的多路复合；其中MPEG-1 system部分定义了如何将压缩后的音、视频及其他数据进行组合和实现同步，其目的在于生成单一数据位流，便于存储和传输。

MPEG-2对MPEG-1进行扩展，其编码系统由两部分组成：视像编码和声音编码、数据打包和多路数据复合。

MPEG-2将视像、声音和其他数据组合在一起，生成适合存储或传输的基本数据流。数据流由两种类型：PS和TS。这两类都由一个或多个打包的基本数据流PES组合生成。

MPEG-2为了避免不同系统的复杂性和带宽浪费，引入profile(配置)和level(等级)。不同profile使用的编码算法不同，且不同profile按照分辨率和帧率、码率等定义有不同的level。

MPEG定义数据位流结构，按照统一的规范组织视像数据流，保证解码器设计的通用性。MPEG-1将视像序列分成若干个GOP(group of picture：图像组)，把GOP中每帧图像分成许多slice，每个slice分成若干宏块，每个宏块又分为若干图块，如下图所示。

![音视频_mpeg_1_视像数据流结构](音视频_mpeg_1_视像数据流结构.png)

## 术语
- ES流（elementary stream）：基本码流，指编码后的音、视频数据，每个ES只包含一种数据类型。
- PES （packet elementary stream）：将ES根据需要分割为不同长度的数据包，加上包头得到PES。通过将连续的长流分割为短流，便于在网络中分组发送。
- TS（transport stream）：传输流。将视频、音频的ES流和辅助信息**复合**在一起用于实际传输的标准信息流。不同的PES根据共同或独立的时间基准组合成一个TS流。
- PS （program stream）：节目流。总以0x000001BA开始，如果为PS文件，则有且只有一个结束码0x000001B9；网传PS通常没有结束码。

PS与TS的区别：
1. TS是固定包长度为188字节，PS包长可变；
2. TS用于出现错误相对较多的环境下；PS用在出现错误相对比较少的环境。信道环境差的情况下，当同步信息被破坏，对于TS流，接收端可检测后面包的同步信息，从而恢复同步，避免信息丢失；PS包变长，接收端无法在特定位置检查同步信息，丢失同步信息，无法正常播放。

从使用范围上看：
- TS、RTP适用于数据流传输，不具有存储属性；
- PS既有存储属性又可以做实时流传输。

## PS基本封装格式

![音视频_PS分装](音视频_PS分装.png)

- PSH：(Program stream pack header:ps包包头)，0x000001BA。存放系统时间、码率、帧号等信息；
- PS system header：(Partial system header，系统头)，0x000001BB，码率、类型说明等；
- PSM：(Program Stream Map:节目映射流)，0x000001BC，提供节目流中基本流的描述及其相互关系。存放版本号、描述子（basic、encrypt、video、clip、audio）、校验码等。
- PES：负载类型：e0为视频、c0为音频、包长度等。

![音视频_PS封装数据解析](音视频_PS封装数据解析.bmp)

## TS基本封装格式

TS数据包大小必须为188字节，，包括头部(TS Header)和荷载(Payload Data)2部分。

- TS Header：主要包含传输流的头信息，用于传输和包分组，包括固定部分（4字节）和可选部分（adaptation field适配域）。

PID用来识别TS包所承载的数据类型。

## 参考
1. [网络流媒体--PS封装格式](https://blog.csdn.net/jisuanji111111/article/details/120326420)
2. [PS和TS](https://blog.csdn.net/qq_37837061/article/details/116293819)
3. [有关视频传输中TS、PS的释疑（转）](https://blog.csdn.net/woshizhanhun/article/details/3589730?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-3589730-blog-116293819.pc_relevant_aa&spm=1001.2101.3001.4242.5&utm_relevant_index=11)
4. [TS 格式：为什么直播回放的切片一般都用它？丨音视频基础](https://cloud.tencent.com/developer/article/2021489)

[学习音视频技术要看什么书？世界读书日图书推荐](https://cloud.tencent.com/developer/article/1985862?from=article.detail.2216858&areaSource=106000.14&traceId=CfFGLGQIXIT7zpFPawge3)
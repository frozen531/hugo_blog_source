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

## PES

![音视频_PES](音视频_PES.bmp)

PES 包中的部分字段解释：
- packet_start_code_prefix，PES包头起始串，0x00000001
- stream_id，表示基本流的类型和编号。
- PES_packet_length，表示 PES 包中在该字段后的数据字节数，该字段 16 比特。
- PTS，表示显示时间戳。分为 3 段，共 33 比特。
- DTS，表示解码时间戳。分为 3 段，共 33 比特。
- ES_rate，基本流速率，在 PES 流情况中，指定解码器接收 PES 包字节的速率。
- trick_mode_control，表示相关视频的特技方式，3 比特字段。这些特技方式包括：快进、慢动作、冻结帧、快速反向、慢反向等。
- PES_packet_data_byte，表示来自包 stream_id 或 PID 所指示的基本流的连贯数据字节。该字段 8 比特一个单位。

## PS基本封装格式

PS流是对PES的进一步封装，是将具有共同时间基准的一个或多个PES包组合而成的单一的数据流。\
PS流由PS包组成，PS包主要由固定包头、系统头和PES包组成。

![音视频_PS分装](音视频_PS分装.png)

![音视频_PS](音视频_PS.png)
- PSH：(Program stream pack header:ps包包头)，0x000001BA。存放系统时间、码率、帧号等信息；
- PS system header：(Partial system header，系统头)，0x000001BB，码率、类型说明等；
- PSM：(Program Stream Map:节目映射流)，0x000001BC，提供节目流中基本流的描述及其相互关系。存放版本号、描述子（basic、encrypt、video、clip、audio）、校验码等。
- PES：负载类型：e0为视频、c0为音频、包长度等。

![音视频_PS封装数据解析](音视频_PS封装数据解析.bmp)

### PS打包/解包过程

1. 找到起始码0x000001BA，解析头部字段；
2. 判断是否有PSM，根据PSM确定负载的ES流类型；
3. 根据ES流类型解析出具体的ES流数据。

## TS基本封装格式

TS数据包大小必须为188字节，，包括头部(TS Header)和荷载(Payload Data)2部分。TS Header主要包含传输流的头信息，用于传输和包分组，包括固定部分（4字节）和可选部分（adaptation field适配域）。

![音视频_TS](音视频_TS.bmp)

TS包中PID用来识别TS包所承载的数据类型。

PID | 类型
---|---
0x0000 | PAT(Program Association Table:节目关联表)
0x0001 | CAT(Conditional Access Table:条件访问表)

TS流中是多路节目复用的，即多个节目的多个基本流复用后在同一个TS流上传输，那么怎么知道各个节目在传输流中的位置，并区分哪个流属于哪个节目呢？所以就还需要一些附加信息，这就是PSI（Program Specific Information，节目专用信息）。\
标准中规定了4个PSI，分别是节目关联表PAT、节目映射表PMT、条件访问表CAT、网络信息表NIT。\
这些表都由一个或多个子表组成，而子表又进一步由一个或多个section组成，在从PSI表到TS包的转换过程中，section起到了中介的作用。不同的表之间可以通过表标识（table_id）进行区分，属于同一个table_id的不同子表一般通过表的扩展标识（table_id_extension）、版本号（version_number）进行区分，对于子表还要加上其它的字段信息条件。其在这些表中的结构如下图：

![音视频_TS_PSI结构](音视频_TS_PSI结构.bmp)

### PAT(Program Association Table，节目关联表)

PAT（Program Association Table，节目关联表）的PID值固定为'0x0000'， 是PSI的根节点。每个TS流中可能包含一个或多个PAT，所有的这些PAT共同组成了这个TS流中包含的节目列表。PAT列出了TS流中存在哪些节目流，指定了TS流中每个节目对应PMT所在TS包的PID。\
当播放器对视频开始检索分析的时候，针对每个TS 包的header中pid成员进行判定，直到找到PAT表开始的地方进行有效数据起始分析。\
PAT的第一条数据指定了NIT所在TS包的PID，其他数据指定了PMT所在TS包的PID，一个TS流含多少个节目就含有多少PMT。节目关联表PAT的结构图、结构代码、字段信息按顺序展示如下：

![音视频_TS_PAT](音视频_TS_PAT.png)

### PMT(Program Map Table，节目映射表)

解析TS流的时候首先要从PID为0的包里找到节目关联表PAT，因为在PAT中指定了PMT（Program Map Table，节目映射表）所在包的PID。由于PMT中指定了一路节目中各个基本流（视频、音频等）的映射关系，即该节目视频或音频所在TS包的PID，根据指定的PID就可以找到对应的音视频流。总结来说，PMT是用来区分单个节目中的各个基本流，PAT则是区分多路复用中的各个节目。节目映射表的结构图、代码结构、字段解释按顺序解释如下：

![音视频_TS_PMT](音视频_TS_PMT.png)

### CAT(条件访问表CAT)
CAT（Conditional Access Table，条件访问表）所在TS包的PID值为'0x0001'，CAT中列出了条件控制信息（ECM）和条件管理信息（EMM）所在分组的PID，用于节目的加密与解密。CAT的结构图、代码结构按顺序展示如下（相关字段在上面的PAT和PMT中已经出现过了，不需要再解释，参考上面即可）：

![音视频_TS_CAT](音视频_TS_PMT.png)

### NIT(Network Information Table,网络信息表NIT)

NIT（Network Information Table，网络信息表）的PID由PAT中的network_PID字段指定，但NIT的内容是私有的、由用户指定的。它提供TS流的传输信息以及网络自身特性信息，比如网络名称、频道频率、调制特征等信息。

## PS与TS转换

## TS流生成和解析
1）TS 流的生成流程大致如下：
1. 将原始的音视频数据编码后，组成基本码流（ES）；
2. 将基本码流（ES）打包成 PES；
3. 在 PES 中加入需要的信息，比如 PTS、DTS 等；
4. 将 PES 包的数据装载到一系列固定长度为 188 字节的传输包（TS Packet）中；
5. 在 TS 包中加入需要的信息，比如 PSI、PCR 等；
6. 连输输出 TS 包形成具有恒定码率的 TS 流。

2）TS 流的解析流程大致如下：
1. 从 TS 流中解析出 TS 包；
2. 从 TS 包中获取流信息，比如 PSI、PCR 等；
3. 获取特定节目的音视频 PID；
4. 通过 PID 获取特定音视频相关的 TS 包，从中解析出 PES 包；
5. 从 PES 包中获取 PTS、DTS 等时间戳信息，并从 PES 中解析出基本码流（ES）；
6. 将基本码流数据交给解码器，解码出原始音视频数据。

## 参考
1. [网络流媒体--PS封装格式](https://blog.csdn.net/jisuanji111111/article/details/120326420)
2. [PS和TS](https://blog.csdn.net/qq_37837061/article/details/116293819)
3. [有关视频传输中TS、PS的释疑（转）](https://blog.csdn.net/woshizhanhun/article/details/3589730?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-8-3589730-blog-116293819.pc_relevant_aa&spm=1001.2101.3001.4242.5&utm_relevant_index=11)
4. [TS 格式：为什么直播回放的切片一般都用它？丨音视频基础](https://cloud.tencent.com/developer/article/2021489)
5. [学习音视频技术要看什么书？世界读书日图书推荐](https://cloud.tencent.com/developer/article/1985862?from=article.detail.2216858&areaSource=106000.14&traceId=CfFGLGQIXIT7zpFPawge3)
6. [MPEG2 -TS流结构详细浅析](https://blog.csdn.net/m0_60259116/article/details/127211160)
7. [H264解码之PS流解析](https://blog.csdn.net/m0_60259116/article/details/127414345)
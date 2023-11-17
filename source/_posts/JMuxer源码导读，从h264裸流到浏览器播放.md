---
title: JMuxer源码导读，从h264裸流到浏览器播放
date: 2023-06-14 22:00:32
tags: 
- 前端音视频
- H264
categories:
- 前端音视频
---

jMuxer - 一个简单的JavaScript mp4 muxer，可以在浏览器和节点环境中工作。它与通信协议无关，旨在借助媒体源扩展在浏览器上播放媒体文件。它需要 原始 H264 视频数据和/或 AAC 音频数据作为输入。本文将从H264码流结构对应到JMuxer源码上，展示软解码H264裸流到浏览器播放的过程。

## H264码流结构
这里做简单介绍，在编码中更多接触到的是NAL层的操作，所以重点理解NALU一小节，另外还有一些小细节比如一帧图像可以分成多个Slice，那同样的也可能分成多个NALU，那么多个NALU怎么辨认自己是否同一帧等。

详细的码流结构说明可以到[H264码流结构一探究竟 - 掘金 (juejin.cn)](https://juejin.cn/post/7092773284000989191)
### 宏块
视频编码的编码最小单元是宏块，那我们可以从宏块入手，由小至大地近距离观看码流的模样。


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d33ebc6d6fb84eb19bf612b4047ad3f3~tplv-k3u1fbpfcp-watermark.image?)
由于视频中的宏块非常多，所以宏块是这样子组织起来的：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d35bd52b79b744d5b353b6a086b75b13~tplv-k3u1fbpfcp-watermark.image?)

### Slice
如果把宏块当做一箱货物的话，那么Slice可以当做集装箱，它制定了相互传输的格式，将宏快 有组织，有结构，有顺序的形成一系列的码流。Slice 其实是为了并行编码设计的，一般来说是为了提高编码速度的，将一帧图像划分成几个 Slice，并且 Slice 之间相互独立、互不依赖、独立编码。所以帧内预测时候，不能跨Slice预测。所以一帧图像包含一或者若干个slice，一个slice包含若干个宏块。

#### Slice type
1.  I Slice：仅包含I宏块
1.  P Slice：包含P宏块和I宏块
1.  B Slice：包含B宏块和I宏块


### SPS和PPS
在与slice同一层上，还有两个重要的东西，分别是SPS（序列参数集）和PPS（图像参数集），其中，SPS 主要包含的是图像的宽、高、YUV 格式和位深等基本信息；PPS 则主要包含熵编码类型、基础 QP 和最大参考帧数量等基本编码信息。

###### *到这里小结：Slice里面存放着具体的视频编码数据，称为 Video Coding Layer (VCL)，PPS和SPS存放编码相关信息供解码端解码。H264 的码流主要是由 SPS、PPS、I Slice、P Slice和B Slice 组成的。*

### NALU
为了区分同一层次的SPS、PPS、I Slice、P Slice和B Slice，NALU应运而生。
NALU主要由 NALU Header 和 NALU Data 组成，其中 NALU Data 则是SPS、PPS、I Slice、P Slice和B Slice以及其他的一些码流片段；而 NALU Header 如下图：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/566d033b1b514abf8ce6e0dd8e109b47~tplv-k3u1fbpfcp-watermark.image?)

**一共有8位的二进制：1位的禁位 + 2位的优先级 + 5位的 NALU Type ，NALU Type 则是区分 NALU Data 的参数，具体参数表如下：**

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f24135bf7e494b3fa6be856bd3ad9b6d~tplv-k3u1fbpfcp-watermark.image?)

其中 1是 P Slice，5是  I Slice，7是 SPS，8是PPS。


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13ff9b85788f452ea5aa0f52623ca229~tplv-k3u1fbpfcp-watermark.image?)

### 起始码
在实际编码过程中接收到的码流其实是一串十六进制数字，起始码的作用就是用于在一串数字中区分一个NALU的起始位置与上一个NALU的结束位置。

起始码有三字节的00 00 01 与四字节的 00 00 00 01，其中四字节的起始码代表该NALU对应的slice是一帧图像的开始，否则就是三字节的。

另外为了避免起始码与图像编码出来的数据冲突，在编码时每遇到两个字节（连续）的00，就插入一字节0x03，以和起始码相区别。解码时，则将相应的0x03删除掉。

## 关键代码展示

代码截图为我自己实现的demo中截取，大部分对应JMuxer源码，区别在于这是一个面向过程的流程，并且添加上一些注释，更容易理解。

大部分代码都是NAL层（这里不展示VCL层的最大原因是JMuxer里也是直接用了flv.js封装的一些函数，里面大部分都是参数，本人的理解能力有限，真的不是偷懒）
### NALU.js

![code3.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aaa678904ffa49b2b2360ab0b15f3d4f~tplv-k3u1fbpfcp-watermark.image?)
### 起始码切分NALU

![code1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/546d13b89caf433fade35f225b301bb8~tplv-k3u1fbpfcp-watermark.image?)

### 把NALU按一帧分开

![code2.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/347ccbb002924957a6b70bf35c6bfa83~tplv-k3u1fbpfcp-watermark.image?)

### 整个过程
以下就是websocket推码流过来之后JMuxer对码流操作的过程，总体就是在NAL层进行分NALU，分帧，而且要避免传输过来的数据有可能不是一个完整的nalu或帧，要留下一部分数据在下一次推流时处理，相当于一个帧缓冲区。还有SPS的参数集解析，SPS/PPS的解码初始化，到最后就是交给MP4类进行VCL层的操作了，得出来的一个是payload以及payload size，拼接之后就算是一个处理好的码流数据了。最后存放到queue变量中，另外设置定时器或其他方法进行queue的数据读取，相当于一个显示缓冲区。

![code4.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/62f7e07e697848f496aa784ac512f4fd~tplv-k3u1fbpfcp-watermark.image?)

## 完整代码
[web-MSE-h264: jmuxer源码阅读，解码h264裸流，配合MSE实现web端播放 (gitee.com)](https://gitee.com/luoyisen/web-mse-h264)

## 扩展
推过来的码流其实是一个ArrayBuffer，要用Uint8Array去进行转换才能给到jmuxer
Uint8Array：无符号8位二进制整数集合，比如：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09e84c9613c147048686cea44a96a243~tplv-k3u1fbpfcp-watermark.image?)
不难看出前面的0,0,0,1就是起始码了，而紧随其后的103、104就是NALU Header，因为这是无符号的8位二进制整数集合，虽然展示出来的是十进制，但还是要转为二进制来看。比如：

103的二进制就是01100111，可以看出禁位是0，优先级是11，NALU Type就是00111，也就是7，对应的就是SPS；

104的二进制就是01101000，可以看出禁位是0，优先级是11，NALU Type就是01000，也就是8，对应的就是PPS；

而且他们都是h264解码必须有的，所以他们的优先级相应的都是最高的3。同样的还有I Slice、P Slice对应的数字就是101、65，可以看出P Slice的优先级只有2，因为是差异帧，丢了也可以继续解码，但不保证解码的正确性。根据这个优先级，其实也提供了一种优化思路，把优先级为0的NALU过滤掉，也是能一定程度的提高解码速度的。

## 为什么不去看VCL层
展示部分代码片段（真的很想看明白）

![code5.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5578c7b729747438336573b037e4e50~tplv-k3u1fbpfcp-watermark.image?)
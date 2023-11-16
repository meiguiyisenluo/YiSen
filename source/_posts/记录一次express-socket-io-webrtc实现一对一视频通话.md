---
title: 记录一次express+socket.io+webrtc实现一对一视频通话
date: 2022-01-15 08:53:47
tags: 
- 前端音视频
- Webrtc
- JavaScript
- NodeJS
- Socket.io
categories:
- 前端音视频

---

## 为什么要做这个？
   从2020年5月开始从事前端开发工作，这一年半内也接触了好几个项目，最终也是能顺利通过验收，算给两年前努力学习前端的自己一个交代吧。而去年12月刚好接触了一个项目其中涉及到了监控视频的内容（监控组件是项目本身就写好的，我一个页面仔就要做好页面仔的事情），但是身为国家努力培养德智体美全面发展的社会主义建设者和接班人，这我还哪能控制得住自己的好奇心，当即就开启学习模式！！！！！！！！！！

## 起步
   当然一开始没任何头绪的我就是一股脑的去看了项目的源码，毕竟这么个大好资源摆在面前不利用不是傻吗，从源码中的确是了解到了一个核心类 RTCPeerConnection 以及几个事件，但是由于webrtc的一些原理实在过于抽象，事件名字也是让人看得一头雾水，虽然是能跟着实现出来并了解到了代码的执行顺序，但代码到底是在做什么完全能不理解，无法满足我的好奇心，所以，百度我来了！

## 正片开始
> tips：本文会把我读文档以及敲代码的部分都写进来，分为 “文档阅读以及信息整理” 部分以及 “代码实现” 部分，如果对文档阅读不敢兴趣只是想动手实现的话可以直接跳到 “代码实现”  部分

## 一、文档阅读以及信息整理

### 文档链接：

https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API

整篇文档都是在介绍webrtc相关的技术，而我们主要看指南这几篇文章即可
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/326c5d1acd7945baac118b61add3b696~tplv-k3u1fbpfcp-watermark.image?)



### WebRTC 架构概述 与 WebRTC 协议 两篇主要是介绍了WebRTC API底层使用到的网络协议以及链接标准（很概念，读的也是有点一头雾水）

ICE：交互式连接设施[Interactive Connectivity Establishment (ICE)](http://en.wikipedia.org/wiki/Interactive_Connectivity_Establishment) 是一个允许你的浏览器和对端浏览器建立连接的协议框架。

STUN：NAT的会话穿越功能[Session Traversal Utilities for NAT (STUN)](http://en.wikipedia.org/wiki/STUN) (缩略语的最后一个字母是NAT的首字母)是一个允许位于NAT后的客户端找出自己的公网地址，判断出路由器阻止直连的限制方法的协议。

NAT：网络地址转换协议[Network Address Translation (NAT)](http://en.wikipedia.org/wiki/NAT) 用来给你的（私网）设备映射一个公网的IP地址的协议。

TURN：NAT的中继穿越方式[Traversal Using Relays around NAT (TURN)](http://en.wikipedia.org/wiki/TURN) 通过TURN服务器中继所有数据的方式来绕过“对称型NAT”。

SDP：会话描述协议[Session Description Protocol (SDP)](http://en.wikipedia.org/wiki/Session_Description_Protocol) 是一个描述多媒体连接内容的协议，例如分辨率，格式，编码，加密算法等。

### WebRTC 基础 
这篇文章大概就是一个简单的实现了，里面提供了一个demo以及里面的很多代码片段的解析，通篇很长，有兴趣可以去看看，而我能看到的重点如下：

1.
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/726f1e5d64a34a26bbde7cf58a287cc3~tplv-k3u1fbpfcp-watermark.image?)

2.
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95ad23c90bf34531be8df9072c1b1227~tplv-k3u1fbpfcp-watermark.image?)

3.文档中原图（信令事务流程）
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/643121b531c143078b3a58eafbafa5a9~tplv-k3u1fbpfcp-watermark.image?)

4.文档中原图（ICE 候选交换过程）
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9670cd39e89c4188bcbd5fb3b515dd4f~tplv-k3u1fbpfcp-watermark.image?)

以及上面文档链接点进去就能看见的简介第一段文字
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/337ce57e3ed6443f8ea6f5aab42b02bd~tplv-k3u1fbpfcp-watermark.image?)

#### 综合以上五点我能确定的是：<br>
1.WebRtc是实现浏览器点对点视频、音频流传输的技术<br>
2.只要两个对等浏览器之间协商好，他们就可以进行音频、视频通话，不需要其他外力来帮助<br>
3.而两个对等浏览器之间协商的过程也叫**信令**,而且我们需要一个信令服务器来进行协商<br>
4.WebRTC并没有规定或者提供信令服务器，你可以任意选择搭建自己的信令服务器<br>
5.协商过程分为 会话描述（sdp）交换 以及 ICE候选交换 两个过程，对应上面的两个文档原图，也可以从中看到了使用到了哪些WebRTC API，如果有疑惑我们就可以去找指南-WebRTC API概览中查了<br>

> 猜想：ICE候选交换文章中并没有直接说明，但是这篇文章是要构建一个可用的视频通信demo的，那么这个ice候选交换的过程也大概率是必须的



### WebRTC 链接
这篇文章中我找到的重要信息就是这么个信令交换过程，如果细心观察你就会发现其实这个不就是上面的 **文档中原图（信令事务流程）** 的文字描述嘛

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ba0f5003e57f439e803dbb0dec4c582f~tplv-k3u1fbpfcp-watermark.image?)

而且看我发现了什么，这不就正好应征了我上面的猜想嘛，信令协商主要交换的就是 会话描述（sdp）交换 以及 ICE候选交换
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64e5360475ed4b15abb54e7feb95ef0c~tplv-k3u1fbpfcp-watermark.image?)


### WebRTC 会话的生命周期
这篇文章里我也是找到了这个过程，仔细看第三点：  <br><br>每个Peer建立一个track事件的响应程序，这个事件会在远程Peer添加一个track到其stream上时被触发。这个响应程序应将tracks和其消受者联系起来，例如video元素。<br><br>
这里就已经告诉了我们协商好之后怎么把摄像头捕获到的视频流传输到对等浏览器上了。
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33c1e7c3f1bd4c1b839258f1a3ae3028~tplv-k3u1fbpfcp-watermark.image?)



## 二、代码实现
这里面socket.io的知识我就不多做解释了，因为基本都是翻文档能找到的事<br><br>
##### 前端：vue2 + socket.io-client<br>
###### socket.io的客户端的两个封装
![code.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8f89ae8b3834593b2ece9e6b85a41fc~tplv-k3u1fbpfcp-watermark.image?)
###### 前端主要代码
![code.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e412d8fb65bd48efb5cd8d2417e83af8~tplv-k3u1fbpfcp-watermark.image?)
结合上面文档阅读，我也完成了如上代码的编写（当然也有参考别人的），其中 会话描述 的交换过程我用到了 WebRTC 链接 一文中的过程做了备注，特点为备注前面带上序号，而 ICE候选 的交换过程其实就是两个事件，如下：
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2d10029803b4c309c2cedcce0799d0f~tplv-k3u1fbpfcp-watermark.image?)

至于代码的执行顺序我就不多解释了，由于两个对等浏览器用到的是同一个文件（startLive），如果你没有理解上面 文档阅读以及信息整理 部分的内容的话，代码读起来可能会有点难以理解。


##### 信令服务器：nodejs + express + socket.io
这里使用了socket.io的rooms来规定两个人一个房间，并且房间加入满两个人就开始信令协商，而其实关键信令协商的代码只有第60行至62行而已，只是一个简单的信息转发给房间里另一个人的功能（毕竟信令服务器不就是这功能嘛）
![code.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd19db46f063452195640294279747e1~tplv-k3u1fbpfcp-watermark.image?)

## 三、效果演示
PC
![IMG_1416.JPG](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97b172348d7545129d50b85b9f33bbed~tplv-k3u1fbpfcp-watermark.image?)

移动端
![IMG_1421.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00314387fee84dd485e434efc000aa03~tplv-k3u1fbpfcp-watermark.image?)

## 四、延时测试
基本在0.1到0.083秒内

![01.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/231fe647f6ea42499153aa424b6f71f7~tplv-k3u1fbpfcp-watermark.image?)

![01.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dddddcc0d24c464eb079dd381917126e~tplv-k3u1fbpfcp-watermark.image?)

![02.jpeg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c35a3eda5c641e5b71646b461c4171a~tplv-k3u1fbpfcp-watermark.image?)

![03.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/39338ea41f3a49799d47c57a9e0f1cf8~tplv-k3u1fbpfcp-watermark.image?)

![04.jpeg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6d0a648c47d49169ed48bef371e42e1~tplv-k3u1fbpfcp-watermark.image?)

![05.jpeg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/79664137537b4825a3469bc7531b959c~tplv-k3u1fbpfcp-watermark.image?)

![06.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3b5a51f862c4b3c82292c3deaddc27a~tplv-k3u1fbpfcp-watermark.image?)

![07.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e42f2943d9314d5b97bc1a09c3f81af8~tplv-k3u1fbpfcp-watermark.image?)

![08.jpeg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/94de55e3905b4e3ab6251b1517668503~tplv-k3u1fbpfcp-watermark.image?)

![09.jpeg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/777afb44fe1d423582f4e5f18211c44b~tplv-k3u1fbpfcp-watermark.image?)
## 五、源码地址
https://gitee.com/luoyisen/webrtc-test

## 六、写在最后
文章里没有过多的解释webrtc以外的东西，比如socket.io和element和nodejs(express)等，项目里用到的大部分除了webrtc以外的技术都是直接文档copy的，没什么难度，大家可以自己去翻阅一下文档。<br><br>
感慨一下：在做这个小项目的时候其实我几乎是从零开始的，比如nodejs启动https还要自己用openssl给自己颁个证书，socket.io我只是知道有这么个东西然后做过一个helloworld，几乎是没用过的，以前做helloworld的时候还留下个跨域问题放弃了，这次短短两天之内阅读了socket.io的文档并解决了所有的问题。另外还看完了上述webrtc的几篇文章并实现出来了功能，不得不说这大半年内我的进步真的让我自己很惊讶。

<br><br>
好了，就这样，撤退
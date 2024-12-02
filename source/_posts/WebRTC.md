---
title: WebRTC介绍
date: 2018-12-28 18:08:42
tags: 
categories: 音视频
---

## 概述
WebRTC是一个支持网页浏览器进行实时语音对话或视频对话的API，对于原生客户端（例如 Android 和 iOS 应用），可以使用具备相同功能的库。
它支持在对等设备之间发送视频、语音和通用数据，使开发者能够构建强大的语音和视频通信解决方案。

## 1.各种平台上(Browser,Mobile,Native)的实现方案
1.1 Desktop Browser对于WebRTC的支持情况：
* Chrome 支持WebRTC, 实现的功能最多
* Firefox 支持WebRTC, 功能比Chrome略少
* MS Edge 支持ORTC, Object RTC和WebRTC接口有很多不同
* Safari 11开始支持WebRTC
* IE 不支持, 将来也不会支持, 如果你需要支持IE, 需要考虑plugin方案

1.2 Mobile平台如何支持WebRTC?
* Iphone Safari 11开始支持WebRTC
* Android chrome/firefox支持WebRTC, 但需要留意的是, 在很多手机上chrome并不是default browser
* Native Code Android & IOS 都支持


## 2.STUN,TURN,ICE,TCP
多数联网设备都位于局域网中, 并位于防火墙后面, 设备本身只有一个内网的私有IP, 在与外部通信时, 会经过1个或多个NAT路由器, 最终得到一个最外端的一个外部IP, 然后与远端目标机器通讯. 这一网络结构对于web应用, c/s型应用等来说不是问题, 但对于VoIP/P2P等应用就是一个问题了. 通信双方并不知道自己或对方的outermost的外网IP:Port, 如何建立直连呢?
STUN
STUN回答了"What is my IP?"这个问题, 并不参与接下来的media data中转.
TURN
TURN会中转media data, 因此TURN server的通讯压力远大于STUN server.
turn 包含了stun的功能. 所以只需要部署turn服务器即可. 

## 3.流程图
WebRTC 是支持音视频通讯功能的，并且WebRTC是开源的，因此我们可以借助WebRTC实现自己的音视频通讯业务，比如视频会议。无论您是使用PC端，Web浏览器，还是Android、IOS系统手机，您都需要遵循以下流程：

流程图 A
![流程图A](/images/WebRTC1.image)

流程图 B
![流程图B](/images/WebRTC2.image)

## 4.简述逻辑
上述的场景是**ClientA** 和 **ClientB** 进行视频聊天。
**Stun Server**的作用：使用UDP的方式穿透NAT，得到准确的IP地址，还可以中转音视频信息。
**Singal Server**代表信令服务器，它作为**ClientA**和**ClientB**之间通讯桥梁，可以传递信息,可以使用Http协议、WebSoket等方法进行信息传输,还可以实现一些业务逻辑(比如聊天房间的管理)。

下面简单介绍一下整个流程：
 1. ClientA创建PeerConnection对象，然后打开本地音视频设备，将音视频数据封装成MediaStream添加到PeerConnection中。
 2. ClientA调用PeerConnection的CreateOffer方法创建一个用于offer的SDP对象，SDP对象中保存当前音视频的相关参数。ClientA通过PeerConnection的SetLocalDescription方法将该SDP对象保存起来，并通过Signal服务器发送给ClientB。
 3. ClientB接收到ClientA发送过的offer SDP对象，通过PeerConnection的SetRemoteDescription方法将其保存起来，并调用PeerConnection的CreateAnswer方法创建一个应答的SDP对象，通过PeerConnection的SetLocalDescription的方法保存该应答SDP对象并将它通过Signal服务器发送给ClientA。
 4. ClientA接收到ClientB发送过来的应答SDP对象，将其通过PeerConnection的SetRemoteDescription方法保存起来。
 5. 在SDP信息的offer/answer流程中，ClientA和ClientB已经根据SDP信息创建好相应的音频Channel和视频Channel并开启Candidate数据的收集，Candidate数据可以简单地理解成Client端的IP地址信息（本地IP地址、公网IP地址、Relay服务端分配的地址）。
 6. 当ClientA收集到Candidate信息后，PeerConnection会通过OnIceCandidate接口给ClientA发送通知，ClientA将收到的Candidate信息通过Signal服务器发送给ClientB，ClientB通过PeerConnection的AddIceCandidate方法保存起来。同样的操作ClientB对ClientA再来一次。
 7. 这样ClientA和ClientB就已经建立了音视频传输的P2P通道，ClientB接收到ClientA传送过来的音视频流，会通过PeerConnection的OnAddStream回调接口返回一个标识ClientA端音视频流的MediaStream对象，在ClientB端渲染出来即可。同样操作也适应ClientB到ClientA的音视频流的传输。
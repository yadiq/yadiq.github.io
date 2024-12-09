---
title: Android TCP服务端的实现
date: 2022-08-22 18:19:20
tags: 
categories: Android
---

> 传输控制协议（TCP，Transmission Control Protocol）是一种面向连接的、可靠的、基于字节流的传输层通信协议。
TCP通信过程，首先打开服务器监听自己的网络通信端口（假设为7628），打开客户端，设置好要连接的ip地址和服务器的网络通信端口（7628），这样服务器监听到网络通信端口有连接，二者就建立了连接。
本文介绍Android TCP服务端的实现，源码见 github 项目 [AndroidTCP](https://github.com/yadiq/AndroidTCP)。

## 1.TCP服务端开发的步骤

### 1.1 创建ServerSocket，绑定指定端口
```java
//开启服务、指定端口号
if (serverSocket == null) {
	serverSocket = new ServerSocket(ServerPort);
	//serverSocket.setPerformancePreferences(1,0,0);//性能首选项 短连接时间、低延迟和高带宽
	//serverSocket.setReceiveBufferSize(cache.length);//内部套接字接收缓冲区的大小，又用于设置通告给远程对等方的 TCP 接收窗口的大小
	//serverSocket.setReuseAddress(true);//关闭Socket时等待一会
	//serverSocket.setSoTimeout(5*1000);//超时默认为0，无限等待
}
```

### 1.2 监听连接请求，获取输入输出流
```java
//等待客户端的连接，Accept会阻塞，直到建立连接，
clientSocket = serverSocket.accept();
//获取输入流
in = clientSocket.getInputStream();
//获取输出流
out = clientSocket.getOutputStream();
```
### 1.3 接收、发送数据
接收数据：
```java
int len = in.read(container);
if (len > 0) {
	//LogUtil.d("收到数据：" + ByteUtil.bytesToHexWithSpace(container));
	dealData(container, len);
} else {
	isReceive = false;//释放 读取线程
	onConnectError();
}
```
发送数据：
```java
byte[] data = msg.getBytes(Charset.forName("GBK"));
out.write(data);
//LogUtil.d("发送数据：" + ByteUtil.bytesToHexWithSpace(data));

```
### 1.4 定时检查服务允许状态
```java
if (timerWorking) {
    if (timerLength % 3000 == 0) {//每3秒
        checkServerSocket();//检查服务端运行状态
    }
    timerLength += TimerPeriod;//计时更新
}
```
### 1.5 关闭服务
```java
if (serverSocket != null) {
	serverSocket.close();
	serverSocket = null;
}
```

### 1.6 源码
源码位置：
https://github.com/yadiq/AndroidTCP/blob/master/app/src/main/java/com/hqumath/tcp/ui/main/TCPServer.java

## 2.效果图

![Server](https://github.com/yadiq/AndroidTCP/raw/master/img/Server.jpg)![Client](https://github.com/yadiq/AndroidTCP/raw/master/img/Client.png)

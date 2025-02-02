---
title: Gstreamer安装和使用
date: 2022-08-15 21:26:21
tags: 音视频
categories: 音视频
---

> [Gstreamer](https://gstreamer.freedesktop.org)是一个非常强大的、多平台的用于开发流媒体应用程序的框架。GStreamer框架的许多优点都来自于它的模块化。
> 本文使用C++实现了Android平台的Gstreamer播放器，源码见 github 项目 [GstreamerPlayer](https://github.com/yadiq/GstreamerPlayer)。

## 1.Windows安装和使用
### 1.1 安装教程
https://gstreamer.freedesktop.org/documentation/installing/on-windows.html

### 1.2 下载安装包
安装程序分为运行时和开发包，对于开发者需要安装这两个软件。
下载地址：https://gstreamer.freedesktop.org/data/pkg/windows/
版本说明：偶数版本是稳定版，例如1.22.x 1.24.x
开发编译平台：MSVC和MinGW，构建方式不同，这里选择MSVC
安装包：runtime和development，使用对象不同
我们选择下载的安装包名称：gstreamer-1.0-msvc-x86_64-1.24.11.msi

### 1.3 软件安装
选择完整安装，安装位置 C:\gstreamer
自动生成环境变量：
GSTREAMER_1_0_ROOT_MSVC_X86_64 
C:\gstreamer\1.0\msvc_x86_64\

### 1.4 手动配置环境变量
Path %GSTREAMER_1_0_ROOT_MSVC_X86_64%\bin

### 1.5 验证安装是否成功
//查看版本信息
gst-inspect-1.0 --version
//播放视频
gst-launch-1.0 playbin uri=https://www.freedesktop.org/software/gstreamer-sdk/data/media/sintel_trailer-480p.ogv

## 2.Android平台使用
### 2.1 安装教程
https://gstreamer.freedesktop.org/documentation/installing/for-android-development.html

### 2.2 下载安装包
下载地址：https://gstreamer.freedesktop.org/data/pkg/android/
版本选择：1.24.11
安装包名称：gstreamer-1.0-android-universal-1.24.11.tar.xz
对应的NDK版本：r25c
对应的最小API：v21 (Lollipop)

### 2.3 使用Android Studio开发
1. Android平台源码
见 github 项目 [GstreamerPlayer](https://github.com/yadiq/GstreamerPlayer)。
2. tutorial1-5模块
为官方教程代码。
3. app模块
为自己实现的播放器，可以执行Gstreamer Pipline。

![MediaGstreamerInstall1](/images/MediaGstreamerInstall1.png)

## 3 Linux平台使用
### 3.1 验证安装是否成功
ubuntu自带gstreamer库 1.16.3
查看版本 
gst-inspect-1.0 --version
测试播放 
gst-launch-1.0 videotestsrc ! autovideosink
gst-launch-1.0 playbin uri=https://www.w3schools.com/html/movie.mp4

### 3.2 安装GStreamer更多库，见官方教程
https://gstreamer.freedesktop.org/documentation/installing/on-linux.html?gi-language=c#InstallingonLinux-Build

### 3.3 GCC编译
gcc basic-tutorial-1.c -o basic-tutorial-1 `pkg-config --cflags --libs gstreamer-1.0`

### 3.4 VSCode编译
需修改配置文件，添加引用库和编译命令
https://stackoverflow.com/questions/59374956/how-to-configure-visual-studio-code-on-ubuntu-to-compile-gstreamer-files



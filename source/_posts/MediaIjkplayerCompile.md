---
title: ijkPlayer编译&功能优化
date: 2022-08-21 21:26:21
tags: 音视频
categories: 音视频
---

## 1.ijkplayer
1. 概念
[ijkplayer](https://github.com/bilibili/ijkplayer)是BiliBili公司维护的一个开源工程，基于ffmpeg开发的一个播放器软件，支持Android和iOS平台。
支持软硬编解码，支持倍速播放，可以定制化集成需要的功能，集成占用体积也很小。

2. 功能
类库：
avformat: 用于封装/解封装，获取解码所需信息
avcodec: 用于编解码
avutil: 提供公共的工具函数
swscale: 提供图像缩放与像素格式转换
postproc: 提供后期效果处理
avdevice: 用于各平台的设备接入
avfilter: 提供滤镜操作
swresample: 提供音频重采样

命令行工具：
ffmpeg: 用来对视频文件转换格式
ffsever: HTTP多媒体实时广播流服务器
ffplay: 简单的播放器,使用ffmpeg库解析和解码，通过SDL显示。
ffprobe: 收集多媒体文件或流的信息

## 2.ijkplayer编译
### 2.1 编译准备
系统：Ubuntu 20.04

配置环境变量：
export ANDROID_SDK=
export ANDROID_NDK= android-ndk-r10e（必须为该版本）

安装软件：
install git, yasm

### 2.2 正式编译
1. 下载 ijkplayer 源码，master分支
//对应版本 ff4.0--ijk0.8.8--20210426--001
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
cd ijkplayer-android

2. 修改编译配置文件
cd config
rm module.sh
ln -s module-lite-hevc.sh module.sh
//修改配置 config/module.sh
#关闭性能优化
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-linux-perf"
#封装 解封装
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtp"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=rtsp"
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-demuxer=sdp"
#协议
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --enable-protocol=rtp"
#禁止eac3
export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-bsf=eac3_core"

3. 下载 ffmpeg 和 openssl 源码 
//下载指定的 ffmpeg 源码 
./init-android.sh
//下载 openssl 源码（用于支持 https,可不编译）
./init-android-openssl.sh

4. 编译
cd android/contrib
//注意：可编译所有架构或指定架构，all armv7a arm64 x86 x86_64
//编译 openssl（用于支持 https,可不编译）
./compile-openssl.sh clean  
./compile-openssl.sh arm64
//编译 FFmpeg 
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh arm64

5. 使用Android NDK编译ijkplayer
将ijkplayer源码移植到Android项目中，见Github项目
将刚才的编译结果（./build/ffmpeg目录下头文件和so文件），拷贝到项目中。

源码见github项目 [ijkPlayer](https://github.com/yadiq/ijkPlayer)。
支持截图录像，支持直播低延迟播放

![MediaIjkPlayer1](/images/MediaIjkPlayer1.png)
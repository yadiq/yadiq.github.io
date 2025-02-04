---
title: FFmpeg安装和使用
date: 2022-08-20 21:26:21
tags: 音视频
categories: 音视频
---

## 1.FFmpeg概念 

[FFmpeg](https://ffmpeg.org/)是领先的多媒体框架，能够解码、编码、 转码、复用、解复用、流式传输、过滤和播放人类和机器创建的几乎所有内容。
它具有高度的可移植性：FFmpeg 可以在各种构建环境、机器架构和配置下在 Linux、Mac OS X、Microsoft Windows、BSD、Solaris 等平台上编译、运行

## 2.Windows平台安装和使用
1. 下载安装
ffmpeg-n4.4.4-6-gd5fa6e3a91-win64-gpl-shared-4.4.zip
https://github.com/BtbN/FFmpeg-Builds/releases?page=1
2. 配置环境变量
C:\0Develop\ffmpeg\windows-build\ffmpeg-win64-gpl-shared-4.4\bin\
C:\0Develop\ffmpeg\windows-build\ffmpeg-win64-gpl-shared-4.4\bin
3. 验证安装是否成功
ffmpeg -version
ffplay https://www.w3schools.com/html/movie.mp4

## 3.Linux平台安装和使用
1. 安装官方包
sudo apt install ffmpeg
2. 验证安装是否成功
ffmpeg -version
ffplay https://www.w3schools.com/html/movie.mp4

## 4.Android平台编译和使用
1. 官方编译脚本
https://trac.ffmpeg.org/wiki/CompilationGuide/Android
选择 ffmpeg-android-maker
提供编译脚本，有文档
2. 使用教程
https://blog.csdn.net/weiwei9363/article/details/134107335
3. 编译FFmpeg 7.0 版本

//下载仓库
git clone https://github.com/Javernaut/ffmpeg-android-maker
//环境变量
ANDROID_SDK_HOME 
ANDROID_NDK_HOME   NDK r23+
//编译
./ffmpeg-android-maker.sh
//指定ABIS
./ffmpeg-android-maker.sh -abis=arm,arm64
//指定API 21,默认19，API21起支持64位cpu
./ffmpeg-android-maker.sh -android=21
//编译工具 默认gnu
./ffmpeg-android-maker.sh -binutils=llvm

//编译结果
build/ffmpeg 目录下，四种架构的 include 和 lib

//注意可裁剪FFmpeg, 安装额外的插件
例如：x264库，开源的H.264/AVC视频编码器
--enable-gpl --enable-libx265
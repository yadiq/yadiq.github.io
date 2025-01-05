---
title: Linux环境为Android编译OpenSSL和curl
date: 2024-07-02 21:26:21
tags:
categories: Linux
---

> 在Android开发过程中，需要使用Http，Java层有大名鼎鼎的okHttp库，Java语言也有HttpURLConnection可以直接使用。但是在用C/C++开发native层代码时，没有现成的接口可用，需要自己编译curl库。在网络通信过程中，需要对一些数据进行加密，我们可以通过OpenSSL库实现一些加解密算法。

## 1.OpenSSL

### 1.1 OpenSSL概念 
[OpenSSL](https://www.openssl.org/source/)是一个开源的软件库，包含了各种加密算法和协议的实现，如SSL/TLS、AES、RSA、DES、SHA等。它可以用来实现网络通信的安全，包括加密、认证和数据完整性验证等功能。

### 1.2 OpenSSL版本选择 
这里使用 1.1.1w 版本，2023-9-11更新。版本太低时缺少aes_256_gcm算法，版本太高时编译后体积太大。
历史版本下载地址：https://openssl-library.org/source/old/1.1.1/index.html。
github地址：https://github.com/openssl/openssl。

## 2.curl

### 2.1 curl概念 
[curl](https://curl.se/)是一个开源的多协议数据传输开源库，该框架具备跨平台性，开源免费，并提供了包括HTTP、FTP、SMTP、POP3等协议的功能。使用libcurl可以方便地进行网络数据传输操作，如发送HTTP请求、下载文件、发送电子邮件等。如果想让curl支持https，需要依赖openssl，因此需要先编译OpenSSL。

### 2.2 curl版本选择 
这里使用 1.1.1w 版本，2023-9-11更新。版本太低

## 3.编译OpenSSL和curl

### 3.1 本次使用的编译脚本
https://github.com/robertying/openssl-curl-android?tab=readme-ov-file
### 3.2 下载源码
//下载编译脚本
gitclone https://github.com/robertying/openssl-curl-android.git
//切换到根目录，下载OpenSSL和curl指定版本的源码
git clone -b OpenSSL_1_1_1w https://github.com/openssl/openssl.git
git clone -b curl-7_88_1 https://github.com/curl/curl.git
### 3.3 修改openssl编译后的so文件名称
1. 原因
默认生成的so命名为libcrypto.so.1.1，这样样式的在Android中使用System.loadLibrary加载会失败,，因为Android系统不能识别这种so。直接修改so的名字是不行的，因为在so的SONAME内会指出该so的名字。
2. 修改so文件名称为"libcrypto.so"
打开文件 openssl/Configurations/15-android.conf，在 "android" => {...} 中增加一行 shared_extension => ".so"，如下图
![LinuxCompileOpensslCurl1](/images/LinuxCompileOpensslCurl1.png)

### 3.4 开始编译

#### 3.4.1 声明环境变量
//android ndk路径
export NDK=/home/gyd/0Develop/AndroidSDK/ndk/23.0.7599858
//当前操作系统，android ndk使用，详见[ndk官方文档](https://developer.android.com/ndk/guides/other_build_systems#overview)
export HOST_TAG=linux-x86_64
//最低android api版本(编译后的库可向上兼容)
export MIN_SDK_VERSION=21

#### 3.4.2 执行编译脚本
chmod +x ./build.sh
./build.sh

#### 3.4.3 编译脚本介绍
1. 工具链路径
export TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/$HOST_TAG
//配置编译工具链，就是配置C/C++编译器、汇编器、链接器的路径
CC=$TOOLCHAIN/bin/$TARGET_HOST$MIN_SDK_VERSION-clang
CXX=$TOOLCHAIN/bin/$TARGET_HOST$MIN_SDK_VERSION-clang++

2. OpenSSL编译选项
./Configure android-arm64 no-shared \
 -D__ANDROID_API__=$MIN_SDK_VERSION \
 --prefix=$PWD/build/$ANDROID_ARCH
//android-arm64 选择CPU架构
//no-shared 不编译动态库。如果想生成openssl动态库，请去掉 "no-shared"。
//--prefix 编译后安装目录

3. curl编译选项
./configure --host=aarch64-linux-android \
 --target=aarch64-linux-android \
 --with-openssl=$SSL_DIR \
 --with-pic \
 --disable-shared \
 --prefix=$PWD/build/$ANDROID_ARCH
//aarch64-linux-android 选择CPU架构
//--with-openssl 依赖openssl的目录
//--disable-shared 不编译动态库
//--prefix 编译后安装目录

## 4. 编译结果

![LinuxCompileOpensslCurl2](/images/LinuxCompileOpensslCurl2.png)

## 5.参考文章

1. [Android NDK编译zlib、openssl、curl](https://blog.csdn.net/lkun2002/article/details/129595631)

2. [OpenSSL Configure选项说明](https://blog.csdn.net/shb8845369/article/details/100833825)

3. [NDK android curl编译](https://blog.51cto.com/u_16213675/7360568)
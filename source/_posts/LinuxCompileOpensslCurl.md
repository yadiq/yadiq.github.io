---
title: Linux环境为Android编译OpenSSL和curl
date: 2024-07-01 21:26:21
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
[curl](https://curl.se/)是一个开源的多协议数据传输开源库，该框架具备跨平台性，开源免费，并提供了包括HTTP、FTP、SMTP、POP3等协议的功能。使用libcurl可以方便地进行网络数据传输操作，如发送HTTP请求、下载文件、发送电子邮件等。

### 2.2 curl版本选择 
这里使用 1.1.1w 版本，2023-9-11更新。版本太低





#### 1.3.1 MD5 (Message Digest Algorithm 5)
+ 优势：速度较快，适用于小数据集。
+ 劣势：较易碰撞（两个不同的输入产生相同的哈希值），不适合安全性要求高的场景。
#### 1.3.2 SHA-1 (Secure Hash Algorithm 1)
+ 优势：速度适中，适用于小到中等大小的数据集，较MD5更安全。
+ 劣势：不适合用于安全性要求极高的场景，因为已经被证明存在漏洞。
#### 1.3.3 SHA-256 (Secure Hash Algorithm 256-bit)
+ 优势：安全性较高，速度适中，适用于各种数据集大小。
+ 劣势：相对于MD5和SHA-1，速度较慢。
#### 1.3.4 SHA-3 (Secure Hash Algorithm 3)
+ 优势：高度安全，适用于各种数据集大小，较SHA-256更安全。
+ 劣势：相对于SHA-256，速度较慢。
#### 1.3.5 MurmurHash
+ 优势：速度非常快，适用于大数据集，良好的随机性。
+ 劣势：不适用于安全哈希需求，因为它不是密码哈希函数。
#### 1.3.6 CityHash
+ 优势：速度非常快，适用于大数据集，良好的随机性。
+ 劣势：不适用于密码学安全哈希需求，主要用于性能优化。

## 2.APP加密算法的实现

### 2.1 OpenSSL库的集成

[OpenSSL](https://www.openssl.org/source/)是一个开源的软件库，包含了各种加密算法和协议的实现，如SSL/TLS、AES、RSA、DES、SHA等。它可以用来实现网络通信的安全，包括加密、认证和数据完整性验证等功能。
OpenSSL编译过程这里不做介绍，我们可以使用别人编译好的库: [openssl-curl-android](https://github.com/robertying/openssl-curl-android)

### 2.2 AES加密算法

1. 高级加密标准(AES,Advanced Encryption Standard)
AES为分组密码，分组密码也就是把明文分成一组一组的，每组长度相等，每次加密一组数据，直到加密完整个明文。（分组长度只能是128位，也就是说，每个分组为16个字节。密钥的长度可以使用128位、192位或256位。
2. 有六种加密模式
ECB模式（The Electronic Codebook Mode）
CBC模式（The Cipher Block Chaining Mode）
CTR模式（The Counter Mode）
GCM模式（The Galois/Counter Mode）,本质上是CTR模式(计数器模式)加上GMAC
CFB模式（The Cipher Feedback Mode）
OFB模式（The Output Feedback Mode）
3. 参数
Key：对称密钥，它的长度可以为128、192、256bits，用来加密明文的密码。
IV(Initialisation Vector)：初始向量，它的选取必须随机。通常以明文的形式和密文一起传送。它的作用和MD5的”加盐”有些类似，目的是防止同样的明文块，始终加密成同样的密文块。如果不设置，则默认16个0。
AAD(Additional Authenticated Data)：附加身份验证数据。AAD数据不需要加密，通常以明文形式与密文一起传递给接收者。可以为空
Mac tag(MAC标签)：将确保数据在传输和存储过程中不会被意外更改或恶意篡改。该标签随后在解密操作期间使用，以确保密文和AAD未被篡改。在加密时，Mac tag由明文、密钥Key、IV、AAD共同产生。
4. 源码
AES-256-GCM加解密, AES_128_ECB加解密
https://github.com/yadiq/AndroidEncrypt/blob/master/app/src/main/jni_secret/my_aes.cpp
5. 验证。在线加解密工具
https://www.lddgo.net/en/encrypt/aes

### 2.3 SHA加密算法

1. 安全散列算法(Secure Hash Algorithm)
五个类型：SHA-1、SHA-224、SHA-256、SHA-384，SHA-512
后四者有时并称为SHA-2
2. SHA1
SHA-1可以生成一个被称为消息摘要的160位（20字节）散列值，散列值通常的呈现形式为40个十六进制数。
Sha1加密是不可逆的，网上虽然有解密的方法，但只能解密很简单的密码。
```c
SHA1(in, strlen((const char *) in), out);
```
3. HMAC_SHA_256
HMAC(散列消息认证码)是一种使用密码散列函数，同时结合一个加密密钥，通过特别计算方式之后产生的消息认证码(MAC)
```c
out = HMAC(EVP_sha256(), key, key_len, in, strlen((const char *) in), out, &outLen);
```
4. 源码
https://github.com/yadiq/AndroidEncrypt/blob/master/app/src/main/jni_secret/my_sha.cpp
5. 验证。在线加解密工具
https://tool.oschina.net/encrypt?type=2

## 3.效果图

![效果图](/images/AndroidEncrypt.awebp)

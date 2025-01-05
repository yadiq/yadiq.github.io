---
title: Android使用OpenSSL实现加密算法
date: 2024-07-03 21:26:21
tags:
categories: Android
---

> 在Android开发过程中，需要对一些数据进行加密，特别是在网络传输过程中。
本文使用OpenSSL库实现了APP常用的加解密算法，源码见 github 项目 [AndroidNative](https://github.com/yadiq/AndroidNative)。

## 1.OpenSSL

### 1.1 OpenSSL概念 
[OpenSSL](https://www.openssl.org/source/)是一个开源的软件库，包含了各种加密算法和协议的实现，如SSL/TLS、AES、RSA、DES、SHA等。它可以用来实现网络通信的安全，包括加密、认证和数据完整性验证等功能。

### 1.2 OpenSSL的编译
1. 编译过程见上一篇文章
https://yadiq.github.io/2024/07/02/LinuxCompileOpensslCurl/
2. 编译结果见本项目源码
https://github.com/yadiq/AndroidNative/tree/main/app/src/main/jni_demo/otherlibs/openssl

## 2.常用加密算法

### 2.1 对称加密(共享密钥加密算法)
对称加密算法，加密和解密使用同一个秘钥。常用的对称加密算法：AES、DES、3DES。在众多的对称加密算法中，目前来讲AES是最好的，最安全的。
+ 优点：对数据没有长度限制，加解密速度快，比非对称加密快好几个数量级。
+ 缺点：秘钥的传输及保管是个问题，任何一方的秘钥泄漏都将导致数据的不安全，不适合互联网，一般用于内部系统

#### 2.1.1 AES算法(高级加密标准,Advanced Encryption Standard)
1. 概念
AES 加密算法是密码学中的高级加密标准，该加密算法采用对称分组密码体制，密钥长度的最少支持为 128 位、 192 位、256 位，分组长度 128 位。
AES 本身就是为了取代 DES 的，AES 具有更好的 安全性、效率 和 灵活性。
2. 六种加密模式
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

### 2.2 非对称加密(公开密钥加密算法)
非对称加密算法，加密和解密使用不同的秘钥，分为公钥和私钥。常用的非对称加密算法：RSA、DSA、ECC。在非对称加密算法中，RSA用的最多的。
+ 优点：无需关心秘钥的传输问题，只要在服务端把私钥保管好即可。
+ 缺点：加密速度慢，对加密的数据长度有限制，适合小数据量加解密或数据签名。

#### 2.2.1 RSA算法
RSA 加密算法是目前最有影响力的 公钥加密算法，并且被普遍认为是目前 最优秀的公钥方案 之一。
RSA 加密算法 基于一个十分简单的数论事实：将两个大 素数 相乘十分容易，但想要对其乘积进行 因式分解 却极其困难，因此可以将 乘积 公开作为 加密密钥。

### 2.3 哈希算法(散列算法、签名加密算法、摘要算法)
哈希算法，不需要密钥。常用的哈希算法：SHA-1、MD5。
用于将数据快速映射为固定长度的值，通常用于数据结构、数据检索、数据完整性验证等各种领域。

#### 2.3.1 MD5 (Message Digest Algorithm 5)
无论是多长的输入，MD5 都会输出长度为 128bits 的一个串 (通常用 16 进制 表示为 32 个字符)。
它的典型应用是对一段信息产生信息摘要，以防止被篡改。
+ 优势：速度较快，适用于小数据集。
+ 劣势：较易碰撞（两个不同的输入产生相同的哈希值），不适合安全性要求高的场景。

#### 2.3.2 SHA-1 (Secure Hash Algorithm 1)
对于长度小于 2 ^ 64 位的消息，SHA1 会产生一个 160 位的 消息摘要 (通常用 16 进制 表示为 40 个字符)。
它一般而言不可逆，可以被应用在检查文件完整性，以及数字签名等场景。
+ 优势：速度适中，适用于小到中等大小的数据集，较MD5更安全。
+ 劣势：不适合用于安全性要求极高的场景，因为已经被证明存在漏洞。

#### 2.3.2 SHA-256 (Secure Hash Algorithm 256-bit)
SHA1 会产生一个 256 位的 消息摘要 (通常用 16 进制 表示为 64 个字符)。
+ 优势：安全性较高，速度适中，适用于各种数据集大小。
+ 劣势：相对于MD5和SHA-1，速度较慢。

#### 2.3.3 HMAC算法
Hmac算法是一种标准的基于密钥的哈希算法，可以配合MD5、SHA-1等哈希算法，计算的摘要长度和原摘要算法长度相同。
Hmac算法总是和某种哈希算法配合起来用的，可选以下多种算法：HmacMD5/HmacSHA1/HmacSHA256/HmacSHA384/HmacSHA51。

## 3.加密算法的实现

### 3.1 AES加解密算法
已实现的加解密算法：aes_256_gcm、aes_256_cbc、aes_128_ecb，对密文进行base64编码。
```
    /**
     * aes_256_gcm加密函数
     * @param plaintext 明文
     * @return 密文base64编码
     */
    static string aes256gcmEncrypt(const char *plaintext, const void *key, const char *iv);
    /**
     * aes_256_gcm解密函数
     * @param ciphertext 密文base64编码
     * @return 明文
     */
    static string aes256gcmDecrypt(const char *cipherBase64, const void *key, const char *iv);

    /**
     * aes_256_cbc加密函数
     * @param plaintext 明文
     * @return 密文base64编码
     */
    static string aes256cbcEncrypt(const char *plaintext, const void *key, const char *iv);
    /**
     * aes_256_cbc解密函数
     * @param ciphertext 密文base64编码
     * @return 明文
     */
    static string aes256cbcDecrypt(const char *cipherBase64, const void *key, const char *iv);

    /**
     * aes_128_ecb加密函数
     * @param plaintext 明文
     * @return 密文base64编码
 */
    static string aes128ecbEncrypt(const char *plaintext, const void *key);
    /**
     * aes_128_ecb解密函数
     * @param ciphertext 密文base64编码
     * @return 明文
     */
    static string aes128ecbDecrypt(const char *cipherBase64, const void *key);
```
源码见：
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/utils_openssl/AesUtil.cpp

### 3.2 哈希加密算法
已实现的哈希加密算法：SHA-1、HMAC算法，密文转十六进制字符串。
```
    /**
     * SHA1加密
     * @param plaintext 明文
     * @return 密文转十六进制字符串
     */
    static string sha1Encrypt(const char *plaintext);

    /**
     * HMAC加密
     * @param algorithm 哈希算法名称：MD5、SHA1、SHA224、SHA256、SHA384、SHA512
     * @param plaintext 明文
     * @param key 密钥
     * @return 密文转十六进制字符串
     */
    static string hmacEncrypt(const char *algorithm, const char *plaintext, const char *key);
```
源码见：
https://github.com/yadiq/AndroidNative/blob/main/app/src/main/jni_demo/utils_openssl/HashUtil.cpp

## 5. 验证
在线加解密工具:
https://www.lddgo.net/en/encrypt/aes
https://tool.oschina.net/encrypt?type=2

## 4.效果图
![AndroidOpensslEncrypt](/images/AndroidOpensslEncrypt.png)

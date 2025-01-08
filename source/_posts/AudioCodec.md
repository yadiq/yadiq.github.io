---
title: 音频编解码
date: 2022-08-03 21:26:21
tags:
categories: 音视频
---


## 1.数字音频基础概览

### 1.1 音频数字信号的实现过程
模拟信号=>采样=>量化=>编码=>封装

### 1.1 采样率

对声音信号每秒采样次数，单位Hz
常见：8000Hz、16000Hz、44100Hz、48000Hz

### 1.2 声道布局
+ 单声道
数量小，缺乏对声音位置的定位
+ 立体声道
由左右声道组成，改善对声音位置的定位
+ 四声环绕
由前左、前右、后左、后右组成。
+ 5.1声道
四声环绕基础上，增加中场声道、低音，例如杜比音效

### 1.3 采样格式/采样位数
每个采样占用多少位
常见：8位、16位、32位、64位

### 1.4 编码格式
音频轨数据是经过编码的
常见：mp3、aac、ac3、opus、g711
+ pcm 
将声音等模拟信号变成符号化的脉冲列
+ wav 
微软,文件头=音频流的编码参数可以是pcm编码(采样率，声道数，采样位数，音频数据大小)
进行音频编辑操作,先解码为wav或pcm文件
+ aac
基于MPEG-4标准，作为MP3的后继者。
在相同的比特率之下，AAC相较于MP3通常可以达到更好的声音质量

### 1.5 封装格式
文件标识头+多媒体信息+音频轨数据
常见：mp3、m4a、flac、wav


## 2.常用音频编码类型

### 2.1 音频编码分两类
speech codec：（语音编解码器）
    G.711, G.723, G.726 , G.729, ILBC, QCELP, EVRC, AMR, SMV
audio codec：
    real audio, AAC, AC3, MP3, WMA, SBC等

### 2.1 G711
对称加密算法，加密和解密使用同一个秘钥。常用的对称加密算法：AES、DES、3DES。在众多的对称加密算法中，目前来讲AES是最好的，最安全的。
+ 优点：对数据没有长度限制，加解密速度快，比非对称加密快好几个数量级。
+ 缺点：秘钥的传输及保管是个问题，任何一方的秘钥泄漏都将导致数据的不安全，不适合互联网，一般用于内部系统

## 3.音频编码的实现

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


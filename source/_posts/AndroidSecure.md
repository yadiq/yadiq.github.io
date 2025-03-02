---
title: Android安全与逆向
date: 2024-07-06 21:26:21
tags:
categories: Android
---

## 1.概述
Android安全和逆向是两个相关但又不完全相同的领域。

Android安全是指保护Android操作系统、应用程序和用户数据免受潜在威胁的一系列措施。
逆向工程是指对已编译的二进制代码、应用程序或系统进行分析以了解其功能、结构和运作方式的过程。

在Android安全中，逆向工程可以帮助开发者发现潜在的漏洞、恶意代码和应用程序的脆弱点。
然而，逆向工程也可以被用于非法活动，例如破解应用程序、盗取敏感信息等。因此，在进行逆向工程时，必须遵守法律和道德准则。

## 2.App安全

### 2.1 基础安全
1. 混淆
包括代码混淆和资源混淆。有助于减小包的大小，增加了阅读难度。

2. 反调试
使用IDA进行so动态调试是基于进程的注入技术，然后使用linux中ptrace机制。可检测TracerPid值，大于0就退出。
https://blog.csdn.net/qq9746/article/details/109673315
https://blog.51cto.com/u_16213606/7844834
https://www.jianshu.com/p/425620c44ef9
https://blog.csdn.net/c_kongfei/article/details/113242964
https://blog.csdn.net/weixin_37459943/article/details/109315103

3. 反Xposed
https://blog.csdn.net/earbao/article/details/82746138
https://blog.csdn.net/weixin_44348983/article/details/131400346
https://tech.meituan.com/2018/02/02/android-anti-hooking.html
https://mp.weixin.qq.com/s/tamMeh2xsi6L37jjizW_rA

4. 反frida
https://blog.csdn.net/qq_44885775/article/details/130189241
https://mp.weixin.qq.com/s/BbtMCG3wEkv0vn2EuBi55w
http://www.yxfzedu.com/article/645
https://mp.weixin.qq.com/s/auxODmgb7FBC5JFaGmAZEQ
https://mp.weixin.qq.com/s/oQ7PhDdKN72D9mGahohN5Q
https://codeantenna.com/a/jn5b8OJIum
https://github.com/qtfreet00/AntiFrida

5. root检测
https://www.jianshu.com/p/f9f39704e30c
https://blog.csdn.net/Xiaoma_Pedro/article/details/120777603
https://blog.csdn.net/niuzhucedenglu/article/details/53199010

6. 模拟器检测
https://blog.csdn.net/qiou2719/article/details/132367804
https://blog.csdn.net/weixin_44348983/article/details/131398127
https://www.jianshu.com/p/c37b1bdb4757  （底部提供了真机检测方法）

7. 防抓包
https://www.jianshu.com/p/a91ec3840be3 (证书颁发机构不做校验，不做证书锁定校验，防止更换证书厂商后不可用，可以做证书权威性校验)
https://blog.csdn.net/luoshufeng1234/article/details/124291860
https://blog.csdn.net/xinqingch/article/details/119104276

8. 录屏、截图检测

9. 加固
apk加固 360免费
so加固 小密盾免费 https://www.xmprotect.com/index.html

### 2.2 数据安全
1. 说明
动态库so实现(加解密算法、数据操作逻辑、密钥存储)

2. 基础
存储密钥，注意隐藏字符串（改为字节数组）
实现加密算法，详见5.2APP加密算法
混淆和隐藏入口
防盗用

3. 混淆 aes_utils
```
#define AES_128_CBC_PKCS5_Encrypt  ll11l1l1ll
#define getKey                     ll11lll1l1
```

4. 加入花指令 junk.h
插入垃圾代码
_JUNK_FUN_0，_JUNK_FUN_1 

5. 修改jni入口
java 层的类名和方法名就不要那么规范了,FooTools.java method01(str)
so 名字也换掉, fooLib.so

6. app签名校验
   
## 3.App逆向
1. 常用的Android逆向分析工具，它们可以用于反编译、调试、内存分析、Hook等方面：
jadx：一款开源的Java反编译工具，可以将Android应用程序的APK文件反编译为Java源代码，方便进行分析和修改。
APKTool：一款开源的反编译和重新打包工具，可以将Android应用程序的APK文件解压为资源文件和代码文件，方便进行修改和重新打包。
Dex2jar：一款用于将Android应用程序的DEX文件转换为JAR文件的工具，可以用于反编译和分析Android应用程序。
Frida：一款跨平台的动态插桩工具，可以用于修改应用程序的行为和数据，具有强大的Hook和脚本化功能。
Xposed：一款基于Frida的Android插件框架，可以用于Hook应用程序的Java层和Native层代码，实现各种功能。
Burp Suite：一款流行的渗透测试工具，可以用于拦截和修改Android应用程序的网络请求和响应，进行漏洞挖掘和安全测试。
JD-GUI：一款Java反编译工具，可以将Android应用程序的APK文件反编译为Java源代码，方便进行分析和修改。
IDA Pro，反汇编so文件，将二进制代码转化为汇编语言，F5功能还能将汇编语言反编译成c/c++程序。
其它：Ghidra、Radare2、JEB
1. 流程
抓包工具, 定位Java层的发包函数
找到使用的so文件
反汇编so文件
native层分析，定位函数

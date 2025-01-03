---
title: 内网穿透工具ZeroTier
date: 2023-08-08 18:19:20
tags: 
categories: 其它学习
---

> 如果你想通过外网访问到内网的资源，目前只能采用内网穿透的软件来实现。而一般常规的内网穿透软件都需要一个公网 IP 才能正常工作，今天介绍一款不需要公网 IP 实现内网穿透的工具 ZeroTier 。
ZeroTier 是一个专门用来建立点对点虚拟专用网（P2P VPN）的工具，它提供在线管理界面和全平台的客户端，不需要复杂设置，只要安装客户端并加入到自己创建的网络即可。

## 概述
本文对 ZeroTier Android端源码进行了修改，APP启动后可以自动加入指定网络ID，方便远程访问外网Android设备，进行APP调试。源码见 github 项目 [ZerotierFix](https://github.com/yadiq/ZerotierFix)。

## 1.官网后台配置

### 1.1 注册账户
官网地址：https://www.zerotier.com/

### 1.2 创建私有局域网，获得 Network ID
如下图，网络ID为 ebe7fbd4452b324b
![ZeroTier1](/images/OtherZeroTier1.png)

## 2.PC客户端配置

### 2.1 安装PC客户端，加入 Network ID
如下图，填写网络ID ebe7fbd4452b324b
![ZeroTier2](/images/OtherZeroTier2.png)

### 2.2 官网后台授权
找到刚才添加的设备，并授权，如下图
![ZeroTier3](/images/OtherZeroTier3.png)

## 3.Android客户端配置

### 2.1 修改Android项目源码，添加上面的 Network ID
1. 博主借鉴Zerotier Android端的开源实现，[ZerotierFix](https://github.com/kaaass/ZerotierFix)。
2. 为简化客户使用流程，博主修改了少量代码，实现了自动加入网络ID的功能，简化客户使用流程。源码见 [ZerotierFix](https://github.com/yadiq/ZerotierFix)。
修改位置： NetworkListActivity.java 第64行

```java
        rootView = findViewById(R.id.fragmentContainer);
        rootView.postDelayed(() -> {
            SharedPreferences sharedPreferences = PreferenceManager.getDefaultSharedPreferences(this);
            //未启用自定义Planet时，执行初始化
            boolean useCustom = sharedPreferences.getBoolean(Constants.PREF_PLANET_USE_CUSTOM, false);
            if (useCustom) {
                return;
            }
            //启用自定义Planet
            //sharedPreferences.edit().putBoolean(Constants.PREF_PLANET_USE_CUSTOM, true).apply();
            //允许移动网络下使用
            sharedPreferences.edit().putBoolean(Constants.PREF_NETWORK_USE_CELLULAR_DATA, true).apply();
            //添加Planet文件
            //addPlanetFile("http://planet.zerotier.epplink.com:3300/planet?key=d47c3b3d6adc84ca");
            //添加自己的 Network ID TODO
            joinNetwork("ebe7fbd4452b324b");
            //更新网络列表
            Fragment currentFragment = getSupportFragmentManager().findFragmentById(R.id.fragmentContainer);
            if (currentFragment instanceof NetworkListFragment) {
                ((NetworkListFragment)currentFragment).updateNetworkListAndNotify();
            }
        }, 100);
```
3. APP启动后，会自动添加一个网络ID，打开开关同意添加VPN链接。然后等待后台授权。
后台授权刚刚接入的设备，同步骤2.2，授权后可看到设备的局域网ip "10.147.19.39"，
然后你就可以进行设备调试了。
![ZeroTier3](/images/OtherZeroTier3.png)

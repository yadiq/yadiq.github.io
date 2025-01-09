---
title: Android系统传感器的使用
date: 2024-01-07 18:19:20
tags: 
categories: Android
---

> 本文实现了手机6轴数据的显示，源码见 github 项目 [AndroidSensor](https://github.com/yadiq/AndroidSensor)。

## 1.Android系统传感器

### 1.1 内置传感器种类

*   运动相关
    测量三个轴上的加速力和旋转力。
    包括：加速度计、重力传感器、陀螺仪和旋转矢量传感器
*   环境相关
    测量环境参数，例如：气温、气压、光照和湿度
    包括：温度计、气压计、光度计和湿度计
*   位置相关
    测量设备的物理位置
    包括：磁力计、距离传感器

### 1.2 基于硬件的传感器列表

*   TYPE\_ACCELEROMETER 加速度计
    三个轴上的加速度(包含重力),单位m/s^2
    数据：SensorEvent.values\[3] xyz轴加速度
    原理：测量惯性力。
    模型：一个小球在一个方盒子中，每面墙都能感测压力。
    检测到力的方向与它惯性力(加速度)的方向是相反的。

*   TYPE\_GYROSCOPE 陀螺仪
    三个轴上的旋转角速度,单位rad/s
    数据：SensorEvent.values\[3] xyz轴旋转角速度
    原理：角动量守恒。高速转动中地的转子具有惯性，
    有抗拒方向改变的趋向，它的旋转轴永远指向一固定方向。
    模型：旋转轴、转子、旋转轮、两个平衡环。
    旋转轮静止，平衡环转动，旋转轮受力。

*   TYPE\_MAGNETIC\_FIELD	磁力计
    三个轴上的环境地磁场强度,单位μT
    原理：测量外界磁场

### 1.3 基于软件的传感器列表(从一个或多个基于硬件的传感器派生其数据)

*   TYPE\_GRAVITY 重力
    三个轴上的重力,单位m/s^2
    数据：SensorEvent.values\[3] xyz轴重力

*   TYPE\_LINEAR\_ACCELERATION 线性加速度
    三个轴上的加速度(不包括重力),单位m/s^2
    数据：SensorEvent.values\[3] xyz轴线性加速度

*   TYPE\_ROTATION\_VECTOR 旋转矢量
    用于运动检测，如检测手势、监测角变化、监测相对方位变化
    依赖：加速度计、磁力计和陀螺仪
    数据：SensorEvent.values\[4]
    沿 xyz 轴的旋转矢量分量+旋转矢量的标量分量

*   TYPE\_GAME\_ROTATION\_VECTOR 游戏旋转矢量传感器
    不使用地磁场，Y轴不会指向北方，而是指向参照物

*   TYPE\_GEOMAGNETIC\_ROTATION\_VECTOR 地磁旋转矢量传感器
    不使用陀螺仪，准确度低于正常旋转矢量传感器，但功耗低

## 2.描述物体运动姿态

### 2.1 坐标系

机身坐标系: 右手直角坐标系(右上外)
X轴水平向右，Y轴垂直向上，Z轴垂直屏幕向外
大地坐标系：东北天坐标系
X轴指向东，Y轴指向北，Z轴指向天空
旋转的方向按右手法则定义
右手大拇指指向轴向，四指弯曲的方向为绕该轴旋转的方向

### 2.2 六自由度方向

X轴平行于船体基线指向船首，Y轴指向船体左侧，Z轴垂直于船体基线向上

*   roll 飞行器横滚，船舶横摇
    围绕X轴的角度，取值范围为\[-180,180]，单位°，当模组完全水平时为0
*   pitch 飞行器俯仰，船舶纵摇
    围绕Y轴的角度，取值范围为\[-90,90]，单位°，当模组完全水平时为0
*   yaw 飞行器偏航，船舶首摇
    围绕Z轴的角度，取值范围为\[-180,180]，单位°，当X轴指向正北时为0
*   surge 飞行器纵移，船舶纵荡
    X轴加速度
*   sway 飞行器横移，船舶横荡
    Y轴加速度
*   heave 飞行器升降，船舶垂荡
    Z轴加速度

![船舶](/images/AndroidSensor1.webp)
![飞行器](/images/AndroidSensor2.webp)

## 3.Android系统传感器

### 3.1 获取传感器数据

```java
//传感器服务
sensorManager = getSystemService(Context.SENSOR_SERVICE);
//获取某种传感器的列表。(可能有多个)
deviceSensors = sensorManager.getSensorList(Sensor.TYPE_ACCELEROMETER);
//获取某种传感器的默认传感器。(是否存在!= null)
accelerometerSensor = sensorManager.getDefaultSensor(Sensor.TYPE_ACCELEROMETER);
//获取传感器的参数(分辨率、量程)
getResolution() 和 getMaximumRange() 
//监控传感器事件(默认频率200ms)
sensorManager.registerListener(sensorEventListener, accelerometerSensor, SensorManager.SENSOR_DELAY_NORMAL);
//当传感器的值发生变化时回调 event.values
onSensorChanged(SensorEvent event)
```

### 3.2 计算屏幕方向

```java
float[] rotationMatrix = new float[9];//屏幕的旋转矩阵
float[] accelerometerData = new float[3];//加速度计数据
float[] magnetometerData = new float[3];//磁力计数据
float[] orientationAngles = new float[3];//屏幕的三个方向角
//基于加速度计和磁力计的当前读数的旋转矩阵
SensorManager.getRotationMatrix(rotationMatrix, null, accelerometerData, magnetometerData);
//将更新的旋转矩阵表示为三个方向角
SensorManager.getOrientation(rotationMatrix, orientationAngles);
```

## 4.效果图
![飞行器](/images/AndroidSensor3.gif)


## 5.官方文档

> [Android传感器](https://developer.android.com/develop/sensors-and-location/sensors/sensors_overview)



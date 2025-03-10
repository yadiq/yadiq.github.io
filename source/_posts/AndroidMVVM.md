---
title: AndroidMVVM快速开发框架
date: 2019-08-22 19:32:36
tags: 快速开发框架
categories: Android
---

## 概述
一款基于MVVM框架，以Jetpack组件DataBinding+LiveData+ViewModel为基础，整合Retrofit+RxJava网络模块的快速开发框架。本文源码见 github 项目 [AndroidMVVM](https://github.com/yadiq/AndroidMVVM)。

## 1.框架流程
![architecture1](/images/AndroidMVVM1.png)
![architecture2](/images/AndroidMVVM2.png)

## 2.框架特点
### 2.1 Jetpack组件
1. ViewBinding & DataBinding
2. Lifecycles
3. LiveData
4. Navigation
5. Paging
6. Room
7. ViewModel

### 2.2 流行框架
1. [retrofit](https://github.com/square/retrofit)+[okhttp](https://github.com/square/okhttp)+[rxJava](https://github.com/ReactiveX/RxJava)负责网络请求
2. [gson](https://github.com/google/gson)负责解析json数据
3. [glide](https://github.com/bumptech/glide)负责加载图片

### 2.3 基类封装
1. BaseActivity
2. BaseFragment
3. BaseViewModel

### 2.4 全局操作
1. 全局的Activity堆栈式管理
2. LoggingInterceptor全局拦截网络请求日志
3. 全局的异常捕获，程序发生异常时不会崩溃，返回上个界面。
4. 使用androidx
5. 不使用kotlin

### 2.5 Room组件
1. 实现了Network only 和 Network & database 两种模式
FollowersFragment 使用 Room 持久化存储列表数据，
Network => DB => LiveData => RecyclerView
![Room组件](/images/AndroidMVVM3.png)

## 3.界面
1. 登录界面（使用任意账户登录）
2. 我的仓库列表
3. 我的star仓库列表
4. 我的following列表
5. 仓库详情
6. 用户详情

## 4.注意
1. 接口使用GitHub API v3，单IP限制每小时60次requests

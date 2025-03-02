---
title: ReactNative学习
date: 2025-01-02 21:26:21
tags: ReactNative
categories: 跨平台
---

### 1.navigation
1. 安装模块
//基础
npm install @react-navigation/native 
npm install @react-navigation/native-stack
//常用依赖
npm install react-native-screens 
npm install react-native-safe-area-context
//其它
npm install @react-navigation/bottom-tabs
2. 栈管理
https://reactnavigation.org/docs/navigation-object
navigation.navigate('Main')
 navigate 入栈新页面。栈内复用，上面的出栈
	如果当前已经是这个 Screen，则不会跳转
	如果页面栈内已经有这个 Screen，则将这个 Screen 上的所有 Screen 出栈
 push 入栈新页面。栈内无则入栈，栈内有则新建栈
	如果当前已经是这个 Screen，则栈内会有两个同样的 Screen
 replace 替换当前页面
 goBack 后退一页 关闭当前页
 pop 默认返回到上一页 ？回到栈
 popTo 返回栈内特定页
 popToTop 返回栈顶页 
3. 导航事件
堆栈导航时，类组件不会被卸载

### 2.生命周期
类组件有生命周期
缺点：性能差，无法使用导航hook
	为了提高性能，尽量使用函数组件
	
https://legacy.reactjs.org/blog/2018/03/27/update-on-async-rendering.html
a. UNSAFE_”前缀
componentWillMount 弃用 改用构造函数
componentWillReceiveProps 改用getDerivedStateFromProps
componentWillUpdate 改用componentDidUpdate
	UNSAFE_componentWillMount
b. 将状态初始化移至构造函数或属性初始化器
减少生命周期函数
c. 获取网络数据,添加监听器
下面二者成对出现
componentDidMount
	总是render在之后立即执行componentWillMount
	asyncRequest();
	props.dataSource.subscribe()
componentWillUnmount 
	asyncRequest.cancel();
	props.dataSource.unsubscribe()
d. 更新state响应变化
getDerivedStateFromProps
它在创建组件时以及每次由于 props 或 state 的更改而重新渲染时被调用
getSnapshotBeforeUpdate

### 3.列表
FlatList
	data={DATA} 数据
	renderItem 渲染
		入参：item对象 index索引 separators分隔符
	keyExtractor 为给定的 item 生成一个不重复的 key
	extraData 除data以外的数据,可刷新界面

### 4.与原生语言通信
1. Turbo 原生模块 
通过 JavaScript 函数和对象访问
本地持久存储， Web Storage API 的实现
新建 /specs/NativeLocalStorage.ts
    export interface Spec extends TurboModule {}
修改 package.json
    "javaPackageName": "com.nativelocalstorage"
新建 NativeLocalStorageModule.java
    extends NativeLocalStorageSpec
    实现sp存储
新建 class NativeLocalStoragePackage 
    extends TurboReactPackage
    注册模块
修改 MainApplication 
    packages.add(new NativeLocalStoragePackage());
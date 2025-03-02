---
title: ReactNative基础
date: 2025-01-01 21:26:21
tags: ReactNative
categories: 跨平台
---

### 1.背景
Meta开源的 JavaScript 框架，用于构建原生应用程序。
跨平台开发框架，支持多个原生平台包括：Android、iOS、macOS、Web、Windows等。

### 2.React
1. 概念
React 是前端 JavaScript 库，用于开发交互式用户界面。
2. 优点
使用单向的数据绑定，改变数据而无需重新加载页面
3. 缺点
变化太快，学习新变化是个挑战

### 3.React Native
1. 概念
React Native 扩展了React的功能，是一个基于JavaScript的移动开发框架。
2. 优点
有现成的组件，开发时间短
跨平台共享代码，更小的团队JavaScript团队代替Android iOS
热重载
3. 缺点
调试非常困难
没有类型安全
4. 原生组件对应关系
React Native UI 组件 => Android 原生视图
```
<View> <ViewGroup>	
<Text> <TextView>	
<Image>	<ImageView>	
<ScrollView> <ScrollView>	
<TextInput>	<EditText>	
```

### 4.React Native开发
#### 4.1 基础demo
```javascript
//导入组件
import React from 'react';
import { Text } from 'react-native';
//声明组件
const Cat = () => {
  return (
    <>
     <Text>Hello, I am your cat!</Text>
    </>
  );
}
//显示组件，只能显示一个
export default Cat;
```

#### 4.2 自定义组件
```javascript
//子组件，有属性
const Cat = (props) => {
  return (
    <>
      <Text>Hello, I am {props.name}!</Text>
    </>
  );
}
//父组件
const Cafe = () => {
  return (
    <>
      <Cat name="Maru" />
      <Image
        source={{uri:"https://"}}
        style={{width:200, height:200}} 
      />
    </>
  );
}
```

#### 4.3 创建状态变量
```javascript
import React, { useState } from 'react';

const Cat = (props) => {
  //创建状态变量。取值 设值 初始值
  const [isHungry, setIsHungry] = useState(true);
  const [text, setText] = useState('');
  return (
    <>
      <Button
        title={isHungry ? 'Pour me some milk, please!' : 'Thank you!'}
        disabled={!isHungry}
        onPress={()=>{setIsHungry(false)}} 
      />
      <TextInput
        style={{height: 40}}
        placeholder="Type here to translate!"
        defaultValue={text}
        onChangeText={text => setText(text)}
      />
    </>
  );
}
```

#### 4.4 其它组件
```javascript
//垂直滚动视图
<ScrollView> 
  <View/>
<ScrollView />
//垂直滚动列表
<FlatList
  data={[
    {key: 'Devin'}
  ]}
  renderItem={({item}) => <Text >{item.key}</Text>}
/>
```
#### 4.5 react hooks 钩子
https://www.ruanyifeng.com/blog/2020/09/react-hooks-useeffect-tutorial.html
https://react.dev/reference/react/useState

任何一个组件，可以用类来写，也可以用函数来写(钩子)
函数组件：
retrun前 保存变量状态，获取数据，订阅事件
retrun 返回HTML代码，显示数据的状态

推荐使用函数组件，更纯粹，跟计算无关的操作使用副效应effect。

 useState()：保存状态
 useContext()：保存上下文
 useRef()：保存引用,
  var driveModel = useRef<DriveStatusModel | null>(null)
  读取driveModel.current
 useMemo(): 缓存计算结果
 useEffect()：通用的
```
//添加变量
const [state, setState] = useState(initialState)
const [age, setAge] = useState(28);
//缓存计算结果
const cachedValue = useMemo(calculateValue, dependencies)
const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
//useEffect
组件每渲染一次，该函数就自动执行一次
useEffect(func, [props.name])
useEffect(func, [])
第二个参数是依赖项，只有依赖项变化才重新渲染
若第二个参数是空数组，没有依赖项，只在组件加载时执行一次
func的返回函数可清理副效应，组件卸载时执行一次，每次副效应函数重新执行之前，也会执行一次

常见用途：
	获取数据：首次加载时获取一次数据
	事件监听、
	改变DOM
	
```

#### 4.6 TypeScritp 语法
TypeScritp是JavaScrtipt的超集

let和const是JavaScript里较新的变量声明方式。
  let和var相似，let的作用域 重定义更加严格
  const是对let的增强，阻止变量再次赋值

  解构数组
  let input = [1, 2];
  let [first, second] = input;
  解构对象
  let o = {
    a: "foo",
    b: 12
  };
  let { a, b } = o;


#### 4.7 Style 样式
1. 例子
<Text style={styles.container}>just red</Text>

//简洁写法 StyleSheet
const styles = StyleSheet.create({
  container: {
    height: 50,//固定像素尺寸
  },
});
2. 大小
//根据内容计算
width: auto
//绝对像素
width: 50, height: 50
//百分比
width: 50%, height: 50%
//主轴比例(副轴铺满)
flex: 1
3. 间距
padding: 10
margin: 10
4. 子元素的布局
//主轴 (row column-reverse row-reverse)
flexDirection: column
//排列方向 (RTL)
direction: LTR
//沿主轴的排列方式 (flex-end center ...)
justifyContent: flex-start
//沿次轴的排列方式 (flex-start center)
alignItems: stretch 子元素拉伸以匹配次轴的高度
//自身沿次轴的排列方式，同alignItems
alignSelf: stretch
//换行方式 (nowrap)
flexWrap: warp
//绝对与相对定位（absolute 绝对定位）
position：relative 相对定位 默认 
	top: 0,
  left: 0,

5. Image组件
//图片缩放 (contain stretch repeat center)
resizeMode: cover 

### 5.React Native运行
#### 5.1 开发工具
VSCode JS 开发者欢迎的 IDE 工具
//管理应用状态的库 Redux 
命令行工具 Terminal+Cmd
//Expo框架
  基于文件的路由，标准的本机模块库
  沙盒开发环境，带有一个已上架的空应用容器

#### 5.2 搭建开发环境
Node 版本>=18
  C:\Program Files\nodejs
  //切换淘宝源
  npm config set registry https://registry.npmmirror.com
	//取消ssl验证：
	npm config set strict-ssl false
  //使用Yarn替代npm工具 
  npm install -g yarn
JDK 版本17
  C:\Program Files Green\java
AS 版本2024.1.2 Patch1
  
#### 5.3 创建独立项目
//创建新项目 中文版，自动生成 0.76.1
npx @react-native-community/cli@latest init AwesomeProject

#### 5.4 现有项目添加一个RN界面
创建空白文件夹
复制Android源码
  /android
安装NPM依赖
  npm install
配置 Gradle
  //修改 /android/settings.gradle
  @react-native/gradle-plugin
  //修改 /android/build.gradle
  react-native-gradle-plugin
  //修改 /android/app/build.gradle
  apply plugin: "com.facebook.react"
  //修改 /android/gradle.properties
添加 ReactActivity

#### 5.5 编译并运行
yarn android --port=1234 //修改端口
//yarn android --mode release //生产
//安装APP，并在另一个命令行中启动Metro服务
//默认8081端口, 某些电脑被禁用
//启动Metro服务，并安装APP

//启动服务
yarn start --port=1234
npm start --port=1235
npx react-native start --reset-cache

//安装App
npm run android
//修改android 服务器端口
adb reverse tcp:8081 tcp:8081

//调试命令
adb shell input keyevent 82
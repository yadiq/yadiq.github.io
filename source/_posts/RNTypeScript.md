---
title: ReactNative TypeScript 语法
date: 2025-01-03 21:26:21
tags: ReactNative
categories: 跨平台
---

### 1.概念
TypeScript 是 JavaScript 的一个超集，支持 ECMAScript 6 标准。
TypeScript 由微软开发的自由和开源的编程语言，在 JavaScript 的基础上增加了静态类型检查的超集。
它可以编译成纯 JavaScript，编译出来的 JavaScript 可以运行在任何浏览器上。

### 2.注意
区分大小写
分号是可选的
单行注释//
多行注释/**/

### 3.静态类型
//可以显式地声明变量、参数、返回值的类型
let name: string = "Alice";
let age: number = 25;
//可以自动推断变量类型
let name = "Alice";

### 4.模块(可以更换的组织代码)
模块里面的变量、函数和类等在模块外部是不可见的

//导出模块
export function add(a: number, b: number): number {
  return a + b;
}
//导入模块
import { add } from "./math";
console.log(add(2, 3));

### 5.箭头函数
相当于匿名函数，简化了函数定义
let A = () => 5;
let A = function() {return 5};

### 6.async/await语法
async 表示函数是一个异步函数，返回值 Promise
await必须在有关键字async的函数中使用，在async（异步）函数中等待一个promise的返回
例如：
async loadData(){
    let data1 = await ...;        //调用接口，获取数据
    console.log(data1,data2,data3);
}

### 7.断言
类型断言，类型转换
let value: any = "this is a string";
//尖括号语法
let length: number = (<string>value).length;
//as 语法
let length: number = (value as string).length;

非空断言, 明确知道某个值不可能为 undefined 和 null 
function fun(value: string | undefined | null) {
	const length: number = value.length;//error
  const length: number = value!.length;//ok
}

确定赋值断言，该属性会被明确地赋值
let count!: number;
initialize();
console.log(2 * count); // Ok

function initialize() {
  count = 10;
}

### 8.判断
=== 等同符。值类型不同时返回false
== 等值符。值类型不同时自动转换，然后再比较值
 boolean、string、number三者中任意两者进行比较时，优先转换为数字进行比较。
 例如 100=="100" 1==true 都返回true

null和undefined是两种基本类型，== true
null属于object类型，表示空值
undefined属于undefined类型，表示未知的值

？访问一个可能为null或undefined的对象的属性或方法
？？ 提供一个默认值

### 9.运算符
! 非空断言运算符。强调对应的元素非null|undefined
	const length: number = value.length;//error
  const length: number = value!.length;//ok
? 可选链运算符。
  判断是否是 null|undefined，如果是则会停止表达式运行
  obj?.prop
?? 空值合并运算符。
  在左侧表达式结果为 null 或者 undefined 时，才会返回右侧表达式。
  let b = a ?? 10
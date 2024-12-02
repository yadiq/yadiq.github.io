---
title: PHP基础
date: 2018-12-29 18:21:43
tags:
categories: 程序设计
---

## 概述
PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。
PHP 是免费的，并且使用广泛。对于像微软 ASP 这样的竞争者来说，PHP 无疑是另一种高效率的选项。


## 1.php基础

```
1.echo printf sprintf区别
echo 输出多个
printf 输出带格式的 %.2f
sprintf 复值到变量
2.按引用传递参数（指针）,函数所做修改可以体现在函数作用域外
$param1 = 20;
$param2 = 0.2;
func(&$param1, &$param2);
3.默认参数值
func($price, $tax=0.2);//可以不指定第二个参数
func($price1, $price2="", $price3="");//可选参数，可不传
4.返回多个值 list() 类似数组
function func(){
	$user = {"red", "blue"};
	return $user;
}
list($red, $blue) = func();
```


## 2.函数

```
1.函数库
GlobalFunctions.php
function instance($interfaceName) {}
2.require_once include_one require 是放在文件开头， include 是放在流程控制的处理部分
interface.php
require_once dirname(__FILE__) . '/common/Common.php';
require_once dirname(__FILE__) . '/common/GlobalFunctions.php';
```


## 3.数组

```
1.格式类似map
包含key value ，key可是数值、关键值
#赋值 数字
$state = array(0 => "alabama", 1 => "wyoming");
#赋值 关键值
$state = array("OH" => "ohio" , "NY" => "New York");
$state["0"] $state["OH"]
#二维数组
$state = array (
	"ohio" => array("population" => "11353140" , "capital" => "Columbus");
);
$state["ohio"]["population"];

2.array()函数
$languages = array("Spain" => "Spanish"
	"United States" => "English");
```
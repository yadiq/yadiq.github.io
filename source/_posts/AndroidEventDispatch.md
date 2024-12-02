---
title: Android View的事件分发机制
date: 2019-01-03 21:26:21
tags:
categories: Android
---

## 1.public boolean dispatchTouchEvent(MotionEvent event)

通过方法名我们不难猜测，它就是事件分发的重要方法。那么很明显，如果一个MotionEvent传递给了View，那么dispatchTouchEvent方法一定会被调用！
返回值：表示是否消费了当前事件。可能是View本身的onTouchEvent方法消费，也可能是子View的dispatchTouchEvent方法中消费。返回true表示事件被消费，本次的事件终止。返回false表示View以及子View均没有消费事件，将调用父View的onTouchEvent方法

## 2.public boolean onInterceptTouchEvent(MotionEvent ev)

事件拦截，当一个ViewGroup在接到MotionEvent事件序列时候，首先会调用此方法判断是否需要拦截。特别注意，这是ViewGroup特有的方法，View并没有拦截方法
返回值：是否拦截事件传递，返回true表示拦截了事件，那么事件将不再向下分发而是调用View本身的onTouchEvent方法。返回false表示不做拦截，事件将向下分发到子View的dispatchTouchEvent方法。

## 3.public boolean onTouchEvent(MotionEvent ev)

真正对MotionEvent进行处理或者说消费的方法。在dispatchTouchEvent进行调用。
返回值：返回true表示事件被消费，本次的事件终止。返回false表示事件没有被消费，将调用父View的onTouchEvent方法
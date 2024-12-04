---
title: Android自定义键盘
date: 2019-01-07 16:56:35
tags:
categories: Android
---

## 概述
Android自定义键盘，仿招商银行。本文源码见 github 项目 [MyKeyboard](https://github.com/yadiq/MyKeyboard)。

## 1.解决方案
轻量级解决方案-PopupWindow实现。
键盘效果：

![键盘效果1](/images/AndroidKeyboard1.gif) ![键盘效果2](/images/AndroidKeyboard2.gif)

## 2.实现原理

### 2.1 通过PopupWindow显示自定义键盘

```
mKeyboardView = (MyKeyboardView) mIncludeKeyboardview.findViewById(R.id.keyboard_view);
mWindow = new PopupWindow(mIncludeKeyboardview, ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.WRAP_CONTENT, false);
mWindow.showAtLocation(mParent, Gravity.BOTTOM, 0, 0);
```

### 2.2 实现自定义键盘（支持切换字母、数字、符号、随机键盘）

```
class MyKeyboardView extends KeyboardView {

    public void init(EditText editText, PopupWindow window, int keyBoardType) {
        this.mEditText = editText;
        this.mWindow = window;
        if (keyBoardType == KEYBOARDTYPE_Pwd)
            isPwd = true;
        setEnabled(true);
        setPreviewEnabled(false);
        setOnKeyboardActionListener(mOnKeyboardActionListener);//按键监听
        setKeyBoardType(keyBoardType);
    }
    
    //键盘类型支持字母、数字、符号、随机键盘
    public void setKeyBoardType(int keyBoardType) {
        switch (keyBoardType) {
            case KEYBOARDTYPE_Num:
                if (keyboardNum == null)
                    keyboardNum = new Keyboard(getContext(), R.xml.keyboard_number);
                setKeyboard(keyboardNum);
                break;
            case KEYBOARDTYPE_ABC:
                if (keyboardABC == null)
                    keyboardABC = new Keyboard(getContext(), R.xml.keyboard_abc);
                setKeyboard(keyboardABC);
                break;
            case KEYBOARDTYPE_Pwd:
                if (keyboardPwd == null)
                    keyboardPwd = new Keyboard(getContext(), R.xml.keyboard_number);
                randomKey(keyboardPwd);
                setKeyboard(keyboardPwd);
                break;
            case KEYBOARDTYPE_Symbol:
                if (keyboardSymbol == null)
                    keyboardSymbol = new Keyboard(getContext(), R.xml.keyboard_symbol);
                setKeyboard(keyboardSymbol);
                break;
        }
    }
}
```

### 2.3 点击空白处隐藏键盘 (重写View的事件分发)
```
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                cancelOvertimeRunable();

                if (KeyboardTool.isTouchView(filterViewByIds(), ev))//过滤的EditText,不做处理
                    return super.dispatchTouchEvent(ev);
                View v = getCurrentFocus();
                if (KeyboardTool.isFocusEditText(v, systemEditViewIds())//当前焦点在系统键盘EditText
                        && !KeyboardTool.isTouchView(systemEditViewIds(), ev)) {//且没有触摸在系统键盘EditText
                    KeyboardTool.hideInputForce(mContext, v);//隐藏系统键盘
                    v.clearFocus();//清空焦点
                } else if (KeyboardTool.isFocusEditText(v, customEditViewIds())//当前焦点在自定义键盘EditText
                        && !KeyboardTool.isTouchView(customEditViewIds(), ev)) {//且没有触摸在自定义键盘EditText
                    if (getKeyboardUtil() != null)//隐藏自定义键盘
                        getKeyboardUtil().hide();
                    v.clearFocus();//清空焦点
                }
                break;
            case MotionEvent.ACTION_UP:
                startOvertimeRunable();
                break;
        }
        return super.dispatchTouchEvent(ev);
    }
```

### 2.4 键盘上推（设置根布局margintop）
```
//调整父控件位置，键盘不挡住编辑框
int nKeyBoardToTopHeight = mHeightPixels - mKeyboardHeight;//屏幕高度-键盘高度
int[] editLocal = new int[2];
editText.getLocationOnScreen(editLocal);
ViewGroup.MarginLayoutParams lp = (ViewGroup.MarginLayoutParams) mParent.getLayoutParams();
if (editLocal[1] + mEditTextHeight * 2 > nKeyBoardToTopHeight) {
    int height = editLocal[1] - lp.topMargin - nKeyBoardToTopHeight;
    int mScrollToValue = height + mEditTextHeight * 2;
    lp.topMargin = 0 - mScrollToValue;
    mParent.setLayoutParams(lp);
    mScrollTo = true;
}
```
---
layout: post
title: "ReactNative问题集(UI篇)"
subtitle: "ReactNative日常问题集结"
date: 2020-06-30
author: "WenDan"
header-img: "img/post-bg.jpg"
tags:
  - 前端开发
  - React Native
---

# ReactNative 遇到的问题（UI篇）

### 1、Text"吞字"啦

在偶然的情况下发现部分 android 机型会出现 Text 组件“吞字”的情况（ios 端正常显示的）。原本我以为是恶作剧 P 图...天真的我哎

我是写了一个基础 Text 组件，里面有设置 font-weight 属性，但是没有设置 font-family。一直都是默认字体。结果就是这个原因导致了字体显示 bug。

#### 解决方案

1. 给外部 View 设置宽度

   方法不适用所有场景，比如数据是带入的，就会导致宽度不定。

2. 设置 fontFamily

   给 Text 组件设置 fontFamily

3. 全局设置 fontFamily

   [参考解决方法](https://github.com/facebook/react-native/issues/15114#issuecomment-341988546)

   ```javascript
   import React from 'react';
   import { StyleSheet, Text } from 'react-native';`
   const styles = StyleSheet.create({
       defaultFontFamily: {
           fontFamily: 'lucida grande', //或者设置为""也可以
       }
   });

   const oldRender = Text.prototype.render;
   Text.prototype.render = function (...args) {
       const origin = oldRender.call(this, ...args);
       return React.cloneElement(origin, {
           style: [styles.defaultFontFamily, origin.props.style]
       });
   };
   ```

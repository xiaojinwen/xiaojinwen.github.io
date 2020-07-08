---
title: fixed定位失效问题
date: 2020-6-11 22:00:49
categories: ['前端'] 
tags: 前端
comments: true
---

# 问题
- 父元素使用`transform`,子元素使用`fixed`定位会失效,

# 解决

## 方法一
- 父元素不适用transform属性或者子元素不使用fixed定位

## 方法二
- 方法一无法解决(可能父元素是使用的某些插件,或者子元素不适用fixed无法实现效果)

- 将子元素剥离出有transform属性的父元素,再使用fixed

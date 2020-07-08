---
title: 快应用卡片absolute定位失效问题
date: 2020-6-11 22:00:49
categories: ['前端'] 
tags: 前端
comments: true
---

# 需求一
> 页面是要求将一个浏览数量显示在列表每个item的下方位置,我很自然的使用了绝对定位,快应用里显示是完全正常的,

# 问题
然而预览卡片的时候就出现怪事了,布局错乱了,布局很简单,所以我一下就觉得是绝对定位失效的问题(当时不知道怎么打开element,所以完全是靠改代码然后定位问题,后来知道了,在快应用开发工具-调试器-DEVTOOLS-右上角的三个点,然后点击Run command-输入show Elements执行,就可以看到页面的结构了),定位失效有可能是因为快应用本身是全局使用的flex布局,也有可能是快应用卡片暂不支持使用绝对定位

# 解决
因为快应用使用的是flex布局,所以使用flex布局的属性解决问题,是我最优先想到的.

## 使用flex属性解决
tips:此处都省略设置display:flex;
### 父元素使用
```css
flex-direction: column;
justify-content: space-between;
```

### 子元素(居上的)
因为我上方是一个两行(或者一行)文字的标题
```css
lines: 2;
text-overflow: ellipsis;
/* 居上显示 不要设置高度 */
align-self: flex-start;
line-height: 60px;
```

### 子元素(居下的)
这个不用设置flex的属性

# 需求二
> 页面需要list的每一个item的一个图片的右下方显示一个文本
# 使用flex属性和margin属性
## 图片的样式
正常设置宽度,高度铺满整个容器

## 在图片右下角显示的文本
```css
margin-left: -132px;
align-self: flex-end;
margin-bottom: 16px;
```
成功解决



---
title: iOS上输入框光标错位问题
date: 2018-09-20 16:26:11
categories: ['前端'] 
tags: 前端
comments: true
---
## iOS上,fixed 元素内的输入元素,获取焦点时的光标错位问题 [转载来源](http://web.blued.cn/2017/12/15/ios-fixed-input-cursor-position/)
如果一个输入元素（input, textarea …）的父容器设置了 position: fixed，当这个元素获取焦点时，滚动网页光标便错位

![无法展示图片](http://web.xiaojw.xyz/blog_file/piaoyi.gif)

> 遗憾的是，截至目前，iOS 11.x 上也有这个问题。

### 解决方法一:不使用fixed 布局
如果一定要使用请看下一种方法

### 解决方法二:动态改变布局方式
当元素获取焦点时，改变父容器的定位方式：fixed > absolute
```bash
for (let evt of ['focus', 'blur']) {
  const isFocus = evt === 'focus'
  const fn = isFocus ? 'add' : 'remove'

  inputDOMNode.addEventListener(evt, () => {
    parentDOMNode.classList[fn]('input-focus')
    htmlDOMNode.classList[fn]('no-scroll')
    bodyDOMNode.classList[fn]('no-scroll')

    isFocus && setScrollTop(0)
  })
}
```

```bash
.input-focus {
  position: absolute;
  ...
}

.no-scroll {
  height: 100%;
  overflow: hidden;
}
```
监听了输入元素 focus 和 blur 事件，为父元素添加或移除某些样式。
> 当 position: absolute 时，输入框的定位方式需要手动设置（这里采取了顶部对齐）；.no-scroll 是为了禁止 body 的滚动，保证输入框可见。

但是这个方案在部分 Android 设备上，当键盘收起时并不会触发输入元素的 blur 事件，往往还需要用户主动点击页面的其他区域，算是一点小遗憾吧。

所以监听改变前先判断手机是ios还是android android就不监听

### 解决方法三:直接给html, body元素设置样式(推荐)
```bash
html,
body {
  -webkit-overflow-scroll: touch !important;
  overflow: auto !important;
  height: 100% !important;
}
```
`-webkit-overflow-scrolling`控制元素在移动设备上是否使用滚动回弹效果。
```bash
-webkit-overflow-scrolling: touch; /* 当手指从触摸屏上移开，会保持一段时间的滚动 */
overflow: auto; /* 由浏览器定夺，如果内容被修剪，就会显示滚动条 */
```
当输入元素获取焦点时，键盘弹起，输入元素被顶到了键盘的上方，此时用户的手指会从触摸屏上移开，输入元素会保持一段时间的滚动，从而光标的位置可以被正确计算。
> !important 在这里是为了防止这些属性会因为浏览器优先级过高而发生变化。有点小遗憾的是，!important 侵入性有些高。

### 解决方法四:IOS系统修复
慢慢等,说不定下一次更新也没有修复

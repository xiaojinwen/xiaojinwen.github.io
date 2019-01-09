---
title: 解决vue不能使用锚标记时的方法
date: 2018-10-23 17:48:24
categories: ['前端'] 
tags: 前端
comments: true
---
## 普通的锚标记转跳

```bash
<a href="#q">转跳</a>

<a name="q">转跳到这</a>

```

## 以上方法不能转跳到对应位置的话,用这种
```bash

<div @click="returnTo('xxx')">转跳</div>

<div id="xxx">转跳到这</h3>

// methods 里
returnTo(name) {
   document.querySelector(`#${name}`).scrollIntoView(true)
}

```
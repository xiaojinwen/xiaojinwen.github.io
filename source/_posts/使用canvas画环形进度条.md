---
title: 使用canvas画环形进度条
date: 2018-08-30 19:07:06
categories: ['前端'] 
tags: 前端
comments: true
---

### 效果展示

![无法展示图片](http://web.xiaojw.xyz/blog_file/canvas.png)


### html

```bash

<canvas id="myCanvas" width="300" height="150" style="border:1px solid #d3d3d3;">
	Your browser does not support the HTML5 canvas tag.
</canvas>
	
```

### js

```bash

let canvas = document.getElementById('myCanvas')
let ctx = canvas.getContext('2d')
// 绘制 灰色层 底层
ctx.beginPath()
ctx.arc(100, 75, 50, 0, 2 * Math.PI)
ctx.lineWidth = 10
ctx.strokeStyle = "#eee";
ctx.stroke()
// 绘制 蓝色层 进度层
ctx.beginPath()
ctx.arc(100, 75, 50, -Math.PI / 2, 0)
ctx.lineWidth = 10
ctx.strokeStyle = "#489cff";
ctx.stroke()
// 绘制文字层
ctx.font = "20px Verdana";
/* 渐变
let gradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
gradient.addColorStop(0, "magenta");
gradient.addColorStop(0.5, "blue");
gradient.addColorStop(1, "red");
ctx.fillStyle = gradient;
*/
ctx.fillStyle = "#666";
ctx.fillText("25%", 80, 84)

```

---
title: 按钮下载进度demo
date: 2022-03-15 11:31:11
categories: ['前端'] 
tags: 前端
---

## 先看效果
![效果图片-运行显示便正常](/images/按钮下载进度demo.gif)

### 实现进度显示能在两种颜色背景下显示清晰
### 使用绝对定位 设置两种颜色的文字


- 按钮 内容设置 进度百分比 文字

- 进度dom设置 百分比宽度 ,用于展示 进度条

- 进度dom的伪元素设置 进度百分比 文字

## 废话不多说,上代码(可运行看效果)
```html
<!DOCTYPE html>
<html>

<head>
    <title>下载进度条demo</title>
    <style type="text/css">
        .btn {
            position: relative;
            display: flex;
            overflow: hidden;
            width: 140px;
            height: 60px;
            background-color: rgba(38, 96, 245, 0.1);
            border-radius: 30px;
            font-size: 30px;
            line-height: 30px;
            text-align: center;
            align-items: center;
            justify-content: center;
            color: #2660f5;
        }

        .btn__text {
            height: 100%;
            font-size: 30px;
            line-height: 30px;
            text-align: center;
            display: flex;
            align-items: center;
            justify-content: center;
            color: #2660f5;
        }

        .btn__progress {
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            width: 40%;
            height: 60px;
            background-color: #2660f5;
            overflow: hidden;
        }

        .btn__progress::after {
            z-index: 100;
            position: absolute;
            left: 0;
            right: 0;
            top: 0;
            bottom: 0;
            width: 140px;
            height: 60px;
            color: #fff;
            display: flex;
            align-items: center;
            justify-content: center;
            text-align: center;
            content: attr(data-progress);
        }
    </style>
</head>

<body>
    <div class="btn">
        <div class="btn__progress" data-progress="40%" style="width:40%"></div>
        <div class="btn__text">40%</div>
    </div>
    <script>
        function changeProgress(p) {
            if (Number.isInteger(p)) {
                p = Math.min(100, p) + '%';
                const progressDom = document.querySelector('.btn__progress');
                const textDom = document.querySelector('.btn__text')
                progressDom.dataset.progress = p;
                progressDom.style.width = p;
                textDom.innerHTML = p
            }
        }
        // 测试
        let p = 0;
        const timer = setInterval(() => {
            p += 3;
            if (p > 100) {
                p = 100;
                clearInterval(timer);
            }
            changeProgress(p);
        }, 120);

    </script>
</body>

</html>

```

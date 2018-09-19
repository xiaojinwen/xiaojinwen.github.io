---
title: 关于vue中修改UI库样式的坑
date: 2018-09-19 11:57:01
categories: ['前端'] 
tags: 前端
comments: true
---

## 一、怎么修改样式都不生效，在浏览器上改就能生效

### 原因一: style标签加了scoped属性`<style scoped>`,去掉即可
```bash
<style>
</style>
```
### 原因二: 那就是选择器没选中咯,检查检查


## 二、修改样式后影响了其他组件的样式

### 原因:直接在UI组件上加的样式

### 解决:在外层再套一层标签 加上限制,便不会影响全局了
例:
```bash
---------------------
template
---------------------

<div class="elinput">
	<el-input v-model="form.VerificationCode" placeholder="输入验证码"
	   auto-complete="ture"></el-input>
</div>

---------------------
style
---------------------
.elinput .el-input{
// 写样式
}
```

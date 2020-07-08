---
title: lodash所遇问题之时间字段排序前后端结果不同
date: 2020-6-11 22:00:49
categories: ['前端'] 
tags: 前端
comments: true
---

# 问题
- `lodash`库,使用_.orderBy方法,
时间字段 是用的包含时区的时间

- 浏览器环境排序正常
- node后端环境排序出错,排序是按照字符串的排序
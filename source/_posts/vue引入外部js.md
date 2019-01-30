---
title: vue引入外部js
date: 2019-01-24 10:26:18
categories: ['前端'] 
tags: 前端
---

### .vue文件
```vue
<template>
    <div>
        <remote-script src="http://vip.niukewang.com/static/niu/js/ksbaoname.js"></remote-script>
    </div>
</template>
<script>
    export default {
        mounted() {
        	// 外部js里的变量
            console.log(listlei)
        }
    }
</script>
```

### main.js
```javascript
// 导入外部js
import Vue from 'vue'
 
Vue.component('remote-script', {
    render: function (createElement) {
        var self = this;
        return createElement('script', {
            attrs: {
                type: 'text/javascript',
                src: this.src
            },
            on: {
                load: function (event) {
                    self.$emit('load', event);
                },
                error: function (event) {
                    self.$emit('error', event);
                },
                readystatechange: function (event) {
                    if (this.readyState == 'complete') {
                        self.$emit('load', event);
                    }
                }
            }
        });
    },
    props: {
        src: {
            type: String,
            required: true
        }
    }
});
```
### 文件转载自[vue 引入外部js文件 - 配置component](https://blog.csdn.net/zjl199303/article/details/82585775)

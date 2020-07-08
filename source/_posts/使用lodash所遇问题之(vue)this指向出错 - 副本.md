---
title: 使用lodash所遇问题之(vue)this指向出错
date: 2020-6-11 22:00:49
categories: ['前端'] 
tags: 前端
comments: true
---

### 问题
- `lodash`库,使用节流/防抖 使用箭头函数 `this`指向出错


### 解决方法
```javascript
// 这样写this指向会错误
methods:{    
  resize: _.throttle(()=>{        // this.xxx  出错    }, 1000),
}
// 应该这样
methods:{    
    resize: _.throttle(function(){        
     // this.xxx  正常    
    }, 1000),
}
// 或者created() {    
     this.resize = _.throttle(this.resize, 300);    
     window.addEventListener('resize', this.resize); 
     // 此处省略remove
},
methods:{    
    resize(){        // this.xxx  正常    },
}
```
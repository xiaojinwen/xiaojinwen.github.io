---
title: 页面切换关闭未完成的axios请求
date: 2020-3-11 10:52:49
categories: ['前端'] 
tags: 前端
comments: true
---


### 实现
使用axios拦截器


```javascript
// ...import

//  获取CancelToken
const CancelToken = axios.CancelToken;
let cancel = [];
// http request 拦截器);
axios.interceptors.request.use(
	(config) => {
		// 全局添加cancelToken
		config.cancelToken = new CancelToken(c => {  //强行中断请求要用到的
			cancel.push(c)
		})
		return config;
	},
	(err) => {
		return Promise.reject(err);
	},
);

// http response 拦截器
axios.interceptors.response.use((response) => {
	return Promise.resolve(response);
}, (error) => {
	if (axios.isCancel(error)) { // 取消请求的情况下，终断Promise调用链
		return new Promise(() => {
		});
	} else {
		return Promise.reject(error);
	}
});

Vue.prototype.$cancelAjax = () => {
	// 关闭上一次axios请求的方法
	cancel.forEach(item => item())
	cancel = []
}


// 页面切换前时候执行 $cancelAjax 方法
```




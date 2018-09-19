---
title: vue的url去掉难看的井号并解决相关问题
date: 2018-09-19 16:26:10
categories: ['前端'] 
tags: 前端
comments: true
---
## 一、首先在路由文件配置history模式

```bash
// 增加 mode: 'history'
export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      redirect: '/login'
    }
  ]
})
```
## 二、使用history后一切都正常,唯有刷新浏览器的时候就不会加载了

因为刷新后 服务器上没有对应的静态页对应

解决 将服务器上路径设置对应的定向

### 后端配置例子

#### Apache
```bash
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```
除了 mod_rewrite，你也可以使用 [FallbackResource](https://httpd.apache.org/docs/2.2/mod/mod_dir.html#fallbackresource)

#### nginx
```bash
location / {
  try_files $uri $uri/ /index.html;
}
```

#### 原生 Node.js
```bash
const http = require('http')
const fs = require('fs')
const httpPort = 80

http.createServer((req, res) => {
  fs.readFile('index.htm', 'utf-8', (err, content) => {
    if (err) {
      console.log('We cannot open "index.htm" file.')
    }

    res.writeHead(200, {
      'Content-Type': 'text/html; charset=utf-8'
    })

    res.end(content)
  })
}).listen(httpPort, () => {
  console.log('Server listening on: http://localhost:%s', httpPort)
})
```
#### 基于 Node.js 的 Express
对于 Node.js/Express，请考虑使用[connect-history-api-fallback 中间件](https://github.com/bripkens/connect-history-api-fallback)

```bash
const express = require('express');
const history = require('connect-history-api-fallback');
const app = express();
app.use(history(
    {
        htmlAcceptHeaders: ['text/html', 'application/xhtml+xml']
    }
));
app.use(express.static('dist'));
const server = app.listen(3002, function () {
  const port = server.address().port;
  console.log(`listening at http://localhost:${port}`);
});
```

#### Internet Information Services (IIS)
1.安装 [IIS UrlRewrite](https://www.iis.net/downloads/microsoft/url-rewrite)
2.在你的网站根目录中创建一个`web.config`文件，内容如下：
```bash
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Handle History Mode and custom 404/500" stopProcessing="true">
          <match url="(.*)" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

#### Caddy
```bash
rewrite {
    regexp .*
    to {path} /
}
```

#### Firebase 主机
在你的`firebase.json`中加入：
```bash
{
  "hosting": {
    "public": "dist",
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```
## 警告
因为这么做以后，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 `index.html` 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。
```bash
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '*', component: NotFoundComponent }
  ]
})
```
或者，如果你使用 Node.js 服务器，你可以用服务端路由匹配到来的 URL，并在没有匹配到路由的时候返回 404，以实现回退。更多详情请查阅 [Vue 服务端渲染文档](https://ssr.vuejs.org/zh/)。


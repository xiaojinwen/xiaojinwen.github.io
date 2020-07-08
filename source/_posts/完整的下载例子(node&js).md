---
title: 完整的下载例子(node&js)
date: 2020-6-11 22:00:49
categories: ["前端", "后端"]
tags: 前端
comments: true
---

# 完整的下载例子(node&js)

接口正常返回,content 字段内容为下载文件的 Buffer 数据

# node(think.js)

重新加载了一个未设置宽度的的图片 取出图片的宽高

```javascript
...省略部分
const fs = require("fs");
const path = `./export/${fileName + suffix}`;
const { ctx } = this;
const reader = fs.createReadStream(path);
const streamToBuffer = (stream) => {
  return new Promise((resolve, reject) => {
    const buffers = [];
    stream.on("error", reject);
    stream.on("data", (data) => buffers.push(data));
    stream.on("end", () => resolve(Buffer.concat(buffers)));
  });
};
const content = await streamToBuffer(reader);
return ctx.success({
  content,
  // mime_type: 'text/comma-separated-values',
  mime_type: "text/csv",
  fileName,
  suffix,
});
```

# node(think.js)

```javascript
...省略部分
//   let ab = new ArrayBuffer(data.content.data.length);
//   let view = new Uint8Array(ab);
//   for (var i = 0; i < data.content.data.length; ++i) {
//     view[i] = data.content.data[i];
//   }
//   console.log(view);
const ab = Buffer.from(data.content, "binary");
const blob = new Blob([ab], {
  type: data.mime_type,
});
const fileName = data.fileName + data.suffix || "unkown";
if (window.navigator.msSaveOrOpenBlob) {
  navigator.msSaveBlob(blob, fileName);
} else {
  const link = document.createElement("a");
  const body = document.querySelector("body");
  link.href = window.URL.createObjectURL(blob); // 创建对象url
  link.download = fileName;
  // fix Firefox
  link.style.display = "none";
  body.appendChild(link);
  link.click();
  body.removeChild(link);
  window.URL.revokeObjectURL(link.href); // 通过调用 URL.createObjectURL() 创建的 URL 对象
}
```

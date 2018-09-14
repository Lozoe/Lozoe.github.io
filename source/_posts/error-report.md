---
title: 错误监控
date: 2017-09-10 16:04:04
categories: [JavaScript]
tags:
  - 错误监控
---

### 前端错误分类
- 即时运行错误： 代码错误
- 资源加载错误

### 错误的捕获方式

即时运行错误捕获：

```js
1) try...cache
try {
        
} catch (error) {

}
2) window.onerror 
window.addEventListener('error', funtion() {}, false)
```
<!-- more -->
资源加载错误捕获：

```js
1) object.onerror(script错误不冒泡到window )
2) performance.getEnties() 获取所有已经加载资源加载时长
for(let [key, value] of performance.getEntries().entries()) {
    console.log(value)
}
document.getElementsByTagName('img')  两者差集

3) Error时间捕获

window.addEventListener('error', function (e) {
  console.log('捕获', e);
}, true); // 捕获阶段
<script src="//xxx.com/test.js" charset="utf-8"></script>  

```

##### ps: 跨域js错误如何处理？

跨域js只可以捕获到Script Error信息，可以通过以下形式来捕获跨域错误。

1. 客户端在`script`标签中添加crossorigin
2. 服务端在资源响应头添加 Access-Control-Allow-Origin: *

### 上报错误的基本原理

1. 利用Ajax通信上报
2. 利用Image对象上报(推荐)

```js
(new Image()).src = 'http://xxx.com/aaa?r=www';
```



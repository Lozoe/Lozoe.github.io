---
title: Web Security
date: 2016-06-26 16:46:59
categories: [安全]
tags:
  - Web安全
  - XSS
  - XSS防范措施
---

## 什么是XSS

### 简介

跨站脚本攻击(Cross Site Scripting)，为不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。恶意攻击者通常往Web页面里插入恶意Script代码，当用户浏览该页之时，嵌入其中Web里面的Script代码会被执行，从而达到恶意攻击用户的目的, 还有另外一种产常见的就是CSRF(Cross-site request forgery)跨站点请求伪造。
<!--more-->

![csrf.png]csrf.png

### XSS主要分类

反射型：发出请求时，XSS代码出现在URL中，作为输入提交到服务器端，服务器端解析响应之后，XSS代码随着响应内容一起传回给浏览器，最后浏览器解析执行XSS代码。
存储型：存储型XSS和反射型XSS的差别在于,提交的代码会存储在服务器中(例如数据库,内存,文件系统等),下次请求页面时不用再提交XSS代码。

### 特点

1.耗时间
2.有一定几率不成功
3.没有响应的软件自动攻击
4.需要有一定的语言基础
5.这是一种被动的攻击手法
6.几乎所有的网站都存在Xss 谷歌，百度，QQ都有

### 其它的攻击

DoS(Denial of Service)拒绝服务攻击、DDoS(Distributed Denial of Service)分布式拒绝服务攻击
这两种攻击方式利用目标系统网络服务功能缺陷或者直接消耗其系统资源，使得该目标系统无法提供正常的服务。单一的DoS攻击一般是采用一对一方式的，而DDoS利用更多的傀儡机（肉鸡）来发起进攻，以比从前更大的规模来进攻受害者。
Server Limit DOS服务器限制拒绝服务攻击
比如： 攻击导致http request header过长而导致web server产生400或者4开头的一个错误。如果浏览器中这些数据保存在cookies中，会导致用户无法正常访问域名或者这个站点

### XSS的反射型攻击演示

构建Node服务进行演示
1.新建文件夹，命令行输入：

express -e ./使用express脚手架，用ejs作为模板引擎，在当前目录执行
npm install安装依赖
2.在routes/index.js下设置路由：

```js
router.get('/', function(req, res, next) {
  res.set('X-XSS-Protection',0); //关掉浏览器对XSS的检测
  res.render('index',{ title:'Express',xss:req.query.xss });
}); //query是express获取search的字段
```

3.在views/index.ejs中的body部分添加：

```html
<div class="">
  <%- xss %><!--'-'表示允许输入html，不需要转义-->
</div>
```

4.npm start 启动服务

5.在http://localhost:3000/后输入

```html
?xss=<iframe src="//baidu.com/h.html"></iframe>或者?xss=<img src="null" onerror="alert("1")">或者>xss=<p onclick="alert("1")">点我</p>
```

进行模仿XSS的放射型攻击。

### XSS的防范措施

XSS不止是==URL注入 ，或者评论代码注入，还有cookie 劫持==等多种形式

对于评论代码注入的三大步骤：

编码：对用户输入的数据进行HTML Entity编码，比如字符"转义成转义字符&quot
过滤：移除用户上传的DOM属性，如onerror等，还有用户上传的Style节点、Script节点，Iframe节点，frame节点等
校正：避免直接对HTML Entity解码；使用DOM Parse转换，校正不配对的DOM标签（DOM Parse指将字符串或文本解析成DOM结构）

### 实战

[github](https://github.com/Lozoe/xss)

## 什么是CSRF

CSRF，全称跨站请求伪造（Cross-site request forgery)，也称one click attack/session riding,还可以缩写为XSRF。通俗的说就是利用被害者的身份去发送请求。

### 浏览器的Cookie保存机制

- Session Cookie,浏览器不关闭则不失效
- 本地Cookie,过期时间内不管浏览器关闭与否均不失效

自动带cookie和浏览器特性有关（safari IE678不可以自动携带cookie）

```html
<body onload="javascript:document.forms[0].submit()">
  <!-- <iframe src="http://lz.qunar.com/xxx" frameborder="0"></iframe> -->
  <form action="http://lz.qunar.com/xxx" method="post">
    <input type="hidden" name="param1" value="123123123" />
    <input type="hidden" name="param2" value="123" />
</body>
```

### CSRF几种攻击方式

- HTML CSRF
- JSON HiJacking
- Flash CSRF

#### HTML CSRF

通过html标签进行发起CSRF请求的，可以发起Get请求的标签

```html
<link href="">
<img src="">
<frame src="">
<script src="">
<video src="">
background: url("");
```

POST攻击： form表单构造
攻击者在自己的网站上创建一个包含恶意操作的表单，并诱使用户访问该网站。当用户在认证过的网站上登录后，攻击者的网站会自动提交表单，触发用户在认证网站上的操作。

```html
<body onload="javascript:document.forms[0].submit()">
  <!-- <iframe src="http://lz.qunar.com/xxx" frameborder="0"></iframe> -->
  <form action="http://lz.qunar.com/xxx" method="post">
    <input type="hidden" name="param1" value="123123123" />
    <input type="hidden" name="param2" value="123" />
</body>
```

#### JSON HiJacking

利用浏览器的同源策略漏洞来获取跨域请求的响应数据(利用浏览器对 JSONP（JSON with Padding）的支持。攻击者在自己的网站上构造一个请求，利用 JSONP 跨域的特性，将请求发送到目标网站。然后，攻击者通过一个特殊的响应来获取目标网站返回的数据) 构造自定义的回调函数

```html
<script>
  function hijack(data) {
    console.log(data);
  }
</script>
<script src="http://www.a.com/json?callback=hijack">
```

#### Flash CSRF

通过flash来实现跨域请求  攻击者在自己的网站上创建一个包含恶意操作的表单，并诱使用户访问该网站。当用户在认证过的网站上登录后，攻击者的网站会自动提交表单，触发用户在认证网站上的操作。

import flash.net.URLRequest

```js
function get() {
  var url = new URLRequest("http://a.com/json?callback=hijack");
  url.method = "GET";
  sendToUrl(url);
}
```

### CSRF 的防御方法

- 通过验证码进行防御
- 检查请求来源（reffer验证） 也可以被篡改
- 增加请求参数 token

## CSRF与XSS的区别

XSS: 利用对用户的输入不严谨然后执行JS语句
CSRF: 通过伪造受信任用户发送请求
CSRF可以通过XSS来实现

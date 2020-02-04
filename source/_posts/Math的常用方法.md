---
title: Math的常用方法
date: 2019-08-01 21:31:56
categories: [JavaScript]
tags:
---

JavaScipt中的Math.ceil() 、Math.floor() 、Math.round()、Math.pow() 三个函数的理解

以前一直会三个函数的使用产生混淆，现在通过对三个函数的原型定义的理解，其实很容易记住三个函数。
<!-- more -->
现在做一个总结：

1. Math.ceil()用作向上取整。
2. Math.floor()用作向下取整。
3. Math.round() 我们数学中常用到的四舍五入取整。

```js
alert(Math.floor(5/2));
alert(Math.ceil(5/2));
```

4.幂

```js
alert(Math.pow(10 , 2)); // 10的2次方
```
---
title: Math的常用方法
date: 2019-08-01 21:31:56
categories: [JavaScript]
tags:
---

## 常见函数

JavaScipt中的Math.ceil() 、Math.floor() 、Math.round()、Math.pow() 三个函数的理解

以前一直会三个函数的使用产生混淆，现在通过对三个函数的原型定义的理解，其实很容易记住三个函数。
<!-- more -->
现在做一个总结：

1. Math.ceil()用作向上取整。
2. Math.floor()用作向下取整。
3. Math.round() 我们数学中常用到的四舍五入取整。
4. Math.pow(x,y) 返回 x 的 y 次幂的值
5. Math.random() 结果为0-1间的一个随机数(包括0,不包括1)

```js
alert(Math.floor(5/2));
alert(Math.ceil(5/2));
```

## 函数配合实现取随机数,例如1-6的

1，用Math.ceil(Math.random()*6);时，主要获取1到6的随机整数，取0的几率极小。

2，用Math.round(Math.random()*5 + 1)，可基本均衡获取1到6的随机整数，其中获取最小值0和最大值6的几率少一半。

3，用Math.floor(Math.random()*6 + 1);时，可均衡获取1到6的随机整数。

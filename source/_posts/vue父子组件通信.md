---
title: vue组件通信
date: 2018-11-22 13:19:09
categories: [Framework]
tags:
---

总结一下在vue中 父子组件通信的形式。

在平时开发中我们可能大多数情况用的是 Prop、$emit

其实大体上有下面几种形式：
props $emit
$emit $on (eventbus)
.sync语法糖
vuex
$attrs 和 $listeners Vue2.4
provide 和 inject Vue2.2
其他方式通信

https://juejin.im/post/5cde0b43f265da03867e78d3#heading-12
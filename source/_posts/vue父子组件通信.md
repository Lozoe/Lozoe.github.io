---
title: vue父子组件通信
date: 2018-11-22 13:19:09
tags:
---

Coding Coding，作为程序猿的我们要边Coding的同时更要学会思考。今天我就斗胆来总结一下在vue中 父子组件通信的形式。

在平时开发中我们可能大多数情况用的是 Prop、$emit

其实大体上有下面几种形式：
- Prop
$emit
.sync语法糖
$attrs 和 $listeners
provide 和 inject
其他方式通信

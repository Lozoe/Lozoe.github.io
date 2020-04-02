---
title: vuex数据流向
date: 2020-03-02 17:31:44
categories: [Framework]
tags:
---

## vuex基础

https://vuex.vuejs.org/zh-cn

多组件共享状态， 之前操作方式，由父组件传递到各个子组件。 当路由等加入后，会变得复杂。 引入viewx 解决共享问题。

原vue结构图

<img src="vue结构图.png" alt="vue结构图" width="487" height="329" align="bottom" />

vuex结构图

![vuex结构图](vuex结构图.png)

Vuex对象结构 （state，mutations，getters，actions）

- state 对象数据

- mutations 操作变更state数据

- getters 计算state

- actions  触发mutations

![vuex解析图优化](vuex解析图优化.png)

参考：
https://www.cnblogs.com/reeber/p/10628935.html

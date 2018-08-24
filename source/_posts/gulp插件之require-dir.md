---
title: gulp插件之require-dir
date: 2018-07-24 17:09:21
tags:
  - gulp 
  - require-dir
---

### gulp之 require-dir 分离任务到多个文件中

分离任务到多个文件中
如果 gulpfile.js 开始变得很大，你可以通过使用 require-dir 模块将任务分离到多个文件

想象如下的文件结构：
```javascript
//gulpfile.js
tasks/
├── dev.js
├── release.js
└── test.js
```
<!-- more -->
安装 `require-dir` 模块：
```
npm install --save-dev require-dir
```
在 gulpfile.js 中增加如下几行代码：
```javascript
var requireDir = require('require-dir');
var dir = requireDir('./tasks');
```

[源码](https://github.com/aseemk/requireDir)

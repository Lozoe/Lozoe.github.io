---
title: rollup
date: 2021-07-01 16:33:12
tags:
---

Rollup 是一个 JavaScript 模块打包器，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。Rollup 对代码模块使用新的标准化格式，这些标准都包含在 JavaScript 的 ES6 版本中，而不是以前的特殊解决方案，如 CommonJS 和 AMD。ES6 模块可以使你自由、无缝地使用你最喜爱的 library 中那些最有用独立函数，而你的项目不必携带其他未使用的代码。ES6 模块最终还是要由浏览器原生实现，但当前 Rollup 可以使你提前体验。

<!-- more -->

## 使用入门

#### 安装

```jsnpm install --global rollup``` 

Rollup 可以通过命令行接口(command line interface)配合可选配置文件(optional configuration file)来调用，或者可以通过 JavaScript API来调用。运行 `rollup --help` 可以查看可用的选项和参数。

#### Tree Shaking

除了使用 ES6 模块之外，Rollup 还静态分析代码中的 import，并将排除任何未实际使用的代码。这允许您架构于现有工具和模块之上，而不会增加额外的依赖或使项目的大小膨胀。

只引入最基本最精简代码，所以可以生成轻量、快速，以及低复杂度的 library 和应用程序。因为这种基于显式的 import 和 export 语句的方式，它远比「在编译后的输出代码中，简单地运行自动 minifier 检测未使用的变量」更有效


参考：
[rollup 中文](https://www.rollupjs.com/guide/introduction#%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8%E6%8C%87%E5%8D%97quick-start)

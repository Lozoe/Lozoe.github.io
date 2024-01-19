---
title: vite打包
date: 2024-01-16 19:13:42
tags:
---
[vite 的文档](https://cn.vitejs.dev/) 可以简单理解为前端工程的构建工具，vite 对自己的定义为下一代的前端工具链。

[为什么选择vite](https://cn.vitejs.dev/guide/why.html)

## 前言

1. 首先为什么需要打包工具？
	a. 随着es 、ts、css 预编译语言，以及vue、react 类的前端框架的快速发展，前端需要在构建阶段加入相应的编译过程才能转化为浏览器可识别的语言。
	b. 同时在浏览器支持 ES 模块之前，JavaScript 并没有提供原生机制让开发者以模块化的方式进行开发。js模块化，社区多种解决方案，一直到 es6 推出 es module 规范。
		基于各种模块化解决方案，对应产生了各种工具，如 browserify，babel，webpack 等
	c. 在开发过程中对于开发效率的问题，冷启动，热更新，以及生产环境的包体压缩，代码混淆等
2. vite 为什么会出现？
	a. 基于 bundle 理念的打包构建工具，对日益复杂的大型前端应用主要问题有
		i. 服务启动慢 分钟级别
		ii. HMR 热更新慢： 即使有 webpack 的热更新加成，项目热更新也在 8~10s
    b. 浏览器原生支持 es 模块, node 支持 esm

## vite介绍学习

- 快 ！！启动快，热更新快 

基于 nobundle 理念，利用浏览器原生 ES 模块的支持，实现开发阶段的 Dev Server，进行模块的按需加载，而不是先整体打包再进行加载。

> 利用浏览器原生 es 模块的支持，按需加载，在构建时只需处理模块的编译而无须打包，把模块间的相互依赖关系完全交给浏览器来处理。
        ■ 浏览器会加载入口模块，分析依赖后，再通过网络请求加载被依赖的模块。
优势：
        ■ 初次构建启动快： 无包构建流程中，模块依赖分析与编译都是在浏览器渲染页面时异步处理的
        ■ 按需编译：在浏览器渲染时，根据入口模块分析加载所需模块，编译过程按需处理，因此相比之下处理内容更少，速度也会更快。
        ■ 增量构建速度快：rebuild 过程中，只需处理编译单个模块。

- Opinionated——配置简单，对于ts 、jsx、css 等支持开箱即用，免去了当 webpack 配置工程师
- 强大插件机制，以及易于扩展的 api，开发者可以自由定制构建过程，在构建流程的生命周期中，做更多的事情。同时插件兼容rollup，插件生态越来越丰富

## vite 实现

### Concept

基于依赖分析的打包工具，打包工具可以基于依赖分析实现 treeshking、code splitting 等优化，可以配合 runtime 代码实现 lazy load等。如 Webpack，就是通过抓取-编译-构建整个应用的代码，生成一份编译后能良好兼容各个浏览器的的生产环境代码。在开发环境流程也基本相同，需要先将整个应用构建打包后，再把打包后的代码交给dev server。

![webpack-dev-server](webpack-dev-server.png)

Vite直接将源码交给浏览器，实现dev server秒开，浏览器显示页面需要相关模块时，再向dev server发起请求，服务器简单处理后，将该模块返回给浏览器，实现真正意义的按需加载

![vite-dev-server](vite-dev-server.png)

### Poc

![yyx](yyx.png)
vite最开始来自己于 evan you 的一个 idea，而在这之前，他就已经有类似的想法，即[vue-dev-server](https://github.com/vuejs/vue-dev-server)

> 想象一下，你可以在浏览器中本地导入 Vue 单文件组件......无需构建步骤。
● vue-dev-server 特点：
	○ 浏览器请求导入作为本机 ES 模块导入 - 没有捆绑（bundling）处理。
	○ 服务器拦截对*.vue文件的请求，即时编译它们，并将它们作为 JavaScript 发送回来。
	○ 对于提供可在浏览器中运行的 ES 模块构建的库，只需直接从 CDN 导入它们即可。
	○ 文件内 npm 包的导入.js（仅包名称）会被动态重写以指向本地安装的文件。目前仅vue支持特殊情况。其他包可能需要进行转换才能作为本地浏览器目标 ES 模块公开。

demo 版的 vite 分别实现了以下功能
![0](0.png)
- 将当前项目目录作为静态文件服务器的根目录
![1](1.png)
- 拦截部分文件请求
![2](2.png)
- 处理代码中import node_modules中的模块
![3](3.png)
- 处理vue单文件组件(SFC)的编译
![4](4.png)
采用@vue/compiler-sfc 来解析单文件组件，将组件解析分 template、script、style 三部分
- 通过WebSocket实现HMR
![5](5.png)
![5_2](5_2.png)
![6](6.png)

### Product

![product](product.png)

vite 的核心架构由 一个基于 ESM 的利用 esbuild 的开发服务器 和 基于 Rollup 的配置化的打包器组成，同时 vite 打造了一套基于rollup 的插件机制与生态。

#### 依赖预构建

> 首次启动 vite 时，Vite 在本地加载你的站点之前预构建了项目依赖。
原因：

1. CommonJS 和 UMD 兼容性: 在开发阶段中，Vite 的开发服务器将所有代码视为原生 ES 模块。因此，Vite 必须先将以 CommonJS 或 UMD 形式提供的依赖项转换为 ES 模块。
2. 性能： 为了提高后续页面的加载性能，Vite将那些具有许多内部模块的 ESM 依赖项转换为单个模块。

vite 是基于 no-bundle 的构建工具，在开发时按需编译而无需打包，但只针对于我们的业务代码，对于第三方依赖，vite 采用 esbuild 将依赖项转换为 ES 模块，这也是 vite 项目启动快的一个重要原因。

![speed](speed.png)

#### HMR 热更新
Vite 的 HMR API 基于[esm-hmr](https://github.com/FredKSchott/esm-hmr/tree/master) 规范来实现，在客户端与服务端建立了一个 websocket 连接，当代码被修改时，服务端发送消息通知客户端去请求修改模块的代码，完成热更新。

![流程](流程.png)

● server：服务端做的就是监听代码文件的改变，在合适的时机向客户端发送 websocket 信息通知客户端去请求新的模块代码。
● client：Vite 中客户端的 websocket 相关代码在处理 html 中时被写入代码中。

Vite 会接受到来自client 的消息。通过不同的消息触发一些事件。做到浏览器端的即时热模块更换。包括 connect、vue-reload、vue-rerender 等事件，分别触发组件vue 的重新加载，render等。

#### rollup
- 兼容 rollup 插件
   	- PluginContainer ：用来模拟 Rollup 调度各个 Vite 插件的执行逻辑

- 生产环境使用 rollup 打包
	- 如何 同一套 Vite 配置文件和源码，在开发环境和生产环境下的表现是一致的？
		- Vite 在开发模式下，模仿 Rollup 仿造出了一套拥有相同的 API 的插件架构，使得插件在两种模式下都能正常使用，保证了两种模式下 Vite 有相同的行为

### 总结

1. vite的核心正是依靠浏览器对ES Module规范的实现， 借助esm +  nobundle 所以构建快
2. 建立 在 esbuild   + rollup  两大基础上
3. 重点概念，依赖预构建，HMR 热更新

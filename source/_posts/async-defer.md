---
title: async & defer
date: 2017-08-30 17:55:17
tags:
  - async
  - defer
---

## 1.script标签的默认行为
首先我们先来看一下 `<script>` 标签 的几个重要特性：

`script`标签的会阻止文档渲染。相关脚本会立即下载并执行。
注意：（不带defer或async属性）
`document.currentScript` 可以获得当前正在运行的脚本(Chrome 29+, FF4+)。
脚本顺序默认情况下和`script`标签出现的顺序一致。

先来看一个例子，用这个例子来说明 script 的加载顺序问题。
<!-- more -->
## 2. script 的加载顺序
假设如下简单代码，我们去看一下，执行的结果是什么样子？
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>script 的加载顺序</title>
</head>
<body>

</body>
    <script src="./A.js"></script>
    <script src="./B.js"></script>
    <script src="./C.js"></script>
</html>
```
三个js文件分别alert各自的文件名字
执行结果如下：

按照正常的逻辑去执行，分别弹出了 ：
```
A
B
C
```

如果把B.js里面的内容换成网络上一个非常大的js文件

运行结果还是一样的

对于加在js，浏览器会做如下处理

- 停止解析 document
- 请求 A.js
- 执行 A.js 中的脚本
- 继续解析 document
- 遇到script B ，停止解析 document
- 请求 B.js
- 执行 B.js 中的脚本
- ......


## 3. script 标签的属性
script 标签 的六个属性。

type:可以看成language的替代属性；表示编写代码使用的脚本语言的内容类型。默认值为text/javascript。必须

src:表示包含要执行代码的外部文件。可选

charset:属性与 src 属性一起使用，告诉浏览器用来编码这个 JavaScript 程序的字符集。它的值是任何一个 ISO 标准字符集编码的名称。由于大多数浏览器会忽略它的值，因此这个属性很少有人用。可选

language:原来用于表示编写代码用的脚本语言，因大多数浏览器都会忽略这个属性，因此也没必要再用了。已废弃

async:表示立即下载该脚本，但不妨碍页面中的其他操作(比如：下载其他资源或等待加载其他脚本)，只对外部文件有效。可选

defer:表示脚本可以延迟到文档完全被解析和显示后再执行。只对外部文件有效。可选


注意：
> 虽然text/javascript 和text/ecmascript 都已经不被推荐使用，
> 但人们一直以来使用的都还是text/javascript。
> 实际上，服务端在传送JavaScript 文件时使用的 MIME 类型通常是application/x-javascript，
> 但在type中设置这个值却可能导致脚本被忽略。
> 另外，在非IE浏览器中
> 还可以使用以下值：application/javascript 和application/ecmascript。
> 考虑到约定成俗和最大限度的浏览器兼容性，
> 目前type 属性的值依旧还是text/javascript。

除此之外，属性 1 ~ 4 是 HTML4.01中定义的，而 5 和 6 则是 HTML 5 中新增的内容，那我们接下来就专门针对这两个新属性，来看一下，script 的阻塞式执行。

## 4. async & defer
两者都会并行下载，不会影响页面的解析。

defer 会按照顺序在 DOMContentLoaded 前按照页面出现顺序依次执行。

async 则是下载完立即执行。

#### 兼容性

[async](https://caniuse.com/#search=async)
[defer](https://caniuse.com/#search=defer)

#### defer
```html
<script src="defer1.js" defer></script>
<script src="defer2.js" defer></script>
```
不阻止解析 document， 并行下载 defer1.js, defer2.js
即使下载完 defer1.js, defer2.js 仍继续解析 document
按照页面中出现的顺序，在其他同步脚本执行后，DOMContentLoaded 事件前 依次执行 defer1.js, defer2.js

#### async
```html
<script src="async1.js" async></script>
<script src="async2.js" async></script>
```
不阻止解析 document, 并行下载 async1.js, async2.js
当脚本下载完后立即执行。（两者执行顺序不确定，执行阶段不确定，可能在 DOMContentLoaded 事件前或者后 ）

#### 其他
如果 `script` 无 `src` 属性，则 `defer, async` 会被忽略
动态添加的 `script` 标签隐含 `async` 属性。

#### 结论
- 两者都不会阻止 `document` 的解析
- `defer` 会在 `DOMContentLoaded` 前依次执行 （可以利用这两点哦！）
`async` 则是下载完立即执行，不一定是在 `DOMContentLoaded` 前
- `async` 因为顺序无关，所以很适合无依赖js

[测试地址](https://lozoe.github.io/async-defer/) 打开控制台
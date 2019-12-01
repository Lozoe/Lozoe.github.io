---
title: canvas学习
date: 2019-07-31 15:35:24
categories: [CSS]
---

## Canvas是用来干嘛的

canvas（画布）是HTML5中新增一个HTML5标签与操作canvas的javascript API，用于在网页实时生成图像，并且可以操作图像内容，可以实现在网页中完成动态的2D与3D图像技术。SVG以之间的一个重要的不同是，有一个基于 JavaScript 的绘图 API，而 SVG使用一个 XML 文档来描述绘图。SVG 绘图很容易编辑与生成，但功能明显要弱一些。 canvas可以完成动画、游戏、图表、图像处理等原来需要Flash完成的一些功能。
<!-- more -->
## 如何使用

### 一、首先新建一个`<canvas>`节点

```html
<canvas id="myCanvas" width="400" height="200">
你的浏览器不支持canvas!
</canvas>
```

复制代码上面代码中，如果浏览器不支持这个API，则就会显示`<canvas>`标签中间的文字——“您de浏览器不支持canvas！”。

每个canvas节点都有一个对应的context对象（上下文对象），Canvas API定义在这个context对象上面，所以需要获取这个对象，方法是使用getContext方法。

```js
var canvas = document.getElementById('myCanvas');
var ctx = canvas && canvas.getContext && canvas.getContext('2d');
```

复制代码上面代码中，getContext方法指定参数2d，表示该canvas节点用于生成2D图案（即平面图案）。如果参数是webgl，就表示用于生成3D图像（即立体图案），这部分实际上单独叫做WebGL API。

### 二、绘图

> canvas画布提供了一个用来作图的平面空间，该空间的每个点都有自己的坐标，x表示横坐标，y表示纵坐标。原点(0, 0)位于图像左上角，x轴的正向是原点向右，y轴的正向是原点向下

#### 1.绘制路径

> beginPath方法表示开始绘制路径，moveTo(x, y)方法设置线段的起点，lineTo(x, y)方法设置线段的终点，stroke方法用来给透明的线段着色。

```js
ctx.beginPath(); // 开始路径绘制
ctx.moveTo(10, 10); // 设置路径起点，坐标为(10,10)
ctx.lineTo(200, 10); // 绘制一条到(200,10)的直线
ctx.lineWidth = 10.0; // 设置线宽
ctx.strokeStyle = 'red'; // 设置线的颜色
ctx.stroke(); // 进行线的着色，这时整条线才变得可见
```

moveto和lineto方法可以多次使用。最后，还可以使用closePath方法，自动绘制一条当前点到起点的直线，形成一个封闭图形，省却使用一次lineto方法。

#### 2.绘制矩形

##### fillRect实心矩形

> fillRect(x, y, width, height)方法用来绘制矩形，它的四个参数分别为矩形左上角顶点的x坐标、y坐标，以及矩形的宽和高。fillStyle属性用来设置矩形的填充色。

```js
ctx.fillStyle = 'yellow';
ctx.fillRect(10, 30, 200, 100);
```

##### strokeRect空心矩形

```js
ctx.strokeRect(10, 150, 50, 50);
```

##### clearRect清除某个矩形区域的内容

```js
ctx.clearRect(10, 30, 50, 50);
```

#### 3.绘制文本

> fillText(string, x, y) 用来绘制文本，它的三个参数分别为文本内容、起点的x坐标、y坐标。使用之前，需用font设置字体、大小、样式（写法类似与CSS的font属性）。与此类似的还有strokeText方法，用来添加空心字。

```js
// 设置字体
ctx.font = "Bold 20px Arial";
// 设置对齐方式
ctx.textAlign = "left";
// 设置填充颜色
ctx.fillStyle = "green";
// 设置字体内容，以及在画布上的位置
ctx.fillText("Hello!", 10, 50);
// 绘制空心字
ctx.strokeText("Hello!", 10, 100);
```

复制代码fillText方法不支持文本断行，即所有文本出现在一行内。所以，如果要生成多行文本，只有调用多次fillText方法。

#### 4.绘制圆形和扇形

> arc方法用来绘制扇形，ctx.arc(x, y, radius, startAngle, endAngle, anticlockwise);
复制代码arc方法的x和y参数是圆心坐标，radius是半径，startAngle和endAngle则是扇形的起始角度和终止角度（以弧度表示），anticlockwise表示做图时应该逆时针画（true）还是顺时针画（false）。

绘制实心圆形

```js
ctx.beginPath();
ctx.arc(120, 180, 50, 0, Math.PI * 2, true);
ctx.fillStyle = "#eee";
ctx.fill();
```

绘制空心圆形

```js
ctx.beginPath();
ctx.arc(230, 180, 50, 0, Math.PI * 2, true);
ctx.lineWidth = 1.0;
ctx.strokeStyle = "#eee";
ctx.stroke();
```

demo:
<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="lozoe" data-slug-hash="JgNLab" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="canvas">
  <span>See the Pen <a href="https://codepen.io/lozoe/pen/JgNLab/">
  canvas</a> by Iona (<a href="https://codepen.io/lozoe">@lozoe</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

参考：

https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage

https://juejin.im/post/5ac437b5f265da238f12c1c6

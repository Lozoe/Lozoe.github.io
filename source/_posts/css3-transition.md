---
title: css3-transition
date: 2016-08-16 09:33:34
categories: [CSS]
tags:
  - CSS3
---

CSS3提供了transition的动画支持，transition动画可以控制HTML组件的某个属性发生改变时会经历一段时间、以平滑渐变的方式发生改变，从而产生动画效果。
### 使用
- transition-property：指定对HTML元素的哪个CSS属性进行平滑渐变处理，如 width、height、background-color、all(所有发生变化的属性)
- transition-duration：过渡的时长，如 1s
- transition-timing-function：指定渐变的速度
```
ease: 动画开始时较慢，然后速度加快，到达最大速度后再减慢速度
linear: 线性速度，开始到结束时的速度保持不变
ease-in: 动画开始时速度较慢，然后速度加快
ease-out: 动画开始时速度很快，然后速度减慢
ease-in-out: 动画开始时速度较慢,然后速度加快到达最大速度后再减慢速度
cubic-bezier(x1, y1, x2, y2): 通过贝塞尔曲线控制动画速度，可以取代以上几个值
```
- transition-delay：指定延迟动画开始的时间，如 0.5s
<!-- more -->
举个栗子：
```html
<div class="transition-1">transition</div>
<div class="transition-2">transition</div>
```
```css
.transition-1 {
  width: 100px;
  height: 100px;
  color: #000;
  line-height: 100px;
  text-align: center;
  background: red;
  transition: all 1s linear 0.5s;
}
.transition-1:hover {
  width: 200px;
  height: 200px;
  line-height: 200px;
  color: #fff;
  background: green;
}

.transition-2 {
  width: 100px;
  height: 100px;
  color: #000;
  line-height: 100px;
  text-align: center;
  background: red;
  transition: width 1s linear 0.5s, height 1s linear 0.5s, background 2s linear 0.5s;
}
.transition-2:hover {
  width: 200px;
  height: 200px;
  line-height: 200px;
  color: #fff;
  background: green;
}
```
效果：https://codepen.io/lozoe/pen/KxxgdQ
 
ps: 只有当过渡状态过程中，transition 属性存在（transition 被选择器应用时）时，才会有过渡的效果，否则没有过渡效果。
若将 transition 属性应用在 div 被 hover 时的选择器上，则离开div是就没有了过渡效果。


### 贝塞尔曲线

https://developer.mozilla.org/zh-CN/docs/Web/CSS/timing-function
贝塞尔曲线（Bezier curve）是计算机图形学中重要的参数曲线，它通过一个方程来描述一条曲线。根据方程的最高阶数，可以分为线性贝塞尔曲线、二次贝塞尔曲线、三次贝塞尔曲线以及更高次的贝塞尔曲线。

在 css3 中使用的 cubic-bezier() 函数，是一个 三次贝塞尔曲线函数。

三次贝塞尔曲线中四个点，在 cubic-bezier() 中：
```
第一个点 p0(0, 0)和最后一个点 p3(1, 1)是固定坐标的
p1(x1, y1) 和 p2(x2, y2) 是传入 cubic-bezier() 函数中的参数的。其中 x∈[0, 1]，y 可以不在 [0, 1] 区间内，但最大值最好不要大于 1.5，最小值不要小于 -0.5
0 和 1 分别表示 0% 和 100%
```

cubic-bezier(x1, y1, x2, y2) 接受的参数便是 p1(x1, h1) 和 p2(x2, y2) 的坐标。

[贝塞尔曲线参数获取](http://cubic-bezier.com/#.14,.56,.79,.35)


- 横坐标：时间。时间是匀速增加的
- 纵坐标：进度。随着时间的增加，进度也会增加
- 斜率：速度

这个 进度 在 css 中，实际指的就是样式变化前后的值。如：

- width 从 100px 变为 200px，纵坐标的起点就为 100px，终点为 200px
- opacity 从 0 变为 1，纵坐标的起点就为 0，终点为 1

平抛： 
- 水平方向：匀速运动
- 垂直方向：加速度不变的匀加速运动
```html
<button class="move">运动</button>
<button class="reset">重置</button>
<div class="ball"></div>
```
```css
button.move {
  position: absolute;
  width: 50px;
  height: 30px;
  top: 20px;
  left: 20px;
}
button.reset {
  position: absolute;
  width: 50px;
  height: 30px;
  top: 50px;
  left: 20px;
}
.ball {
  width: 10px;
  height: 10px;
  border: 1px solid red;
  border-radius: 50%;
  position: absolute;
  left: 80px;
  top: 30px;
}
.ball.end {
  left: 180px;
  top: 230px;
  transition: left 0.2s linear, top 0.2s cubic-bezier(.48,0,.94,.28);
}
```
```js
const moveBtn = document.querySelector('.move'),
      resetBtn = document.querySelector('.reset'),
      ball = document.querySelector('.ball');
moveBtn.addEventListener('click', function() {
  ball.className += ' end';
})
resetBtn.addEventListener('click', function() {
  let classNameArr = ball.className.split(' ');
  ball.className = classNameArr[0];
})
```

http://cubic-bezier.com/#.48,0,.94,.28

css3 中的贝塞尔曲线其实很简单：一条以 时间为横坐标，以 进度为纵坐标 的 和速度相关 的曲线，它表示了 过渡/动画 中间状态的 变化快慢。

[贝塞尔曲线扫盲](http://www.html-js.com/article/1628)
http://myst729.github.io/bezier-curve/


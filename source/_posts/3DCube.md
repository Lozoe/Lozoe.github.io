---
title: css 3D立方体
date: 2019-06-16 12:01:43
categories: [CSS3]
tags:
  - css
---

曾经被问过实现一个3d立方体，太久不写css了，有些css3的属性一时还有点陌生了，这次通过这个小case也来复习一下css3的变换相关的内容。

## html结构

````html
<div class="box">
  <div class="container">
    <div class="side front">front</div>
    <div class="side top">top</div>
    <div class="side bottom">bottom</div>
    <div class="side left">left</div>
    <div class="side right">right</div>
    <div class="side back">back</div>
  </div>
</div>
````
<!-- more -->
## 设置透视效果

给box设置景深透视效果（类似于threejs中的景深概念），因为设置景深透视效果的元素，其效果是作用在子元素上，而不是元素自身。

````css
.box {
  // 这里给box定宽高的原因是透视中心好确定，否则需要计算子元素container相对于box的位置而设置perspective－origin属性。
  height: 200px;
  width: 200px;
  // 这里仅仅为了留白
  margin: 100px 0 0 100px;
  // 关键点1: 设置透视效果。让眼睛离屏幕远一些，这样看立方体的时候就比较远，效果好些（如果设为100px，那正好是在front的位置看立方体）
  perspective: 800px;
  user-select: none;
}
````

## 设置立方体六个面的容器，让它动起来，方便看立体效果

````css
.container {
  position: relative;
  height: 200px;
  width: 200px;
  // 因为该元素的animation内有transform，所以设置下transform-origin，但其实默认值就是center，可不设置
  transform-origin: 50% 50% 100px;
  // 关键点2：设置transform的效果是否启用3d
  transform-style: preserve-3d;
  animation: rotate-frame 8s infinite linear;
  animation-play-state: paused;
}
.container:hover {
  animation-play-state: running;
}

@keyframes rotate-frame{
  0% {
    transform: rotateX(0deg);
  }
  25% {
    transform: rotateX(180deg);
  }
  50% {
    transform: rotateX(360deg) rotateY(0deg);
  }
  75% {
    transform: rotateX(360deg) rotateY(180deg);
  }		
  100% {
    transform: rotateX(360deg) rotateY(360deg);
  }
}
````

## 设置六个面的通用样式，让面可以被看得到

````css
.side {
  // 整个都是在设置六个面的与3d效果无关的外观，不必关注
  position: absolute;
  display: inline-block;
  width: 200px;
  height: 200px;
  line-height: 200px;
  text-align: center;
  opacity: 0.5;
  box-shadow: inset 0 0 50px rgba(0, 0, 0, 0.9);
}
````

## 设置每个面的位置，使得六个面围成一个立方体

````css
.top {
  transform-origin: top;
  transform: rotateX(-90deg) translateY(-100px);
}
.bottom {
  transform-origin: bottom;
  transform: rotateX(90deg) translateY(100px);
}
.left {
  // 设置变换动作的基准
  transform-origin: left;
  // 设置了两个变换
  // 1 rotateY，让它竖起来
  // 2 让它向屏幕外部移动100px，以达到立方体中心点在屏幕上
  // 其他五个面同理
  transform: rotateY(90deg) translateX(-100px);
}
.right {
  transform-origin: right;
  transform: rotateY(-90deg) translateX(100px);
}
.back {
  transform: translate3d(0, 0, -100px);
}
.front {
  transform: translate3d(0, 0, 100px);
}
````

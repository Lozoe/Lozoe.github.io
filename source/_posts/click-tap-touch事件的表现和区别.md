---
title: click/tap/touch事件的表现和区别
date: 2020-04-06 11:10:27
categories: [JavaScript]
---

## click

click是手指触碰屏幕和离开屏幕的偏移量很小情况下并在短时间内执行而发生的事件，用于移动端会有200到300毫秒的延时

要模拟click事件，touchstart、touchmove、touchend必须结合使用，通过touchstart、touchend纪录时间、手指移动的偏移量，符合click标准则触发click事件。而现有的解决方案FastClick就是这么做的。

```js
$(function(){
    newFastClick(document.body);
})
```

移动端的click300ms延迟现象：移动端的click事件300ms延迟是因为浏览器为了判断当前事件是否双击（double click）而在触发touchend事件后等待用户约300ms（双击事件可以缩放视图内容），若用户没有再次点击则默认触发click事件。

用户点击触发事件的过程：touchstart －> touchmove(没有移动则不触发) －> touchend －>(300ms后) click

事件对象及兼容

```js
事件对象：e = e || window.event;
事件冒泡： e.stopPropagation() || e.cancelBubble = true
事件源：e.target || e.srcElement
阻止默认事件：e.preventDefault() || e.returnValue = false;
鼠标位置：
if(e.pageX) {
    xPos = evt.pageX;
    yPos = evt.pageY;
} else {
    // ie6到8不兼容
    xPos = e.clientX +（document.documentElement.scrollLeft || document.body.scrollLeft;
    yPos = e.clientY +（document.documentElement.scrollTop || document.body.scrollTop;
}
```

## touch 类事件

属于原生移动端事件

```js
touchstart: 当手指触摸到屏幕会触发；
touchmove: 当手指在屏幕上移动时，会触发；
touchend: 当手指离开屏幕时，会触发；
touchcancel: 是在拖动中断时候触发。
```

Touch对象代表一个触点，可以通过event.touches[0]获取，每个触点包含位置，大小，形状，压力大小，和目标 element属性。

事件对象及兼容

```js
touches：表示当前跟踪的触摸操作的touch对象的数组。
targetTouches：特定于事件目标的Touch对象的数组。
changeTouches：表示自上次触摸以来发生了什么改变的Touch对象的数组。

每个touch事件包含下面的属性：

clientX：触摸目标在视口中的x坐标。
clientY：触摸目标在视口中的y坐标。
identifier：标识触摸的唯一ID。
pageX：触摸目标在页面中的x坐标。
pageY：触摸目标在页面中的y坐标。
screenX：触摸目标在屏幕中的x坐标。
screenY：触摸目标在屏幕中的y坐标。
target：触摸的DOM节点目标。
```

关于touch的滑动事件

```js
ele.addEventListener('touchstart', start, false);
ele.addEventListener('touchmove', move, false);
ele.addEventListener('touchend', end, false);
function start(e){
    this.startX = e.touches[0].pageX
    this.startY = e.touches[0].pageY;
}
function move(e){
    this.moveEndX = e.changedTouches[0].pageX;
    this.moveEndY = e.changedTouches[0].pageY;
    this.X = this.moveEndX - this.startX;
    this.Y = this.moveEndY - this.startY;
    if (Math.abs(this.X) > Math.abs(this.Y)&&Math.abs(this.X)>30&& this.X > 0) {
        //向右滑
        console.log('right');
    }else if (Math.abs(this.X) > Math.abs(this.Y)&&Math.abs(this.X)>30&& this.X < 0) {
        //向左滑
        console.log('left');
    }
}
function end(e){
    ele.removeEventListener('touchmove', start, false);
    ele.removeEventListener('touchend', move, false);
}
```

## tap

属于第三方事件，需要引入第三方库，比如zepto.js

常用的方法有tap，singleTap, doubleTap, longTap分别代表点击、单次点击、双次点击、长按。

tap封装了touchstart、touchmove、touchend三个事件的处理（touchstart之后如果有产生touchmove则取消此次tap事件的产生）click则只是单纯的绑定了浏览器的click事件。

```js
bug: tap点透事件。(点击会触发非当前层的点击事件，这就是点透)
原因: tap事件是通过监听绑定document上的touch事件来模拟的，并且tap事件是冒泡到document上才触发的。
解决: 为元素绑定touchend事件，并在内部加上e.preventDefault();
```

参考：

https://zhuanlan.zhihu.com/p/26351643

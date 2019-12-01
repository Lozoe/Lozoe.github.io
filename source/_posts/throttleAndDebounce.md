---
title: 函数节流和函数防抖
date: 2018-07-04 15:33:02
categories: [JavaScript]
tags:
  - throttle
  - debounce
---

### 一、概述

函数节流和去抖的出现场景，一般都伴随着客户端 DOM 的事件监听。举个例子，实现一个原生的拖拽功能（不能用 H5 Drag&Drop API），需要一路监听 mousemove 事件，在回调中获取元素当前位置，然后重置 dom 的位置（样式改变）。如果我们不加以控制，每移动一定像素而触发的回调数量是会非常惊人的，回调中又伴随着 DOM 操作，继而引发浏览器的重排与重绘，性能差的浏览器可能就会直接假死，这样的用户体验是非常糟糕的。我们需要做的是降低触发回调的频率，比如让它 500ms 触发一次，或者 200ms，甚至 100ms，这个阈值不能太大，太大了拖拽就会失真，也不能太小，太小了低版本浏览器可能就会假死，这样的解决方案就是函数节流，英文名字叫「throttle」。函数节流的核心是，让一个函数不要执行得太频繁，减少一些过快的调用来节流。指定时间间隔内只会执行一次任务。
<!-- more -->
说完函数节流，再看它的好基友函数去抖（debounce）。思考这样一个场景，对于浏览器窗口，每做一次 resize 操作，发送一个请求，很显然，我们需要监听 resize 事件，但是和 mousemove 一样，每缩小（或者放大）一次浏览器，实际上会触发 N 多次的 resize 事件，用节流？节流只能保证定时触发，我们一次就好，这就要用去抖。简单的说，任务频繁触发的情况下，只有任务触发的间隔超过指定间隔的时候，任务才会执行。

### 二、节流(throttle)

设定一个时间间隔，某个频繁触发的函数，在这个时间间隔内只会执行一次。也就是说，这个频繁触发的函数会以一个固定的周期执行。

#### 应用场景

1. DOM 元素的拖拽功能实现(mousemove),当某个div节点被拖动的时候频繁触发拖拽函数
2. window.onresize事件
3. 监听滚动事件判断是否到页面底部自动加载更多()
4. 射击游戏的 mousedown/keydown 事件
5. 计算鼠标移动的距离(mousemove)
6. 上传进度

我们发现这些场景共同的问题就是函数被触发的频率太高。

#### 代码实现

关于函数节流的代码实现有很多种，今天来介绍两种思路。

主要思想都是一样：
将即将被执行的函数用setTimeout延迟一段时间执行。如果该次延迟执行还没有完成，则忽略接下来调用该函数的请求。

第一种：通过timer来标识

```js
function throttle(func, interval) {
    var timer;
    var firstTime = true;
    return function () {
        // 如果是第一次调用不需要延迟执行
        if (firstTime) {
            func.apply(this, arguments);
            firstTime = false;
            return;
        }

        // 如果定时器还在，说明前一次延迟执行还没有完成
        if (timer) {
            return false;
        }

        // 延迟一段时间执行
        timer = setTimeout(() => {
            clearTimeout(timer);
            timer = null;
            func.apply(this, arguments);
        }, interval || 500);
    }
}
// 测试
var n = 0;
window.onresize = throttle(function () {
    console.log("window resized: " + n++);
}, 3000);
```

第二种：通过时间差和间隔时间对比

```js
function throttle(func, delay, context) {
    var last, deferTimer
    return function() {
        var _this = this
        var _args = arguments
        var now = +new Date()
        //  小于delay触发
        if(last && now - last < delay) {
            deferTimer = setTimeout(function() {
                clearTimeout(deferTimer)
                last = now
                func.apply(_this, _args)
            }, delay)
        } else {
            last = now
            func.apply(_this, _args)
        }
    }
}
// 测试一
var n = 0;
window.onresize = throttle(function () {
    console.log("window resized: " + n++);
}, 3000);

// 测试二
function T() {
  document.body.innerHTML += 'T<br>'
}

window.addEventListener('mousemove', throttle(T, 1000));
```

要点：其判断的条件有一条关键就是不管通过什么条件来判断，每次在setTimeout执行的函数体中需要重置这个条件，比如timer,last的时间

tips:其实发现throttle 把if else条件去掉就是debounce函数

### 三、防抖(debounce)

设定一个时间间隔，当某个频繁触发的函数执行一次后，在这个时间间隔内不会再次被触发，如果在此期间尝试触发这个函数，则时间间隔会重新开始计算

#### 应用场景

1. 每次 resize/scroll 触发统计事件
2. 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，验证一次就好）

#### 代码实现

```js
function debounce(func, delay = 300, context) {
    return function() {
        var _context = context || this
        var _args = arguments
        clearTimeout(func.id)
        func.id = setTimeout(function() {
            func.apply(_context, _args)
        }, delay)
    }
}

function D() {
  document.body.innerHTML += 'D<br>'
}

window.addEventListener('mousemove', debounce(D, 1000));

```

### 四、小结

函数节流和函数去抖的核心其实就是限制某一个方法被频繁触发，而一个方法之所以会被频繁触发，大多数情况下是因为 DOM 事件的监听回调，而这也是函数节流以及去抖多数情况下的应用场景。

参考： https://www.jianshu.com/p/54b75c6f2caa
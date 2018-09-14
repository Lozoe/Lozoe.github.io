---
title: 移动端1px边框解决方案
date: 2015-11-10 18:25:26
categories: [CSS]
tags:
  - css
---

在移动端web页面开发中，为了使css中使用的尺寸与设计稿一致，通常会采用 rem 单位配合 [lib-flexible](https://github.com/amfe/lib-flexible) 来实现移动端的适配，在IOS设备上 flexible.js 会根据设备的分辨率动态的调整 viewport 的 width 和 scale 来达到目的。

但是，现在很多的安卓手机也是高分辨率的屏幕，有些1px边框的按钮和列表会显得特别粗，flexible.js 只处理了IOS的手机，所以安卓手机需要我们自己处理。

网友众说纷纭，方案也很多，今天就小白的理解来重新总结一下。
<!-- more -->
##### JS解决方案：
可以通过 window.devicePixelRatio 拿到设备的像素比，然后给 html 标签加上的相应的样式。

```js
function retina () { // 高分辨率屏幕处理
    if (navigator.userAgent.toUpperCase().includes('IPHONE OS')) return // IOS会缩放，不处理
    let classNames = []
    let pixelRatio = window.devicePixelRatio || 1
    let html = document.querySelector('html')
    classNames.push('pixel-ratio-' + Math.floor(pixelRatio))
    if (pixelRatio >= 2) {
        classNames.push('retina')
    }
    for(let clsName of classNames) {
        html.classList.add(clsName);
    }
}
```
这样一来我们可以通过, html.pixel-ratio-2 给不同分辨率的屏幕加上特殊的样式处理。

### 伪类 + 缩放 

retina 屏的浏览器可能不认识0.5px的边框，将会把它解释成0px，没有边框。包括 iOS 7 和之前版本，OS X Mavericks 及以前版本，还有 Android 设备。所以可以使用伪元素相对父元素放大对应倍数，再缩小对应devicePixelRatio倍数，此处可以配合媒体查询来使用

```css
.item {
  position: relative;
}

.pixel-ratio-3 .item:after {
  width: 300%;
  height: 300%;
  -webkit-transform: scale(0.33333);
  transform: scale(0.33333);
}

.pixel-ratio-2 .item:after {
  width: 200%;
  height: 200%;
  -webkit-transform: scale(0.5);
  transform: scale(0.5);
}

.item:after {
  pointer-events: none;
  position: absolute;
  z-index: 999;
  top: 0;
  left: 0;
  content: "\0020";
  border-color: #ddd;
  border-style: solid;
  border-width: 1px 0 0;
  -webkit-transform-origin: 0 0;
  transform-origin: 0 0;
}

```

##### 优点：

- 所有场景都能满足
- 支持圆角(伪类和本体类都需要加border-radius)

##### 缺点：
- 对于已经使用伪类的元素(例如clearfix)，可能需要多层嵌套

### viewport + rem

同时通过设置对应viewport的rem基准值，这种方式就可以像以前一样轻松愉快的写1px了。
这种兼容方案相对比较完美，适合新的项目，老的项目修改成本过大。
对于这种方案，可以看看[手淘H5的终端适配](https://github.com/amfe/article/issues/17)

[百度](http://caibaojian.com/mobile-responsive-example.html)

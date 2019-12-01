---
title: css实现自适应正方形
date: 2019-07-26 14:07:30
categories: [CSS]
---

在处理移动端页面时，我们有时会需要将 banner 图做成与屏幕等宽的正方形以获得最佳的体验效果，那么应该怎么使用纯 CSS 制作出能够自适应大小的正方形呢？

## 方案一： CSS3 vw 单位

CSS3 中新增了一组相对于可视区域百分比的长度单位 vw, vh, vmin, vmax。其中 vw 是相对于视口宽度百分比的单位，1vw = 1% viewport width， vh 是相对于视口高度百分比的单位，1vh = 1% viewport height；vmin 是相对当前视口宽高中 较小 的一个的百分比单位，同理 vmax 是相对当前视口宽高中 较大 的一个的百分比单位。该单位浏览器兼容性如下：
<!-- more -->
![兼容性](兼容性.png)

利用 vw 单位，我们可以很方便做出自适应的正方形：

```html
<style>
.placeholder {
  width: 10%;
  height: 10vw;
}
</style>
<div class="placeholder"></div>
```

优点：简洁方便
缺点：浏览器兼容不好

## 方案二：设置垂直方向的 padding 撑开容器

在 CSS 盒模型中，一个比较容易被忽略的就是 margin, padding 的百分比数值计算。按照规定，margin, padding 的百分比数值是相对 父元素宽度 的宽度计算的。由此可以发现只需将元素垂直方向的一个 padding 值设定为与 width 相同的百分比就可以制作出自适应正方形了：

但有一个问题，当内容有高度时，会高度溢出：

显然这就不是一个正方形了。结局方案是：

```css
.placeholder {  
  height: 0;
}
```

简洁明了，且兼容性好。

## 方案三：利用伪元素的 margin(padding)-top 撑开容器

在方案二中，我们利用百分比数值的 padding-bottom 属性撑开容器内部空间，但是这样做会导致在元素上设置的 max-height 属性失效。

其实失效的原因不言而喻，max-height 属性只限制于 height。也就是只会对元素的 content height 起作用。那么我们是不是能用一个子元素撑开 content 部分的高度，从而使 max-height 属性生效呢？我们来试试：

```css
.placeholder {
    width: 100%;
    background:green;
}

.placeholder:after {
    content: '';
    display: block;
    padding-top: 100%; /* margin 百分比相对父元素宽度计算 */
}
```

这里就涉及到 [margin collapse](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing) 的概念了，由于容器与伪元素在垂直方向发生了外边距折叠，所以我们想象中的撑开父元素高度并没有出现。而应对的方法是在**父元素上触发 BFC**

```css
.placeholder {
  overflow: hidden;
}
```

注：若使用垂直方向上的 padding 撑开父元素，则不需要触发 BFC

即:

```html
<style>
.placeholder3 {
    width: 10%;
    background: yellow;
}

.placeholder3:after {
    content: '';
    display: block;
    /* padding-top: 100%; */ /* margin 百分比相对父元素宽度计算 */
}
</style>
<div class="placeholder3"></div>
```

## 效果

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="css,result" data-user="lozoe" data-slug-hash="bXwaEW" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="自适应正方形">
  <span>See the Pen <a href="https://codepen.io/lozoe/pen/bXwaEW/">
  自适应正方形</a> by Iona (<a href="https://codepen.io/lozoe">@lozoe</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

对于元素内部添加内容时高度会溢出这样的情况，可以将内容放到独立的内容块中，利用绝对定位消除空间占用。

## 结语

以上就是三种制作自适应正方形的方案，抛去 CSS3 中的视口相对单位，主要利用到 margin, padding 的百分比数值相对父元素宽度的宽度计算得出 来制作宽高相等、且相对视口宽度自适应的正方形。

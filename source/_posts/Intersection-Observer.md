---
title: Intersection Observer
date: 2021-06-28 14:41:06
tags:

---

IntersectionObserver接口 (从属于Intersection Observer API) 提供了一种异步观察目标元素与其祖先元素或顶级文档视窗(viewport)交叉状态的方法。祖先元素与视窗(viewport)被称为根(root)。（简言之，提供给web开发者，一种异步查询元素相对于其他元素或窗口位置的能力。它常用于追踪一个元素在窗口的可视问题。）

当一个IntersectionObserver对象被创建时，其被配置为监听根中一段给定比例的可见区域。一旦IntersectionObserver被创建，则无法更改其配置，所以一个给定的观察者对象只能用来监听可见区域的特定变化值；然而，你可以在同一个观察者对象中配置监听多个目标元素。

在 Intersection Observer 出来之前，传统位置计算的方式，依赖于对DOM状态的轮询计算，然而这种方式会在主线程里密集执行从而造成页面性能问题，getBoundingClientRect()的频繁调用也可能引发浏览器的样式重计算和布局。如果是在iframe里，因为同源策略，我们不能直接访问元素，也就很难用传统方式去处理iframe里的元素。

Intersection Observer 的设计，就是为了更方便的处理元素的可视问题。使用 Intersection Observer 我们可以很容易的监控元素进入和离开可视窗口，实现节点的预加载和延迟加载。Intersection Observer 并不是基于像素变化的实时计算，它的反馈会有一定的延时，这种异步的方式减少了对DOM和style查询的昂贵计算和持续轮询，相比传统方式降低了CPU、GPU的消耗。

<!-- more -->

## Intersection Observer 使用方法
```javascript
// 定义相交监视器
var observer = new IntersectionObserver(changes => {
  for (const change of changes) {
    console.log(change.time);               // 发生变化的时间
    console.log(change.rootBounds);         // 根元素的矩形区域的信息
    console.log(change.boundingClientRect); // 目标元素的矩形区域的信息
    console.log(change.intersectionRect);   // 目标元素与视口（或根元素）的交叉区域的信息
    console.log(change.intersectionRatio);  // 目标元素与视口（或根元素）的相交比例
    console.log(change.target);             // 被观察的目标元素
  }
}, {});

// 开始观察某个目标元素
observer.observe(target);

// 停止观察某个目标元素
observer.unobserve(target);

// 关闭监视器
observer.disconnect();

```

#### 1. IntersectionObserver Callback

当目标元素和根元素相交发生改变时，会触发监视器的回调。

```javascript
callback IntersectionObserverCallback = void (
    sequence<IntersectionObserverEntry> entries, 
    IntersectionObserver observer
);

```

回调的参数，entries 是一个IntersectionObserverEntry数组，数组的长度是我们监控目标元素的个数，另一个参数是 IntersectionObserver 对象。

#### 2. IntersectionObserver Entry

IntersectionObserverEntry 对象提供了目标元素与跟元素相交的详细信息。他有如下几个属性。

```javascript
interface IntersectionObserverEntry {
  readonly attribute DOMHighResTimeStamp time;
  readonly attribute DOMRectReadOnly? rootBounds;
  readonly attribute DOMRectReadOnly boundingClientRect;
  readonly attribute DOMRectReadOnly intersectionRect;
  readonly attribute boolean isIntersecting;
  readonly attribute double intersectionRatio;
  readonly attribute Element target;
};

{
    time: 3893.92,
    rootBounds: ClientRect {
        bottom: 920,
        height: 1024,
        left: 0,
        right: 1024,
        top: 0,
        width: 920
    },
    boundingClientRect: ClientRect {
        // ...
    },
    intersectionRect: ClientRect {
        // ...
    },
    isIntersecting: 0.54,
    intersectionRatio: 0.54,
    target: element
}

```

[![](https://user-gold-cdn.xitu.io/2019/8/29/16cdccc7f71311ee?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2019/8/29/16cdccc7f71311ee?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- time：发生相交到相应的时间，毫秒。
- rootBounds：根元素矩形区域的信息，如果没有设置根元素则返回null，图中蓝色部分区域。
- boundingClientRect：目标元素的矩形区域的信息，图中黑色边框的区域。
- intersectionRect：目标元素与视口（或根元素）的交叉区域的信息，图中蓝色方块和粉红色方块相交的区域。
- isIntersecting：目标元素与根元素是否相交
- intersectionRatio：目标元素与视口（或根元素）的相交比例。
- target：目标元素，图中黑色边框的部分。

#### 3. IntersectionObserver Option

IntersectionObserver Option 是 IntersectionObserver 构造函数的第二个参数，用来配置监视器的部分信息。

```javascript
dictionary IntersectionObserverInit {
  Element?  root = null;
  DOMString rootMargin = "0px";
  (double or sequence<double>) threshold = 0;
};

```
- root：设置监视器的根节点，不传则默认为视口。
- rootMargin： 类似于 CSS 的 margin 属性。用来缩小或扩大 rootBounds，从而影响相交的触发。
- threshold：属性决定相交比例为多少时，触发回调函数。取值为 0 ~ 1，或者 0 ~ 1的数组。
如下图，当我们把  threshold 设置为 [0, 0.25, 0.5, 0.75, 1]，绿色方块分别在 0%，25%，50%，75%，100% 可见时，触发回调函数。

[![](https://user-gold-cdn.xitu.io/2019/8/29/16cdcce2aeb7262b?imageslim)](https://user-gold-cdn.xitu.io/2019/8/29/16cdcce2aeb7262b?imageslim)

## Intersection Observer 使用场景

#### 1. 构建自定义的预加载或延迟加载DOM和数据。

比如说我们可以用 Intersection Observer 实现图片的懒加载，当图片达到用户可视区的时候再进行加载。
优化大型网站性能，优先渲染用户可视的视图，根据用户页面滚动，按需加载视图。

#### 2. 实现高性能的滚动列表。

实现列表的上拉加载，减少长列表的滑动卡顿。

#### 3. 计算元素的可视性。

优化广告推广的有效性，只有广告进入用户可视区，才算是有效的广告计数。
实现用户把播放器移除可视区时的自动暂停，进入可视区再进行播放。
还可以高效的实现视差动画。

## Intersection Observer 兼容性

[![](https://user-gold-cdn.xitu.io/2019/8/29/16cdccd6278eb25a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2019/8/29/16cdccd6278eb25a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

IntersectionObserver 目前已经被主流的浏览器支持，我们可以使用渐进支持的方式使用它。对于目前浏览器的生态，我们也要做好向下降级的措施。
我们也可以使用 IntersectionObserver polyfill 增加浏览器的兼容，具体可以查看 polyfill.io。

[![](https://user-gold-cdn.xitu.io/2019/8/29/16cdccd7bbc36332?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2019/8/29/16cdccd7bbc36332?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


参考：
[Intersection Observer](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
[深入理解 Intersection Observer](https://juejin.cn/post/6844903927419256846)

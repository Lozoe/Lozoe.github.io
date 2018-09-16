---
title: css3-animation
date: 2016-08-16 11:08:56
categories: [CSS]
tags:
  - CSS3
---

和transition相比，animation还可以自定义多个关键帧，浏览器将会负责计算和插入关键帧的虚拟动画帧
### 用法
1. animation-name	规定需要绑定到选择器的 keyframe 名称。。
2. animation-duration	规定完成动画所花费的时间，以秒或毫秒计。
3. animation-timing-function	规定动画的速度曲线。
4. animation-delay	规定在动画开始之前的延迟。
5. animation-iteration-count	规定动画应该播放的次数。
6. animation-direction	规定是否应该轮流反向播放动画。
<!-- more -->
详见：http://www.w3school.com.cn/cssref/pr_animation.asp

举个栗子：

```html
<span class="cursor"></span>
```
```css
/* keyframes */
@keyframes fade-in-out {
    0% {
        opacity: 0;
    }
    0.1% {
        opacity: 1;
    }
    52% {
        opacity: 1;
    }
    52.1% {
        opacity: 0;
    }
    100% {
        opacity: 0;
    }
}
@-webkit-keyframes fade-in-out {
    0% {
        opacity: 0;
    }
    0.1% {
        opacity: 1;
    }
    52% {
        opacity: 1;
    }
    52.1% {
        opacity: 0;
    }
    100% {
        opacity: 0;
    }
}
.cursor {
    position: absolute;
    display: inline-block;
    vertical-align: middle;
}
.cursor:after {
    content: '|';
    width: .3em;
    font-weight: 300;
    font-size: 30px;
    text-align: center;
    color: black;
    animation: fade-in-out 1s linear infinite;
    -webkit-animation: fade-in-out 1s linear infinite;
}
```

效果：https://codepen.io/lozoe/pen/oPPzMx
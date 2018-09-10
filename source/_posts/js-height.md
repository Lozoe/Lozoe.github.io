---
title: 原生js获取元素高度
date: 2016-02-10 18:42:45
tags:
---

有时在写页面时，需要获取一个div的高度。怎么才能获取呢？哈哈，先上结论。有两种方法。

- offsetHeight 、clientHeight
- getComputedStyle
### offsetHeight 与 clientHeight
这两个属性都能获取元素的高度，它们有什么区别呢？
<!-- more -->
```html
<!DOCTYPE html>
<html>
    <head>
        <title>demo</title>
        <meta charset="utf-8">
        <style type="text/css">
            #demo {
                width: 100px;
                height: 200px;
                margin: 10px;
                padding: 20px;
                border: 2px solid #bdbdbd;
            }
        </style>
    </head>
    <body>
        <div id="demo">hello</div>
        <script type="text/javascript">
            var div = document.getElementById('demo');
            console.log(div.offsetHeight); // 244
            console.log(div.clientHeight); // 240
        </script>
    </body>
</html>
```

可以看出
```js
offsetHeight = content + border + padding = 200 + 2 * 2 + 20 * 2 = 244
clientHeight = content + padding = 200 + 2 * 20 = 240
```
两个属性的差距于是就显而易见了。同样返回元素的高度，**offsetHeight的值包括元素内容+内边距+边框**，而**clientHeight的值等于元素内容+内边距**。区别就在于有没有边框~
但是，问题来了，我们想获取元素本身的高度呢？于是——
请往下看~

有小伙伴可能会说，我们可以直接利用style属性获取元素高度。然而在上面的代码中，
```
console.log(div.style.height) // ''
```

**为空！**
这是因为style属性只能获取元素标签style属性里的值。对于一个光秃秃的`<div>`，style的输出是空的。
下面把内部样式表中的高度注释掉，写在行内样式中，改动如下。
```css
#demo {
    width: 100px;
    /*height: 200px;*/
    background: yellow;
    margin: 10px;
    padding: 10px;
    border: 2px solid blue;
}
```
```html
<div id="demo" style="height: 200px">hello</div>
```
这个时候style属性的输出为
```
console.log(div.style.height) // 200px
```
完美获得元素高度。
可是。问题又来了，如果我们每次都要写成内联样式，也太费事了吧。那么，该怎么办？

### getComputedStyle

getComputedStyle方法获取的是最终应用在元素上的所有CSS属性对象（即使没有CSS代码，也会把默认的祖宗八代都显示出来）；这和style属性只能获取内联样式的行为形成了鲜明的对比。除此之外，getComputedStyle是只读的，但是style能文能武，可读可写，我们也可以利用它动态设置元素的高度。
我们只需输入这么一行代码。

```js
window.getComputedStyle(div);
```

getComputedStyle方法取得了元素的所有样式。
可是我们只想要高度呀。那就让getPropertyValue方法来帮忙getPropertyValue方法可以获取CSS样式申明对象上的属性值。
```js
console.log(window.getComputedStyle(div).getPropertyValue('height')); // 200px
```

### Element.getBoundingClientRect()
https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect

转载：https://www.jianshu.com/p/58c12245c2cc

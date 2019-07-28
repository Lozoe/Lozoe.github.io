---
title: es6之Proxy
date: 2019-07-27 10:15:06
tags:
  - es6
---

Vue3.0开始使用Proxy取代defineProperty，关于Proxy，今天小编也来研究一番。

Proxy: 代理。在js中，就是对已有对象做些处理，得到一个新的对象，这个新对象的属性读取与设置都可以加入我们自己的处理逻辑。
<!-- more -->
> Proxy: 代理。在js中，就是对已有对象做些处理，得到一个新的对象，这个新对象的属性读取与设置都可以加入我们自己的处理逻辑。

```js
const proxyObj = new Proxy(origin, {
  get: (target, prop) => {
    return prop in target ? target[prop] : '不存在该属性'
  },
  set: (target, prop, value) => {
    target[prop] = 'xxx';
  }
})

```

先看个🌰，来个整体认识：

```js
const origin = {
  name: 'Lz_Tiramisu',
  age: 25
}

const proxyObj = new Proxy(origin, {
  get: (target, prop) => {
    return prop in target ? target[prop] : '不存在该属性'
  },
  set: (target, prop, value) => {
    target[prop] = 'xxx';
  }
})

console.log(proxyObj.name); // Lz_Tiramisu
console.log(proxyObj.age); // 25
proxyObj.name = 'lucy';
console.log(proxyObj.name); // xxx
```

![proxyResult](proxy.png)

看上去跟defineProperty无区别，不过：

![proxyResult](proxy1.png)

对origin追加属性的操作会同步至proxyObj，这特性很nice。
此外我们可以留意：

- 对原始对象的属性修改会同步至代理对象上；
- 对代理对象的属性修改也会同步至原始对象上。

## 语法

```js
const proxy = new Proxy(origin, controller);
```
controller可以是空对象{},表示对 proxy 的属性操作都会同步至origin上。

## 数据双向绑定

html:

```html
<div id="bind"></div>
<input type="text" oninput="handleInput" id="input">
```

js:

```js
function handleInput(e) {
  var newVal = e.target.value;
  proxy.name = newVal;
}

function changeView() {
  console.log(proxy.name, origin.name);
  // 数据的修改是异步的，所以这里也需要异步读取才会是最新值
  setTimeout(() => {
    bind.innerText = proxy.name;
  });
}

var bind = document.getElementById('bind');
var input = document.getElementById('input');
input.oninput = handleInput;
var origin = {
  name: '初始值'
}

var proxy = new Proxy(origin, {
  set: function(target, prop, value) {
    if (prop === 'name') {
      changeView();
    }
    if(!value) {
      target[prop] = '默认值';
    } else {
      target[prop] = value;
    }
  }
})

changeView();
```

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="lozoe" data-slug-hash="mNOOKx" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="proxy">
  <span>See the Pen <a href="https://codepen.io/lozoe/pen/mNOOKx/">
  proxy</a> by Iona (<a href="https://codepen.io/lozoe">@lozoe</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

参考：

http://es6.ruanyifeng.com/?search=proxy&x=0&y=0#docs/proxy

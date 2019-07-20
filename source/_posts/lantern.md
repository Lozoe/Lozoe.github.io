---
title: 红黄绿灯
date: 2019-06-22 16:33:10
categories: [JavaScript]
tags:
  - 异步
---

### 问题

如何实现一个页面中红绿灯的效果：一个div，依次呈现红灯3s，绿灯2s，黄灯1s，不间断的交替。

### 多种方案实现

- css
- setTimeout
- promise
- generator
- async
<!-- more -->
#### css

结构：

```html
<div class="lantern css">css</div>
```

样式：

```css
.lantern {
  width: 100px;
  line-height: 100px;
  text-align: center;
  color: #333;
  display: inline-block;
  height: 100px;
  border-radius: 50%;
  border: 1px solid #ccc;
  &.css {
    animation: turnColor 6s steps(1) infinite running backwards; 
    &:hover {
      animation-play-state: paused;
    }
  }
}

@keyframes turnColor {
  // 0% {
  //   background-color: red;
  // }
  // 50% {
  //   background-color: green;
  // }
  // 66.66% {
  //   background-color: yellow;
  // }
  0% {
    background-color: red;
  }
  50% {
    background-color: green;
  }
  83.33% {
    background-color: yellow;
  }
}
```

关键： 阶跃函数

#### setTimeout

结构：

```html
<div class="lantern jst">js-timeout</div>
```

js：

```javascript
/* common start */
function common(interval, callback) {
    return new Promise(function(resolve, reject) {
        callback()
        setTimeout(function() {
            resolve()
        }, interval * 1000)
    })
}
function updateColor(dom, color) {
  dom.style.backgroundColor = color;
}
/* common end */

/* setTimeout start */
var jsDOMTimeout = document.querySelector('.jst');
function redt() {
  updateColor(jsDOMTimeout, 'red');
}
function greent() {
  updateColor(jsDOMTimeout, 'green');
}
function yellowt(){
  updateColor(jsDOMTimeout, 'yellow');
}
function loopt() {
    redt();
    setTimeout(function() {
        greent();
        setTimeout(function() {
            yellowt();
            setTimeout(function() {
                loopt();
            }, 1000);
        }, 2000);
    }, 3000);
}
loopt();
/* setTimeout end */
```

#### promise

结构：

```html
<div class="lantern jsp">js-promise</div>
```

js:

```javascript
/* promise */
var jsDOMPromise = document.querySelector('.jsp');
function redp() {
  updateColor(jsDOMPromise, 'red');
}
function greenp() {
  updateColor(jsDOMPromise, 'green');
}
function yellowp(){
  updateColor(jsDOMPromise, 'yellow');
}
var promise = new Promise(function(resolve, reject) {
    resolve();
});
function loop() {
    promise.then(function() {
        return common(3, redp);
    }).then(function() {
        return common(2, greenp);
    }).then(function() {
        return common(1, yellowp);
    }).then(function() {
        loop();
    })
}
loop();
/* promise end*/
```

#### generator

结构：

```html
<div class="lantern jsg">js-generator</div>
```

js:

```javascript
/* generator start */
var jsDOMGenerator = document.querySelector('.jsg');
function redg() {
  updateColor(jsDOMGenerator, 'red');
}
function greeng() {
  updateColor(jsDOMGenerator, 'green');
}
function yellowg() {
  updateColor(jsDOMGenerator, 'yellow');
}
function* light() {
    yield common(3, redg);
    yield common(2, greeng);
    yield common(1, yellowg);
}
function loopg(iterator, gen) {
    var result = iterator.next();
    if(result.done) {
        loopg(gen(), gen);
    } else {
        result.value.then(function() {
            loopg(iterator, gen);
        });
    }
}
loopg(light(), light);
/* generator end*/
```

#### async await

结构：

```html
<div class="lantern jsa">js-async</div>
```

```javascript
/* async await start*/
function keep(interval, dom, color) {
  return new Promise(function(resolve, reject) {
    updateColor(dom, color);
    setTimeout(function() {
      resolve()
    }, interval * 1000)
  })
}
var jsDOMAsync = document.querySelector('.jsa');
// function reda() {
//   updateColor(jsDOMAsync, 'red');
// }
// function greena() {
//   updateColor(jsDOMAsync, 'green');
// }
// function yellowa(){
//   updateColor(jsDOMAsync, 'yellow');
// }
(async function() {
  while(true) {
    // await common(3, reda);
    // await common(2, greena);
    // await common(1, yellowa);
    await keep(3, jsDOMAsync, 'red');
    await keep(2, jsDOMAsync, 'green');
    await keep(1, jsDOMAsync, 'yellow');
  }
})();
/* async await end*/
```

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="lozoe" data-slug-hash="xoJmZN" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="lantern">
  <span>See the Pen <a href="https://codepen.io/lozoe/pen/xoJmZN/">
  lantern</a> by Iona (<a href="https://codepen.io/lozoe">@lozoe</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### 思考

尽可能从API设计角度实现一个红绿灯效果，红灯3s, 黄灯2s，绿灯1s；

所谓的api设计角度，简单的理解就是通过.来调用实现。因此在这里我们初步设计实现的方式为：

```
ele(标签名).turn()
```

在实现点方法实现之前我们可以先来看下，最基本的效果代码，即只要写一个函数即可：

```html
<div id="lantern" class="lantern"></div>
```

```css
#lantern {
    height: 100px;
    width: 100px;
    border: 1px solid #bdbdbd;
}

.red {  
    background-color: red;
}

.yellow {
    background-color: yellow;
}

.green {
    background-color: green;
}
```

```javascript
let lantern = document.querySelector('.lantern');

function updateBgColor(colors, clsString, curColor) {
  let res = clsString
  for(let color of colors) {
    res = res.replace(color, '')
  }
  res += ` ${curColor}`
  return res.replace(/\s{2,}/g, ' ')
}

function LatternTurn(id) {
  this.el = document.getElementById(id);
}

LatternTurn.prototype.turn = function() {
  var lantern = this.el;

  (function turnColor() {
    var that = this;
    lantern.className = updateBgColor(['red', 'yellow', 'green'], lantern.className, 'red')
    setTimeout(function() {
      lantern.className = updateBgColor(['red', 'yellow', 'green'], lantern.className, 'yellow')
      setTimeout(function() {
        lantern.className = updateBgColor(['red', 'yellow', 'green'], lantern.className, 'green')
        setTimeout(turnColor, 1000);
      }, 2000)
    }, 3000)
  })()
}

function ele(id) {
  var latternTurn = new LatternTurn(id);
  return latternTurn;
}

ele('lantern').turn();
```

<p class="codepen" data-height="265" data-theme-id="0" data-default-tab="js,result" data-user="lozoe" data-slug-hash="EqYWgd" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="lantern-api">
  <span>See the Pen <a href="https://codepen.io/lozoe/pen/EqYWgd/">
  lantern-api</a> by Iona (<a href="https://codepen.io/lozoe">@lozoe</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

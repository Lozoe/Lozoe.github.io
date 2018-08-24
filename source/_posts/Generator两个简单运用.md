---
title: Generator两个简单运用
date: 2018-08-24 16:43:37
tags:
  - 长轮询
---

偶尔在论坛中发现了两个非常优秀的代码实践，特此学习。
```javascript
{
  let draw = function count(count) {
    //抽奖逻辑 TODO
    console.log(`剩余${count}次机会`)
  }
  let residue = function* (count) {
    while(count > 0) {
      count--;
      yield draw(count);
    }
  }
  let star = residue(5); // 抽奖机会数从后端返回
  let drawHandler = function() {
      star.next();
  }; // 添加到点击抽奖按钮的逻辑
  /*
  let btn = document.createElement('button');
  btn.id = 'start';
  btn.textContent = '抽奖';
  document.body.appendChild(btn);
  document.getElementById('start').addEventListener('click', drawHandler, false);
  */
}
```
<!-- more -->
```javascript
{
  // 浏览器定期向服务器取状态，http是无状态链接，可以通过长轮询和websocket
  /**
   * 长轮询
   */
  let ajax = function* () {
    yield new Promise(function(resolve, reject) {
      setTimeout(function() {
        resolve({code: 0});
      }, 200); // setTimeout内部逻辑为请求服务端结束的回调
    })
  }

  let pull = function() {
    let generator = ajax();
    let step = generator.next();
    step.value.then(function(d) {
      if(d.code !== 0) {
        setTimeout(function() {
          console.info('wait');
          pull();
        }, 1000)
      } else {
          console.info(d);
      }
    });
  }
  pull();
}
```


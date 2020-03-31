---
title: 实现cacheRequest，相同资源ajax只发一次请求
date: 2020-03-30 16:18:17
categories: [JavaScript]
---

请实现一个cacheRequest方法，保证当前ajax请求相同资源时，真实网络层中，实际只发出一次请求（假设已经存在request方法用于封装ajax请求）

一开始看到第一感觉是，设个Http Headercache-control，把expire调大，不就自然会找浏览器缓存拿了。但是看到后面说提供自带的request方法和只发起一次ajax。那估计是想让笔者在业务层自己用代码解决这个cache问题。

<!-- more -->

接下来，我们抛开实际场景价值，思考下怎样实现。

一般我们很简单的就可以得出以下思路：

- 利用闭包或者模块化的设计，引用一个Map，存储对应的缓存数据。
- 每次请求检查缓存，有则返回缓存数据，无则发起请求。
- 请求成功后，再保存缓存数据并返回，请求失败则不缓存。

```js
// 构建Map，用作缓存数据
let cache = new Map();
// 这里简单的把url作为cacheKey
const cacheRquest = (url) => {
    if (cache.has(url)) {
        return Promise.resolve(cache.get(url))
    } else {
        // 无缓存，发起真实请求，成功后写入缓存
        return request(url).then(res => {
            cache.set(url, res);
            return res;
        }).catch(err => Promise.reject(err));
    }
};
```

写到这里，你以为这篇文章就这么容易的结束了吗～～

当然不是，我觉得还有一个小坑：

有这么一个小概率边缘情况，并发两个或多个相同资源请求时，第一个请求处于pending的情况下，实际上后面的请求依然会发起，不完全符合题意。

因此，我们再重新设计出以下逻辑：

1. 当第一个请求处于pending时，设一个状态值锁住，后面并发的cacheRequest识别到pending时，则不发起请求，封装成异步操作，并塞进队列。
2. 当请求响应后，取出队列的异步resolve，把响应数据广播到每一个异步操作里。
3. 当发生请求错误时，同理：广播错误信息到每一个异步操作。
4. 之后的异步cacheRequest则正常的取出success的响应数据。

至此，并发的请求只要第一个返回成功了，后边都会马上响应，且真实只发起一个ajax请求。

首先，定义好我们缓存数据的schema，称为cacheInfo以此存入我们的Map里

```js
{
  status: 'PENDING', // ['PENDING', 'SUCCESS'， 'FAIL']
  response: {},      // 响应数据
  resolves: [],      // 成功的异步队列
  rejects: []        // 失败的异步队列
}
```

函数的主体我们梳理下主干的逻辑：

- 额外的加多一个option，参数可传入自定义的cacheKey
- 真实请求的handleRequest逻辑单独进行封装，因为不止一处用到，我们下面再单独实现

```js
const cacheMap = new Map();

const cacheRequest = function (target, option = {}) {
    const cacheKey = option.cacheKey || target;

    const cacheInfo = cacheMap.get(cacheKey);
    // 无缓存时，发起真实请求并返回
    if (!cacheInfo) {
        return handleRequest(target, cacheKey);
    }

    const status = cacheInfo.status;
    // 已缓存成功数据，则返回
    if (status === 'SUCCESS') {
        return Promise.resolve(cacheInfo.response);
    }
    // 缓存正在PENDING时，封装单独异步操作，加入队列
    if (status === 'PENDING') {
        return new Promise((resolve, reject) => {
            cacheInfo.resolves.push(resolve)
            cacheInfo.rejects.push(reject)
        })
    }
    // 缓存的请求失败时，重新发起真实请求
    return handleRequest(target, cacheKey);
}
```

接下来，就是发起真实请求的handleRequest，我们把改写status的操作和写入cacheInfo的操作一并封装在里面。当中抽离出两个公共函数：写入缓存的setCache和用于广播异步操作的notify。

首先是setCache，逻辑非常简单，浅合并原来的cacheInfo并写入

```js
// ... cacheMap = new Map()

const setCache = (cacheKey, info) => {
  cacheMap.set(cacheKey, {
    ...(cacheMap.get(cacheKey) || {}),
    ...info
  })
}
```

接下来是handleRequest：改写状态锁，发起真实请求，进行响应成功和失败后的广播操作

```js
const handleRequest = (url, cacheKey) => {
    setCache(cacheKey, {
        status: 'PENDING',
        resolves: [],
        rejects: []
    });

    const ret = request(url);

    return ret.then(res => {
        // 返回成功，刷新缓存，广播并发队列
        setCache(cacheKey, {
            status: 'SUCCESS',
            response: res
        });
        notify(cacheKey, res);
        return Promise.resolve(res);
    }).catch(err => {
        // 返回失败，刷新缓存，广播错误信息
        setCache(cacheKey, { status: 'FAIL' });
        notify(cacheKey, err);
        return Promise.reject(err);
    })
}
```

最后是广播函数notify的实现，取出队列，逐个广播然后清空

```js
// ... cacheMap = new Map()

const notify = (cacheKey, value) => {
    const info = cacheMap.get(cacheKey);

    let queue = [];

    if (info.status === 'SUCCESS') {
        queue = info.resolves
    } else if (info.status === 'FAIL') {
        queue = info.rejects
    }

    while(queue.length) {
        const cb = queue.shift();
        cb(value);
    }

    setCache(cacheKey, { resolves: [], rejects: [] });
}
```

后续优化

1. 缓存：本地全局变量缓存，也可以放在storage里，或者是用闭包，用时间戳或者md5建立唯一索引。  
2. 请求节流： 外层套个节流函数

节流函数：

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

需要在handleRequest时节流

参考：

https://juejin.im/post/5e745f85f265da57340262c3

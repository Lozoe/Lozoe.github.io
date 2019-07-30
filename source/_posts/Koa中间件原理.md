---
title: Koa中间件原理
date: 2019-05-29 19:20:56
tags:
---

对于一个node web服务框架来说，它的核心功能，就是在处理HTTP从流入到流出应用这一过程。而中间件，正是该过程的主要参与者，重要性不言而喻。

Koa的中间件设计机制可谓风骚无比，本篇笔记，就来彻底搞清楚中间件到底怎么玩起来的
<!-- more -->

## Koa中间件的使用方式

中间件的运行方式遵循洋葱模型。Koa中，要应用一个中间件，我们使用 app.use()，传参是一个函数：

```js
app.use(async function(ctx, next) {
  await next();
  if (ctx.body || !ctx.idempotent) return;
  ctx.redirect('/404.html');
});
```

## use函数搞了什么事情

```js
use(fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    fn = convert(fn);
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
}
```

不难看出，use就做了一件事：把fn push到Koa的middleware数组中。

## 如何建立起了洋葱模型

依靠compose，Koa的compose来自于koa-compose项目，[源码](https://github.com/koajs/compose/blob/master/index.js)也比较简单：

```js
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

接下来，我们就分析一下compose是如何做到中间件重组的

## compose详解

我们需明白一点，compose的作用：让注册的中间件按洋葱模型执行。去掉这件事情的细枝末节，问题可以抽象为：
有一个middleware数组，每个元素都为async函数，如下：

```js
var middleware = [
  async function(ctx, next) {
    console.log(11);
    await next();
    console.log(12);
  },
  async function(ctx, next) {
    console.log(21);
    await next();
    console.log(22);
  },
  async function(ctx, next) {
    console.log(31);
    await next();
    console.log(32);
  },
  async function(ctx, next) {
    console.log(4);
  },
];
```

如何处理该函数，能够让上述middleware函数数组的最终输出为

```js
11
21
31
4
32
22
12
```

接下来，我们通过四个阶段的代码逐渐解决该问题：
设定我们解决该问题的函数是

```js
function compose(middleware) {
  return function () {
    // 真正函数体
  }
}
```

1. 无脑式执行，依次执行middleware中的函数，并在执行时，将第二个参数写为空函数

```js
for (var fn of middleware) {
    fn('ctx', () => {});
}
```

结果当然感人：

```js
11
21
31
4
12
22
32
```

从11到4是ok的，但是后半部分是按照middleware[0]到middleware[3]执行的，这不符合洋葱模型的执行顺序。

2. 第一段代码的写法问题出在middleware执行时的第二个参数设置为空函数导致的，第二个参数next在middleware函数中的作用是await next();。由于await必须等待其后的Promise状态确定后才往后执行这一特性，所以，middleware[0]执行时其next应该设置为下一个元素即middleware[1]，这样middleware[0]中next之前的代码执行后，会等待middleware[1]的代码执行完毕后再接着执行next后面的代码。

```js
var i = 0;
var fn = middleware[i];
fn('ctx', () => { middleware[i + 1]('ctx', () => { })});
```

执行结果：

```js
11
21
22
12
```

思路没问题，但是middleware需要都执行啊，再改造一把

3. 代码如下

```js
var i = 0;
var fn = middleware[i];
fn('ctx', middleware[i+1].bind(null, 'ctx', middleware[i+2].bind(null,'ctx', middleware[i+3].bind(null,'ctx', () => {}))));
```

> 为节省代码，用fn.bind(null, arg)取代了() => { fn(arg) }

执行结果很理想：

```js
11
21
31
4
32
22
12
```

到第三段代码，已经显而易见了，我们需要一个递归，来依次处理所有的middleware，而递归的结束条件，是处理到middleware的最后一个元素，代码如下

```js
deal(0);

function deal(i) {
  var fn = middleware[i];
  if (!fn) {
    return () => {};
  }
  return fn('ctx', () => { deal(i + 1) })

}
```

4. 最终compose函数

```js
function compose(middleware) {
  return function () {
    deal(0);
    function deal(i) {
      var fn = middleware[i];
      if (!fn) {
        return () => {};
      }
      return fn('ctx', () => { deal(i + 1) })
    }
  }
}
```

和koa-compose项目的源码有所差别，接下来的代码片段慢慢抹平差异，阅读至此koa-compose的原理已经阐述完毕

6. 改变第五段代码，增加异常判断、修改函数名称、接受把普通函数包装为pormise函数（其实await会自动包装其后的函数为promise，不包也ok），得到：

```js
function compose(middleware) {
  if (!Array.isArray(middleware))
    throw new TypeError('Middleware stack must be an array!')
  for (var fn of middleware) {
    if (typeof fn !== 'function')
      throw new TypeError('Middleware must be composed of functions!')
  }
  return function () {
    // 为了最后的调用者仍可以使用then，所以使用了return
    return dispatch(0);
    function dispatch(i) {
      var fn = middleware[i];
      if (!fn) {
        return Promise.resolve();
      }
      try {
        return Promise.resolve(fn('ctx', () => { dispatch(i + 1) }));
      } catch (err) {
        return Promise.reject(err);
      }
    }
  }
}
```

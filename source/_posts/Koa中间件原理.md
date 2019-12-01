---
title: Koa中间件原理
date: 2019-05-29 19:20:56
categories: [Framework, Node]
---

Koa是用Node.js实现的服务框架。在Koa中middleware的使用场景是在请求到来和发送响应之间（也就是在处理HTTP从流入到流出应用这一过程），对代码按照功能进行插件化管理，其重要性不言而喻。

熟悉redux的同学都知道，redux的中间件是采用函数式编程的compose方式对middleware进行组合。而Koa的中间件其实和redux大同小异。

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

## Koa中间件的实现原理

要实现一套middleware,主要分为三步：

1. 搜集middleware
2. 组合middleware
3. 调用执行

```js
// 随便写一个中间件
export const debugMiddleware = () => next => action => {
    if(DEBUG || IS_BETA) {
        console.log(JSON.stringify(action, null, 4));
    }
    return next(action);
};
```

在Redux中，前两步是有applymiddleware实现的，调用执行则是首先由createStore的第三个参数传入`applyMiddleware(debugMiddleware)`作为enhancer，再传入createStore进行createStore的加强，在store.dispatch调用时自动触发各个中间件，最终触发dispatch函数。

而Koa中间件是怎么做的呢？接下来详细分析。

## 搜集middleware

app.use函数搞了什么事情,其实他所做的工作正是手机middleware,源码如下：

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

## 组合middleware

中间件如何建立起了洋葱模型，其实就是依靠compose，Koa的compose来自于koa-compose项目，[源码](https://github.com/koajs/compose/blob/master/index.js)也比较简单：

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

### compose详解

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

## 调用执行

接下来我们来讨论一下中间件怎么执行，当一个请求到来的时候，是怎样经过中间件，怎么跑起来的

首先我们只要知道下面这段callback函数就是请求到来的时候执行的回调

```js
/**
* Return a request handler callback
* for node's native http server.
*
* @return {Function}
* @api public
*/
callback() {
    const fn = compose(this.middleware);

    if (!this.listeners('error').length) this.on('error', this.onerror);

    const handleRequest = (req, res) => {
        // res.statusCode = 404;
        const ctx = this.createContext(req, res);
        const onerror = err => ctx.onerror(err);
        const handleResponse = () => respond(ctx);
        onFinished(res, onerror);
        return fn(ctx).then(handleResponse).catch(onerror);
    };

    return handleRequest;
}
```

这段代码可以分成两个部分

- 请求前的中间件初始化处理部分
- 请求到来时的中间件运行部分

### 请求前的中间件初始化处理部分

关于请求前的中间件初始化处理部分也就是`const fn = compose(this.middleware);`在前面的部分已经做过详述；本质是就是对中间件集合建立起一个洋葱模型。

### 请求到来时的中间件运行部分

接下来开始讨论请求到来时，初始化好的中间件是怎么跑的
其实执行的核心代码是

```js
const ctx = this.createContext(req, res);
fn(ctx).then(handleResponse).catch(onerror);
```

到此呢，Koa中间件的面纱就完全揭开了。

ps:
其实呢，在koa更早的v1版本中间件组合是采用的generator代码如下：

```js
function compose(middleware){
    return function *(next){
        // 第一次得到next是由于*noop生成的generator对象
        if (!next) next = noop();

        var i = middleware.length;
        // 从后往前开始执行middleware中的generator函数
        while (i--) {
            // 把后一个中间件得到的generator对象传给前一个作为第一个参数存在
            next = middleware[i].call(this, next);
        }
        return yield *next;
    }
}

function *noop(){}
```

1、执行所有的generator函数，得到generator
2、将generator作为下一次generator函数执行的参数next
3、返回一个入口generator

参考：

https://github.com/qianlongo/resume-native/issues/1
http://www.shadowvip.com/topic/5b9a22996a5aa70cdafd141d

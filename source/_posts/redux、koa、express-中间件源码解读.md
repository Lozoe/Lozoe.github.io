---
title: redux、koa、express 中间件源码解读
date: 2020-03-06 10:42:58
categories: [Framework]
tags:
---

express 和 koa 的中间件是用于处理 http 请求和响应的，但是二者的设计思路确不尽相同。大部分人了解的express和koa的中间件差异在于：

- express采用尾递归方式，中间件一个接一个的顺序执行, 习惯于将response响应写在最后一个中间件中；
- 而koa的中间件支持 generator, 执行顺序是“洋葱圈”模型。

所谓的“洋葱圈”模型:

![onion-model](onion-model.png)

不过实际上，express 的中间件也可以形成“洋葱圈”模型，在 next 调用后写的代码同样会执行到，不过express中一般不会这么做，因为 express的response一般在最后一个中间件，那么其它中间件 next() 后的代码已经影响不到最终响应结果了;

## express

### 入口

首先看一下 express 的实现：

```js
// express.js

var proto = require('./application');
var mixin = require('merge-descriptors');

exports = module.exports = createApplication;

function createApplication() {
    // app 同时是一个方法，作为http.createServer的处理函数
    var app = function(req, res, next) {
        app.handle(req, res, next);
    }

    mixin(app, proto, false);
    return app;
}
```

这里其实很简单，就是一个 createApplication 方法用于创建 express 实例，要注意返回值 app 既是实例对象，上面挂载了很多方法，同时它本身也是一个方法，作为 http.createServer的处理函数, 具体代码在 application.js 中：

```js
// application.js
var http = require('http');
var flatten = require('array-flatten');
var app = exports = module.exports = {}

app.listen = function listen() {
  var server = http.createServer(this)
  return server.listen.apply(server, arguments)
}
```

这里 app.listen 调用 nodejs 的http.createServer 创建web服务，可以看到这里 var server = http.createServer(this) 其中 this 即 app 本身, 然后真正的处理程序即 app.handle;

### 中间件处理 - express

express 本质上就是一个中间件管理器，当进入到 app.handle 的时候就是对中间件进行执行的时候，所以，最关键的两个函数就是：

- `app.handle` 尾递归调用中间件处理 req 和 res
- `app.use` 添加中间件

全局维护一个stack数组用来存储所有中间件，app.use 的实现就很简单了，可以就是一行代码

```js
// app.use
app.use = function(fn) {
    this.stack.push(fn);
}
```

express 的真正实现当然不会这么简单，它内置实现了路由功能，其中有 router, route, layer 三个关键的类，有了 router 就要对 path 进行分流，stack 中保存的是 layer实例，app.use 方法实际调用的是 router 实例的 use 方法, 有兴趣的可以自行去阅读。

app.handle 即对 stack 数组进行处理

```js
app.handle = function(req, res, callback) {
    var stack = this.stack;
    var idx = 0;
    function next(err) {
        if (idx >= stack.length) {
            callback('err');
            return;
        }
        var mid;
        while(idx < stack.length) {
            mid = stack[idx++];
            mid(req, res, next);
        }
    }
    next();
}
```

这里就是所谓的"尾递归调用"，next 方法不断的取出stack中的“中间件”函数进行调用，同时把next 本身传递给“中间件”作为第三个参数，每个中间件约定的固定形式为 (req, res, next) => {}, 这样每个“中间件“函数中只要调用 next 方法即可传递调用下一个中间件。

之所以说是”尾递归“是因为递归函数的最后一条语句是调用函数本身，所以每一个中间件的最后一条语句需要是next()才能形成尾递归，否则就是普通递归，尾递归相对于普通递归的好处在于节省内存空间，不会形成深度嵌套的函数调用栈。

至此，express 的中间件实现就完成了。

## koa

不得不说，相比较 express 而言，koa 的整体设计和代码实现显得更高级，更精炼；代码基于ES6 实现，支持generator(async await), 没有内置的路由实现和任何内置中间件，context 的设计也很是巧妙。

### 入口及目录

一共只有4个文件：

- application.js 入口文件，koa应用实例的类
- context.js ctx 实例，代理了很多request和response的属性和方法，作为全局对象传递
- request.js koa 对原生 req 对象的封装
- response.js koa 对原生 res 对象的封装

request.js 和 response.js 没什么可说的，任何 web 框架都会提供req和res 的封装来简化处理。所以主要看一下 context.js 和 application.js的实现

```js
// context.js
/**
 * Response delegation.
 */

delegate(proto, 'res')
  .method('setHeader')

/**
 * Request delegation.
 */

delegate(proto, 'req')
  .access('url')
  .setter('href')
  .getter('ip');
```

context 就是这类代码，主要功能就是在做代理，使用了 delegate 库。

简单说一下这里代理的含义，比如delegate(proto, 'res').method('setHeader') 这条语句的作用就是：当调用proto.setHeader时，会调用proto.res.setHeader 即，将proto的 setHeader方法代理到proto的res属性上，其它类似。

```js
// application.js 中部分代码

constructor() {
    super()
    this.middleware = []
    this.context = Object.create(context)
}

use(fn) {
    this.middleware.push(fn)
}

listen(...args) {
    debug('listen')
    const server = http.createServer(this.callback());
    return server.listen(...args);
}

callback() {
    // 这里即中间件处理代码
    const fn = compose(this.middleware);

    const handleRequest = (req, res) => {
        // ctx 是koa的精髓之一, req, res上的很多方法代理到了ctx上, 基于 ctx 很多问题处理更加方便
        const ctx = this.createContext(req, res);
        return this.handleRequest(ctx, fn);
    };

    return handleRequest;
}

handleRequest(ctx, fnMiddleware) {
    ctx.statusCode = 404;
    const onerror = err => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
}  
```

同样的在listen方法中创建 web 服务, 没有使用 express 那么绕的方式，`const server = http.createServer(this.callback())`; 用`this.callback()`生成 web 服务的处理程序

### 中间件处理 - koa

构造函数 constructor 中维护全局中间件数组 this.middleware和全局的this.context 实例（源码中还有request，response对象和一些其他辅助属性）。和 express 不同，因为没有router的实现，所有this.middleware 中就是普通的”中间件“函数而非复杂的 layer 实例，
this.handleRequest(ctx, fn); 中 ctx 为第一个参数，fn = compose(this.middleware) 作为第二个参数, handleRequest 会调用 fnMiddleware(ctx).then(handleResponse).catch(onerror); 所以中间处理的关键在compose方法, 它是一个独立的包koa-compose, 把它拿了出来看一下里面的内容：

```js
// compose.js
'use strict'
module.exports = compose
function compose (middleware) {
  return function (context, next) {
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```

和express中的next 是不是很像，只不过他是promise形式的，因为要支持异步，所以理解起来就稍微麻烦点：每个中间件是一个`async (ctx, next) => {}`, 执行后返回的是一个promise, 第二个参数 next的值为 dispatch.bind(null, i + 1) , 用于传递”中间件“的执行，一个个中间件向里执行，直到最后一个中间件执行完，resolve 掉，它前一个”中间件“接着执行 await next() 后的代码，然后 resolve 掉，在不断向前直到第一个”中间件“ resolve掉，最终使得最外层的promise resolve掉。

这里和express很不同的一点就是koa的响应的处理并不在"中间件"中，而是在中间件执行完返回的promise resolve后：

```js
return fnMiddleware(ctx).then(handleResponse).catch(onerror);
```

通过 handleResponse 最后对响应做处理，中间件会设置ctx.body, handleResponse也会主要处理 ctx.body ，所以 koa 的”洋葱圈“模型才会成立，await next()后的代码也会影响到最后的响应。

至此，koa的中间件实现就完成了。

## redux

### 入口及目录 - redux

本文还是主要看一下它的中间件实现, 先简单说一下 redux 的核心处理逻辑, createStore 是其入口程序，工厂方法，返回一个 store 实例，store实例的最关键的方法就是 dispatch , 而 dispatch 要做的就是一件事：

```js
currentState = currentReducer(currentState, action);
```

即调用reducer, 传入当前state和action返回新的state。

所以要模拟基本的 redux 执行只要实现 createStore , dispatch 方法即可。其它的内容如 bindActionCreators, combineReducers 以及 subscribe 监听都是辅助使用的功能，可以暂时不关注。

### 中间件处理 - redux

然后就到了核心的”中间件" 实现部分即 applyMiddleware.js：

```js
// applyMiddleware.js

import compose from './compose'

export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```

redux 中间件提供的扩展是在 action 发起之后，到达 reducer 之前，它的实现思路就和express 、 koa 有些不同了，它没有通过封装 store.dispatch, 在它前面添加 中间件处理程序，而是通过递归覆写 dispatch ，不断的传递上一个覆写的 dispatch 来实现。

每一个 redux 中间件的形式为 `store => next => action => { xxx }`

这里主要有两层函数嵌套：

最外层函数接收参数store, 对应于 applyMiddleware.js 中的处理代码是 `const chain = middlewares.map(middleware => middleware(middlewareAPI))`, middlewareAPI 即为传入的store 。这一层是为了把 store 的 api 传递给中间件使用，主要就是两个api:

getState, 直接传递store.getState.
dispatch: (...args) => dispatch(...args)，这里的实现就很巧妙了，并不是store.dispatch, 而是一个外部的变量dispatch, 这个变量最终指向的是覆写后的dispatch, 这样做的原因在于，对于 redux-thunk 这样的异步中间件，内部调用store.dispatch 的时候仍然后走一遍所有“中间件”。

返回的chain就是第二层的数组，数组的每个元素都是这样一个函数next => action => { xxx }, 这个函数可以理解为 接受一个dispatch返回一个dispatch, 接受的dispatch 是后一个中间件返回的dispatch.

还有一个关键函数即 compose, 主要作用是 `compose(f, g, h)` 返回 `() => f(g(h(..args)))`

现在在来理解 `dispatch = compose(...chain)(store.dispatch)` 就相对容易了，原生的 store.dispatch 传入最后一个“中间件”，返回一个新的 dispatch , 再向外传递到前一个中间件，直至返回最终的 dispatch, 当覆写后的dispatch 调用时，每个“中间件“的执行又是从外向内的”洋葱圈“模型。

至此，redux中间件就完成了。

参考：

尾调用：http://www.ruanyifeng.com/blog/2015/04/tail-call.html
redux: https://redux.js.org/advanced/middleware/

---
title: express之router
date: 2020-03-29 16:05:55
tags:
---

## 前言

从Express 4.0开始，路由器功能成了一个单独的组件Express.Router。它好像小型的express应用程序一样，有自己的use、get、param和route方法。

## 基本用法

首先，Express.Router是一个构造函数，调用后返回一个路由器实例。然后，使用该实例的HTTP动词方法，为不同的访问路径，指定回调函数；最后，挂载到某个路径。

```js
var router = express.Router();

router.get('/', function(req, res) {
    res.send('首页');
});

router.get('/about', function(req, res) {
    res.send('关于');
});

app.use('/', router);
```

上面代码先定义了两个访问路径，然后将它们挂载到根目录。如果最后一行改为app.use(‘/app’, router)，则相当于为/app和/app/about这两个路径，指定了回调函数。

这种路由器可以自由挂载的做法，为程序带来了更大的灵活性，既可以定义多个路由器实例，也可以为将同一个路由器实例挂载到多个路径。

## router.route方法

router实例对象的route方法，可以接受访问路径作为参数。

```js
var router = express.Router();

router.route('/api')
    .post(function(req, res) {
        // ...
    })
    .get(function(req, res) {
        Bear.find(function(err, bears) {
            if (err) res.send(err);
            res.json(bears);
        });
    });

app.use('/', router);
```

## router中间件

use方法为router对象指定中间件，即在数据正式发给用户之前，对数据进行处理。下面就是一个中间件的例子。

```js
router.use(function(req, res, next) {
    console.log(req.method, req.url);
    next();
});
```

上面代码中，回调函数的next参数，表示接受其他中间件的调用。函数体中的next()，表示将数据传递给下一个中间件。

注意，中间件的放置顺序很重要，等同于执行顺序。而且，中间件必须放在HTTP动词方法之前，否则不会执行。

## 对路径参数的处理

router对象的param方法用于路径参数的处理，可以

```js
router.param('name', function(req, res, next, name) {
    // 对name进行验证或其他处理……
    console.log(name);
    req.name = name;
    next();
});

router.get('/hello/:name', function(req, res) {
    res.send('hello ' + req.name + '!');
});
```

上面代码中，get方法为访问路径指定了name参数，param方法则是对name参数进行处理。注意，param方法必须放在HTTP动词方法之前。

## app.route

假定app是Express的实例对象，Express 4.0为该对象提供了一个route属性。app.route实际上是express.Router()的缩写形式，除了直接挂载到根路径。因此，对同一个路径指定get和post方法的回调函数，可以写成链式形式。

```js
app.route('/login')
    .get(function(req, res) {
        res.send('this is the login form');
    })
    .post(function(req, res) {
        console.log('processing');
        res.send('processing the login form!');
    });
```

上面代码的这种写法，显然非常简洁清晰。

参考：

https://javascript.ruanyifeng.com/nodejs/express.html#toc21

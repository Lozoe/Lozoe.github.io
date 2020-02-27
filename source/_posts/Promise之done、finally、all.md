---
title: Promise之done、finally、all
date: 2020-02-27 16:03:20
tags:
categories: [JavaScript]
---

## done()

Promise 对象的回调链，不管以then方法或catch方法结尾，要是最后一个方法抛出错误，都有可能无法捕捉到（因为 Promise内部的错误不会冒泡到全局）。因此，我们可以提供一个done方法，总是处于回调链的尾端，保证抛出任何可能出现的错误。

```js
asyncFunc()
    .then(f1)
    .catch(r1)
    .then(f2)
    .done();
```
<!-- more -->

实现也非常简单

```js
Promise.prototype.done = function(onFulfilled, onRejected) {
    this
        .then(onFulfilled, onRejected)
        .catch(function (reason) {
            // 抛出一个全局错误
            setTimeout(() => {
              throw reason;
            }, 0);
         });
}
```

从上面代码可见，done方法的使用，可以像then方法那样用，提供fulfilled和rejected状态的回调函数，也可以不提供任何参数。但不管怎样，done都会捕捉到任何可能出现的错误，并向全局抛出。

## finally()

finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。它与done方法的最大区别，它接受一个普通的回调函数作为参数，该函数不管怎样都必须执行。

下面是一个例子，服务器使用Promise处理请求，然后使用finally方法关掉服务器。

```js
server.listen(0)
    .then(function () {
        // run test
    })
    .finally(server.stop);
 ```

实现如下：

```js
Promise.prototype.finally = function (callback) {
    let P = this.constructor;
    return this.then(
        value => P.resolve(callback()).then(() => value),
        reason => P.resolve(callback()).then(() => {
            throw reason;
        });
    );
}
```

上面代码中，不管前面的 Promise是fulfilled还是rejected，都会执行回调函数callback。

这个是转换为ES5之后的。

```js
Promise.prototype.finally = function (callback) {
    var P = this.constructor;
    return this.then(function (value) {
        return P.resolve(callback()).then(function () {
            return value
        })
    }, function (reason) {
        return P.resolve(callback()).then(function () {
            throw reason
        })
    })
}
```

要这么写的原因是在于，finally其实并不一定是这个promise链的最后一环，相对而言，其实done才是。

因为finally可能之后还有then和catch等等，所以其必须要返回一个promise对象。

## all()

### Promise.all 功能

Promise.all(iterable)可以将多个Promise实例包装成一个新的Promise实例。此实例在iterable参数内所有的Promise都fulfilled或者参数中不包含Promise时，状态变成fulfilled,如果参数中Promise有一个失败rejected ，此实例回调失败，失败原因的是第一个失败Promise的返回结果。也就是说成功和失败的返回值是不同的，成功的时候返回的是一个结果数组，而失败的时候则返回最先被reject失败状态的值

```js
let p1 = new Promise((resolve, reject) => {
  resolve('成功了')
})

let p2 = new Promise((resolve, reject) => {
  resolve('success')
})

let p3 = Promse.reject('失败')

Promise.all([p1, p2]).then((result) => {
  console.log(result)               //['成功了', 'success']
}).catch((error) => {
  console.log(error)
})

Promise.all([p1,p3,p2]).then((result) => {
  console.log(result)
}).catch((error) => {
  console.log(error)      // 失败了，打出 '失败'
})
```

p的状态由 p1,p2,p3决定，分成以下；两种情况：

（1）只有p1、p2、p3的状态都变成 fulfilled，p的状态才会变成 fulfilled，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被 rejected，p的状态就变成 rejected，此时第一个被reject的实例的返回值，会传递给p的回调函数。

### Promise.all 的特点

Promse.all在处理多个异步处理时非常有用，比如说一个页面上需要等两个或多个ajax的数据回来以后才正常显示，在此之前只显示loading图标

Promise.all的返回值是一个Promise实例

> Promise.all 里的任务列表[asyncTask(1),asyncTask(2),asyncTask(3)],我们是按照顺序发起的。
> 但是根据结果来说，它们是异步的，互相之间并不阻塞，每个任务完成时机是不确定的，尽管如此，所有任务结束之
> 后，它们的结果仍然是按顺序地映射到resultList里,这样就能和Promise.all里的任务列表。这带来了一个绝大的好处：在前端开发请求数据的过程中，偶尔会遇到发送多个请求并根据请求顺序获取和使用数据的场景，使用Promise.all毫无疑问可以解决这个问题

### Promise.all 的源码实现

```js
Promise.all = arr => {
    let aResult = [];    // 用于存放每次执行后返回结果
    return new _Promise(function (resolve, reject) {
        let i = 0;
        next();    // 开始逐次执行数组中的函数(重要)
        function next() {
            arr[i].then(function (res) {
                aResult.push(res);    // 存储每次得到的结果
                i++;
                if (i == arr.length) {
                    // 如果函数数组中的函数都执行完，便resolve
                    resolve(aResult);
                } else {
                    next();
                }
            })
        }
    })
};
```

## race()

`Promise.race([p1, p2, p3])`里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态

我们简单看一下例子，返回结果为3，因为我们设置了定时器，第三个Promise执行的最快

```js
Promise.race([
    new Promise(function(resolve, reject) {
        setTimeout(() => resolve(1), 1000)
    }),
    new Promise(function(resolve, reject) {
        setTimeout(() => resolve(2), 100)
    }),
    new Promise(function(resolve, reject) {
        setTimeout(() => resolve(3), 10)
    })
]).then(value => {
    console.log(value) // 3
});
```

```js
let p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('success')
    },1000)
})

let p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        reject('failed')
    }, 500)
})

Promise.race([p1, p2]).then((result) => {
    console.log(result)
}).catch((error) => {
    console.log(error)  // 打开的是 'failed'
})
```

源码实现：

```js
static race(promiseAry) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promiseAry.length; i++) {
      promiseAry[i].then(resolve, reject)
    }
  })
}
Promise.race = arr => {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < arr.length; i++) {
            arr[i].then(resolve, reject);
        }
    });
};
```

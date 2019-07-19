---
title: js中的eventloop
date: 2019-07-18 07:09:24
tags:
  - 事件循环
---

## 背景

Event Loop可以说是js老生常谈的一个话题了。作为一枚前端开发，需要清楚的认知浏览器中与node中事件循环与执行机制的不同，不可混为一谈。

浏览器的[Event loop](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)是在HTML5中定义的规范，而node中则由[libuv](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)库实现。

以下是参考一系列文章后的总结。

## 与事件循环相关的概念

说到事件循环，务必要了解清楚一些名词，宏任务，微任务，单线程，同步，异步，都是什么鬼，今天小编就带着大家来聊聊我的认识。

### 线程与异步

> Javascript是单线程，所以JavaScript的任务必须一个接一个的执行而不能并行。对于耗时的任务，js采用异步的方式去执行，以达到不阻塞主线程（这特么的主线程又是啥时候冒出来的）的目的。
> 在这里，引出了同步和异步的概念。

#### 同步

如果一个语句在执行后，调用者必须等到该语句执行结束后，才能继续执行后续代码，则称为同步语句，最直观的就是window.alert();

#### 异步

如果一个语句在执行后，调用者并不能立即得到执行结果，需要在之后的某个时刻才能得到，那么这条语句就是异步的。所以当执行异步程序的时候，我们可以去干些别的事情，当异步执行结束后，会通知到我们，但是！！，程序有可能在做其他事情，所以即使异步结束了也需要在一旁等待，等程序空闲下来了才有时间去看看哪些异步执行完了，再去执行
（明白为何setTimeout并不是真正的延时n后执行了吧，算了我再贴一段代码来说明setTimeout的机制粑）

```js
setTimeout(() => {
    console.log(0);
}, 1999);
for (var i = 0; i < 80000000; i++) {
    var a = [1,2,3];
    var b = [].concat(a);
}
setTimeout(() => {
    console.log(1);
}, 10);
```

- 以上代码0和1谁先被打印？
- 两次打印的时间间隔是多久？

想搞清上面两个问题，我们需要对setTimeout的延迟本质有清晰的认知。延迟n ms意思是告诉js引擎，过n ms之后把这个回调放入宏任务队列中等待执行。从eventLoop的角度，我们也可以这么看，setTimeout是个异步语句，被放入task pool中，n ms后异步结束，告知js引擎把他的回调放入宏任务队列中。

据此，代码的执行情况可以如下解释：

![setTimeout](https://1shark.com/word-img-1256165110/timeout.png)

由此可知，0比1先被打印（如果for耗时很短，那肯定是1先打印了），时间间隔大概10ms。
嘿嘿嘿，然鹅时间间隔并非一定是10ms，这取决于for循环执行耗时。

### 执行栈与任务队列

栈：先进后出
队列：先进先出

#### 执行栈

当调用某个方法的时候，js会为此生成一个执行上下文，它里面保存着方法的私有作用域、上层作用域(作用域链)、方法的参数，以及这个作用域中定义的变量和this的指向。由于js是单线程的，这些方法就会按顺序被排列在一个单独的地方，这个地方就是所谓执行栈。

#### 任务队列

任务队列里面放的都是异步任务，任务队列一次仅执行一个任务。执行栈则是一个类似于函数调用栈的运行容器，当执行栈为空时，JS 引擎便检查任务队列，如果不为空，任务队列便将第一个任务压入执行栈中运行。

#### 其他概念

关于什么是宏任务，什么是微任务，我们在接下来的过程一步一步抛出。

## 浏览器中的事件循环

在异步代码完成后仍有可能要在一旁等待，因为此时程序可能在做其他的事情，等到程序空闲下来才有时间去看哪些异步已经完成了。所以 JavaScript 有一套机制去处理同步和异步操作，那就是事件循环 (Event Loop)。

简言之，

1. 同步任务进入执行栈；
2. 异步任务放入task pool中，异步任务有了运行结果，就将该函数移入任务队列（任务队列中的都是 已经完成的异步操作的，而不是注册一个异步任务就会被放在这个任务队列中（它会被放到 Task Pool 中））
3. 执行栈为空时，读取任务队列，将任务队列中的第一个任务压入执行栈中运行
主线程不断重复第三步，也就是只要主线程空了，就会去读取任务队列，该过程不断重复， 直到执行栈和任务队列都为空，这就是所谓的事件循环。

### 宏任务与微任务

在之前没有具体介绍这两个概念，是觉得在这里引出宏任务与微任务更合适。

异步任务分为宏任务(macrotask)与微任务 (microtask)。宏任务会进入一个队列，而微任务会进入到另一个不同的队列，且微任务要优于宏任务执行。这两类任务存放都是已经完成的异步操作的回调.
为何有宏任务和微任务之分，或许是因为每次执行完某个宏任务后，这个宏任务又带了个异步代码小尾巴，引擎认为这个小尾巴不应该进入到任务队列末尾重新排队等待执行，而应该紧接着本次宏任务执行，所以用宏任务和微任务来处理这种场景。

#### 宏任务

script、setTimeout、setInterval、I/O、事件、postMessage、 MessageChannel、setImmediate (Node.js) （settimeout0的优先级高于setImmediate）

#### 微任务

Promise.then process.nextTick MutaionObserver（且preocess.nextTick优先级大于promise.then，因为nextTick在eventloop当前阶段生效，即当前操作执行完，就执行nextTick）

宏任务，微任务，执行栈，task pool，事件循环 这些概念用图可表示为：

![event-loop](http://word-img-1256165110.cos.ap-beijing.myqcloud.com/excute-loop.png)

！！ 微任务创建的微任务仍然放入微任务队列中的最后，仍然早于宏任务执行

## Node.js 与 浏览器环境下事件循环的区别

Node.js 在升级到 11.x 后，Event Loop 运行原理发生了变化，一旦执行一个阶段里的一个宏任务(setTimeout,setInterval 和 setImmediate) 就立刻执行微任务队列，这点就跟浏览器端一致。之前的就不管了。

## 考一考

### 简单宏任务和微任务（即任务中只有同步语句）

```js
setTimeout(() => {
  console.log('A');
}, 0);
function func() {
  setTimeout(function() {
    console.log('B');
  }, 0);
  return new Promise(function(resolve) {
    console.log('C');
    resolve();
  });
}
func().then(function() {
  console.log('D');
});
console.log('E');
```

执行过程分析：

```js
第一个setTimeout放到宏任务队列，此时宏任务队列为[console.log(‘A’)]；
func()被执行，
第二个setTimeout被放入宏队列，此时宏任务队列为；[console.log(‘A’), console.log(‘B’)]；
new Promise被执行，里面同步语句console.log(‘C’)被执行，输出C；
第三步的Promise已处于resolve状态，他的then被放入微任务队列中，此时微任务队列为[console.log(‘D’)]；
同步语句console.log(‘E’)被执行，输出E；
查看并执行微任务队列，输出D；
查看宏任务队列，并依次执行，输出a,b
结果：c e d a b
```

### 复杂宏任务（即里面包含新的微任务等）

```js
console.log('1');
setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
}, 0)
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})
setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
}, 0)
```

执行过程分析：

1. 第一句console.log(‘1’)是同步语句，直接输出1；
2. setTimeout是异步语句，0ms后将其回调放入宏任务中，此时宏任务队列为[第一个setTimeout的回调，]；
3. nextTick是微任务，被放入微任务队列中，此时微任务队列为[console.log(‘6’)]；
4. 执行promise，console.log(‘7’)是同步语句，直接输出7，由于promise立即被resolve，所以.then被立即放入微任务队列中，此时微任务队列为[console.log(‘6’),console.log(‘8’)]；
5. 执行外部的第二个setTimeout，将其回调放入宏任务中，此时宏任务为[第一个setTimeout的回调, 第二个setTimeout的回调]；
6. 至此第一轮事件循环结束，此时：宏任务队列为第一个setTimeout的回调, 第二个setTimeout的回调]，微任务队列为[console.log(‘6’),console.log(‘8’)]；
7. 清理第一轮事件循环的微任务队列，依次输出6，8；
8. 执行第一个宏任务：第一个setTimeout的回调

```js
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
```

- console.log(‘2’)是同步语句，直接输出2；
- nextTick被放入微任务中[console.log(‘3’)];
- 直接输出4，并把.then放入微任务中[console.log(‘3’), console.log(‘5’)]
- 又一轮循环结束，清理微任务，依次输出3，5

9. 执行第二个宏任务：第二个setTimeout的回调

```js
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
```

- console.log(‘9’)是同步语句，直接输出9；
- nextTick被放入微任务中[console.log(‘10’)];
- 直接输出11，并把.then放入微任务中[console.log(‘10’), console.log(‘12’)]
- 又一轮循环结束，清理微任务，依次输出10，12
- 代码执行完毕

结果：1,7,6,8,2,4,3,5,9,11,10,12

### 复杂宏任务基础上理解任务队列加入时机

和上一题一样，只是把两个setTimeout的延迟依次修改为100和99

```js
console.log('1');
setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
}, 100)
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})
setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
}, 99)
```

直接说运行结果吧：
1,7,6,8,2,4,3,5,9,11,10,12或1,7,6,8,9,11,10,12,2,4,3,5

原因解释：两个setTimeout相差只有1ms，第一个setTimeout，js引擎知道了100ms后把他的回调放入宏任务中，然后就往下执行了，走到第二个setTimeout时，或许已经用时3ms了，此时js引擎知道了99ms后把第二个setTimeout的回调放入宏任务中，3+99 > 100，所以在放第二个的回调之前，第一个setTimeout的回调已经在宏任务中了。而如果js走到第二个setTimeout时用时连1ms都没到，那就意味着第二个setTimeout的回调先进入宏任务队列中。

### 有async/await的情况

async/await的本质是Promise的语法糖
做个转换：

```js
async function foo() {
  // await 前面的代码
  await bar();
  // await 后面的代码
}
async function bar() {
  // do something...
}
foo();
```

> 其中 await 前面的代码 是同步的，调用此函数时会直接执行；而 await bar(); 这句可以被转换成 Promise.resolve(bar())；await 后面的代码 则会被放到 Promise 的 then() 方法里。因此上面的代码可以被转换成如下形式，这样是不是就很清晰了？

```js
function foo() {
  // await 前面的代码
  Promise.resolve(bar()).then(() => {
    // await 后面的代码
  });
}
function bar() {
  // do something...
}
foo();
```

题目：

```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(function() {
  console.log('setTimeout');
}, 0);
async1();
new Promise(function(resolve) {
  console.log('promise1');
  resolve();
}).then(function() {
  console.log('promise2');
});
console.log('script end');
```

分析：
代码变异：

```js
function async1() {
  console.log('async1 start'); // 2

  Promise.resolve(async2()).then(() => {
    console.log('async1 end'); // 6
  });
}
function async2() {
  console.log('async2'); // 3
}
console.log('script start'); // 1
setTimeout(function() {
  console.log('settimeout'); // 8
}, 0);
async1();
new Promise(function(resolve) {
  console.log('promise1'); // 4
  resolve();
}).then(function() {
  console.log('promise2'); // 7
});
console.log('script end'); // 5
```

### then层级嵌套

```js
var p1 = new Promise((resolve) => {
    console.log('promise1');
    resolve();
}).then(() => {
    console.log('then11');
    new Promise((resolve) => {
        console.log('promise2');
        resolve();
    }).then(() => {
        console.log('then21');
    }).then(() => {
        console.log('then22');
    });
}).then(() => {
    console.log('then12');
});
var p3 = new Promise((resolve) => {
    console.log('promise3');
    resolve();
}).then(() => {
    console.log('then31');
});
```

执行过程分析：

1. 运行到p1被赋值，第一个promise被执行，输出promise1，其立即resolve，所以then被立即放入微任务中micro:[p1的then]；
2. 运行到p3被赋值，第三个promise被执行，输出promise3，其立即resolve，所以then被立即放入微任务中micro:[p1的then，p3的then]；
3. 执行栈为空，查看微任务并执行，先执行p1的then，输出then11，
接着一个promise被创建，输出promise2，其then被加入微任务中micro:[p3的then,p2的then]，
接着p1的then.then被放入micro:[p3的then,p2的then，p1的then的then]
再执行微任务中的p3的then，输出then31,micro: [p2的then，p1的then的then]
4. 执行p2的then，输出then21，然后p2的then的then被放入微任务中micro：[p1的then的then,p2.then.then]，再执行p1的then的then，输出then12
5. 查看微任务third-micro：[p2.then.then]，执行输出then22

执行结果：

promise1，promise3，then11，promise2，then31，then21，then12，then22

### 变异

then里面的new Promise变为return new Promise

```js
const p1 = new Promise((resolve, reject) => {
    console.log('promise1');
    resolve();
}).then(() => {
    console.log('then11')
    new Promise((resolve, reject) => {
        console.log('promise2');
        resolve();
    }).then(() => {
        console.log('then21');
    }).then(() => {
        new Promise((resolve, reject) => {
            console.log('promise3');
            resolve();
        }).then(() => {
            console.log('then31');
        }).then(() => {
            console.log('then32');
        });
    }).then(() => {
        console.log('then23');
    }).then(() => {
        console.log('then24');
    });
}).then(() => {
    console.log('then12');
}).then(() => {
    console.log('then13');
});
```

执行结果：
promise1,then11,promise2,then21,promise3,then31,then32,then23,then24,then12,then13

分析：
return new Promise语句会将其then都追加在最外层的promise的then中，顺序就是谁先出现谁在前面。

### nextTick的优先级高于Promise回调

```js
console.log('golb1');

process.nextTick(function () {
  console.log('glob1_nextTick');
})
new Promise(function (resolve, rej) {
  console.log('glob1_promise');
  rej();
}).catch(function () {
  console.log('glob1_catch')
})

process.nextTick(function () {
  console.log('glob2_nextTick');
})
new Promise(function (resolve, rej) {
  console.log('glob2_promise');
  rej();
}).catch(function () {
  console.log('glob2_catch')
})
```

执行过程分析：

```js
第一次宏任务执行后，输出同步语句：
golb1
glob1_promise
glob2_promise
而对于立即resolve的Promise回调，他的优先级低于nextTick（更不用说那些未来某个时刻才变成resolve状态的Promise的then了），所以nextTick先于then/catch被执行。
随后的微任务执行输出：
golb1
glob1_promise
glob2_promise
glob1_nextTick
glob2_nextTick
glob1_catch
glob2_catch

所以最终输出结果为：
golb1
glob1_promise
glob2_promise
glob1_nextTick
glob2_nextTick
glob1_catch
glob2_catch
```

### setTimeout0与setImmediate的优先级

```js
setTimeout(function() {
    console.log('timeout1');
    process.nextTick(function() {
        console.log('timeout1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('timeout1_promise');
        resolve();
    }).then(function() {
        console.log('timeout1_then')
    })
})

setImmediate(function() {
    console.log('immediate1');
    process.nextTick(function() {
        console.log('immediate1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('immediate1_promise');
        resolve();
    }).then(function() {
        console.log('immediate1_then')
    })
})
```

或者

```js
setTimeout(function() {
    console.log('timeout1');
    process.nextTick(function() {
        console.log('timeout1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('timeout1_promise');
        resolve();
    }).then(function() {
        console.log('timeout1_then')
    })
}, 1)

setImmediate(function() {
    console.log('immediate1');
    process.nextTick(function() {
        console.log('immediate1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('immediate1_promise');
        resolve();
    }).then(function() {
        console.log('immediate1_then')
    })
})
```
不管谁先谁后，结果也都不是固定的
结果可能为：
```
timeout1
timeout1_promise
timeout1_nextTick
timeout1_then
immediate1
immediate1_promise
immediate1_nextTick
immediate1_then
```
也可能为：
```
immediate1
immediate1_promise
immediate1_nextTick
immediate1_then
timeout1
timeout1_promise
timeout1_nextTick
timeout1_then
```

### 变异及简化

```js
setTimeout(function() {
    console.log('timeout1');
})

setImmediate(function () {
  console.log('immediate1');
})

setTimeout(function() {
    console.log('timeout2');
})

setImmediate(function() {
    console.log('immediate2');
})
```

结果不一定…
```
timeout1
timeout2
immediate1
immediate2
```
或
```
timeout1
immediate1
immediate2
timeout2
```
或
```
immediate1
immediate2
timeout1
timeout2
```


参考：

https://segmentfault.com/a/1190000013660033?utm_source=channel-hottest

https://html.spec.whatwg.org/multipage/webappapis.html#event-loops

https://1shark.com/u/article?id=5ccbb62552d6f36598d84dd8

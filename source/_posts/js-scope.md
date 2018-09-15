---
title: js中的作用域
date: 2018-09-15 17:07:31
categories: [JavaScript]
tags:
  - js作用域
---

在上一次学习分享了函数式编程后对JavaScript 的作用域颇有感想，今天小编就本人的理解来谈一下js中的作用域以及作用域的改变

# 一、作用域分类
js中作用域主要分为一下几种：

- 全局作用域
- 函数作用域
- 块级作用域
- 词法作用域
- 动态作用域
<!-- more -->
### （一）全局作用域
作用域，是指变量的生命周期（一个变量在哪些范围内保持一定值）。

###### 1. 全局变量

生命周期将存在于整个程序之内。

能被程序中任何函数或者方法访问。

在 JavaScript 内默认是可以被修改的。

全局变量，虽然好用，但是是极其可怕的，当程序变得复杂全局作用域将会一定程度上造成开发者的困扰以及难题，不论从命名冲突还是垃圾回收机制来看，都不赞成过多使用全局变量。

###### 2. 显式声明
带有关键字 var 的声明；

```js
var val = 111

var fun = function () { 
    console.log('test') 
}

console.log(window.fun)		// ƒ () { console.log('test') }

console.log(window.val)		// 111
    
```

其实，我们写的函数如果不经过封装，也会是全局变量，他的生命周期也就是全局作用域；

###### 3. 隐式声明
不带有声明关键字的变量，JS引擎 会默认将之声明一个全局变量

```js
function foo(value) {
  result = value + 1    // 没有用 var 修饰
  return result
}
foo(1)    // 2
console.log(window.result)    // 2 
```

现在，变量 result 被挂载到 window 对象上了！！！

### （二） 函数作用域
函数作用域内，对外是封闭的，从外层的作用域无法直接访问函数内部的作用域！！！
```js
function bar() {
  var bar = 'bar'
}
console.log(bar)    // 报错：ReferenceError: bar is not defined
```

通过 return 访问函数内部变量：
```js
function bar(value) {
  var bar = 'bar';
  
  return bar + value;
}
console.log(bar('foo'));		// "barfoo"
```

函数就像一个工厂，我们输入一些东西，它在内部加工，然后给我们一个加工产物；

###### 1. 闭包访问函数内部变量
```js
function bar(value) {
  var bar = 'bar'
  var rusult = bar + value
  
  function inner() {
     return rusult
  }
  return inner()
}
console.log(bar('foo'));		// "barfoo"
```

###### 2. 立即执行函数

这是个很实用的函数，很多库都用它分离全局作用域，形成一个单独的函数作用域；

```js
const pingpong = (() => {
let PRIVATE = 0
  return {
    increase(n) {
        return PRIVATE += n
    },
    decrease(n) {
        return PRIVATE -= n
    }
  }
})()
```


它能够自动执行闭包里面包裹的内容，能够很好地消除全局变量的影响；

###  （三） 块级作用域

JS在在 ES6 之前，是没有块级作用域的概念的。

```js
for(var i = 0; i < 5; i++) {
  // ...
}
console.log(i)    // 5
```

很明显，用 var 关键字声明的变量，在 for 循环之后仍然被保存这个作用域里；

这可以说明： `for() { }`仍然在，全局作用域里，并没有产生像函数作用域一样的封闭效果；

如果想要实现 块级作用域 那么我们需要用 let 关键字声明！！！
```js
for(let i = 0; i < 5; i++) {
  // ...
}

console.log(i)    // 报错：ReferenceError: i is not defined
```

在 for 循环执行完毕之后 i 变量就被释放了，它已经消失了！！！

同样能形成块级作用域的还有 const 关键字：
```js
if (true) {
  const a = 'inner';
}

console.log(a)    // 报错：ReferenceError: a is not defined
```

######  1. 块级作用域之let 和 const

二者创建块级作用域的条件是必须有一个 { } 包裹：
```js
{
  let a = 'inner';
}
  
if (true) {
   let b = 'inner'; 
}

var i = 0;

// ......
```
不要小看块级作用域，它能帮我们做很多事情，举个栗子：
```js
for(var i = 0; i < 5; i++) {
  setTimeout(function() {
     console.log(i);			// 5 5 5 5 5
  }, 200)
}
```

这几乎是作用域的必考题目，你会觉得这种结果很奇怪，但是事实就是这么发生了；

这里的 i 是在全局作用域里面的，只存在 1 个值，等到回调函数执行时，用词法作用域捕获的 i 就只能是 5

因为这个循环计算的 i 值在回调函数结束之前就已经执行到 5 了；我们应该如何让它恢复正常呢？？？

**解法1：调用函数，创建函数作用域：**
```js
for(var i = 0; i < 5; i++) {
  abc(i)
}

function abc(i) {
  setTimeout(function() {
     console.log(i);			// 0 1 2 3 4 
  }, 200); 
}
```
这里相当于创建了5个函数作用域来保存，我们的 i 值

**解法2：采用立即执行函数，创建函数作用域；**
```js
for(var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j)
    }, 200)
  })(i)
}
```

原理同上，只不过换成了自动执行这个函数罢了，这里保存了 5 次 i 的值；

**解法3：let 创建块级作用域**
```
for(let i = 0; i < 5; i++) {
    setTimeout(function() {
      console.log(i)
    }, 200)
}
```

### （四）词法作用域

函数的作用域在函数定义的时候就决定了，词法作用域是指一个变量的可见性，及其文本表述的模拟值，摘自《JavaScript函数式编程》

听起来，十分地晦涩，不过将代码拿来分析就非常浅显易懂了；
```js
val = 'outer'
function fun() {
  var val = 'inner'
  console.log(val)    // "inner"
  function innerFun() {
    var val = 'innerfun'
    
    console.log(val)    // "innerfun"
  }
  return innerFun()
}
fun()
console.log(val);			// "outer"
```

当我们访问变量时，JS引擎总会从最近的作用域向外层域查找
再比如：
```js
var val = 'outer';
function foo() {
  console.log(val);		// "outer"
}

function bar() {
  var val = 'inner';
  
  foo()
}

bar()
```
显然，当 JS 引擎查找这个变量时，发现全局的 val 离得更近一些，这恰好和 动态作用域 相反

如上图所示，下面将讲述与 词法作用域相反的动态作用域；

### （五）动态作用域

与词法作用域相反，动态作用域是在函数调用的时候才决定的。js中this的引用一般情况从词法作用域查找，但可以通过call/apply/bind改变，不过需要谨慎使用，免的埋坑。在编程中，最容易被低估和滥用的概念就是动态作用域（《JavaScript函数式编程》）。

> 动态作用域，作用域是基于调用栈的，而不是代码中的作用域嵌套；

###### 1. call/apply/bind
1. 都是用来改变函数的this对象的指向的。
2. 第一个参数都是this要指向的对象。
3. 都可以利用后续参数传参。
```js
var xw={
    name: "小王",
    gender: "男",
    age: 24,
    say: function(){
        alert(this.name+" , "+this.gender+" ,今年"+this.age);
    }
}
var xh={
    name: "小红",
    gender: "女",
    age: 18
}
xw.say();
```
本身没什么好说的，显示的肯定是小王 ， 男 ， 今年24。那么如何用xw的say方法来显示xh的数据呢。对于call可以这样：
```js
xw.say.call(xh);
```
对于apply可以这样：
```js
xw.say.apply(xh);
```
而对于bind来说需要这样：
```js
xw.say.bind(xh)();
```
如果直接写xw.say.bind(xh)是不会有任何结果的，看到区别了吗？call和apply都是对函数的直接调用，而bind方法返回的仍然是一个函数，因此后面还需要()来进行调用才可以。那么call和apply有什么区别呢？我们把例子稍微改写一下。

```js
var xw={
    name: "小王",
    gender: "男",
    age: 24,
    say: function(school,grade){
        alert(this.name+" , "+this.gender+" ,今年"+this.age+" ,在"+school+"上"+grade);
    }
}
var xh={
    name: "小红",
    gender: "女",
    age: 18
}

```
可以看到say方法多了两个参数，我们通过call/apply的参数进行传参。对于call来说是这样的
```js
xw.say.call(xh,"实验小学","六年级");  
```
而对于apply来说是这样的
```js
xw.say.apply(xh,["实验小学","六年级"]);
```
看到区别了吗，call后面的参数与say方法中是一一对应的，而apply的第二个参数是一个数组，数组中的元素是和say方法中一一对应的，这就是两者最大的区别。那么bind怎么传参呢？它可以像call那样传参。
```
xw.say.bind(xh,"实验小学","六年级")();
```
但是由于bind返回的仍然是一个函数，所以我们还可以在调用的时候再进行传参。
```
xw.say.bind(xh)("实验小学","六年级");
```
###### 2. bind实现

bind之所以可以这样调用呢，还得追溯到源码。
```js
Function.prototype.bind = function(context) {
    var _this = this,
    _args = Array.prototype.slice.call(arguments, 1);
    return function() {
        return _this.apply(context, _args.concat(Array.prototype.slice.call(arguments)));
    }
}
```

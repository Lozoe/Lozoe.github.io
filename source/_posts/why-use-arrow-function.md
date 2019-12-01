---
title: 为什么使用箭头函数
date: 2018-09-14 18:15:37
categories: [JavaScript]
---


# 有什么优点？
### 更简洁的语法

```js
function funcName(params) {
   return params + 2;
 }
funcName(2);
// 4
```
<!-- more -->
该函数使用箭头函数可以使用仅仅一行代码搞定！
```js
var funcName = params => params + 2
funcName(2);
// 4

```

无参：
```js
() => { statements }
```
只有一个参数(省略括号)
```js
parameters => { statements }
```
如果返回值仅仅只有一个表达式(expression), 还可以省略大括号
```js
parameters => expression

// 等价于:
function (parameters){
  return expression;
}
```
### 没有局部this的绑定
箭头函数不会改变this本来的绑定。

```js
function Counter() {
  var that = this;
  this.timer = setInterval(() => {
    console.log(this === that);
  }, 1000);
}
var b = new Counter();
// true
// true
// ...
```
# 其他特点
语法汇总：
```js
(参数1, 参数2, …, 参数N) => { 函数声明 }
(参数1, 参数2, …, 参数N) => 表达式（单一）
//相当于：(参数1, 参数2, …, 参数N) =>{ return 表达式; }

// 当只有一个参数时，圆括号是可选的：
(单一参数) => {函数声明}
单一参数 => {函数声明}

// 没有参数的函数应该写成一对圆括号。
() => {函数声明}

//加括号的函数体返回对象字面表达式：
参数=> ({foo: bar})

//支持剩余参数和默认参数
(参数1, 参数2, ...rest) => {函数声明}
(参数1 = 默认值1,参数2, …, 参数N = 默认值N) => {函数声明}

//同样支持参数列表解构
let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c;
f();  // 6
```

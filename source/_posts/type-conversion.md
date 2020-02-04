---
title: 类型转换
date: 2018-09-14 10:46:07
categories: [JavaScript]
tags:
  - 类型转换
---

# 类型转换

## 分类

- 显式类型转换
- 隐式类型转换

最新的ECMAScript标准定义了7种数据类型：
- 原始类型
Boolean Null Undefined Number String Symbol
- 对象类型
Object
<!-- more -->
## 显式类型转换

1. Number函数
2. String函数
3. Boolean函数

### Number函数
#### 原始类型转换
- 数值：转换后还是原来的值
- 字符串：如果可以被解析为数值，则转换为相应的数值，否则得到NaN，空字符串转为0
- 布尔值：true转换为1，false转换为0
- undefined：转为NaN
- null： 转为0
```
// Number函数：转换成数值
// 转换后还是原来的值
console.log(Number(324)); // 324
// 如果可以被解析为数值，则转换为相应的数值，否则得到NaN。空字符串转为0
console.log(Number('324'));// 324
console.log(Number('324abc')); // NaN

console.log(Number('')); // 0
// true转成1，false转成0
console.log(Number(false)); // 0
console.log(Number(true));// 1
// 转成NaN
console.log(Number(undefined)); // NaN
// 转成0
console.log(Number(null)); // 0
```
#### 对象类型（Object）转换：
  
1）先调用对象的valueOf方法，如果该方法返回原始类型的值（Number,String,Boolean），则直接对该值使用Number方法，不再进行后续步骤。

2）如果valueOf返回复合类型的值，再调用对象自身的toString方法，如果toString方法返回原始类型的值，则对该值使用Number方法，不再进行后续步骤。如果toString返回复合类型的值，则报错。

```js
var a = {a: 1};
console.log('a', Number(a)); // "a NaN"

// 如果valueOf返回原始类型
var b = {
    b: 1,
    valueOf: function () {
        console.log('valueOf excuted')
        // return null; // 123, 'abc', '', null, undefined, true, false 如果是原始类型，则直接对该值调用Number方法
        return {b: 'this is b object valueOf'} // 如果不是原始类型，则接着调用toString
    },
    toString: function () {
        console.log('toString excuted')
        return 123// 123, 'abc', '', null, undefined, true, false  如果toString方法返回原始类型的值，则对该值使用Number方法
        // return {
        //     b: 'b toString'
        // }; // 如果toString返回复合类型的值，则报错
    },
};
console.log('b', Number(b));

// 如果valueOf不返回原始类型
var c = {
    c: 1,
    toString: function () {
        return {
            c: 2,
        };
    }
};
console.log('c', String(c)); 
/* 报错 VM152:10 Uncaught TypeError: Cannot convert object to primitive value
    at String (<anonymous>)
    at <anonymous>:10:18 */
```
### String函数
#### 原始类型转换
- 数值：转为相应的字符串
- 字符串：转换后还是原来的值
- 布尔值：true转换为'true' false转换为'false'
- undefined：转为'undefined'
- null：转为'null'
```js
// 数值：转为相应的字符串
console.log(String(123)); // "123"
// 字符串：转换后还是原来的值
console.log(String('abc')); // "abc"
// 布尔值：true转为“true”，false转为“false”
console.log(String(true)); // "true"
// undefined：转为“undefined”
console.log(String(undefined)); // "undefined"
// null：转为“null”
console.log(String(null)); // "null"
```

#### 对象类型（Object）转换：
  
1）先调用对象的toString方法，如果该方法返回原始类型的值（Number,String,Boolean），则直接对该值使用String方法，不再进行后续步骤。

2）如果toString返回复合类型的值，再调用对象自身的valueOf方法，如果valueOf方法返回原始类型的值，则对该值使用String方法，不再进行后续步骤。如果valueOf返回复合类型的值，则报错。

```js
// 先调用toString方法，如果toString方法返回的是原始类型的值，则对该值使用String方法;
// 如果toString方法返回的是复合类型的值，再调用valueOf方法，如果valueOf方法返回的是原始类型的值，则对该值使用String方法
var a = {
    a: 1
}
console.log('a', String(a));  // a "[object Object]"
var b = {
    b: 1,
    toString: function () {
        return {
            b: 2,
        };
    },
    valueOf: function () {
        return 'b';
    },
};
console.log('b', String(b)); 

// 如果toString不返回原始类型
var c = {
    c: 1,
    toString: function () {
        return {
            c: 2,
        };
    }
};
console.log('c', String(c)); 
/* 报错 VM152:10 Uncaught TypeError: Cannot convert object to primitive value
    at String (<anonymous>)
    at <anonymous>:10:18 */
```

#### Boolean函数
- undefined：false
- null：false
- -0：false
- +0：false
- NaN：false
- ''(空字符串)：false

```js
console.log('undefined', Boolean(undefined)); // false

console.log('null', Boolean(null)); // false

console.log('0', Boolean(0)); // false

console.log('NaN', Boolean(NaN)); // false

console.log('', Boolean('')); // false
```

除了以上情况其他一律为 转换结果一律为 true

## 隐式类型转换

- 四则运算 + - * /
- 判断语句 if 三目运算符
- native调用 console.log alert自动转换为String类型

常见：http://2ality.com/2012/01/object-plus-object.html
```js
[] + [] = "" // String([])+String([])

[] + {} = "[object Object]" // "String([])+String({})"

{} + [] = 0 // Chrome:{}代码块不执行 相当于 +[]  即数学中的求正运算 所以先调用Number([])([].valueOf() = [], [].toString() = "", Number("") = 0, +0 = 0)

{} + {} // Chrome: "[object Object][object Object]" eval 相当于字符串拼接 String({})+String({})
{} + {} // Firefox: NaN 第一个{}当成代码块不执行 也就是直接+{}求正运算+Number({})（{}.valueOf()={}, {}.toString() = "[object Object]",Number("[object Object]")=NaN, +NaN=NaN）

true + true = 2

"a" + {a: 1} = "a[object Object]"

var a = [1, 2, 3]
var b = [1, 2, 4]
console.log( a < b )
console.log( a == b )
console.log( a === b )
```
---
title: Object.assign
date: 2017-08-27 12:01:43
tags:
  - assign
---

Object.assign函数的使用,使用该函数我们可以快速的复制一个或者多个对象到目标对象中，本文内容涉及es6,es7相关的对象复制的内容，以及一些es5的替代方案的介绍。

## 函数原型
首先看一下函数的定义：
函数参数为一个目标对象（该对象作为最终的返回值）,源对象(此处可以为任意多个)。通过调用该函数可以拷贝所有可被枚举的自有属性值到目标对象中。
```javascript
Object.assign(target, ...sources);
```
这里我们需要强调的三点是：

- 可被枚举的属性
- 自有属性
- string或者Symbol类型是可以被直接分配的

拷贝过程中将调用源对象的getter方法，并在target对象上使用setter方法实现目标对象的拷贝。

## 函数实例
这里我们通过几个MDN上的例子来介绍一下使用方法：

### 实例一

我们参考上面的原型函数说明即可知道其最开始的o1因为设置为target，则调用其setter方法设置了其他对象的属性到自身。
```javascript
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };

var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, target object itself is changed.
```
### 实例二

我们自定义了一些对象，这些对象有一些包含了不可枚举的属性,另外注意使用 Object.defineProperty 初始化的对象默认是不可枚举的属性。对于可枚举的对象我们可以直接使用Object.keys()获得,或者使用for-in循环遍历出来.

对于不可枚举的属性，使用Object.assign的时候将被自动忽略。
```javascript
var obj = Object.create({ foo: 1 }, { // foo is an inherit property.
  bar: {
    value: 2  // bar is a non-enumerable property.
  },
  baz: {
    value: 3,
    enumerable: true  // baz is an own enumerable property.
  }
});

var copy = Object.assign({}, obj);
console.log(copy); // { baz: 3 }  
```
### 实例三

对于只读的属性，当分配新的对象覆盖他的时候，将抛出异常:
```javascript
var target = Object.defineProperty({}, 'foo', {
  value: 1,
  writable: false
}); 

Object.assign(target, { bar: 2 })

//{bar: 2, foo: 1}

Object.assign(target, { foo: 2 })
//Uncaught TypeError: Cannot assign to read only property 'foo' of object '#<Object>'(…)
```
## Polyfill
这里我们简单的看下如何实现es5版本的Object.assign：

实现步骤：

1. 判断是否原生支持该函数，如果不存在的话创建一个立即执行函数，该函数将创建一个assign函数绑定到Object上。
2. 判断参数是否正确(目的对象不能为空，我们可以直接设置{}传递进去,但必须设置该值)
3. 使用Object在原有的对象基础上返回该对象，并保存为out
4. 使用for…in循环遍历出所有的可枚举的自有对象。并复制给新的目标对象(hasOwnProperty返回非原型链上的属性)
源码如下：
```
if (typeof Object.assign != "function") {
  (function() {
    Object.assign = function(target) {
      "use strict";
      if (target === undefined || target === null) {
        throw new TypeError("Cannot convert undefined or null to object");
      }

      var output = Object(target);
      for (var index = 1; index < arguments.length; index++) {
        var source = arguments[index];
        if (source !== undefined && source !== null) {
          for (var nextKey in source) {
            if (source.hasOwnProperty(nextKey)) {
              output[nextKey] = source[nextKey];
            }
          }
        }
      }
      return output;
    };
  })();
}
// 或者 MDN
if (typeof Object.assign != 'function') {
  // Must be writable: true, enumerable: false, configurable: true
  Object.defineProperty(Object, "assign", {
    value: function assign(target, varArgs) { // .length of function is 2
      'use strict';
      if (target === undefined || target == null) { // TypeError if undefined or null
        throw new TypeError('Cannot convert undefined or null to object');
      }

      var to = Object(target);

      for (var index = 1; index < arguments.length; index++) {
        var nextSource = arguments[index];

        if (source !== undefined && source !== null) { // Skip over if undefined or null
          for (var nextKey in nextSource) {
            // Avoid bugs when hasOwnProperty is shadowed
            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
              to[nextKey] = nextSource[nextKey];
            }
          }
        }
      }
      return to;
    },
    writable: true,
    configurable: true
  });
}
```
## 扩展内容
### 1.深度复制

当我们调用下面的函数的时候，由于Object.assign将覆盖之前的内容，所以并不能完全的做到融合对象，而是全部替换掉，所以返回的对象内容将变成最后一个值; {a: {c: 3}
```javascript
Object.assign({a: {b: 0}}, {a: {b: 1, c: 2}}, {a: {c: 3}});
```
如何深层次的融合对象，比如我们期望的输出结果为：
```
{a:{b:1,c:3}}
```
这样我们必须实现自己的算法来完成深层复制了,不过github上已经有很多好的解决方案，比如[deep-merge](https://github.com/sindresorhus/deep-assign/blob/master/index.js) 通过递归的方式逐层的去调用assign函数。

2.ES2016实现

在es7中我们使用rest属性可以捕获所有剩余的对象内容比如下面的例子(可使用babel-repl页面测试，浏览器一般尚未支持)：
```javascript
let { fname, lname, ...rest } = { fname: "Hemanth", lname: "HM", location: "Earth", type: "Human" };
fname; //"Hemanth"
lname; //"HM"
rest; // {location: "Earth", type: "Human"}
```
这样我们就可以使用该特性来实现assign函数
```javascript
let oldObj1 = {a:"a",b:{b1:"b1"}}
let oldObj2 = {a:"a1",b:{b2:"b2"},c:"c"}

let newObject={...oldObj1,...oldObj2};
console.log(newObject) //{"a":"a1","b":{"b2":"b2"},"c":"c"}
```

不过仍旧只是浅层的替换，并没有实现深层次的合并。

转自：https://cnodejs.org/topic/56c49662db16d3343df34b13
参考：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign
---
title: js遍历
date: 2018-08-25 17:04:01
categories: [JavaScript]
tags:
  - 遍历
  - for-of
---

## JavaScript 数组遍历方法的对比
原文首发至 lozoe.github.io 转载请标注原作者和附带原文连接

### 前言
JavaScript 发展至今已经发展出多种数组的循环遍历的方法,不同的遍历方法运行起来那个比较快,不同循环方法使用在那些场景,下面将进行比较:
<!--more-->
### 各种数组遍历的方法
##### 1、for 语句
```javascript
var arr = [1,2,4,6]
for(var i = 0, len = arr.length; i < len; i++){
    console.log(arr[i])
}
```

这是标准也是最传统的for循环语句的写法，字符串也支持，定义一个变量i作为索引，以跟踪访问的位置，len是数组的长度，循环条件就是i不能超过len。

##### 2、forEach 语句
forEach 方法对数组的每一项运行传入的参数，并且执行一次提供的callback函数,forEach只可用于数组.遍历一个数组让数组每个元素做一件事情.那些已删除（使用delete方法等情况）或者未初始化的项将被跳过（但不包括那些值为 undefined 的项）（例如在稀疏数组上)。
```javascript
var arr = [1,5,8,9]
arr.forEach(function(item) {
    console.log(item);
})
```
##### 3、for-in 语句
for-in语句是一种精准的迭代语句，可以用来枚举对象的属性 for-in 循环只遍历可枚举属性。一般常用来遍历对象，包括++非整数类型的名称和继承的那些原型链上面的属性也能被遍历++。像 Array和 Object使用内置构造函数所创建的对象都会继承自Object.prototype和String.prototype的不可枚举属性就不能遍历了.
```javascript
var obj = {
    name: 'lz',
    age: 25
}
for (var key in obj) {
    console.log(obj[key])
}
```

##### 4、for-of 语句 (ES6) 今天的重中之重
for-of语句不仅遍历数组，而且也能否遍历其他类型的数据。通过Iterator接口的实现为各种数据提供统一简便的访问接口。即强大的for-of.


for-of语句在可迭代对象（包括 Array，Map，Set，String，TypedArray，函数的arguments 对象、NodeList对象以及其它部署过Iterator接口的数据结构）上创建一个迭代循环，Array，Map，Set，String，TypedArray，函数的arguments 对象、NodeList对象原生具备Iterator接口。调用自定义迭代钩子，并为每个不同属性的值执行语句。for-of是ES6创建的一种新的遍历命令。
```javascript
var arr = [1,2,3,4]
// 遍历数组
for (let item of arr) {
    console.log(item)
}
// 计算生成的数据结构（es6数组，set, map都部署了entries,keys,values）
// entries
for (let pair of arr.entries()) {
    console.log(pair) 
} // [0, 1] [1, 2] [2, 3] [3, 4]
// keys
for (let key of arr.keys()) {
    console.log(key) 
} // 0 1 2 3

// 遍历字符串
let str = 'hello';
for (let item of str) {
    console.log(item)
} // h e l l o

// 遍历DOM NodeList
let paras = document.querySelectorAll('div');
for (let item of paras) {
    console.log(item)
} 

// 遍历arguments
function printArgs() {
    for(let x of arguments) {
        console.log(x);
    }
}
printArgs('a', 'b'); // 'a' 'b'

// 没有Iterator接口的类数组对象 可以使用Array.from()将其转换为数组，再使用for-of
let arrLike = {length: 3, 0: 'a', 1: 'b', 2: 'c'};
for(let item of Array.from(arrLike)) {
    console.log(item)
}

```
对于普通的对象不能直接使用for-of,因其没有被部署Iterator接口。但是for-in依然可以遍历对象的键名。
```javascript
let obj = {
    a: 1,
    b: 2, 
    c: 3
}
for(let key in obj) {
    console.log(key)
}// a b c

for(let key of obj) {
    console.log(key)
} // Uncaught TypeError: obj is not iterable
```
解决方案一：
使用Object.keys()方法将对象的键名生成一个带有iterator接口的数组，再对之遍历
```javascript
let obj = {
    a: 1,
    b: 2, 
    c: 3
}

for(let key of Object.keys(obj)) {
    console.log(key + ': ' + obj[key]);
} // a: 1 b: 2 c: 3

Object.keys(obj).forEach((key, index) => {
    console.log(key + ': ' + obj[key])
})

```
解决方案二：
Generator将对象重新包装
```javascript
function* entries(obj) {
    for(let key of Object.keys(obj)) {
        yield [key, obj[key]];
    }
}
let obj = {
    a: 1,
    b: 2, 
    c: 3
}

for(let [key, value] of entries(obj)) {
    console.log(key + ': ' + value);
} // a: 1 b: 2 c: 3
```
### 各种遍历语法的比较
##### 1、for循环
最原始的写法，但是写法比较麻烦，但是性能是最好的。
##### 2、forEach
所以呢数据提供了forEach

不过问题在于无法跳出forEach循环，break,return 全都不奏效。

##### 3、for-in
for-in可以遍历数组键名
但是有如下缺点：
- 数组的键名是数字，但是for-in循环是以字符串作为键名‘0’，‘1’；
- for-in循环不仅可以遍历数字键名，还会遍历手动添加其他类型的键名，也会遍历原型链上的属性，所以会更加耗费时间；
- 默写情况下，以任意顺序遍历键名。

for-in是为对象设计，不适合遍历数组。

##### 4、for-of
- 和for-in一样语法简洁
- 不同于forEach，可是使用break, continue, return
- 提供了遍历所有数据结构的统一操作接口

### 其他数组迭代的方法

##### 1、map 方法 (不改变原数组)
map 方法对数组中每一项运行给定函数，返回每次函数调用结果组成的数组。
 callback 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 delete 删除的索引则不会被调用。
```javascript
var arr = [1, 2, 3, 4]
var mapResult = arr.map((item, index, array) => item * 2); // 2 4 6 8
```

##### 2、some 方法 (不改变原数组)
some 方法对数组中每一项运行给定函数，如果该函数对任意一项返回true,则返回true。
 callback 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 delete 删除的索引则不会被调用。
 
```javascript
var arr = [1, 2, 3, 4]
var someResult = arr.some((item, index, array) => item > 2); // true
```

##### 3、every 方法 (不改变原数组)
every 方法对数组中每一项运行给定函数，如果该函数对每一项返回true,则返回true，否则返回false。
 callback 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 delete 删除的索引则不会被调用。
 
```javascript
var arr = [1, 2, 3, 4]
var everyResult = arr.every((item, index, array) => item > 0); // true
```

##### 4、filter 方法 (不改变原数组)
map 方法对数组中每一项运行给定函数，返回true的项组成的数组。
 callback 函数只会在有值的索引上被调用；那些从来没被赋过值或者使用 delete 删除的索引则不会被调用。
 
```javascript
var arr = [1, 2, 3, 4]
var filterResult = arr.filter((item, index, array) => item > 2); // [3, 4]
```
### 归并方法

##### 1、reduce 方法
从数组的第一项开始，逐个遍历到最后。
接收两个参数：
- 在每一项上调用的函数；
- 作为归并基础的基础值。（可选）

回调接收四个参数：前一个值、当前值、当前索引、数组对象，这个函数返回的任何值都会作为第一个参数自动传给下一项。第一次迭代发生在数组的第二项上，因此第一个参数是数组的第一项第二个参数是数组的第二项。

```javascript
// 迭代求和
let arr = [1, 2, 3, 4];
let sum = arr.reduce((prev, cur, index, arr) => {
    return prev + cur;
}) // 10
```
 浏览器支持： IE9+ Firefox3+ safari 4+ Opera 10.5 Chrome

##### 1、reduceRight 方法
从数组的最后一项开始，逐个遍历到第一项。 其他特点同reduce

### 遍历性能

这里我使用了[jsPerf](https://jsperf.com/)平台进行测试.

JavaScritp loop 对比
```
var nullarr = new Array(100);
var arr = [];
for(var i = 0; i < 100; i++){
  if (i % 2 === 0) {
      arr[i] = i.toString();
  } else {
      arr[i] = i;
  }
}
for(var i = 0; i < 100; i++){
  if( i % 2 ===0) {
     nullarr[i] = new Object(i)
  } else {
     nullarr[i] = i
  }
}
```

那我们对比一下 for for len forEach for-in for-of map filter 循环的速度
[测试地址](https://jsperf.com/js-loop-compare123)

![工作原理](js-loop-compare.png)


可以看到 for循环的速度是最快的,是最老的循环,也是优化得最好的,其次是for-of这个是es6才新增的循环非常好用,最慢是for-in我们可以作一下速度排序
```
for > for-of > forEach > filter > map > for-in
```
这很明显处理大量循环数据的时候还是要使用古老for循环效率最好,但也不是不使用for-in,其实很多时候都要根据实际应该场景的,for-in更多使用在遍历对象属性上面,for-in在遍历的过程中还会遍历继承链,所以这就是它效率比较慢的原因,比如map 速率不高,不过处理在ES6实现数组功能上面非常好用方便,轻松影射创建新数组.或者例如使用Iterator属性也是行的,所以每个循环都有合适使用的地方.




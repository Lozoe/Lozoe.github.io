---
title: 前端代码规范之JavaScript
date: 2019-07-22 10:38:07
tags:
  - 代码规范
---

## 目录

* [代码格式](#代码格式)
* [推荐使用](#推荐使用)
* [不推荐使用](#不推荐使用)

## 代码格式

### 1. 缩进4个空格

保留之前的习惯，对老代码的保护；
4个空格更清晰；
不用tab：有自己的编码表示

### 2. 行宽

100字符(换行 & 拆分 & 缩进)
<!-- more -->
### 3. 变量声明

使用前声明

### 4. 命名驼峰式

（A-Z a-z 0-9 _$）

- "_"开头代表变量为私有
- 大多数变量和方法名应该以小写字母开始
- 全局变量应该全部使用大写字母
- 类名使用大写字母开头
- jQuery对象使用$开头

### 5. 注释

多行

```js
/** xxx */
/**
 * make() returns a new element
 * based on the passed in tag name
 */
function make(tag) {
    return element;
}
```

单行使用 //：

```js
// bad
const active = true;  // is current tab

// good
// is current tab
const active = true;

// bad
function getType() {
    console.log('fetching type...');
    // set the default type to 'no type'
    const type = this._type || 'no type';

    return type;
}

// good
function getType() {
    console.log('fetching type...');

    // set the default type to 'no type'
    const type = this._type || 'no type';

    return type;
}

// also good
function getType() {
    // set the default type to 'no type'
    const type = this._type || 'no type';

    return type;
}
```

在要注释部分的上边一行放置注释. 如果不是块的第一行则在注释前留一个空行.

ps:

- 所有注释都要以空格开头便于阅读
- 使用 // FIXME: 来标注问题.
- 使用 // TODO: 来标明问题的解决方案.

### 6. 空格和空行

1.在起始花括号前加一个空格

```js
function test() {
    console.log('test');
}

dog.set('attr', {
    age: '1 year',
    breed: 'Bernese Mountain Dog',
});
```

2.控制语句的圆括号前加一个空格 (if, while 等)

```js
// bad
if(isJedi) {
    fight ();
}

// good
if (isJedi) {
    fight();
}
```

3.操作符两边放置空格

```js
// bad
const x=y+5;

// good
const x = y + 5;
```

4.空行结束文件

```js
// bad
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;

// bad
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;↵
↵

// good
import { es6 } from './AirbnbStyleGuide';
  // ...
export default es6;↵
```

5.函数调用和声明在参数列表和函数名间不要放置空格

```js
// bad
function fight () {
    console.log ('Swooosh!');
}

// good
function fight() {
    console.log('Swooosh!');
}
```

6.圆括号、方括号中不要添加空格

```js
// bad
function bar( foo ) {
    return foo;
}

// good
function bar(foo) {
    return foo;
}

// bad
if ( foo ) {
    console.log(foo);
}

// good
if (foo) {
    console.log(foo);
}

// bad
const foo = [ 1, 2, 3 ];

// good
const foo = [1, 2, 3];
```

7.花括号中添加空格

```js
// bad
const foo = {clark: 'kent'};

// good
const foo = { clark: 'kent' };
```

8.逗号前不要加空格，逗号后要加空格.

9.对象字面量属性的键和值之间要强制使用空格

```js
var obj = { "foo": 42 };
```

## 推荐使用

- 单个文件不超过1000行
- 单个函数不超过100行
- 单个函数复杂度不超过10
- 单个函数参数个数不超过7个（解决：把参数封装成对象）
- 字符串推荐使用单引号（html片段中的属性字符串用双引号）

## 不推荐使用

- 不要批量格式化他人代码
- 不推荐使用new Object()、new Array()（前者性能较低：new Object() => {}、  new Array() => []）
- 判断时避免使用 ```js==、 !=```（前者不严谨，会类型转换： == => === 、!= => !==）
- if嵌套不要超过3层(逻辑封装)
- 不推荐回调嵌套（promise、Async/await (es7)推荐）
- 不推荐使用魔法数字（注释、语义化常量）

参考： https://github.com/airbnb/javascript

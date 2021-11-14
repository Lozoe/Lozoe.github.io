---
title: 了不起的WeakMap
date: 2021-11-14 17:24:20
tags:
---

ES2015（ES6）中新增了几种数据类型，包括Map WeakMap Set WeakSet等。其中Map可以与我们熟悉的对象Object进行对照，他们的功能都是提供一个键值对集合，主要的区别在于Object的key只能是字符串，而Map的key可以是任意类型。

今天要讨论的主角是WeakMap。

WeakMap 对象是键/值对的集合，且其中的键是弱引用的。其键只能是对象，而值则可以是任意的。WeakMap的作用就是可以更有效的垃圾回收、释放内存。所以今天将从以下几个方面学习WeakMap

![概要](概要.png)

<!-- more -->

## 一、什么是垃圾回收

在计算机科学中，垃圾回收（Garbage Collection，缩写为 GC）是指一种自动的存储器管理机制。当某个程序占用的一部分内存空间不再被这个程序访问时，这个程序会借助垃圾回收算法向操作系统归还这部分内存空间。垃圾回收器可以减轻程序员的负担，也减少程序中的错误。

垃圾回收最早起源于 LISP 语言，它有两个基本的原理：

考虑某个对象在未来的程序运行中，将不会被访问；
回收这些对象所占用的存储器。
JavaScript 具有自动垃圾回收机制，这种垃圾回收机制原理其实很简单：找出那些不再继续使用的变量，然后释放其所占用的内存，垃圾回收器会按照固定的时间间隔周期性地执行这一操作。

![垃圾回收](gc-cycle.jpeg)

局部变量只有在函数执行的过程中存在，在这个过程中，一般情况下会为局部变量在栈内存上分配空间，然后在函数中使用这些变量，直至函数执行结束。垃圾回收器必须追踪每个变量的使用情况，为那些不再使用的变量打上标记，用于将来能及时回收其占用的内存，用于标识无用变量的策略主要有引用计数法和标记清除法。

### 1.1 引用计数法

最早的也是最简单的垃圾回收实现方法，这种方法为占用物理空间的对象附加一个计数器，当有其他对象引用这个对象时计数器加一，反之引用解除时减一。这种算法会定期检查尚未被回收的对象的计数器，为零的话则回收其所占物理空间，因为此时的对象已经无法访问。

引用计数法实现比较简单，但它却无法回收循环引用的存储对象，比如：

```JavaScript
function f() {
  var o1 = {};
  var o2 = {};
  o1.p = o2; // o1引用o2
  o2.p = o1; // o2引用o1
}

f();
```
为了解决这个问题，垃圾回收器引入了标记清除法。

### 1.2 标记清除法

标记清除法主要将 GC 的垃圾回收过程分为标记阶段和清除两个阶段：

- 标记阶段：把所有活动对象做上标记；
- 清除阶段：把没有标记（也就是非活动对象）销毁。

JavaScript 中最常用的垃圾回收方式就是标记清除（mark-and-sweep），当变量进入环境时，就将这个变量标记 “进入环境”，当变量离开环境时，就将其标记为 “离开环境”。标记清除法具体的垃圾回收过程如下图所示：

![标记清除](gc_mark_sweep.gif)

在日常工作中，对于不再使用的对象，通常我们会希望它们会被垃圾回收器回收。这时，你可以使用 null 来覆盖对应对象的引用，比如：

```js
let zhuo = { name: "liuzhuo" };
// 该对象能被访问，zhuo是它的引用
zhuo = null; // 覆盖引用
// 该对象将会被从内存中清除
```

但是，当对象、数组这类数据结构在内存中时，它们的子元素，如对象的属性、数组的元素都是可以访问的。例如，如果把一个对象放入到数组中，那么只要这个数组存在，那么这个对象也就存在，即使没有其他对该对象的引用。比如：

```js
let zhuo = { name: "liuzhuo" };
let array = [ zhuo ];
zhuo = null; // 覆盖引用

// zhuo 被存储在数组里, 所以它不会被垃圾回收机制回收
// 我们可以通过 array[0] 来获取它
```

同样，如果我们使用对象作为常规 Map 的键，那么当 Map 存在时，该对象也将存在。它会占用内存，并且不会被垃圾回收机制回收。比如：

```js
let zhuo = { name: "liuzhuo" };

let map = new Map();
map.set(zhuo, "zhuo");
zhuo = null; // 覆盖引用

// zhuo被存储在map中
// 我们可以使用map.keys()来获取它
```

那么如何解决上述 Map 的垃圾回收问题呢？这时我们就需要来了解一下 WeakMap。

## 二、为什么需要 WeakMap

### 2.1 Map 和 WeakMap 的区别

相信很多读者对 ES6 中 Map 已经不陌生了，已经有了 Map，为什么还会有 WeakMap，它们之间有什么区别呢？Map 和 WeakMap 之间的主要区别：

- Map 对象的键可以是任何类型，但 WeakMap 对象中的键只能是对象引用；
- WeakMap 不能包含无引用的对象，否则会被自动清除出集合（垃圾回收机制）；
- WeakMap 对象是不可枚举的，无法获取集合的大小。

在 JavaScript 里，Map API 可以通过使其四个 API 方法共用两个数组（一个存放键，一个存放值）来实现。给这种 Map 设置值时会同时将键和值添加到这两个数组的末尾。从而使得键和值的索引在两个数组中相对应。当从该 Map 取值的时候，需要遍历所有的键，然后使用索引从存储值的数组中检索出相应的值。

但这样的实现会有两个很大的缺点，首先赋值和搜索操作都是 O(n) 的时间复杂度（n 是键值对的个数），因为这两个操作都需要遍历全部整个数组来进行匹配。另外一个缺点是可能会导致内存泄漏，因为数组会一直引用着每个键和值。 这种引用使得垃圾回收算法不能回收处理他们，即使没有其他任何引用存在了。

相比之下，原生的 WeakMap 持有的是每个键对象的 “弱引用”，这意味着在没有其他引用存在时垃圾回收能正确进行。 原生 WeakMap 的结构是特殊且有效的，其用于映射的 key 只有在其没有被回收时才是有效的。

正由于这样的弱引用，WeakMap 的 key 是不可枚举的 (没有方法能给出所有的 key)。如果key 是可枚举的话，其列表将会受垃圾回收机制的影响，从而得到不确定的结果。因此，如果你想要这种类型对象的 key 值的列表，你应该使用 Map。而如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。

所以对于前面遇到的垃圾回收问题，我们可以使用 WeakMap 来解决，具体如下：

```js
let zhuo = { name: "liuzhuo" };

let map = new WeakMap();
map.set(zhuo, "全栈修仙之路");
zhuo = null; // 覆盖引用
```

### 2.2 WeakMap 与垃圾回收

WeakMap 真有介绍的那么神奇么？下面我们来动手测试一下同个场景下 Map 与 WeakMap 对垃圾回收的影响。首先我们分别创建两个文件：`map.js` 和 `weakmap.js`。

#### map.js

```js
//map.js
function usageSize() {
  const used = process.memoryUsage().heapUsed;
  return Math.round((used / 1024 / 1024) * 100) / 100 + "M";
}

global.gc();
console.log(usageSize()); // ≈ 3.19M

let arr = new Array(10 * 1024 * 1024);
const map = new Map();

map.set(arr, 1);
global.gc();
console.log(usageSize()); // ≈ 83.19M

arr = null;
global.gc();
console.log(usageSize()); // ≈ 83.2M
```

创建完 `map.js` 之后，在命令行输入 `node --expose-gc map.js` 命令执行 `map.js` 中的代码，其中 `--expose-gc` 参数表示允许手动执行垃圾回收机制。

#### weakmap.js

```js
function usageSize() {
  const used = process.memoryUsage().heapUsed;
  return Math.round((used / 1024 / 1024) * 100) / 100 + "M";
}

global.gc();
console.log(usageSize()); // ≈ 3.19M

let arr = new Array(10 * 1024 * 1024);
const map = new WeakMap();

map.set(arr, 1);
global.gc();
console.log(usageSize()); // ≈ 83.2M

arr = null;
global.gc();
console.log(usageSize()); // ≈ 3.2M
```

同样，创建完 weakmap.js 之后，在命令行输入 node --expose-gc weakmap.js 命令执行 weakmap.js 中的代码。通过对比 map.js 和 weakmap.js 的输出结果，我们可知 weakmap.js 中定义的 arr 被清除后，其占用的堆内存被垃圾回收器成功回收了。

下面我们来大致分析一下出现上述区别的主要原因：

对于 map.js 来说，由于在 arr 和 Map 中都保留了数组的强引用，所以在 Map 中简单的清除 arr 变量内存并没有得到释放，因为 Map 还存在引用计数。而在 WeakMap 中，它的键是弱引用，不计入引用计数中，所以当 arr 被清除之后，数组会因为引用计数为 0 而被垃圾回收清除。

了解完上述内容之后，下面我们来正式介绍 WeakMap。

## 三、WeakMap 简介

WeakMap 对象是一组键/值对的集合，其中的键是弱引用的。WeakMap 的 key 只能是 Object 类型。 原始数据类型是不能作为 key 的（比如 Symbol）。

### 3.1 语法

```js
new WeakMap([iterable])
```
`iterable`：是一个数组（二元数组）或者其他可迭代的且其元素是键值对的对象。每个键值对会被加到新的 WeakMap 里。null 会被当做 undefined。

### 3.2 属性

`length`：属性的值为 0；
`prototype`：`WeakMap` 构造器的原型。 允许添加属性到所有的 WeakMap 对象。

### 3.3 方法

`WeakMap.prototype.delete(key)`：移除 key 的关联对象。执行后 `WeakMap.prototype.has(key)` 返回false。
`WeakMap.prototype.get(key)`：返回 key 关联对象，或者 undefined（没有 key 关联对象时）。
`WeakMap.prototype.has(key)`：根据是否有 key 关联对象返回一个布尔值。
`WeakMap.prototype.set(key value)`：在 WeakMap 中设置一组 key 关联对象，返回这个 WeakMap 对象。

### 3.4 示例

```js
const wm1 = new WeakMap(),
      wm2 = new WeakMap(),
      wm3 = new WeakMap();
const o1 = {},
      o2 = function(){},
      o3 = window;

wm1.set(o1, 37);
wm1.set(o2, "azerty");
wm2.set(o1, o2); // value可以是任意值,包括一个对象或一个函数
wm2.set(o3, undefined);
wm2.set(wm1, wm2); // 键和值可以是任意对象,甚至另外一个WeakMap对象

wm1.get(o2); // "azerty"
wm2.get(o2); // undefined,wm2中没有o2这个键
wm2.get(o3); // undefined,值就是undefined

wm1.has(o2); // true
wm2.has(o2); // false
wm2.has(o3); // true (即使值是undefined)

wm3.set(o1, 37);
wm3.get(o1); // 37

wm1.has(o1);   // true
wm1.delete(o1);
wm1.has(o1);   // false
```

介绍完 WeakMap 相关的基础知识，下面我们来介绍一下 WeakMap 的应用。

## 四、WeakMap 应用

### 4.1 通过 WeakMap 缓存计算结果

使用 WeakMap，你可以将先前计算的结果与对象相关联，而不必担心内存管理。以下功能 countOwnKeys() 是一个示例：它将以前的结果缓存在 WeakMap 中 cache。

```js
const cache = new WeakMap();

function countOwnKeys(obj) {
  if (cache.has(obj)) {
    return [cache.get(obj), 'cached'];
  } else {
    const count = Object.keys(obj).length;
    cache.set(obj, count);
    return [count, 'computed'];
  }
}
```
创建完 countOwnKeys 方法，我们来具体测试一下：

```js
let obj = { name: "kakuqo", age: 30 };
console.log(countOwnKeys(obj));
// [2, 'computed']
console.log(countOwnKeys(obj));
// [2, 'cached']
obj = null; // 当对象不在使用时，设置为null
```

### 4.2 在 WeakMap 中保留私有数据

在以下代码中，WeakMap _counter 和 _action 用于存储以下实例的虚拟属性的值：

```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  
  dec() {
    let counter = _counter.get(this);
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}
```

创建完 Countdown 类，我们来具体测试一下：

```js
let invoked = false;

const countDown = new Countdown(3, () => invoked = true);
countDown.dec();
countDown.dec();
countDown.dec();

console.log(`invoked status: ${invoked}`)
```
说到类的私有属性，我们不得提一下 ECMAScript Private Fields。

## 五、总结

本文主要介绍了 JavaScript 中 WeakMap 的作用和应用场景，其实除了 WeakMap 之外，还有一个 WeakSet，只要将对象添加到 WeakMap 或 WeakSet 中，GC 在触发条件时就可以将其占用内存回收。

但实际上 JavaScript 的 WeakMap 并不是真正意义上的弱引用：其实只要键仍然存活，它就强引用其内容。WeakMap 仅在键被垃圾回收之后，才弱引用它的内容。为了提供真正的弱引用，TC39 提出了 WeakRefs 提案。

WeakRef 是一个更高级的 API，它提供了真正的弱引用，并在对象的生命周期中插入了一个窗口。同时它也可以解决 WeakMap 仅支持 object 类型作为 Key 的问题。

参考：
MDN - WeakMap： https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap
exploringjs - ch_weakmaps： https://exploringjs.com/impatient-js/ch_weakmaps.html
typescriptlang - ecmascript-private-fields： https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-8.html#ecmascript-private-fields
what-are-the-actual-uses-of-es6-weakmap： https://stackoverflow.com/questions/29413222/what-are-the-actual-uses-of-es6-weakmap
JavaScript垃圾回收：https://lz5z.com/JavaScript%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6/
What’s New in JavaScript：https://zhuanlan.zhihu.com/p/65409226
简单了解 JavaScript 垃圾回收机制：https://codertw.com/%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80/682114/
javascript.info - weakmap-weakset：https://javascript.info/weakmap-weakset
为什么 JavaScript 的私有属性使用 # 符号： https://zhuanlan.zhihu.com/p/47166400
weakmap: http://www.semlinker.com/you-dont-know-weakmap/#%E4%BA%8C%E3%80%81%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81-WeakMap


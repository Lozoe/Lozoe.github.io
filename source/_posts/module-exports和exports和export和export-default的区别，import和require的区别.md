---
title: module.exports和exports和export和export default的区别，import和require的区别
date: 2019-07-19 18:38:20
categories: [Javascript]
tags:
---

在es5中，用module.exports和exports导出模块，用require引入模块。
es6新增export和export default导出模块，import导入模块。

## module.exports和exports & require

在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象， 而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域。

```js
exports = module.exports = {};
//utils.js
let a = 100;
console.log(module.exports); //能打印出结果为：{}
console.log(exports); //能打印出结果为：{}
exports.a = 200; //这里辛苦劳作帮 module.exports的内容给改成 {a : 200}
exports = '指向其他内存区'; //这里把exports的指向指走
```

<!-- more -->
```js
//test.js
var a = require('/utils');
console.log(a) // 打印为 {a : 200}
```

从上面可以看出，其实require导出的内容是module.exports的指向的内存块内容，并不是exports的。 简而言之，区分他们之间的区别就是 exports 只是 module.exports的引用，辅助后者添加内容用的。

用白话讲就是，exports只辅助module.exports操作内存中的数据，辛辛苦苦各种操作数据完，累得要死，结果到最后真正被require出去的内容还是module.exports的，真是好苦逼啊。

其实大家用内存块的概念去理解，就会很清楚了。

然后呢，为了避免糊涂，尽量都用 module.exports 导出，然后用require导入。

## export和export default & import

### export 和 export default

首先我们讲这两个导出，下面我们讲讲它们的区别

- export与export default均可用于导出常量、函数、文件、模块等
- 在一个文件或模块中，export、import可以有多个，export default仅有一个
- 通过export方式导出，在导入时要加{ }，export default则不需要
- export能直接导出变量表达式，export default不行。

export能直接导出变量表达式，export default不行。

//testEs6Export.js

```js
'use strict'
// 导出变量
export const a = '100';
// 导出方法
export const dogSay = function() {
    console.log('wang wang');
}
// 导出方法第二种
function catSay() {
   console.log('miao miao');
}
export { catSay };

//export default导出
const m = 100;
export default m;
//export defult const m = 100;// 这里不能写这种格式。
```

//index.js

```js
'use strict'
var express = require('express');
var router = express.Router();
import { dogSay, catSay } from './testEs6Export'; // 导出了 export方法
import m from './testEs6Export';  // 导出了 export default
import * as testModule from'./testEs6Export';//把所有 export集合到 testModule 对象导出

/* GET home page. */
router.get('/',function(req, res, next) {
  dogSay();
  catSay();
  console.log(m);
  testModule.dogSay();
  console.log(testModule.m);// undefined ,因为 as导出是把零散的 export聚集在一起作为一个对象，而export default是导出为 default属性。
  console.log(testModule.default);// 100
  res.send('恭喜你，成功验证');
});
module.exports = router;
```

export default 和 export 可以同时存在一个 js 文件里。引用的时候如果不用 {}，接收到的就是 export default 的值。引用的时候如果用 {}，接收到的就是 export 的值。

从上面可以看出，确实感觉 ES6的模块系统非常灵活的。

在有 import 的情况下，直接 export {} 也能成立，但导出的形式只能是下列格式。

```js
import name from './a';
export {
    name
}
```

import 文件的另一种形式：

```js
import('./test').then(res => {
    console.log(res)
})
```

## 其他

除了在语法上的区别，在编译加载上es5和es6的模式也是有区别的。

历史上，JavaScript 一直没有模块（module）体系，无法将一个大程序拆分成互相依赖的小文件，再用简单的方法拼装起来。其他语言都有这项功能，比如 Ruby 的require、Python 的import，甚至就连 CSS 都有@import，但是 JavaScript 任何这方面的支持都没有，这对开发大型的、复杂的项目形成了巨大障碍。

在 ES6 之前，社区制定了一些模块加载方案，最主要的有 CommonJS 和 AMD 两种。前者用于服务器，后者用于浏览器。ES6 在语言标准的层面上，实现了模块功能，而且实现得相当简单，完全可以取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西。比如，CommonJS 模块就是对象，输入时必须查找对象属性。

```js
// CommonJS模块
let { stat, exists, readFile } = require('fs');

// 等同于
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

上面代码的实质是整体加载fs模块（即加载fs的所有方法），生成一个对象（_fs），然后再从这个对象上面读取 3 个方法。这种加载称为“运行时加载”，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。

```js
// ES6模块
import { stat, exists, readFile } from 'fs';
```

上面代码的实质是从fs模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。当然，这也导致了没法引用 ES6 模块本身，因为它不是对象。

由于 ES6 模块是编译时加载，使得静态分析成为可能。有了它，就能进一步拓宽 JavaScript 的语法，比如引入宏（macro）和类型检验（type system）这些只能靠静态分析实现的功能。

除了静态加载带来的各种好处，ES6 模块还有以下好处。

- 不再需要UMD模块格式了，将来服务器和浏览器都会支持 ES6 模块格式。目前，通过各种工具库，其实已经做到了这一点。
- 将来浏览器的新 API 就能用模块格式提供，不再必须做成全局变量或者navigator对象的属性。
- 不再需要对象作为命名空间（比如Math对象），未来这些功能可以通过模块提供。

参考： http://es6.ruanyifeng.com/#docs/module#export-%E5%91%BD%E4%BB%A4

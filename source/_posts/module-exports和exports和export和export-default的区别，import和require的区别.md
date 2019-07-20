---
title: module.exports和exports和export和export default的区别，import和require的区别
date: 2019-07-19 18:38:20
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

---
title: module.exports和exports和export和export default的区别，import和require的区别
date: 2019-07-19 18:38:20
categories: [JavaScript]
tags:
---

在es5中，用module.exports和exports导出模块，用require引入模块。
es6新增export和export default导出模块，import导入模块。

## CommonJS规范之module.exports和exports & require

### 模块定义和使用

在 Commonjs 中，一个文件就是一个模块。在一个node执行一个文件时，会给这个文件内生成一个 exports和module对象， 定义一个模块导出通过 exports 或者 module.exports 挂载即可，而module又有一个exports属性。他们之间的关系如下图，都指向一块{}内存区域。

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

CommonJS 的模块主要由原生模块 module 来实现，这个类上的一些属性对我们理解模块机制有很大帮助。

```js
Module {
  id: '.', // 如果是 mainModule id 固定为 '.'，如果不是则为模块绝对路径
  exports: {}, // 模块最终 exports
  filename: '/absolute/path/to/entry.js', // 当前模块的绝对路径
  loaded: false, // 模块是否已加载完毕
  children: [], // 被该模块引用的模块
  parent: '', // 第一个引用该模块的模块
  paths: [ // 模块的搜索路径
   '/absolute/path/to/node_modules',
   '/absolute/path/node_modules',
   '/absolute/node_modules',
   '/node_modules'
  ]
}
```

### require 从哪里来

在编写 CommonJS 模块的时候，我们会使用 require 来加载模块，使用 exports 来做模块输出，还有 module，__filename, __dirname 这些变量，为什么它们不需要引入就能使用？

原因是 Node 在解析 JS 模块时，会先按文本读取内容，然后将模块内容进行包裹，在外层裹了一个 function，传入变量。再通过 vm.runInThisContext 将字符串转成 Function形成作用域，避免全局污染。

```js
let wrap = function(script) {
  return Module.wrapper[0] + script + Module.wrapper[1];
};

const wrapper = [
  '(function (exports, require, module, __filename, __dirname) { ',
  '\n});'
];
```

于是在 CommmonJS 的模块中可以不需要 require，直接访问到这些方法，变量。

参数中的 module 是当前模块的的 module 实例（尽管这个时候模块代码还没编译执行），exports 是 module.exports 的别名，最终被 require 的时候是输出 module.exports 的值。require 最终调用的也是 Module._load 方法。__filename，__dirname 则分别是当前模块在系统中的绝对路径和当前文件夹路径。

### 模块的查找过程

开发者在使用 require 时非常简单，但实际上为了兼顾各种写法，不同类型的模块，node_modules packages 等模块的查找过程稍微有点麻烦。

首先，在创建模块对象时，会有 paths 属性，其值是由当前文件路径计算得到的，从当前目录一直到系统根目录的 node_modules。可以在模块中打印 module.paths 看看。

```js
[
    '/Users/evan/Desktop/demo/node_modules',
    '/Users/evan/Desktop/node_modules',
    '/Users/evan/node_modules',
    '/Users/node_modules',
    '/node_modules'
]
```

除此之外，还会查找全局路径（如果存在的话）

```js
[
    execPath/../../lib/node_modules, // 当前 node 执行文件相对路径下的 lib/node_modules
    NODE_PATH, // 全局变量 NODE_PATH
    HOME/.node_modules, // HOME 目录下的 .node_module
    HOME/.node_libraries // HOME 目录下的 .node-libraries
]
```

按照官方文档给出的[查找过程](https://nodejs.org/dist/latest-v12.x/docs/api/modules.html#modules_all_together)已经足够详细，这里只给出大概流程。

```js
从 Y 路径运行 require(X)

1. 如果 X 是内置模块（比如 require('http'）)
    a. 返回该模块。
    b. 不再继续执行。

2. 如果 X 是以 '/' 开头、
   a. 设置 Y 为 '/'

3. 如果 X 是以 './' 或 '/' 或 '../' 开头
    a. 依次尝试加载文件，如果找到则不再执行
        - (Y + X)
        - (Y + X).js
        - (Y + X).json
        - (Y + X).node
    b. 依次尝试加载目录，如果找到则不再执行
        - (Y + X + package.json 中的 main 字段).js
        - (Y + X + package.json 中的 main 字段).json
        - (Y + X + package.json 中的 main 字段).node
    c. 抛出 'not found'
4. 遍历 module paths 查找，如果找到则不再执行
5. 抛出 'not found'
```

### 模块加载相关

#### MainModule

当运行 `node index.js` 时，Node 调用 Module 类上的静态方法 `_load(process.argv[1])` 加载这个模块，并标记为主模块，赋值给 process.mainModule 和 require.main，可以通过这两个字段判断当前模块是主模块还是被 require 进来的。

CommonJS 规范是在代码运行时同步阻塞性地加载模块，在执行代码过程中遇到 require(X) 时会停下来等待，直到新的模块加载完成之后再继续执行接下去的代码。

![mainModule](mainModule.png)

虽说是同步阻塞性，但这一步实际上非常快，和浏览器上阻塞性下载、解析、执行 js 文件不是一个级别，硬盘上读文件比网络请求快得多。

#### 缓存和循环引用

文件模块查找挺耗时的，如果每次 require 都需要重新遍历文件夹查找，性能会比较差；还有在实际开发中，模块可能包含副作用代码，例如在模块顶层执行 addEventListener ，如果 require 过程中被重复执行多次可能会出现问题。

CommonJS 中的缓存可以解决重复查找和重复执行的问题。模块加载过程中会以模块绝对路径为 key, module 对象为 value 写入 cache。在读取模块的时候会优先判断是否已在缓存中，如果在，直接返回 module.exports；如果不在，则会进入模块查找的流程，找到模块之后再写入 cache。

模块缓存可以打印 require.cache 进行查看。

```js
{
    '/Users/evan/Desktop/demo/main.js':
        Module {
            id: '.',
            exports: {},
            parent: null,
            filename: '/Users/evan/Desktop/demo/main.js',
            loaded: false,
            children: [ [Object] ],
            paths:
            [ '/Users/evan/Desktop/demo/node_modules',
                '/Users/evan/Desktop/node_modules',
                '/Users/evan/node_modules',
                '/Users/node_modules',
                '/node_modules'
            ]
        },
    '/Users/evan/Desktop/demo/a.js':
        Module {
            id: '/Users/evan/Desktop/demo/a.js',
            exports: { foo: 1 },
            parent: 
            Module {
                id: '.',
                exports: {},
                parent: null,
                filename: '/Users/evan/Desktop/demo/main.js',
                loaded: false,
                children: [Array],
                paths: [Array] },
            filename: '/Users/evan/Desktop/demo/a.js',
            loaded: true,
            children: [],
            paths:
            [ '/Users/evan/Desktop/demo/node_modules',
                '/Users/evan/Desktop/node_modules',
                '/Users/evan/node_modules',
                '/Users/node_modules',
                '/node_modules' ] } }
```

缓存还解决了循环引用的问题。举个例子，现在有模块 a require 模块 b；而模块 b 又 require 了模块 a。

```js
// main.js
const a = require('./a');
console.log('in main, a.a1 = %j, a.a2 = %j', a.a1, a.a2);

// a.js
exports.a1 = true;
const b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.a2 = true;

// b.js
const a = require('./a.js');
console.log('in b, a.a1 = %j, a.a2 = %j', a.a1, a.a2);
```

程序执行结果如下：

```js
in b, a.a1 = true, a.a2 = undefined
in a, b.done = undefined
in main, a.a1 = true, a.a2 = true
```

实际上在模块 a 代码执行之前就已经创建了 Module 实例写入了缓存，此时代码还没执行，exports 是个空对象。

```js
{
    '/Users/evan/Desktop/module/a.js':
        Module {
            exports: {},
            //...
        }
}
```

代码 exports.a1 = true; 修改了 module.exports 上的 a1 为 true, 这时候 a2 代码还没执行。

```js
{
    '/Users/evan/Desktop/module/a.js':
        Module {
            exports: {
                a1: true
            },
            //...
        }
}
```

进入 b 模块，require a.js 时发现缓存上已经存在了，获取 a 模块上的 exports 。打印 a1, a2 分别是 true，和 undefined。

运行完 b 模块，继续执行 a 模块剩余的代码，exports.a2 = true; 又往 exports 对象上增加了 a2 属性，此时 module a 的 export 对象 a1, a2 均为 true。

```js
{
    '/Users/evan/Desktop/module/a.js':
        Module {
            exports: {
                a1: true,
                a2: true
            },
            //...
        }
}
```

再回到 main 模块，由于 require('./a.js') 得到的是 module a export 对象的引用，这时候打印 a1, a2 就都为 true。

### 小结

CommonJS 模块加载过程是同步阻塞性地加载，在模块代码被运行前就已经写入了 cache，同一个模块被多次 require 时只会执行一次，重复的 require 得到的是相同的 exports 引用。

## ES6 模块之export和export default & import

ES6 模块是前端开发同学更为熟悉的方式，使用 import, export 关键字来进行模块输入输出。ES6 不再是使用闭包和函数封装的方式进行模块化，而是从语法层面提供了模块化的功能。

ES6 模块中不存在 require, module.exports, __filename 等变量，CommonJS 中也不能使用 import。两种规范是不兼容的，一般来说平日里写的 ES6 模块代码最终都会经由 Babel, Typescript 等工具处理成 CommonJS 代码。

使用 Node 原生 ES6 模块需要将 js 文件后缀改成 mjs，或者 package.json "type" 字段改为 "module"，通过这种形式告知 Node 使用 ES Module 的形式加载模块。

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

### ES6 模块 加载过程

ES6 模块的加载过程分为三步：

#### 1. 查找，下载，解析，构建所有模块实例

ES6 模块会在程序开始前先根据模块关系查找到所有模块，生成一个无环关系图，并将所有模块实例都创建好，这种方式天然地避免了循环引用的问题，当然也有模块加载缓存，重复 import 同一个模块，只会执行一次代码。

#### 2. 在内存中腾出空间给即将 export 的内容（此时尚未写入 export value）。然后使 import 和 export 指向内存中的这些空间，这个过程也叫连接

这一步完成的工作是 living binding import export，借助下面的例子来帮助理解。

```js
// counter.js
let count = 1;

function increment () {
  count++;
}

module.exports = {
  count,
  increment
}

// main.js
const counter = require('counter.cjs');

counter.increment();
console.log(counter.count); // 1
```

上面 CommonJS 的例子执行结果很好理解，修改 count++ 修改的是模块内的基础数据类型变量，不会改变 exports.count，所以打印结果认为 1。

```js
// counter.mjs
export let count = 1;

export function increment () {
  count++;
}

// main.mjs
import { increment, count } from './counter.mjs'

increment();
console.log(count); // 2
```

从结果上看使用 ES6 模块的写法，当 export 的变量被修改时，会影响 import 的结果。这个功能的实现就是 living binding，具体规范底层如何实现可以暂时不管，但是知道 living binding 比网上文章描述为 "ES6 模块输出的是值的引用" 更好理解。

更接近 ES6 模块的 CommonJS 代码可以是下面这样：

```js
exports.counter = 1;

exports.increment = function () {
    exports.counter++;
}
```

#### 3. 运行模块代码将变量的实际值填写在第二步生成的空间中

到第三步，会基于第一步生成的无环图进行深度优先后遍历填值，如果这个过程中访问了尚未初始化完成的空间，会抛出异常。

```js
// a.mjs
export const a1 = true;
import * as b from './b.mjs';
export const a2 = true;

// b.mjs
import { a1, a2 } from './a.mjs'
console.log(a1, a2);
```

上面的例子会在运行时抛出异常 ReferenceError: Cannot access 'a1' before initialization。如果改成 import * as a from 'a.mjs' 可以看到 a 模块中 export 的对象已经占好坑了。

```js
// b.mjs
import * as a from './a.mjs'
console.log(a);
```

将输出 `{ a1: <uninitialized>, a2: <uninitialized> }`可以看出，ES6 模块为 export 的变量预留了空间，不过尚未赋值。这里和 CommonJS 不一样，CommonJS 到这里是知道 a1 为 true, a2 为 undefined

除此之外，我们还能推导出一些 ES6 模块和 CommonJS 的差异点：

- CommonJS 可以在运行时使用变量进行 require, 例如 require(path.join('xxxx', 'xxx.js'))，而静态 import 语法（还有动态 import，返回 Promise）不行，因为 ES6 模块会先解析所有模块再执行代码。

![变量引入](变量引入.png)

- require 会将完整的 exports 对象引入，import 可以只 import 部分必要的内容，这也是为什么使用 Tree Shaking 时必须使用 ES6 模块 的写法。
- import 另一个模块没有 export 的变量，在代码执行前就会报错，而 CommonJS 是在模块运行时才报错。

### 为什么平时开发可以混写

使用转换工具处理 ES6 模块的时候，常看到打包之后出现 __esModule 属性，字面意思就是将其标记为 ES6 Module。这个变量存在的作用是为了方便在引用模块的时候加以处理。

例如 ES6 模块中的 `export default` 在转化成 CommonJS 时会被挂载到 `exports['default']` 上，当运行 `require('./a.js')` 时  是不能直接读取到 `default` 上的值的，为了和 ES6 中 `import a from './a.js'` 的行为一致，会基于 `__esModule` 判断处理。

```js
// a.js
export default 1;

// main.js
import a from './a';

console.log(a);
```

转化后

```JS
// a.js
Object.defineProperty(exports, "__esModule", {
  value: true
});
exports.default = 1;
```

```js
// main.js
'use strict';

var _a = require('./a');

var _a2 = _interopRequireDefault(_a);

function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }

console.log(_a2.default);
```

`a` 模块 `export defualt` 会被转换成 `exports.default = 1;`，这也是平时前端项目开发中使用 `require` 为什么还常常需要 `.default` 才能取到目标值的原因。
接着当运行 `import a from './a.js'` 时，`es module` 预期的是返回 `export` 的内容。工具会将代码转换为 `_interopRequireDefault` 包裹，在里面判断是否为 esModule，是的话直接返回，如果是 commonjs 模块的话则包裹一层 `{default: obj}`，最后获取 a 的值时，也会被装换成 _a1.default。

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

参考:

http://es6.ruanyifeng.com/#docs/module#export-%E5%91%BD%E4%BB%A4
https://juejin.im/post/5e5f10176fb9a07cd443c1e2

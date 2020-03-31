---
title: webpack优化
date: 2020-03-31 17:46:24
categories: [Tool]
---


## webpack优化的最终目的

快 + 小 + 按需加载 + 利用浏览器缓存

一个是速度、一个是体积、一个是用到什么加载什么、公用代码可以利用浏览器缓存

## 一、 减少webpack打包的时间

### 1. 优化 Loader 配置

对于 Loader 来说，影响打包效率首当其冲必属 Babel 了。因为 Babel 会将代码转为字符串生成 AST，然后对 AST 继续进行转变最后再生成新的代码，项目越大，转换代码越多，效率就越低。当然了，我们是有办法优化的。

首先我们可以优化 Loader 的文件搜索范围（Loader处理的文件路径，与不处理的文件路径指定。以及作用到什么类型的文件）

```js
module.exports = {
    module: {
        rules: [
        {
            // js 文件才使用 babel
            test: /\.js$/,
            loader: 'babel-loader',
            // 只在 src 文件夹下查找
            include: [resolve('src')],
            // 不会去查找的路径
            exclude: /node_modules/
        }
        ]
    }
}
```

对于 Babel 来说，我们肯定是希望只作用在 JS 代码上的，然后 node_modules 中使用的代码都是编译过的，所以我们也完全没有必要再去处理一遍。

当然这样做还不够，我们还可以将 Babel 编译过的文件缓存起来，下次只需要编译更改过的代码文件即可，这样可以大幅度加快打包时间

```js
loader: 'babel-loader?cacheDirectory=true'
```

总结起来loader的优化点：

- 比如babel-loader查找的路径，与不查找的路径指定
- 缓存babel编译过的文件

### 2. HappyPack开启多线程打包

受限于 Node 是单线程运行的，所以 Webpack 在打包的过程中也是单线程的，特别是在执行 Loader 的时候，长时间编译的任务很多，这样就会导致等待的情况。

HappyPack 可以将 Loader 的同步执行转换为并行的，这样就能充分利用系统资源来加快打包效率了

```js
module: {
    loaders: [
        {
            test: /\.js$/,
            include: [resolve('src')],
            exclude: /node_modules/,
            // id 后面的内容对应下面
            loader: 'happypack/loader?id=happybabel'
        }
    ]
},
plugins: [
    new HappyPack({
        id: 'happybabel',
        loaders: ['babel-loader?cacheDirectory'],
        // 开启 4 个线程
        threads: 4
    })
]
```

通过此插件开启多线程打包，充分利用系统资源

### 3. DllPlugin （提前打包指定的类库）

DllPlugin 可以将特定的类库提前打包然后引入。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案。

```js
// 单独配置在一个文件中
// webpack.dll.conf.js
const path = require('path')
const webpack = require('webpack')
module.exports = {
    entry: {
        // 想统一打包的类库
        vendor: ['react']
    },
    output: {
        path: path.join(__dirname, 'dist'),
        filename: '[name].dll.js',
        library: '[name]-[hash]'
    },
    plugins: [
        new webpack.DllPlugin({
            // name 必须和 output.library 一致
            name: '[name]-[hash]',
            // 该属性需要与 DllReferencePlugin 中一致
            context: __dirname,
            path: path.join(__dirname, 'dist', '[name]-manifest.json')
        })
    ]
}
```

然后我们需要执行这个配置文件生成依赖文件，接下来我们需要使用 DllReferencePlugin 将依赖文件引入项目中

```js
// webpack.conf.js
module.exports = {
    // ...省略其他配置
    plugins: [
        new webpack.DllReferencePlugin({
            context: __dirname,
            // manifest 就是之前打包出来的 json 文件
            manifest: require('./dist/vendor-manifest.json'),
        })
    ]
}
```

总结：

- 将特订的类库提前打包然后引入，减少打包类库的次数，加快打包速度
- 同时将公共代码抽离成单独的文件

### 4. 代码并行压缩

在 Webpack3 中，我们一般使用 UglifyJS 来压缩代码，但是这个是单线程运行的，为了加快效率，我们可以使用 webpack-parallel-uglify-plugin 来并行运行 UglifyJS，从而提高效率。

在 Webpack4 中，我们就不需要以上这些操作了，只需要将 mode 设置为 production 就可以默认开启以上功能。代码压缩也是我们必做的性能优化方案，当然我们不止可以压缩 JS 代码，还可以压缩 HTML、CSS 代码，并且在压缩 JS 代码的过程中，我们还可以通过配置实现比如删除 console.log 这类代码的功能。

总结：

- webpack4 生产模式，默认开启多个子进程（线程）由之前的一个一个压缩，变为并行压缩

### 5. 其他

- resolve.extensions：用来表明文件后缀列表，默认查找顺序是 ['.js', '.json']，如果你的导入文件没有添加后缀就会按照这个顺序查找文件。我们应该尽可能减少后缀列表长度，然后将出现频率高的后缀排在前面
- resolve.alias：可以通过别名的方式来映射一个路径，能让 Webpack 更快找到路径
- module.noParse：如果你确定一个文件下没有其他依赖，就可以使用该属性让 Webpack 不扫描该文件，这种方式对于大型的类库很有帮助

## 二、减少 Webpack 打包后的文件体积

### 1. Scope Hoisting

Scope Hoisting 会分析出模块之间的依赖关系，尽可能的把打包出来的模块合并到一个函数中去。代码体积更小，因为函数申明语句会产生大量代码，代码在运行时因为创建的函数作用域更少了，内存开销也随之变小

由于scope hositing 需要分析出模块之间的依赖关系，因此源码必须使用ES6模块化语句，不然就不能生效，原因和tree shaking一样

比如我们希望打包两个文件

```js
// test.js
export const a = 1
// index.js
import { a } from './test.js'
```

对于这种情况，我们打包出来的代码会类似这样

```js
[
    /* 0 */
    function (module, exports, require) {
        //...
    },
    /* 1 */
    function (module, exports, require) {
        //...
    }
]
```

但是如果我们使用 Scope Hoisting 的话，代码就会尽可能的合并到一个函数中去，也就变成了这样的类似代码

```js
[
    /* 0 */
    function (module, exports, require) {
        //...
    }
]
```

这样的打包方式生成的代码明显比之前的少多了。如果在 Webpack4 中你希望开启这个功能，只需要启用 `optimization.concatenateModules` 就可以了。

```js
module.exports = {
    optimization: {
        concatenateModules: true
    }
}
```

webpack3需要使用ModuleConcatenationPlugin

```js
new webpack.optimize.ModuleConcatenationPlugin()
```

总之，尽可能的合并打包出来的模块到一个函数中去

### 2. Tree Shaking （webpack 4 生产环境 默认开启）

tree shaking是一个术语、通常用于打包时移除js中未引用的代码(dead-code)，它依赖于ES6模块系统中的import 和 export 的***静态结构特性***

Tree Shaking 可以实现删除项目中未被引用的代码，比如

```js
// test.js
export const a = 1
export const b = 2
// index.js
import { a } from './test.js'
```

对于以上情况，test 文件中的变量 b 如果没有在项目中使用到的话，就不会被打包到文件中。

如果使用 Webpack 4 的话，开启生产环境就会自动启动这个优化功能。

## 三、按需加载(懒加载) + 动态加载 + 异步加载

当页面中一个文件过大并且还不一定用到的时候，我们希望在使用到的时候才开始加载这个文件俗称按需加载

这样可以减少页面的响应时间，提高访问速度。

webpack按需加载有 `require.ensure`和 `import`两种方法。

### require.ensure 按需异步加载

CommonJS解决方案， webpack 特有的，已经被 import() 取代

```js
require.ensure(
    dependencies: String[],
    callback: function(require),
    errorCallback: function(error),
    chunkName: String
)
```

给定 dependencies 参数，将其对应的文件拆分到一个单独的 bundle 中，此 bundle 会被异步加载。当使用 CommonJS 模块语法时，这是动态加载依赖的唯一方法。意味着，可以在模块执行时才运行代码，只有在满足某些条件时才加载依赖项。

### import() 动态地加载模块

调用 import() 之处，被作为分离的模块起点，意思是，被请求的模块和它引用的所有子模块，会分离到一个单独的 chunk 中。

> ES2015 loader 规范 定义了 import() 方法，可以在运行时动态地加载 ES2015 模块。

eg1. 当点击按钮时 再加载lazy.js 输出里面内容

```js
// lazy.js
export default 'lazy loader';

// main.js
let output = () => {
    import('./lazy').then(module => {
        console.log(module.default);
    });
};
ReactDOM.render(
    <div>
    <button onClick={output}>点击</button>
    </div>,
    document.querySelector('#root')
)
```

eg2. vue中的懒加载

```js
const Login = () => import(/* webpackChunkName: "login" */'./login')

new VueRouter({
  routes: [
    { path: '/login', component: Login }
  ]
})
```

ps： require.ensure 和 import() 这个特性依赖于内置的 Promise。如果想在低版本浏览器使用 require.ensure，记得使用像 es6-promise 或者 promise-polyfill 这样 polyfill 库，来预先填充(shim) Promise 环境。

## 利用浏览器缓存

### 提取公共代码与第三方代码

将多个入口重复加载的公共资源提取出来

- 相同的资源被重复的加载，浪费用户的流量和服务器的成本；
- 每个页面需要加载的资源太大，导致网页首屏加载缓慢，影响用户体验。如果能把公共代码抽离成单独文件进行加载能进行优化，可以减少网络传输流量，降低服务器成本

```js
optimization: {
    splitChunks: {
        // 默认打包node_modules到venders.js
        chunks: 'all'
    },
    // 将webpack运行时生成代码打包到runtime.js
    runtimeChunk: true
},
```

在webpack4.0 optimization.splitChunks替代了CommonsChunkPlugin

## 总结

### 优化角度

- 打包速度
- 产物体积
- 按需加载、异步加载、动态加载
- 利用浏览器缓存
- 开发流程规范和体验

#### 打包速度

- 优化Loader配置 - 1）查找路径include exclude 2）缓存babel编译过的文件babel-loader?cacheDirectory=true
- HappyPack开启多线程打包
- DllPlugin提前打包指定的类库 webpack.dll.conf.js DllReferencePlugin，减少打包类库的次数，同时将公共代码抽离成单独的文件
- 代码并行压缩 - webpack3 UglifyJS压缩混淆代码，webpack-parallel-uglify-plugin并行运行UglifyJS；Webpack4 mode = production时自动开启
- 压缩html、css
- resolve.extensions ['.js', '.json']尽可能减少后缀列表长度,出现频率高的后缀排在前面
- resolve.alias 别名映射路径,Webpack 不扫描该文件
- module.noParse

#### 产物体积

- Scope Hoisting 尽可能的合并打包出来的模块到一个函数中去 optimization.concatenateModules ModuleConcatenationPlugin
- js/css Tree Shaking webpack 4prod js 默认开启。css 用PurifyCssPlugin
- 文件处理 & base64 & css-sprites
- 代码压缩 UglifyJS

#### 按需加载、异步加载、动态加载

require.ensure、import 路由懒加载

#### 利用浏览器缓存做长缓存优化

提取公共代码与第三方代码做长缓存优化

#### 开发流程规范和体验

- 模块热更新
- sourceMap
- webpack-dev-server-proxy 接口代理
- ts
- BundleAnalyzerPlugin分析打包情况

参考：
https://juejin.im/post/5ac76a8f51882555677ecc06#heading-5

https://juejin.im/post/5dcc45136fb9a02b450c2818

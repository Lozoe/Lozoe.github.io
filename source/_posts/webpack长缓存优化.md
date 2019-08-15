---
title: webpack长缓存优化
date: 2019-07-30 20:55:54
tags:
  - webpack
  - 性能优化
---

## 什么是长缓存优化，webpack如何做到

浏览器在用户访问页面的时候，为了加快加载速度，会对用户访问的静态资源进行存储，但是每一次代码升级或者更新，都需要浏览器去下载新的代码，最方便和简单的更新方式就是引入新的文件名称。

在webpack中，可以在output给输出的文件指定chunkhash,并且分离经常更新的代码和框架代码，通过NamedModulesPlugin和NamedChunksPlugin使再次打包的文件名字不变。

<!-- more -->
## 循序渐进

### 场景一： 改变app（业务代码）时，vendor变化

```js
// foo.js
import react from 'react'
console.log('This is foo.js~!')
```

```js
// webpack.config.js
var path = require('path')
var Webpack = require('webpack')
module.exports = {
    entry: {
        main: './src/foo'
    },

    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[hash].js'
    }
}
```

打包产物：

![result1](result1.png)

然后改变代码：

```js
// foo.js
import react from 'react'
console.log('This is foo.js~!!')
```

再看产物：

![result2](result2.png)

对于一个项目来说，第三方库一般是不会变化的，如何做到改变app，vendor版本不变化？

其实提取vendor即可。

改变配置如下：

```js
// webpack.config.js
var path = require('path')
var Webpack = require('webpack')
module.exports = {
    entry: {
        main: './src/foo',
        vendor: ['react']
    },

    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[hash].js'
    }
}
```

产物：

![result3](result3.png)

变化代码
```js
// foo.js
import react from 'react'
console.log('This is foo.js~!!!')
```

再看产物：

![result4](result4.png)

诶。。。怎么还是变化了,大小差不多

别忘了配置plugin: commonChunkPlugin把公用的代码提取出来

```js
// webpack.config.js
var path = require('path')
var Webpack = require('webpack')
module.exports = {
    entry: {
        main: './src/foo',
        vendor: ['react']
    },

    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[hash].js'
    },
    plugins:[
        new Webpack.optimize.CommonsChunkPlugin({
            name: ['vendor'],
            minChunks: Infinity
        }),
    ]
}
```

打包前后变化：

![result5](result5.png)

二者版本一样，此时使用chunkHash试试

```js
// webpack.config.js
var path = require('path')
var Webpack = require('webpack')
module.exports = {
    entry: {
        main: './src/foo',
        vendor: ['react']
    },

    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].[chunkHash].js'
    },

    plugins:[
        new Webpack.optimize.CommonsChunkPlugin({
            name: ['vendor'],
            minChunks: Infinity
        })
    ]
}
```

对比：
![result6](result6.png)

此时还差一步：提取runtime 

```js
new Webpack.optimize.CommonsChunkPlugin({
    name: 'manifest'
})
```

![result7](result7.png)

至此 场景一解决

### 场景二： 引入新的模块，模块顺序变化，vendor hash发生变化

```js
// foo.js
import react from 'react'
import module from './module'
console.log('This is foo.js~!')
```

```js
// module.js
export default 'This is my module.'
```

对比图

![result8](result8.png)

解决方案：

NamedChunksPlugin  chunkId => name
NamedModulesPlugin  moduleId => moduleName

```js
new Webpack.NamedChunksPlugin()
```

![result9](result9.png)

隐藏的modules也来给指定一个名字：

```js
new Webpack.NamedModulesPlugin()
```

![result10](result10.png)

场景二顺利解决！！！

### 场景三： 动态引入，hash变化

```js
import('./async').then(function(a) {
    console.log(a)
});
```

![result11](result11.png)

解决方案动态模块给定模块名称： 

```js
import(/* webpackChunkName: 'async' */'./async').then(function(a) {
    console.log(a)
});
```

![result12](result12.png)

ok，场景三也搞定了~

### 最后五步走

1、独立打包vendor、commonChunksPlugin、hash=>chunkHash
2、抽取出manifest，inline到html
3、NamedChunksPlugin  chunkId => name
4、NamedModulesPlugin  moduleId => moduleName
5、动态模块给定模块名称

最终配置拿走不谢

```js
var path = require('path')
var Webpack = require('webpack')
module.exports = {
    entry: {
        main: './src/foo',
        vendor: ['react']
    },

    output: {
        path: path.resolve(__dirname, 'dist'),
        // filename: '[name].[hash].js'
        filename: '[name].[chunkHash].js'
    },

    plugins:[
        new Webpack.NamedChunksPlugin(),

        new Webpack.NamedModulesPlugin(),

        new Webpack.optimize.CommonsChunkPlugin({
            name: ['vendor'],
            minChunks: Infinity
        }),

        new Webpack.optimize.CommonsChunkPlugin({
            name: 'manifest'
        })
    ]
}
```

```js
import(/* webpackChunkName: 'async' */'./async').then(function(a) {
    console.log(a)
})
```

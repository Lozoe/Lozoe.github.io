---
title: vue-router
date: 2019-08-14 20:44:54
tags:
  - vue
---

## vue-router是什么

这里的路由并不是指我们平时所说的硬件路由器，**这里的路由就是SPA（单页应用）的路径管理器**。再通俗的说，vue-router就是WebApp的链接路径管理系统。
vue-router是Vue.js官方的路由插件，它和vue.js是深度集成的，适合用于构建单页面应用。vue的单页面应用是基于路由和组件的，路由用于设定访问路径，并将路径和组件映射起来。传统的页面应用，是用一些超链接来实现页面切换和跳转的。在vue-router单页面应用中，则是路径之间的切换，也就是组件的切换。**路由模块的本质 就是建立起url和页面之间的映射关系。**
<!-- more -->
至于我们为啥不能用a标签，这是因为用Vue做的都是单页应用（当你的项目准备打包时，运行 npm run build时，就会生成dist文件夹，这里面只有静态资源和一个index.html页面），所以你写的标签是不起作用的，你必须使用vue-router来进行管理。

## vue-router实现原理

SPA(single page application):单一页面应用程序，只有一个完整的页面；它在加载页面时，不会加载整个页面，而是只更新某个指定的容器中内容。单页面应用(SPA)的核心之一是: 更新视图而不重新请求页面;vue-router在实现单页面前端路由时，提供了两种方式：**Hash模式和History模式**；根据mode参数来决定采用哪一种方式。

### 1、Hash模式

vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。hash（#）是URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说hash 出现在 URL 中，但不会被包含在 http 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面；同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置；所以说**Hash模式通过锚点值的改变，根据不同的值，渲染指定DOM位置的不同数据**。hash 模式的原理是 onhashchange 事件(监测hash值变化)，可以在 window 对象上监听这个事件。

### 2、History模式

由于hash模式会在url中自带#，如果不想要很丑的 hash，我们可以用路由的 history 模式，只需要在配置路由规则时，加入"mode: 'history'",这种模式充分利用了html5 history interface 中新增的 pushState() 和 replaceState() 方法。这两个方法应用于浏览器记录栈，在当前已有的 back、forward、go 基础之上，它们提供了对历史记录修改的功能。只是当它们执行修改时，虽然改变了当前的 URL ，但浏览器不会立即向后端发送请求。

```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
});
```

当你使用 history 模式时，URL 就像正常的 url，例如 http://yoursite.com/user/id，也好看！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 http://oursite.com/user/id 就会返回 404，这就不好看了。

所以呢，**你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。**

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

注意：
这么做以后，你的服务器就不再返回 404 错误页面，因为对于所有路径都会返回 index.html 文件。为了避免这种情况，你应该在 Vue 应用里面覆盖所有的路由情况，然后在给出一个 404 页面。后端只管返回index.html,js去判断跳哪个组件页面,以及是否404.
```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    // 后端old入口页面
    { 
        name: 'a',
        path: '/myproject/a.do', 
        component: aPage, 
        meta: { isEnter: true } 
    }, 
    // 本地入口页面
    { 
        name: 'b',
        path: '/myproject/b.html', 
        component: bPage, 
        meta: { isEnter: true } 
    }, 
    { 
        path: '*', 
        component: NotFoundComponent
    }
  ]
})
```

### 3、使用路由模块来实现页面跳转的方式

方式1：直接修改地址栏

方式2：this.$router.push(‘路由地址’)

方式3： <router-linkto="路由地址"></router-link>

## vue-router使用

1、下载 `npm i vue-router --save`

2、在main.js中引入 `import VueRouter from 'vue-router'`;

3、安装插件 `Vue.use(VueRouter)`;

4、创建路由对象并配置路由规则

```js
let router = new VueRouter({
    routes: [{
        path: '/home',
        component: Home
    }]
});
```

5、将其路由对象传递给Vue的实例，options中加入 `router:router`

6、在app.vue中留坑 `<router-view></router-view>`

使用：

```js
//main.js
// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)
import Vue from 'vue'
import VueRouter from 'vue-router'
Vue.use(VueRouter)

// 1. 定义 (路由) 组件。也可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是通过 Vue.extend() 创建的组件构造器，或者，只是一个组件配置对象。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
const router = new VueRouter({
  routes 
})

// 4. 创建和挂载根实例。记得要通过 router 配置参数注入路由，从而让整个应用都有路由功能
 
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

最后别忘记在app.vue中留坑。

## vue-router参数传递

声明式的导航 `<router-link:to="...">`和编程式的导航 router.push(...)都可以传参，本文主要介绍前者的传参方法，同样的规则也适用于编程式的导航。

### 1、用name传递参数

在路由文件src/router/index.js里配置name属性

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      name: 'user',
      component: User
    }
  ]
})
```

模板里(src/App.vue)用 `$route.name`来接收
比如： `<p>{{ $route.name}}</p>`

### 2、通过 <router-link> 标签中的to传参

这种传参方法的基本语法：

```js
<router-link :to="{name:xxx,params:{key:value}}">valueString</router-link>
```

比如先在src/App.vue文件中

```js
<router-link :to="{name: 'hi1', params:  {username: 'jspang', id: '555'}}">Hi页面1</router-link>
```

然后把src/router/index.js文件里给hi1配置的路由起个name,就叫hi1.

```js
{
    path: '/hi1',
    name: 'hi1',
    component: Hi1
}
```

最后在模板里(src/components/Hi1.vue)用 $route.params.username进行接收.

```js
{{$route.params.username}}-{{$route.params.id}}
```

### 3、利用url传递参数----在配置文件里以冒号的形式设置参数。
我们在/src/router/index.js文件里配置路由

```js
const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
```

User组件代码通过`$route.params.id`

```js
const User = {
  template: '<div>User {{ $route.params.id }}</div>'
}
```

在App.vue文件里加入我们的 `<router-view>`标签。这时候我们可以直接利用url传值了

```js
<router-link to="/params/198/jspang website is very good">params</router-link>
```
### 4、使用path来匹配路由，然后通过query来传递参数

```js
<router-link :to="{ name:'Query',query: { queryId:  status }}">router-link跳转Query</router-link>
```

对应路由配置：
```js
{

    path: '/query',
    name: 'Query',
    component: Query
}
```

于是我们可以获取参数：

```js
this.$route.query.queryId
```

## vue-router配置子路由(二级路由)

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL中各段动态路径也按某种结构对应嵌套的各层组件，例如：

```
/user/foo/profile                     /user/foo/posts
+------------------+                  +-----------------+
| User             |                  | User            |
| +--------------+ |                  | +-------------+ |
| | Profile      | |  +------------>  | | Posts       | |
| |              | |                  | |             | |
| +--------------+ |                  | +-------------+ |
+------------------+                  +-----------------+
```

借助 vue-router，使用嵌套路由配置，就可以很简单地表达这种关系。

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

要注意，以 / 开头的嵌套路径会被当作根路径。 这让你充分的使用嵌套组件而无须设置嵌套的路径。

## 单页面多路由区域操作

// app.vue
```html
<router-view></router-view>
<router-view name="left" style="float: left;width: 50%; height: 300px;background-color: #ccc;"></router-view>
<router-view name="right" style="float: left;width: 50%; height: 300px;background-color: #898;"></router-view>
```

// index.js
```js
import Vue from 'vue'
import Router from 'vue-router'
import Hello from '@/components/Hello'
import First1 from '@/components/first1'
import First2 from '@/components/first2'

Vue.use(Router)

export default new Router ({
  routes : [
    {
      path : '/',
      name : 'Hello',
      components : {
        default : Hello,
        left : First1,
        right : First2
      }
    }, {
      path : '/first',
      name : 'First',
      components : {
        default : Hello,
        left : First2,
        right : First1
      }      
    }
  ]
})
```

参考：https://router.vuejs.org/zh/
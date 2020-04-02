---
title: vue之keep-alive
date: 2020-04-02 10:37:36
categories: [Framework]
tags:
---

## 页面缓存

在Vue构建的单页面应用（SPA）中，路由模块一般使用vue-router。vue-router不保存被切换组件的状态，它进行push或者replace时，旧组件会被销毁，而新组件会被新建，走一遍完整的生命周期。

但有时候，我们有一些需求，比如跳转到详情页面时，需要保持列表页的滚动条的深度，等返回的时候依然在这个位置，这样可以提高用户体验。在Vue中，对于这种“页面缓存”的需求，我们可以使用keep-alive组件来解决这个需求。

## keep-alive是什么

keepalive 是 Vue 内置的一个组件，可以使被包含的组件保留状态，或避免重新渲染，将组件进行持久化。也就是所谓的组件缓存

## keep-alive使用方式

keep-alive是个抽象组件（或称为功能型组件），实际上不会被渲染在DOM树中。它的作用是在内存中缓存组件（不让组件销毁），等到下次再渲染的时候，还会保持其中的所有状态，并且会触发activated钩子函数。因为缓存的需要通常出现在页面切换时，所以常与router-view一起出现：

```js
<keep-alive>
    <router-view />
</keep-alive>
```

如此一来，每一个在router-view中渲染的组件，都会被缓存起来。

如果只想渲染某一些页面/组件，可以使用keep-alive组件的include/exclude属性。include属性表示要缓存的组件名（即组件定义时的name属性），接收的类型为string、RegExp或string数组；exclude属性有着相反的作用，匹配到的组件不会被缓存。exclude优先级更高。假如可能出现在同一router-view的N个页面中，我只想缓存列表页和详情页，那么可以这样写：

```html
<keep-alive :include="['ListView', 'DetailView']">
    <router-view />
</keep-alive>
```

被keepalive包含的组件不会被再次初始化，也就意味着不会重走生命周期函数。但是有时候是希望我们缓存的组件可以能够再次进行渲染，这时 Vue 为我们解决了这个问题 被包含在 keep-alive 中创建的组件，会多出两个生命周期的钩子: activated 与 deactivated：

- activated 当 keepalive 包含的组件再次渲染的时候触发
- deactivated 当 keepalive 包含的组件销毁的时候触发

keepalive是一个抽象的组件，缓存的组件不会被 mounted,为此提供activated和deactivated钩子函数

## 实现条件缓存：全局的include数组

上面include的写法不是常用的，因为它固定了哪几个页面缓存或不缓存，假如有下面这个场景：

- 现有页面：首页（A）、列表页（B）、详情页（C），一般可以从：A->B->C；
- B到C再返回B时，B要保持列表滚动的距离；
- B返回A再进入B时，B不需要保持状态，是全新的

很明显，这个例子中，B是“条件缓存”的，C->B时保持缓存，A->B时放弃缓存。其实解决方案也不难，只需要将B动态地从include数组中增加/删除就行了。具体步骤是：

1.在Vuex中定义一个全局的缓存数组，待传给include：

```js
// global.js

export default {
    namespaced: true,
    state: {
        keepAliveComponents: [] // 缓存数组
    },
    mutations: {
        keepAlive (state, component) {
            // 注：防止重复添加（当然也可以使用Set）
            !state.keepAliveComponents.includes(component) && state.keepAliveComponents.push(component)
        },
        noKeepAlive (state, component) {
            const index = state.keepAliveComponents.indexOf(component)
            index !== -1 && state.keepAliveComponents.splice(index, 1)
        }
    }
}
```

2.在父页面中定义keep-alive，并传入全局的缓存数组：

```html
// App.vue
<div class="app">
    <!--传入include数组-->
    <keep-alive :include="keepAliveComponents">
        <router-view></router-view>
    </keep-alive>
</div>

export default {
    computed: {
        ...mapState({
            keepAliveComponents: state => state.global.keepAliveComponents
        })
    }
}
```

3.缓存：在路由配置页中，约定使用meta属性keepAlive，值为true表示组件需要缓存。在全局路由钩子beforeEach中对该属性进行处理，这样一来，每次进入该组件，都进行缓存：

```js
const router = new Router({
    routes: [
        {
            path: '/A/B',
            name: 'B',
            component: B,
            meta: {
                title: 'B页面',
                keepAlive: true // 这里指定B组件的缓存性
            }
        }
    ]
})

router.beforeEach((to, from, next) => {
    // 在路由全局钩子beforeEach中，根据keepAlive属性，统一设置页面的缓存性
    // 作用是每次进入该组件，就将它缓存
    if (to.meta.keepAlive) {
        store.commit('global/keepAlive', to.name)
    }
})
```

4.取消缓存的时机：对缓存组件使用路由的组件层钩子beforeRouteLeave。因为B->A->B时不需要缓存B，所以可以认为：当B的下一个页面不是C时取消B的缓存，那么下次进入B组件时B就是全新的：

```js
export default {
    name: 'B',
    created () {
        // ...设置滚动条在最顶部
    },
    beforeRouteLeave (to, from, next) {
        // 如果下一个页面不是详情页（C），则取消列表页（B）的缓存
        if (to.name !== 'C') {
            this.$store.commit('global/noKeepAlive', from.name)
        }
        next()
    }
}
```

因为B的条件缓存，是B自己的职责，所以最好把该业务逻辑写在B的内部，而不是A中，这样不至于让组件之间的跳转关系变得混乱。

5.一个需要注意的细节：因为keep-alive组件的include数组操作的对象是组件名、而不是路由名，因此我们定义每一个组件时，都要显式声明name属性，否则缓存不起作用。而且，一个显式的name对Vue devtools有提示作用。



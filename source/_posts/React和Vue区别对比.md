---
title: React和Vue区别对比
date: 2020-03-24 17:17:38
categories: [Framework]
---

## React和Vue区别

1）设计思想
    vue的官网中说它是一款渐进式框架，采用自底向上增量开发的设计。
    react主张函数式编程，所以推崇纯组件，数据不可变，单向数据流，当然需要双向的地方也可以手动实现，
    比如借助 onChange 和 setState 来实现一个双向的数据流。
2）编写语法
    Vue推荐的做法是webpack+vue-loader的单文件组件格式，vue保留了html、css、js分离的写法
    React的开发者可能知道，react是没有模板的，直接就是一个渲染函数，它中间返回的就是一个虚拟DOM树，
    React推荐的做法是  JSX + inline style, 也就是把HTML和CSS全都写进JavaScript了,即'all in  js'。
3）构建工具
    vue提供了CLI 脚手架，可以帮助你非常容易地构建项目。
    React 在这方面也提供了create-react-app，但是现在还存在一些局限性，不能配置等等
4）数据绑定（状态管理和对象属性）
    vue是实现了双向数据绑定的mvvm框架，数据驱动页面的正向过程和页面变化映射到data的反向过程，state对象并不是必须的，数据由data属性在Vue对象中进行管理。
    react是单向数据流，react中属性是不允许更改的，状态是允许更改的。
    react中组件不允许通过this.state这种方式直接更改组件的状态。自身设置的状态，可以通过setState来进行更改。
    (注意：React中setState是异步的，导致获取dom可能拿的还是之前的内容，
    所以我们需要在setState第二个参数（回调函数）中获取更新后的新的内容。)
5）diff算法
    vue中diff算法实现流程：
        1在内存中构建虚拟dom树
        2将内存中虚拟dom树渲染成真实dom结构
        3数据改变的时候，将之前的虚拟dom树结合新的数据生成新的虚拟dom树
        4将此次生成好的虚拟dom树和上一次的虚拟dom树进行一次比对(diff算法进行比对)，来更新只需要被替换的DOM，而不是全部重绘。在Diff算法中，只平层的比较前后两棵DOM树的节点，没有进行深度的遍历。
        5会将对比出来的差异进行重新渲染
    react中diff算法实现流程:
        DOM结构发生改变-----直接卸载并重新create
        DOM结构一样-----不会卸载,但是会update变化的内容
        所有同一层级的子节点.他们都可以通过key来区分-----同时遵循1.2两点
        (其实这个key的存在与否只会影响diff算法的复杂度,换言之,你不加key的情况下,diff算法就会以暴力的方式去根据一二的策略更新,但是你加了key,diff算法会引入一些另外的操作)
React会逐个对节点进行更新，转换到目标节点。而最后插入新的节点，涉及到的DOM操作非常多。diff总共就是移动、删除、增加三个操作，而如果给每个节点唯一的标识（key），那么React优先采用移动的方式，能够找到正确的位置去插入新的节点。
vue会跟踪每一个组件的依赖关系，不需要重新渲染整个组件树。而对于React而言,每当应用的状态被改变时,全部组件都会重新渲染,当然react通过shouldComponentUpdate这个生命周期函数方法来进行控制。
6）其他：
维护：Vue主要是由尤雨溪个人维护，React由如Facebook这类大公司维护
社区活跃度： vue star: 160k react star: 146k
开发者工具：vue-devtols react-dev-tools
原生支持：React Native能在手机上创建原生应用，React在这方面处于领先位置。使用JavaScript, CSS和HTML创建原生移动应用，这是一个重要的革新。Vue社区与阿里合作开发Vue版的React Native——Weex也很不错，但仍处于开发状态且并没经过实际项目的验证。

## 相似之处

组件化: Vue单文件组件,React是JS + JSX
Props: 父组件往子组件传送数据
构建工具: vue-cli create-react-app
开发调试: vue-dev-tools react-dev-tools
周边生态: 路由、状态管理
vue-router、vuex   react-router redux mobx
UI生态：
vue   -  pc（iview/elementUI）、h5（有赞vant/mintUI）、混合开发（weex）、微信小程序（iview weapp/zanui）
react - pc（ant design）、h5（ant design mobile）、混合开发（react-native-elements）、微信小程序（taro ui）
使用场景：web app混合 微信小程序(mpvue/taro) 结合electron开发桌面程序

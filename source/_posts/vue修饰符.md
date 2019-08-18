---
title: vue修饰符
date: 2019-08-18 20:23:10
tags:
---

## 分类

事件修饰符、按键修饰符、键值修饰符

## 事件修饰符

.stop ：阻止事件冒泡,是阻止冒泡行为,不让当前元素的事件继续往外触发,如阻止点击div内部事件,触发div事件
.prevent ：阻止默认事件，提交事件不再重载页面
.capture ：添加事件侦听器时使用事件捕获模式 （若有多个该修饰符的元素则由外而内触发）
.self ：当事件在该元素本身（而不是子元素）触发时触发回调(即直接作用在该元素上的事件才可以执行)
.once ：事件只能触发一次
.native 我们经常会写很多的小组件，有些小组件可能会绑定一些事件，但是，像下面这样绑定事件是不会触发的
<!-- more -->
```js
<My-component @click="shout(3)"></My-component>
```

必须使用.native来修饰这个click事件（即`<My-component @click.native="shout(3)"></My-component>`），可以理解为该修饰符的作用就是把一个vue组件转化为一个普通的HTML标签，注意：使用.native修饰符来操作普通HTML标签是会令事件失效的

### @click.prevent.self和@click.self.prevent区别

@click.prevent.self 会阻止所有的点击
@click.self.prevent 只会阻止对元素自身的点击

例如：

```js
<div @click="alert(1)">
  <a href="/#" @click.prevent.self="alert(2)">
    <div @click="alert(3)"></div>
  </a>
</div>
```

点击div3，会alert3,alert1。不但阻止了alert(2)，还阻止了a的默认跳转。因为点击的时候会先prevent，阻止默认事件，阻止了跳转;然后判断是否是self，因为点击到的是div3，所以不是self，阻止了alert(2)。

```js
<div @click="alert(1)">
  <a href="/#" @click.self.prevent="alert(2)">
    <div @click="alert(3)"></div>
  </a>
</div>
```

点击div3，会alert3,alert1,跳转到/#。只阻止了alert(2)。
因为会先判断self，点击到div3，不是self，所以不会执行click事件，就不会执行 阻止默认事件和alert(2) ，所以可以跳转

### .stop和.self的区别

.self只是阻止自身不执行冒泡触发,不会阻止冒泡继续向外部触发,.stop是从自身开始不向外部发射冒泡信号

## 按键修饰符

刚刚我们讲到这个click事件，我们一般是会用左键触发，有时候我们需要更改右键菜单啥的，就需要用到右键点击或者中间键点击，这个时候就要用到鼠标按钮修饰符

.left 左键点击
.right 右键点击
.middle 中键点击

待完善......

https://segmentfault.com/a/1190000016786254
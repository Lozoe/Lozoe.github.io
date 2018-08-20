---
title: oop
date: 2016-06-20 21:32:28
tags:
---


# 1、类的声明
```
/**
* 类的声明
*/
var Animal = function () {
    this.name = 'Animal';
};

/**
* es6中class的声明
*/
class Animal2 {
    constructor () {
        this.name = 'Animal2';
    }
}
```
# 2、实例化
```
/**
* 实例化
*/
console.log(new Animal(), new Animal2());
```
# 3、继承的方式
1）借助构造函数实现继承
```
/**
* 借助构造函数实现继承
*/
function Parent1 () {
  this.name = 'parent1';
}
Parent1.prototype.say = function () {

};
function Child1 () {
  Parent1.call(this); //apply
  this.type = 'child1';
}
console.log(new Child1(), new Child1().say());
```

缺点是父类原型链上的属性或者方法并没有被继承。只是实现了部分继承。

2)借助原型链实现继承
```
/**
* 借助原型链实现继承
*/
function Parent2 () {
  this.name = 'parent2';
  this.play = [1, 2, 3];
}
function Child2 () {
  this.type = 'child2';
}
Child2.prototype = new Parent2();

// 缺点
var s1 = new Child2();
var s2 = new Child2();
console.log(s1.play, s2.play);
s1.play.push(4); // 改变一个对象的时候另外的对象也随之改变。
```
缺点：改变一个对象的时候其他对象的属性或者方法也会随之改变，原因是因为原型链的原型对象他俩是共用的。`s1.__proto__ === s2.__proto__` true

3)组合方式继承
```
/**
* 组合方式
*/
function Parent3 () {
  this.name = 'parent3';
  this.play = [1, 2, 3];
}
function Child3 () {
  Parent3.call(this);
  this.type = 'child3';
}
Child3.prototype = new Parent3();
var s3 = new Child3();
var s4 = new Child3();
s3.play.push(4);
console.log(s3.play, s4.play);
console.log(s3.constructor); // Parent3
```

- 父级构造函数执行了两次，事实没有必要
- 子类构造函数的prototype直接拿的是父类的实例，他没有自己的constructor,子类构造函数的constructor是从父类实例中继承的，也就是原型链的上一级拿过来的，他拿过的constructor是Parent3的constructor
```
/**
* 组合继承的优化1
* @type {String}
*/
function Parent4 () {
  this.name = 'parent4';
  this.play = [1, 2, 3];
}
function Child4 () {
  Parent4.call(this);
  this.type = 'child4';
}
Child4.prototype = Parent4.prototype;
var s5 = new Child4();
var s6 = new Child4();
console.log(s5, s6);

console.log(s5 instanceof Child4, s5 instanceof Parent4); // true true
console.log(s5.constructor); // Parent4
```
缺点：

构造函数是一个，无法区分实例是有子类还是父类构造函数生成。

子类构造函数的prototype直接拿的是父类的实例，他没有自己的constructor,子类构造函数的constructor是从父类实例中继承的，也就是原型链的上一级拿过来的，他拿过的constructor是Parent3的constructor
```
/**
* 组合继承的优化2
*/
function Parent5 () {
  this.name = 'parent5';
  this.play = [1, 2, 3];
}
function Child5 () {
  Parent5.call(this);
  this.type = 'child5';
}
Child5.prototype = Object.create(Parent5.prototype);
```

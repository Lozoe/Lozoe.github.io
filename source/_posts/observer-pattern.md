---
title: 观察者模式
date: 2017-09-16 20:00:11
categories: [Javascript]
tags:
  - 设计模式
---

最常用的模式之一，在很多语言里面都得到大量的应用，包括平时我们接触的DOM事件是js和DOM之间的一种观察者模式，很好的实现了模块之间的解耦
<!-- more -->
```js
/*对ES5的forEach迭代兼容处理*/
if (!Array.prototype.forEach) {
    Array.prototype.forEach = function (fn, thisObj) {
        var scope = thisObj || window;
        for (var i = 0, j = this.length; i < j; ++i) {
            fn.call(scope, this[i], i, this);
        }
    };
}
/*对ES5的filter迭代兼容处理*/
if (!Array.prototype.filter) {
    Array.prototype.filter = function (fn, thisObj) {
        var scope = thisObj || window;
        var arr = [];
        for (var i = 0, j = this.length; i < j; ++i) {
            if (!fn.call(scope, this[i], i, this)) {
                continue;
            }
            arr.push(this[i]);
        }
        return arr;
    };
}
/*此处利用js原型特性实现观察者模式*/
/*1、定义一个观察者对象，初始化对象所订阅的函数集合fns为空数组*/
function Observer() {
    this.fns = [];
}
/*1、在观察者对象的原型添加订阅，发布以及退订功能*/
Observer.prototype = {
    /*订阅功能，传入订阅的函数，push到当前观察者对象的函数集合*/
    subscribe: function(fn) {
        this.fns.push(fn);
    },         
    /*发布，传入发布的信息和发布对象*/
    publish: function(info, thisObj) {
        var scope = thisObj || window;
        this.fns.forEach(
            function(item,index,array) {
                item.call(scope, info);
            }
        );
    },
    /*退订功能，传入退订的函数，将之从当前对象的函数集合中remove掉*/
    unsubscribe: function(fn) {
        this.fns = this.fns.filter(
            function(item,index,array) {
                if (item !== fn) {
                    return item;
                }
            }
        );
    }
};
/*测试,创建闭包测试，避免全局变量污染*/
(function(){   
    /*创建一个观察者对象*/
    var obj = new Observer();
    /*创建订阅的函数f1*/
    var f1 = function(data) {
        console.log("I am the first subscriber.I need an "+data+".");
    };
    /*创建订阅的函数f2*/
    var f2 = function(data) {
        console.log("I am the second subscriber.I also need an "+data+".");
    };
    /*创建订阅的函数f3*/
    var f3 = function(data) {
        console.log("I am the third subscriber.I also need an "+data+".");
    };
    /*为观察者对象订阅f1函数*/
    obj.subscribe(f1);
    /*为观察者对象订阅f2函数*/
    obj.subscribe(f2);
    /*为观察者对象订阅f3函数*/
    obj.subscribe(f3);
    /*1、发布消息*/
    /*2、运行结果：I am the first subscriber.I need an apple.
                   I am the second subscriber.I also need an apple.
                   I am the third subscriber.I also need an apple.*/
    obj.publish('apple');
    /*1、退订f2*/
    obj.unsubscribe(f2);
    /*1、再来发布消息验证，失效*/
    /*2、运行结果：I am the first subscriber.I need an apple.
                   I am the third subscriber.I also need an apple.*/
    /*3、f2功能已经被退订，只能发布f1,f3的功能*/
    obj.publish('apple');
})();
```
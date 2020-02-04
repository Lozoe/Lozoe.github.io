---
title: reduce
date: 2018-09-27 15:34:11
categories: [JavaScript]
tags: reduce
---
经常用到的方法有push、join、indexOf、slice等等，但是有一个经常被我们忽略的方法：reduce，这个方法简直强大的不要不要的。

官方：reduce() 方法接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始缩减，最终为一个值。`arr.reduce(callback,[initialValue])`
<!-- more -->

其实reduce接收的就是一个回调函数，去调用数组里的每一项，直到数组结束。
```js
var total = [0,1,2,3,4].reduce((a, b)=>a + b); //10
```
回调函数接受四个参数：

1、上一次的值;

2、当前值;

3、当前值的索引;

4、数组;
```js
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, index, array) {
 console.log(index)
 return previousValue + currentValue
})  //10
```

分析一下这个结果，这个回调函数共调用了4次，因为第一次没有previousValue，所以直接从数组的第二项开始，一只调用到数组结束。

reduce还有第二个参数，我们可以把这个参数作为第一次调用callback时的第一个参数，上面这个例子因为没有第二个参数，所以直接从数组的第二项开始，如果我们给了第二个参数为5，那么结果就是这样的：
```js
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, index, array){
    console.log(index)
    return previousValue + currentValue
},5) //15
```

reduce可以帮助我们轻松的完成很多事，除了累加，还有扁平化一个二维数组：
```js
var flattened = [[0, 1], [2, 3], [4, 5]].reduce(function(a, b) {
 return a.concat(b);
}, []);
// flattened == [0, 1, 2, 3, 4, 5]
```




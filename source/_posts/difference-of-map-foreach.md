---
title: map和forEach区别
date: 2017-09-15 19:53:09
categories: [JavaScript]
tags:
---

# 一、原生js forEach()和map()遍历

## (一)共同点

1. 都是循环遍历数组中的每一项。
2. forEach() 和 map() 里面每一次执行匿名函数都支持3个参数：数组中的当前项item,当前项的索引index,原始数组arr。
3. 匿名函数中的this都是指Window。
4. 只能遍历数组。
<!-- more -->

## (二)区别

### 1. forEach()

没有返回值。内部如果操作原始数组，会改变原始数组。
```js
var arr = [1, 2, 3, 4, 5];  
var res = arr.forEach(function (item, index, originArr) {  
  originArr[index] = item * 10;  
})  
console.log(res);  // undefined;  
console.log(arr);  // [10, 20, 30, 40, 50]
```


### 2. map()
有返回值，可以return 出来。内部如果操作原始数组，会改变原始数组。
```js
var arr = [1, 2, 3, 4, 5];  
var res = arr.map(function (item, index, originArr) {  
    return item * 10;  
})  
console.log(res);  //[10, 20, 30, 40, 50]
console.log(arr);  //[1, 2, 3, 4, 5]
```
## (三)兼容性
IE6-8下都不兼容
[forEach](https://caniuse.com/#search=forEach)
[map](https://caniuse.com/#search=map)

```js
 
/** 
* forEach 遍历数组 
* @param callback [function] 回调函数； 
* @param context [object] 上下文； 
*/  
Array.prototype.forEach = function (callback,context){  
    context = context || window;  
    if('forEach' in Array.prototype) {  
        this.forEach(callback, context);  
        return;  
    }  
    // IE6-8下自己编写回调函数执行的逻辑  
    for(var i = 0, len = this.length; i < len; i++) {  
        callback && callback.call(context,this[i],i,this);  
    }  
}
 
/** 
* map遍历数组 
* @param callback [function] 回调函数； 
* @param context [object] 上下文； 
*/  
Array.prototype.map = function (callback,context){  
    context = context || window;  
    if('map' in Array.prototype) {  
        return this.map(callback,context);  
    }  
    // IE6-8下自己编写回调函数执行的逻辑  
    var newArr = [];  
    for(var i = 0, len = this.length; i < len; i++) {  
        if(typeof callback === 'function') {  
            var val = callback.call(context,this[i],i,this);  
            newArr[newArr.length] = val;  
        }  
    }  
    return newArr;  
}
```

# 二、jQuery $.each()和$.map()遍历

## 共同点：
即可遍历数组，又可遍历对象。

### 1. $.each()

没有返回值。$.each()里面的匿名函数支持2个参数：当前项的索引i，数组中的当前项n。如果遍历的是对象，k 是键，n 是值。
```js
$.each( ["a","b","c"], function(i, n){  
     console.log( i + ": " + n );  
})

$("span").each(function(i, n){  
     console.log( i + ": " + n );  
})

$.each( { name: "John", lang: "JS" }, function(k, n){  
     console.log( "Name: " + k + ", Value: " + n );  
});  
```

### 2. $.map()

有返回值，可以return 出来。$.map()里面的匿名函数支持2个参数和$.each()里的参数位置相反：数组中的当前项n，当前项的索引i。如果遍历的是对象，i 是值，n 是键。如果是$("span").map()形式，参数顺序和$.each()  $("span").each()一样。
```js
var arr = $.map( [0, 1, 2], function(n){  
     return n * 2;  
});
console.log(arr);  
 
$.map({"name":"Jim","age":17}, function(i, n){ 
     console.log(i + ":" + n);  
});
```
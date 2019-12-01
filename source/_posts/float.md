---
title: js精度问题解决方案
date: 2018-08-24 08:40:39
categories: [CSS]
tags:
---

作为一个一直做收银台技术业务的人，对这个问题也是耿耿于怀。本以为简单的toFixed可以解决一切，其实不然，翻看各大论坛，就这个话题给出一个完美解决方案。

### 先明白原因
<!-- more -->
##### 浮点数运算后的精度问题
```js
// 加法 =====================
0.1 + 0.2 = 0.30000000000000004
0.7 + 0.1 = 0.7999999999999999
0.2 + 0.4 = 0.6000000000000001

// 减法 =====================
1.5 - 1.2 = 0.30000000000000004
0.3 - 0.2 = 0.09999999999999998
 
// 乘法 =====================
19.9 * 100 = 1989.9999999999998
0.8 * 3 = 2.4000000000000004
35.41 * 100 = 3540.9999999999995

// 除法 =====================
0.3 / 0.1 = 2.9999999999999996
0.69 / 10 = 0.06899999999999999
```
##### toFixed不是万能

在遇到浮点数运算后出现的精度问题时，刚开始我是使用toFixed(2)来解决的，因为在W3school上明确写着定义：toFixed()方法可把Number四舍五入为指定小数位数的数字。

但是在chrome下测试结果不太令人满意：
```
1.35.toFixed(1) // 1.4 正确
1.335.toFixed(2) // 1.33  错误
1.3335.toFixed(3) // 1.333 错误
1.33335.toFixed(4) // 1.3334 正确
1.333335.toFixed(5)  // 1.33333 错误
1.3333335.toFixed(6) // 1.333333 错误
```

而IE确实正确的
```js
1.35.toFixed(1) // 1.4 正确
1.335.toFixed(2) // 1.34  正确
1.3335.toFixed(3) // 1.334 正确
1.33335.toFixed(4) // 1.3334 正确
1.333335.toFixed(5)  // 1.33334 正确
1.3333335.toFixed(6) // 1.333334 正确
```
果然IE才是爸爸。难道是浏览器兼容性问题？兼容性问题难道不应该是出在IE中吗？既然找到问题所在，就好下手

### 为什么会产生

让我们来看一下为什么0.1+0.2会等于0.30000000000000004，而不是0.3。首先，想要知道为什么会产生这样的问题，那就回忆一下计算机组成原理。虽然已经全部还给大学老师了，但是没关系，我们还有百度嘛。

### 浮点数的存储

和其它语言如Java和Python不同，JavaScript中所有数字包括整数和小数都只有一种类型 — Number。它的实现遵循 IEEE 754 标准，使用64位固定长度来表示，也就是标准的 double 双精度浮点数（相关的还有float 32位单精度）。


这样的存储结构优点是可以归一化处理整数和小数，节省存储空间。

64位比特又可分为三个部分：
- 符号位S：第 1 位是正负数符号位（sign），0代表正数，1代表负数

- 指数位E：中间的 11 位存储指数（exponent），用来表示次方数

- 尾数位M：最后的 52 位是尾数（mantissa），超出的部分自动进一舍零

### 浮点数的运算

那么JavaScript在计算0.1+0.2时到底发生了什么呢？
首先，十进制的0.1和0.2会被转换成二进制的，但是由于浮点数用二进制表示时是无穷的：
```js
0.1 -> 0.0001 1001 1001 1001...(1100循环)
0.2 -> 0.0011 0011 0011 0011...(0011循环)
```
IEEE 754 标准的 64 位双精度浮点数的小数部分最多支持53位二进制位，所以两者相加之后得到二进制为：
```js
0.0100110011001100110011001100110011001100110011001100 
```
```js
 /*** method **
 *  add / subtract / multiply /divide
 * floatObj.add(0.1, 0.2) >> 0.3
 * floatObj.multiply(19.9, 100) >> 1990
 *
 */
var floatObj = function() {

    /*
     * 判断obj是否为一个整数
     */
    function isInteger(obj) {
        return Math.floor(obj) === obj
    }

    /*
     * 将一个浮点数转成整数，返回整数和倍数。如 3.14 >> 314，倍数是 100
     * @param floatNum {number} 小数
     * @return {object}
     *   {times:100, num: 314}
     */
    function toInteger(floatNum) {
        var ret = {times: 1, num: 0}
        if (isInteger(floatNum)) {
            ret.num = floatNum
            return ret
        }
        var strfi  = floatNum + ''
        var dotPos = strfi.indexOf('.')
        var len    = strfi.substr(dotPos+1).length
        var times  = Math.pow(10, len)
        var intNum = Number(floatNum.toString().replace('.',''))
        ret.times  = times
        ret.num    = intNum
        return ret
    }

    /*
     * 核心方法，实现加减乘除运算，确保不丢失精度
     * 思路：把小数放大为整数（乘），进行算术运算，再缩小为小数（除）
     *
     * @param a {number} 运算数1
     * @param b {number} 运算数2
     * @param digits {number} 精度，保留的小数点数，比如 2, 即保留为两位小数
     * @param op {string} 运算类型，有加减乘除（add/subtract/multiply/divide）
     *
     */
    function operation(a, b, digits, op) {
        var o1 = toInteger(a)
        var o2 = toInteger(b)
        var n1 = o1.num
        var n2 = o2.num
        var t1 = o1.times
        var t2 = o2.times
        var max = t1 > t2 ? t1 : t2
        var result = null
        switch (op) {
            case 'add':
                if (t1 === t2) { // 两个小数位数相同
                    result = n1 + n2
                } else if (t1 > t2) { // o1 小数位 大于 o2
                    result = n1 + n2 * (t1 / t2)
                } else { // o1 小数位 小于 o2
                    result = n1 * (t2 / t1) + n2
                }
                return result / max
            case 'subtract':
                if (t1 === t2) {
                    result = n1 - n2
                } else if (t1 > t2) {
                    result = n1 - n2 * (t1 / t2)
                } else {
                    result = n1 * (t2 / t1) - n2
                }
                return result / max
            case 'multiply':
                result = (n1 * n2) / (t1 * t2)
                return result
            case 'divide':
                result = (n1 / n2) * (t2 / t1)
                return result
        }
    }

    // 加减乘除的四个接口
    function add(a, b, digits) {
        return operation(a, b, digits, 'add')
    }
    function subtract(a, b, digits) {
        return operation(a, b, digits, 'subtract')
    }
    function multiply(a, b, digits) {
        return operation(a, b, digits, 'multiply')
    }
    function divide(a, b, digits) {
        return operation(a, b, digits, 'divide')
    }

    // exports
    return {
        add: add,
        subtract: subtract,
        multiply: multiply,
        divide: divide
    }
}();
```

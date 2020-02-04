---
title: js函数式编程
date: 2017-08-10 21:51:21
categories: [JavaScript]
---

### 前言

在聊函数式编程时先聊聊编程范式，函数式编程是一种编程范式，常见的编程范式有命令式(过程化)编程（Imperative programming），面向对象编程，函数式编程等等。编程范式不关注如何用写程序来解决问题，而是关注各种不同的设计程序的思考方式。是一个抽象的概念。

##### 命令式编程：
变量（对应着存储单元），赋值语句（获取，存储指令），表达式（内存引用和算术运算）和控制语句（跳转指令），一句话，命令式程序就是一个冯诺依曼机的指令序列 。java、c、c++、rubby、php等都属此类。关心解决问题的步骤

<!-- more -->

##### 面向对象编程：
类、对象、封装、继承、多态等等。java、c++
##### 函数式编程：
是面向数学的抽象，将计算描述为一种表达式求值 是将程序描述为表达式和变换，以数学方程的形式建立模型，并且尽量避免可变的状态，关心数据的映射。haskell、erlang、lisp、javascript
Imperative programming & Functional programming
![区别](difference-between-ip-and-fun.png)

OOP programming & Functional programming
![区别](difference-between-oop-and-fun.png)


### JavaScript函数式编程？

那么JavaScript函数式编程是什么？

JavaScript 函数式编程是一个存在了很久的话题，但似乎从 2016 年开始，它变得越来越火热。这可能是因为 ES6 语法对于函数式编程更为友好，也可能是因为诸如 RxJS (ReactiveX) 等函数式框架的流行。
        
函数式编程（Functional programming），将计算机运算视为数学上的函数计算，并且避免状态以及可变数据，是一种声名式编程，通过表达式和声明而非语句完成编程，功能代码里，函数的输出值只取决于输入的参数，因此使用同一值作为参数x的值调用f两次，f(x)每次输出结果都一样，无副作用。

鄙见：以函数作为主要载体的编程方式，用函数去拆解、抽象一般的表达式


面向对象对数据进行抽象，将行为以对象方法的方式封装到数据实体内部，从而降低系统的耦合度。而函数式编程，选择对过程进行抽象，将数据以输入输出流的方式封装进过程内部，从而也降低系统的耦合度。两者虽是截然不同，然而在系统设计的目标上可以说是殊途同归的。二者不矛盾。
庞大的系统，可能既要对数据进行抽象，又要对过程进行抽象，或者一个局部适合进行数据抽象，另一个局部适合进行过程抽象
![抽象](common-target.png)

### JS函数式编程
1. 一等公民函数
2. 纯函数
3. 柯里化
4. 自执行函数、匿名函数、闭包特性
5. 函数组合
6. 高阶函数
7. 方法链
8. 递归

#### 一等公民函数
先看这样一段代码
```js
// 一般写法
const arr = ["apple", "pen", "apple-pen"];
for (const i of arr) {
  const c = arr[i][0];
  arr[i] = c.toUpperCase() + arr[i].slice(1);
}

console.log(arr); //["Apple", "Pen", "Apple-pen"]
```

明显缺点：
1. 表意不明显，逐渐变得难以维护
2. 复用性差，会产生更多的代码量
3. 会产生很多中间变量

下面改进一下：
```js
// 函数式写法一
function upperFirst(word) {
  return word[0].toUpperCase() + word.slice(1);
}

function wordToUpperCase(arr) {
  return arr.map(upperFirst);
}

wordToUpperCase(["apple", "pen", "apple-pen"]);

// 函数式写法二
arr.map(
  ["apple", "pen", "apple-pen"],
  word => word[0].toUpperCase() + word.slice(1)
);
```

从而：
1. 利用了函数封装性将功能做拆解（粒度不唯一）
2. 封装为不同的函数，
3. 高阶函数，Array.map 代替 for…of 做数组遍历，减少了中间变量和操作
4. 表意清晰，易于维护、复用以及扩展。


#### 一等公民-函数

一等公民，函数可以去任何值可以去的地方
```js
/** 函数是一等公民 **/
const ten = () => 10; //变量值

const tens = [10, () => 10]; //数组元素

const tenObj = { number: 10, fun: () => 10 }; //对象成员

10 + (() => 10)(); //按需创建

const add = (number, fun) => number + fun(); //作为函数参数
add(10, () => 11); //21

//作为函数返回值
const currying = (fn, ...rest) => {
	return (...arguments) => {
		let newArgs = rest.concat([].slice.call(arguments));
		return fn.apply(null, newArgs);
	}
}
```

#### 纯函数
直接看代码
```js
/* 纯函数 */
PI = 3.14;

function areaOfACircle(radius) {
  return PI * sqr(radius);
}

areaOfACircle(3);
//=> 28.26

//网页加载一段js 或者开发人员修改了外部变量

//... some code

PI = 'xxx'
areaOfACircle(3);
// NaN
```
![纯函数](no-side-effect.png)
特点
1. 结果只能从参数值来计算
2. 不能依赖能被外部操作改变的数据
3. 不能改变的外部状态

#### 柯里化

柯里化（Currying），又称部分求值（Partial Evaluation），是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

先看看下面这段：
```js
function add(a){
    var sum = 0;
        sum += a;
    return function(b){
        sum += b;
        return function(c) {
            sum += c;
            return sum;
        }
    }
}
var result = add(1)(2)(3); //6
```

实际上是符合柯里化的概念的但是，如果对调用的次数不加限制，比如 四次，那上面的代码就不行了。
那该怎么办呢？
```js
var curring = function(fn) {
	var _args = [];
	return function cb(){
		if(arguments.length === 0) {
			return fn.apply(this, _args);
		}
		Array.prototype.push.apply(_args, [].slice.call(arguments));
		return cb;
	}
}
var multi = function(){
	var total = 0;
	var argsArray = Array.prototype.slice.call(arguments);
		argsArray.forEach(function(item){
			total += item;
		})
	return total
};
var calc = curring(multi);
calc(1,2)(3)(4,5,6);
console.log(calc()); //空白调用时才真正计算
```
柯里化：
```
const curry = (fn, ...rest) => {
	return (...arguments) => {
		let innerArgs = Array.prototype.slice.call(arguments);
		let finalArgs = rest.concat(innerArgs);
		return fn.apply(null, finalArgs);
	}
}

//功能函数
const add = (num1, num2) => num1 + num2 
curriedAddFive = curry(add, 5)
curriedAddFive(3)
```

###### 为什么柯里化？

函数柯里化允许和鼓励开发者分隔复杂功能变成更小更容易分析的部分。这些小的逻辑单元显然是更容易理解和测试的，然后你的应用就会变成干净而整洁的组合，由一些小单元组成的组合。

1. 提高通用性
```js
function square(i) {
    return i * i;
}
function double(i) {
    return i *= 2;
}
function map(handeler, list) {
    return list.map(handeler);
}

// 数组的每一项平方
map(square, [1, 2, 3, 4, 5]);
map(square, [6, 7, 8, 9, 10]);
map(square, [10, 20, 30, 40, 50]);
// ......

// 数组的每一项加倍
map(double, [1, 2, 3, 4, 5]);
map(double, [6, 7, 8, 9, 10]);
map(double, [10, 20, 30, 40, 50]); 
```

柯里化后：
```js
const square = i => i * i
const double = i => i *= 2
const map = (handeler, list) => list.map(handeler)
const currying = (fn, ...rest) => (...args) => fn.apply(null, rest.concat([].slice.call(args)))

const mapSquare = currying(map, square);
mapSquare([1, 2, 3, 4, 5]); //[1, 4, 9, 16, 25]

const mapDouble = currying(map, double);
mapDouble([1, 2, 3, 4, 5]); //[2, 4, 6, 8, 10]
```

2. 延迟执行

柯里化的另一个应用场景是延迟执行。不断的柯里化，累积传入的参数，最后执行。

3. 固定易变因素

柯里化特性决定了应用场景。提前把易变因素，传参固定下来，生成一个更明确的应用函数。典例：bind（用以固定this这个易变对象）

```js
Function.prototype.bind = function(context) {
    var _this = this,
    _args = Array.prototype.slice.call(arguments, 1);
    return function() {
        return _this.apply(context, _args.concat(Array.prototype.slice.call(arguments)));
    }
}

/* 
Function.prototype.bind 方法也是柯里化应用与 call/apply 方法直接执行不同，bind 方法 将第一个参数设置为函数执行的上下文，其他参数依次传递给调用方法（函数的主体本身不执行，可以看成是延迟执行），并动态创建返回一个新的函数， 这符合柯里化特点
*/
// 调用
var foo = {
	x: 666
};
var bar = function () {
	console.log(this.x);
}.bind(foo); // 绑定
console.log(bar); // Function (bind后返回一个延迟执行的新函数)
bar(); //666
```
通常，使用柯里化会有一些开销。取决于你正在做的是什么，可能会或不会，以明显的方式影响你。也就是说，几乎大多数情况，你的代码的拥有性能瓶颈首先来自其他原因，而不是这个。
有关性能，这里有一些事情必须牢记于心：

1. 存取arguments对象通常要比存取命名参数要慢一点.
2. 一些老版本的浏览器在arguments.length的实现上是相当慢的.
3. 创建大量嵌套作用域和闭包函数会带来花销，无论是在内存还是速度上.

#### 自执行函数、匿名函数、闭包特性
```js
const pingpong = (() => {
	let PRIVATE = 0
	return {
		increase(n) {
			return PRIVATE += n
		},
		decrease(n) {
			return PRIVATE -= n
		}
	}
})()
```

上面代码有以下特点：
1. 防止变量污染
2. 外部无法访问私有变量，无法进行修改，只有通过指定的in de 方法来操作私有变量，保护私有变量
3. 使用了匿名函数，代码更加简洁易懂，节省中间变量命名

#### 函数组合
组合（compose）本质即是饲养函数。利用已知函数结合，产下崭新的函数。

eg1.
```js
const compose = (f,g) => x => f(g(x))

const toUpperCase = (x) => x.toUpperCase()
const exclaim = (x) => (x + '!')
const shout = compose(exclaim, toUpperCase);

shout("send in the clowns");
//=> "SEND IN THE CLOWNS!"
```

eg2.

```js
const compose = (f, g) => x => f(g(x))

const head = x => x[0]; //取第一个元素
const reverse = arr => arr.reduce((acc, x) => [x].concat(acc), []); //反转
const toUpperCase = x => x.toUpperCase(); //大写

//以往的方式，直接嵌套调用
toUpperCase(head(reverse(['jumpkick', 'roundhouse', 'uppercut']))) //"UPPERCUT"

//组合调用
const last = compose(head, reverse); //取最后一个元素
const lastUpper = compose(toUpperCase, compose(head, reverse));
// or const lastUpper = compose(toUpperCase, last);
const lastUpperAnother = compose(compose(toUpperCase, head), reverse);

last(['jumpkick', 'roundhouse', 'uppercut']); //"uppercut"
lastUpper(['jumpkick', 'roundhouse', 'uppercut']); //"UPPERCUT"
lastUpperAnother(['jumpkick', 'roundhouse', 'uppercut']); //"UPPERCUT"
```

结合律：

函数组合优点：

- 利用了函数封装性将功能做拆解
- 增强函数复用性和可维护性
- 代码实现一目了然，逻辑清晰
- 数据流从右向左，可读性高于嵌套一堆的函数调用
![区别](association.png)

一起来实现一个通用的组合函数

```js
const head = x => x[0]; // 取第一个元素
const reverse = arr => arr.reduce((acc, x) => [x].concat(acc), []); // 反转
const toUpperCase = x => x.toUpperCase(); // 大写
const removeSpace = x => x.replace(/\s*/g, ''); // 去除空格
function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args))) // 参数从右向左执行
    return funcs.reduce((a, b) => {
        return (...args) => {
            return a(b(...args);
        };
    })
}
const last = compose(removeSpace, toUpperCase, head, reverse);
last(['hello world', 'Bye', 'good morning']); // GOODMORNING
```

#### 高阶函数

- 高阶函数是一等公民
- 以一个函数作为参数
- 以一个函数作为返回结果

eg. map,filter,reduce,柯里化应用，函数组合应用

```js
let people = [{name: "Fred", age: 65}, {name: "Lucy", age: 36}];

_.max(people, p => p.age); //=> {name: "Fred", age: 65}

```

eg2.
```js
const best = (fun, coll) => _.reduce(coll, (x, y) => fun(x, y) ? x : y )
best(function(x, y) {return x > y}, [1, 2, 3, 4,5 ,6])
```

eg3.对象校验器
```js
function always(VALUE) {
	return function() {
	  return VALUE;
	};
}
function checker(/* validators */) {
	var validators = _.toArray(arguments);
  
	return function(obj) {
	  return _.reduce(validators, function(errs, check) {
		if (check(obj))
		  return errs;
		else
		  return _.chain(errs).push(check.message).value();
	  }, []);
	}
}  
var alwaysPasses = checker(always(true), always(true));
alwaysPasses({}); //[]

var fails = always(false);
fails.message = 'a failture in life';
var alwaysFails = checker(fails);
alwaysFails({}); //["a failture in life"]
```
![区别](result.png)
```js
function validator(message, fun) {
	var f = function(/* args */) {
	  return fun.apply(fun, arguments);
	};
  
	f['message'] = message;
	return f;
}
  
var gonnaFail = checker(validator('ZONG!', always(false)))
gonnaFail(100) //['ZONG']
```

#### 方法链、Pipeline

- 没有级联调用，我们可能需要重复很多代码
- 级联调用的秘密在于每个函数都返回this
- 函数式编程 注重链式调用思想

eg1.jQuery的cascade

```js
/* 方法链 */
$('div')
	.css('color', 'red')
	.css('width', 100)
	.click(onClick)

let $el = $('div');
$el.css('color', 'red')
$el.css('width', 100)
$el.click(onClick)

```
eg2.
```
// 定义一个人的数组
let people = [
    {name: 'alice', age: 1}, 
    {name: 'bob', age: 22},
    {name: 'carol', age: 24}];
// 获取满18岁的名单
let list = people
    .filter(p => p.age >= 18)
    .map(p => p.name)
    .reduce((prev, cur) => prev + ',' + cur);
console.log(list);      // bob,carol
```
eg3.
```js
let output = console.log.bind(console),
    toUpper = str => str.toUpperCase();
Promise
	.resolve({name: 'alice', age: 1})
	.then(p => p.name)
	.then(toUpper)
	.then(output)
```
写得非常优雅

#### 递归

斐波那契数列问题

传统过程式编程的思想
```
0、1、1、2、3、5、8、13、21、34
F(0)=0，F(1)=1, F(n)=F(n-1)+F(n-2)（n>=2，n∈N*）
```
```js
fib = n => {
	if(n == 0) return 0;
	if(n == 1) return 1;
	let i = 2;
	let n_1 = 1;//n-1的值
	let n_2 = 0;//n-2的值
	let tmp_n_1 = 0;
	while(i < n){
		tmp_n_1 = n_1;//保存n-1的值
		n_1 = n_2 + n_1;//计算新的n-1的值
		n_2 = tmp_n_1;//n-2的值为上次n-1的值
		i++;
	}
	return n_1+n_2;
}

```
换个角度
n!=n*(n-1)! 当n=0时，结果为1；

一般来说，递归这种方式于循环相比被认为是更符合人的思维的，即告诉机器做什么而不是告诉机器怎么做

所以：

- 循环实现，从1开始迭代，计算n-1和n-2，并迭代替换，直到n
- 递归实现，归纳出递推式f(n)=f(n-1)+f(n-2),当 n=0时为0，n=1时为1


传统过程式编程的思想
到此，可以发现使用递归来实现程序，明显“简单”很多，而且符合人们直观的理解，而使用循环来实现程序很难一目了然的知道程序在做什么。这便是函数式编程的一大优势

那么为什么诸如阶乘这样的问题，人们的第一想法是用循环来考虑呢？这也许是最初学习编程的时候“先入为主”造成的，而且阶乘的递推关系不是十分直观，因为你要向一个不知道阶乘是何物的人解释什么是阶乘，恐怕不会使用递推关系来解释；返过来对斐波那契数列，就比较适合用递归关系来解释。

在实际的编程过程中我们很少用递归来解决问题，我想，一方面是受到传统过程式编程思维的影响，另一方面，大部分问题更容易用过程化的思维来思考。所以，我一直信奉“编程语言会影响人的思维方式，思维方式也会影响对语言的学习”

### 总结
说了这么多，就是要夸夸学学函数式编程

- 代码清晰，简洁，行数少，方便调试，测试和维护
- 代码更加模块化。函数式编程强迫把大问题分解成待解决的相同问题的小问题实例
- 增加可重用性。函数式编程共享了一类通用辅助函数，模块化
- 减少耦合。函数式编程人员的工作是写函数，高阶函数和纯函数，都是完全独立其无副作用的
- 算数正确性

### 其他
库：
- Underscore
- RxJS
- Functional JavaScript
- ......

参考：

1. https://www.gitbook.com/book/llh911001/mostly-adequate-guide-chinese/details
2. http://www.zhangxinxu.com/wordpress/2013/02/js-currying/
3. http://underscorejs.org/
4. https://github.com/funjs
5. http://www.ruanyifeng.com/blog/2017/02/fp-tutorial.html


---
title: Promise对象池
date: 2024-01-25 22:37:30
tags:
---

```js
/**
 * 递归辅助函数
 * 跟踪函数数组 (functionIndex) 中的当前索引和正在执行的 promise 的当前数量 (inProgressCount)。我们定义一个递归函数 helper，它让我们能够异步执行代码。所有这些代码都包含在返回的 promise 的回调中。

1、每次我们执行一个新的函数，我们都会增加 functionIndex，并且我们会增加 inProgressCount。
2、每次一个 promise 解决，我们都会减少 inProgressCount，并且在 inProgressCount < n 和还有剩余需要执行的函数时重复步骤 1
3、如果在任何时刻，functionIndex == functions.length 和 inProgressCount == 0，我们都可以完成并应该解决返回的 promise。

 */
var promisePool = async function(functions, n) {
   return new Promise((resolve) => {
       let inProgressCount = 0;
       let functionIndex = 0;
       function helper() {
           if (functionIndex >= functions.length) {
               if (inProgressCount === 0) resolve();
               return;
           }

           while (inProgressCount < n && functionIndex < functions.length) {
               inProgressCount++;
               const promise = functions[functionIndex]();
               functionIndex++;
               promise.then(() => {
                   inProgressCount--;
                   helper();
               });
           }
       }
       helper();
   });
};
/**
 * 异步/等待 + Promise.all() + Array.shift()
1、如果没有要执行的函数，立即返回（基本情况）。
2、从函数列表中删除第一个函数（使用 Array.shift）。
3、执行同一个第一个函数，并等待它完成。
4、递归调用自己并等待自己完成。这样，只要有任何函数完成，队列中的下一个函数就会被处理。

 */

var promisePool2 = async function(functions, n) {
    async function evaluateNext() {
        if (functions.length === 0) return;
        const fn = functions.shift();
        await fn();
        await evaluateNext();
    }
    const nPromises = Array(n).fill().map(evaluateNext);
    await Promise.all(nPromises);
};

/**
 * 修改方法二的一般想法，并用一些语法花样使它变得非常短。

代替用 Array.shift 移除数组的第一个元素的做法，我们可以将变量 n 当作当前的索引使用。
代替用 if 语句检查是否还有函数要执行的做法，我们可以在函数调用时使用 可选链接 (functions[n++]?.())。这种语法会在 functions[n++] 是 null 或 undefined 时立即返回 undefined。没有这种语法，就会抛出一个错误。
举例说明，我们可以使用 promise 链式调用 (.then(evaluateNext))，而不是在不同的行中使用 await。
在最初并行执行前 n 个 promise 时，我们需要写 functions.slice(0, n).map(f => f().then(evaluateNext)) 而不是简单地写 functions.slice(0, n).map(evaluateNext)。那样，第 n 个 promise 就会在辅助函数外部立即执行，这样我们就可以正确地使用 n 作为索引变量。

 */
var promisePool3 = async function(functions, n) {
   const evaluateNext = () => functions[n++]?.().then(evaluateNext);
   return Promise.all(functions.slice(0, n).map(f => f().then(evaluateNext)));
};
```
---
title: TypeScript获取input的value值
date: 2019-07-26 11:53:45
categories: [Javascript]
tags:
  - typescript
---

直接获取报错：

```js
let el = document.getElementById("idName");
let val = el.value;
```

error:TS2339: Property 'value' does not exist on type 'HTMLElement'.
<!-- more -->
```js
// 第一种方法
let el = <HTMLInputElement>document.getElementById("idName");
let val = el.value;
```

```js
//第二种方法
let el = document.getElementById("idName") as HTMLInputElement;
let val = el.value;
```

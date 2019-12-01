---
title: vue-loader源码解析
date: 2019-08-01 22:54:07
categories: [Framework]
tags:
---

## 观察输入输出
### 输入
测试是最好的文档，所以我们从测试用例开始分析，找到test/fixture/basic.vue，内容如下：
<!-- more -->
```
<template>
  <h2 class="red">{{msg}}</h2>
</template>

<script>
export default {
  data () {
    return {
      msg: 'Hello from Component A!'
    }
  }
}
</script>

<style>
comp-a h2 {
  color: #f00;
}
</style>
```

## 输出

通过运行测试之后，可以得到以下输出，但是由于文件巨大，笔者只抽出部分开始分析，如下

```
/* 2 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
Object.defineProperty(__webpack_exports__, "__esModule", { value: true });

// CONCATENATED MODULE: ./node_modules/babel-loader/lib!./lib/selector.js?type=script&index=0&bustCache!./test/fixtures/basic.vue

/* harmony default export */ 
var basic = ({
  data() {
    return {
      msg: 'Hello from Component A!'
    };
  }
});
// CONCATENATED MODULE: ./lib/template-compiler?{"id":"data-v-b647d0ce","hasScoped":false,"buble":{"transforms":{}}}!./lib/selector.js?type=template&index=0&bustCache!./test/fixtures/basic.vue
var render = function() {
  var _vm = this
  var _h = _vm.$createElement
  var _c = _vm._self._c || _h
  return _c("h2", { staticClass: "red" }, [_vm._v(_vm._s(_vm.msg))])
}
var staticRenderFns = []
render._withStripped = true
var esExports = { render: render, staticRenderFns: staticRenderFns }
/* harmony default export */ var fixtures_basic = (esExports);
if (false) {
  module.hot.accept()
  if (module.hot.data) {
    require("vue-hot-reload-api")      .rerender("data-v-b647d0ce", esExports)
  }
}
// CONCATENATED MODULE: .!./test/fixtures/basic.vue
var disposed = false
function injectStyle (ssrContext) {
  if (disposed) return
  __webpack_require__(3)
}
var normalizeComponent = __webpack_require__(8)
/* script */

/* template */

/* template functional */
var __vue_template_functional__ = false
/* styles */
var __vue_styles__ = injectStyle
/* scopeId */
var __vue_scopeId__ = null
/* moduleIdentifier (server only) */
var __vue_module_identifier__ = null
var Component = normalizeComponent(
  basic,
  fixtures_basic,
  __vue_template_functional__,
  __vue_styles__,
  __vue_scopeId__,
  __vue_module_identifier__
)
Component.options.__file = "test/fixtures/basic.vue"
if (Component.esModule && Object.keys(Component.esModule).some(function (key) {  return key !== "default" && key.substr(0, 2) !== "__"})) {  console.error("named exports are not supported in *.vue files.")}

})()}
```

## 分析输出

以上的输出就是最终可以拿到浏览器上运行的 javaScript，尽管笔者已经删除了一些会影响理解的部分代码，但是这么直接观察这个文件，难免还是无从下手。
那么我们继续细化分析步骤，vue-loader 将 basic.vue 编译到最终输出的 bundle.js 的过程中，其实调用了四个小的 loader。它们分别是：

- selector
- style-compiler
- template-compiler
- babel-loader

以上四个 loader ，除了 babel-loader 是外部的package，其他三个都存在于 vue-loader 的内部（lib/style-compiler 和 lib/template-compiler 和 lib/selector）。
首先 vue-loader 将 basic.vue 编译成以下内容
```js
/* script */
import __vue_script__ from "!!babel-loader!../../lib/selector?type=script&index=0&bustCache!./basic.vue"
/* template */
import __vue_template__ from "!!../../lib/template-compiler/index?{\"id\":\"data-v-793be54c\",\"hasScoped\":false,\"buble\":{\"transforms\":{}}}!../../lib/selector?type=template&index=0&bustCache!./basic.vue"
/* styles */
import __vue_styles__ from "!!vue-style-loader!css-loader!../../lib/style-compiler/index?{\"vue\":true,\"id\":\"data-v-793be54c\",\"scoped\":false,\"hasInlineConfig\":false}!../../lib/selector?type=styles&index=0&bustCache!./basic.vue"
var Component = normalizeComponent(
  __vue_script__,
  __vue_template__,
  __vue_template_functional__,
  __vue_styles__,
  __vue_scopeId__,
  __vue_module_identifier__
)
```

为了方便理解，笔者删除修改了一些内容。
在三个 import 语句中，不管它们用了多少个不同的 loader 去加载，loader chain 的源头都是 basic.vue。
JavaScript 部分
#### 首先分析 script 部分
```
/* script */
import __vue_script__ from "!!babel-loader!../../lib/selector?type=script&index=0&bustCache!./basic.vue"
```

从右到左，也就是 basic.vue 被先后被 selector 和 babel-loader 处理过了。
selector（参数type=script） 的处理结果是将 basic.vue 中的 javaScript 抽出来之后交给babel-loader去处理，最后生成可用的 javaScript

#### Template 部分
再来分析 template 部分

```
/* template */
import __vue_template__ from "!!../../lib/template-compiler/index?{\"id\":\"data-v-793be54c\",\"hasScoped\":false,\"buble\":{\"transforms\":{}}}!../../lib/selector?type=template&index=0&bustCache!./basic.vue"
```

同样的，从左到右，basic.vue 先后被 selector 和 template-compiler 处理过了。
selector (参数type=template) 的处理结果是将 basic.vue 中的 template 抽出来之后交给 template-compiler 处理，最终输出成可用的 HTML。
#### Style 部分

最后分析 style 部分
```js
/* styles */
import __vue_styles__ from "!!vue-style-loader!css-loader!../../lib/style-compiler/index?{\"vue\":true,\"id\":\"data-v-793be54c\",\"scoped\":false,\"hasInlineConfig\":false}!../../lib/selector?type=styles&index=0&bustCache!./basic.vue"
```
style 涉及的 loader 较多，一个一个来分析， 从上代码可知，basic.vue 先后要被 selector, style-compiler, css-loader 以及 vue-style-loader 处理。
selector (参数type=style) 的处理结果是将 basic.vue 中的 css 抽出来之后交给 style-compiler 处理成 css, 然后交给 css-loader 处理生成 module, 最后通过 vue-style-loader 将 css 放在 `<style>` 里面，然后注入到 HTML 里。
注意，这里之所以没有用 style-loader 是因为 vue-style-loader 是在 fork 了 style-loader 的基础上，增加了后端绘制 (SSR) 的支持。具体的不同，读者可以查看官方文档，笔者这里不再累述。
后续

出处：https://juejin.im/post/5a1cb7d7f265da4319560139

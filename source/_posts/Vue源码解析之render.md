---
title: Vue之render源码解析
date: 2020-06-03 20:42:15
tags:
---

## 前言

render函数可以作为一道分割线，render函数的左边可以称之为编译期，将Vue的模板转换为渲染函数。render函数的右边是Vue的运行时，主要是基于渲染函数生成Virtual DOM树，Diff和Patch。

render渲染函数将结合数据生成Virtual DOM
有了虚拟的DOM树后，再交给Patch函数，负责把这些虚拟DOM真正施加到真实的DOM上。在这个过程中，Vue有自身的响应式系统来侦测在渲染过程中所依赖到的数据来源。在渲染过程中，侦测到数据来源之后就可以精确感知数据源的变动。
根据需要重新进行渲染。当重新进行渲染之后，会生成一个新的树，将新的树与旧的树进行diff对比，就可以最终落实到真实DOM上的改动。

一个简单的实例：

![render-example](render-example.png)

vue的渲染机制可以总结如下：

![render-process](render-process.png)

## render前置操作

1. 为何要用with(this){}包裹？
2. 何时将render函数的字符串转成函数？

### 1. 为何要用with(this){}包裹

> with通常被当做重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身。引自《你不知道的JavaScript》上2.2.2节with

### 2. 何时将render函数的字符串转成函数(vue2.6.11)

> 经过parse生成AST和optimize对AST树的优化，会生成render函数的字符串。在createCompilerCreator（在src\compiler\create-compiler.js）中，调用createCompileToFunctionFn将字符串转成函数。

```js
// src\compiler\index.js
import { parse } from './parser/index'
import { optimize } from './optimizer'
import { generate } from './codegen/index'
import { createCompilerCreator } from './create-compiler'

// `createCompilerCreator` allows creating compilers that use alternative
// parser/optimizer/codegen, e.g the SSR optimizing compiler.
// Here we just export a default compiler using the default parts.
// createCompilerCreator做了render的前置操作
export const createCompiler = createCompilerCreator(function baseCompile (
  template: string,
  options: CompilerOptions
): CompiledResult {
  const ast = parse(template.trim(), options)
  if (options.optimize !== false) {
    optimize(ast, options)
  }
  const code = generate(ast, options)
  return {
    ast,
    render: code.render, // 此时还是render函数的字符串
    staticRenderFns: code.staticRenderFns
  }
})
```

```js
// src\compiler\create-compiler.js
import { extend } from 'shared/util'
import { detectErrors } from './error-detector'
import { createCompileToFunctionFn } from './to-function'

export function createCompilerCreator (baseCompile: Function): Function {
  return function createCompiler (baseOptions: CompilerOptions) {
    function compile (
      template: string,
      options?: CompilerOptions
    ): CompiledResult {
      const finalOptions = Object.create(baseOptions)
      const errors = []
      const tips = []
      finalOptions.warn = (msg, tip) => {
        (tip ? tips : errors).push(msg)
      }

      if (options) {
        // merge custom modules
        if (options.modules) {
          finalOptions.modules =
            (baseOptions.modules || []).concat(options.modules)
        }
        // merge custom directives
        if (options.directives) {
          finalOptions.directives = extend(
            Object.create(baseOptions.directives || null),
            options.directives
          )
        }
        // copy other options
        for (const key in options) {
          if (key !== 'modules' && key !== 'directives') {
            finalOptions[key] = options[key]
          }
        }
      }

      const compiled = baseCompile(template, finalOptions)
      if (process.env.NODE_ENV !== 'production') {
        errors.push.apply(errors, detectErrors(compiled.ast))
      }
      compiled.errors = errors
      compiled.tips = tips
      return compiled
    }

    return {
      compile,
      compileToFunctions: createCompileToFunctionFn(compile)
    }
  }
}
```

createCompileToFunctionFn（在src\compiler\to-function.js）中，会优先读缓存信息，若没有才会执行编译方法，同时将render字符串通过createFunction调用New Function()的方法，创造render函数，然后缓存信息。

```js
/* @flow */

import { noop, extend } from 'shared/util'
import { warn as baseWarn, tip } from 'core/util/debug'
import { generateCodeFrame } from './codeframe'

type CompiledFunctionResult = {
  render: Function;
  staticRenderFns: Array<Function>;
};

function createFunction (code, errors) {
  try {
    return new Function(code)
  } catch (err) {
    errors.push({ err, code })
    return noop
  }
}

export function createCompileToFunctionFn (compile: Function): Function {
  const cache = Object.create(null)

  return function compileToFunctions (
    template: string,
    options?: CompilerOptions,
    vm?: Component
  ): CompiledFunctionResult {
    options = extend({}, options)
    const warn = options.warn || baseWarn
    delete options.warn

    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      // detect possible CSP restriction
      try {
        new Function('return 1')
      } catch (e) {
        if (e.toString().match(/unsafe-eval|CSP/)) {
          warn(
            'It seems you are using the standalone build of Vue.js in an ' +
            'environment with Content Security Policy that prohibits unsafe-eval. ' +
            'The template compiler cannot work in this environment. Consider ' +
            'relaxing the policy to allow unsafe-eval or pre-compiling your ' +
            'templates into render functions.'
          )
        }
      }
    }

    // check cache
    const key = options.delimiters
      ? String(options.delimiters) + template
      : template
    if (cache[key]) {
      return cache[key]
    }

    // compile
    const compiled = compile(template, options)

    // check compilation errors/tips
    if (process.env.NODE_ENV !== 'production') {
      if (compiled.errors && compiled.errors.length) {
        if (options.outputSourceRange) {
          compiled.errors.forEach(e => {
            warn(
              `Error compiling template:\n\n${e.msg}\n\n` +
              generateCodeFrame(template, e.start, e.end),
              vm
            )
          })
        } else {
          warn(
            `Error compiling template:\n\n${template}\n\n` +
            compiled.errors.map(e => `- ${e}`).join('\n') + '\n',
            vm
          )
        }
      }
      if (compiled.tips && compiled.tips.length) {
        if (options.outputSourceRange) {
          compiled.tips.forEach(e => tip(e.msg, vm))
        } else {
          compiled.tips.forEach(msg => tip(msg, vm))
        }
      }
    }

    // turn code into functions
    const res = {}
    const fnGenErrors = []
    res.render = createFunction(compiled.render, fnGenErrors)
    res.staticRenderFns = compiled.staticRenderFns.map(code => {
      return createFunction(code, fnGenErrors)
    })

    // check function generation errors.
    // this should only happen if there is a bug in the compiler itself.
    // mostly for codegen development use
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production') {
      if ((!compiled.errors || !compiled.errors.length) && fnGenErrors.length) {
        warn(
          `Failed to generate render function:\n\n` +
          fnGenErrors.map(({ err, code }) => `${err.toString()} in\n\n${code}\n`).join('\n'),
          vm
        )
      }
    }

    return (cache[key] = res)
  }
}
```

参考：

https://juejin.im/post/5d09c587e51d454f71439ce2

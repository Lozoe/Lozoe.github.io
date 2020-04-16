---
title: Vue源码解析值sfc
date: 2020-04-08 14:17:35
tags:
---

## 前言

在vue项目中，.vue 文件称为 SFC(Single File Components)，在vue的源码中，有一个sfc模块专门负责.vue单文件组件的解析

vue 会先对 .vue 文件进行解析，分成 template、script、styles、customBlocks 四个部分，称为 descriptor。之后，再对这四个部分分别进行编译最终得到可以在浏览器中执行的 .js 文件。

SFCDescriptor，是表示 .vue 各个代码块的对象，为以下数据格式：

```js
// an object format describing a single-file component.
declare type SFCDescriptor = {
    template: ?SFCBlock;
    script: ?SFCBlock;
    styles: Array<SFCBlock>;
    customBlocks: Array<SFCBlock>;
};
```

vue 提供了一个 [compiler.parseComponent(file, [options])](https://github.com/vuejs/vue/tree/dev/packages/vue-template-compiler#compilerparsecomponentfile-options)方法，来将 .vue 文件解析成一个 SFCDescriptor。

## 1. 文件入口

解析 sfc 文件的源码入口在 src/sfc/parser.js 中，编译后的产出在 /packages/vue-template-compiler 和 /packages/vue-server-renderer 下的 build.js 中。

build.js 文件中直接 export 出了parseComponent方法。

## 2. parseComponent方法

```js
/**
 * Parse a single-file component (*.vue) file into an SFC Descriptor Object.
 */
export function parseComponent (
    content: string,
    options?: Object = {}
): SFCDescriptor {
    const sfc: SFCDescriptor = {
        template: null,
        script: null,
        styles: [],
        customBlocks: []
    }
    let depth = 0
    let currentBlock: ?SFCBlock = null

    function start (
        tag: string,
        attrs: Array<Attribute>,
        unary: boolean,
        start: number,
        end: number
    ) {
        // ...
    }

    function checkAttrs (block: SFCBlock, attrs: Array<Attribute>) {
        for (let i = 0; i < attrs.length; i++) {
            const attr = attrs[i]
            if (attr.name === 'lang') {
                block.lang = attr.value
            }
            if (attr.name === 'scoped') {
                block.scoped = true
            }
            if (attr.name === 'module') {
                block.module = attr.value || true
            }
            if (attr.name === 'src') {
                block.src = attr.value
            }
        }
    }

    function end (tag: string, start: number, end: number) {
        // ...
    }

    function padContent (block: SFCBlock, pad: true | "line" | "space") {
        if (pad === 'space') {
            return content.slice(0, block.start).replace(replaceRE, ' ')
        } else {
            const offset = content.slice(0, block.start).split(splitRE).length
            const padChar = block.type === 'script' && !block.lang
                ? '//\n'
                : '\n'
            return Array(offset).join(padChar)
        }
    }

    parseHTML(content, {
        start,
        end
    })

    return sfc
}
```

以上代码中，parseComponent方法中主要定义了start和end两个函数，之后调用了parseHTML方法来对 .vue 文件内容践行编译。start和end两个函数作为参数传给了parseHTML

## 3. parseHTML方法

parseHTML是一个html-parser，分解分析 .vue 的关键,parseHTML的代码细节较多，遍历解析查找文件中的各个标签，解析到每个起始标签时，调用 option 中的 start 方法进行处理；解析到每个结束标签时，调用 option 中的 end 方法进行处理。（即parseComponent中定义的 start 和 end）

由于我们这里只是想要找到第一层标签，也就是 template、script这些。因此可以在parseComponent中维护一个 depth 变量，在start中将depth++，在end中depth--。那么，每个depth === 1的标签就是我们需要获取的信息，包含 template、script、style 以及一些自定义标签。

```js

export function parseHTML (html, options) {
    const stack = []
    const expectHTML = options.expectHTML
    const isUnaryTag = options.isUnaryTag || no
    const canBeLeftOpenTag = options.canBeLeftOpenTag || no
    let index = 0
    let last, lastTag
    while (html) {
        last = html
        if (!lastTag || !isPlainTextElement(lastTag)) {
            // 这里分离了template
        } else {
            // 这里分离了style/script
        }

    // 略

    // 前进n个字符
    function advance (n) {
        // 略
    }

    // 解析 openTag 比如 <template>
    function parseStartTag () {
        // 略
    }

    // 处理 openTag
    function handleStartTag (match) {
        // 略
        if (options.start) {
            options.start(tagName, attrs, unary, match.start, match.end)
        }
    }

    // 处理 closeTag
    function parseEndTag (tagName, start, end) {
        // 略
        if (options.start) {
        options.start(tagName, [], false, start, end)
        }
        if (options.end) {
        options.end(tagName, start, end)
        }
    }
}
```

1）一个 while 循环
在 while 循环中，存在两个大的分支，一个用来分析 template ，一个是用来分析 script 和 style。

2）函数 advance
向前跳过文本

3）函数 parseStartTag
判断当前的 node 是不是 openTag

4）函数 handleStartTag
处理 openTag, 这里就用到了之前提到的 start() 函数

5）函数 parseEndTag
判断当前的 node 是不是 closeTag，同时这里也用到了 end() 函数

通过以上各个函数的组合，在while循环中就将 sfc 分割成了三个不同的部分

## 4. start

```js
function start (
    tag: string,
    attrs: Array<Attribute>,
    unary: boolean,
    start: number,
    end: number
) {
    if (depth === 0) {
        currentBlock = {
            type: tag,
            content: '',
            start: end,
            attrs: attrs.reduce((cumulated, { name, value }) => {
                cumulated[name] = value || true
                return cumulated
            }, {})
        }
        if (isSpecialTag(tag)) {
            checkAttrs(currentBlock, attrs)
            if (tag === 'style') {
                sfc.styles.push(currentBlock)
            } else {
                sfc[tag] = currentBlock
            }
        } else { // custom blocks
            sfc.customBlocks.push(currentBlock)
        }
    }
    if (!unary) {
        depth++
    }
}
```

1）记录下 currentBlock。每个 currentBlock 包含以下内容

```js
declare type SFCBlock = {
    type: string;
    content: string;
    start?: number;
    end?: number;
    lang?: string;
    src?: string;
    scoped?: boolean;
    module?: string | boolean;
};
```

2）根据 tag 名称，将 currentBlock 对象保存在在返回结果对象中。

返回结果对象定义为 sfc，如果tag不是 script,style,template 中的任一个，就放在 sfc.customBlocks 中。如果是style，就放在 sfc.styles 中。script 和 template 则直接放在 sfc.script 和 sfc.template 下。

## 5. end

每当遇到一个结束标签时，执行end函数。

```js
function end (tag: string, start: number, end: number) {
    if (depth === 1 && currentBlock) {
        currentBlock.end = start
        let text = deindent(content.slice(currentBlock.start, currentBlock.end))
        // pad content so that linters and pre-processors can output correct
        // line numbers in errors and warnings
        if (currentBlock.type !== 'template' && options.pad) {
            text = padContent(currentBlock, options.pad) + text
        }
        currentBlock.content = text
        currentBlock = null
    }
    depth--
}
```

如果当前是第一层标签(depth === 1)，并且 currentBlock 变量存在，那么取出这部分text，放在 currentBlock.content 中。

在将 .vue 整个遍历一遍后，得到的 sfc 对象即为我们需要的 SFCDescriptor。

## 6. 生成 .js

compiler.parseComponent(file, [options])得到的只是一个组件的 SFCDescriptor，最终编译成.js 文件是交给 vue-loader 等库来做的。


https://github.com/vuejs/vue/blob/dev/packages/vue-template-compiler/index.js
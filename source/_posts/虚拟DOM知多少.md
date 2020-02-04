---
title: 虚拟DOM知多少
date: 2019-08-06 17:03:24
tags:
---

## 什么是Virtual DOM

Virtual DOM是对DOM的抽象,本质上是JavaScript对象,这个对象就是更加轻量级的对DOM的描述.

![virtual dom](virtual-dom.png)

- virtual dom，虚拟dom
- 用js模拟dom结构
- DOM变化的对比，放在JS层来做
- 提高重回性能

<!-- more -->

### 真实的DOM

```html
<ul id='list'>
    <li class='item'>item1</li>
    <li class='item'>item2</li>
</ul>
```

### 虚拟DOM:只模拟了DOM的一部分节点，实际DOM节点中的属性是很复杂的

```js
{
    tag: 'ul',
    attrs:{
        id: 'list'
    },
    children:[
       {
            tag: 'li',
            attrs: {
                className: 'item',
                children: ['item']
            }
            tag: 'li',
            attrs:{
                className: 'item',
                children: ['item2']
            }
       }
    ]
}
```

## 为什么需要Virtual DOM

既然我们已经有了DOM,为什么还需要额外加一层抽象?

首先,我们都知道在前端性能优化的一个秘诀就是尽可能少地操作DOM,不仅仅是DOM相对较慢,更因为频繁变动DOM会造成浏览器的回流或者重绘,这些都是性能的杀手,因此我们需要这一层抽象,在patch过程中尽可能地一次性将差异更新到DOM中,这样保证了DOM不会出现性能很差的情况.（DOM操作是昂贵的，js的运行效率高；尽量减少DOM操作，找出DOM必须更新的节点来更新，其他的不更新）

其次,现代前端框架的一个基本要求就是无须手动操作DOM,一方面是因为手动操作DOM无法保证程序性能,多人协作的项目中如果review不严格,可能会有开发者写出性能较低的代码,另一方面更重要的是省略手动DOM操作可以大大提高开发效率.（项目越复杂，影响越严重）

最后,也是Virtual DOM最初的目的,就是更好的跨平台,比如Node.js就没有DOM,如果想实现SSR(服务端渲染),那么一个方式就是借助Virtual DOM,因为Virtual DOM本身是JavaScript对象.

## Virtual DOM的关键要素

### 虚拟DOM的核心类库 —— snabbdom

#### 核心函数有h函数和patch函数

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <div id="container"></div>
    <button id="btn-change">change</button>

    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/snabbdom.js"></script>
    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/snabbdom-class.js"></script>
    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/snabbdom-props.js"></script>
    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/snabbdom-style.js"></script>
    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/snabbdom-eventlisteners.js"></script>
    <script src="https://cdn.bootcss.com/snabbdom/0.7.0/h.js"></script>
    <script type="text/javascript">
        var snabbdom = window.snabbdom
        // 定义关键函数 patch
        var patch = snabbdom.init([
            snabbdom_class,
            snabbdom_props,
            snabbdom_style,
            snabbdom_eventlisteners
        ])

        // 定义关键函数 h
        var h = snabbdom.h

        // 原始数据
        var data = [
            {
                name: '张三',
                age: '20',
                address: '北京'
            },
            {
                name: '李四',
                age: '21',
                address: '上海'
            },
            {
                name: '王五',
                age: '22',
                address: '广州'
            }
        ]
        // 把表头也放在 data 中
        data.unshift({
            name: '姓名',
            age: '年龄',
            address: '地址'
        })

        var container = document.getElementById('container')

        // 渲染函数
        var vnode
        function render(data) {
            var newVnode = h('table', {}, data.map(function (item) {
                var tds = []
                var i
                for (i in item) {
                    if (item.hasOwnProperty(i)) {
                        tds.push(h('td', {}, item[i] + ''))
                    }
                }
                return h('tr', {}, tds)
            }))

            if (vnode) {
                // re-render
                patch(vnode, newVnode)
            } else {
                // 初次渲染
                patch(container, newVnode)
            }

            // 存储当前的 vnode 结果
            vnode = newVnode
        }

        // 初次渲染
        render(data)


        var btnChange = document.getElementById('btn-change')
        btnChange.addEventListener('click', function () {
            data[1].age = 30
            data[2].address = '深圳'
            // re-render
            render(data)
        })

    </script>
</body>
</html>
```

### Virtual DOM的创建

我们已经知道Virtual DOM是对真实DOM的抽象,根据不同的需求我们可以做出不同的抽象,比如snabbdom.js的抽象方式是这样的.

```js
export interface VNode {
    sel: string | undefined;
    data: VNodeData | undefined;
    children: Array<VNode | string> | undefined;
    elm: Node | undefined;
    text: string | undefined;
    key: Key | undefined;
}
```

当然,snabbdom.js由于是面向生产环境的库,所以做了大量的抽象,我们由于仅仅作为理解,采用最简单的抽象方法:

```js
{
  type, // String，DOM 节点的类型，如 'div'
  data, // Object，包括 props，style等等 DOM 节点的各种属性
  children // Array，子节点
}
```

在明确了我们抽象的Virtual DOM构造之后,我们就需要一个函数来创建Virtual DOM.

```js
/**
 * 生成 vnode
 * @param  {String} type     类型，如 'div'
 * @param  {String} key      key vnode的唯一id
 * @param  {Object} data     data，包括属性，事件等等
 * @param  {Array} children  子 vnode
 * @param  {String} text     文本
 * @param  {Element} elm     对应的 dom
 * @return {Object}          vnode
 */
// function vnode(type, key, data, children, text, elm) {
//   const element = {
//     __type: VNODE_TYPE,
//     type, key, data, children, text, elm
//   }

//   return element
// }
export function vnode(sel: string | undefined,
                      data: any | undefined,
                      children: Array<VNode | string> | undefined,
                      text: string | undefined,
                      elm: Element | Text | undefined): VNode {
  let key = data === undefined ? undefined : data.key;
  return {sel: sel, data: data, children: children,
          text: text, elm: elm, key: key};
}

export default vnode;

```

这个函数很简单,接受一定的参数,再根据这些参数返回一个对象,这个对象就是DOM的抽象.

### Virtual DOM Tree的创建

上面我们已经声明了一个vnode函数用于单个Virtual DOM的创建工作,但是我们都知道DOM其实是一个Tree,我们接下来要做的就是声明一个函数用于创建DOM Tree的抽象 -- Virtual DOM Tree.

```js
export function h(sel: string): VNode;
export function h(sel: string, data: VNodeData): VNode;
export function h(sel: string, children: VNodeChildren): VNode;
export function h(sel: string, data: VNodeData, children: VNodeChildren): VNode;
export function h(sel: any, b?: any, c?: any): VNode {
    var data: VNodeData = {}, children: any, text: any, i: number;
    if (c !== undefined) {
        data = b;
        if (is.array(c)) { children = c; }
        else if (is.primitive(c)) { text = c; }
        else if (c && c.sel) { children = [c]; }
    } else if (b !== undefined) {
        if (is.array(b)) { children = b; }
        else if (is.primitive(b)) { text = b; }
        else if (b && b.sel) { children = [b]; }
        else { data = b; }
    }
    if (children !== undefined) {
        for (i = 0; i < children.length; ++i) {
        if (is.primitive(children[i])) children[i] = vnode(undefined, undefined, undefined, children[i], undefined);
        }
    }
    if (
        sel[0] === 's' && sel[1] === 'v' && sel[2] === 'g' &&
        (sel.length === 3 || sel[3] === '.' || sel[3] === '#')
    ) {
        addNS(data, children, sel);
    }
    return vnode(sel, data, children, text, undefined);
};
export default h;
```

### Virtual DOM 的更新

Virtual DOM 归根到底是JavaScript对象,我们得想办法将Virtual DOM与真实的DOM对应起来,也就是说,需要我们声明一个函数,此函数可以将vnode转化为真实DOM.

先来个简化版本

```js
function createElement(vnode) {
    var tag = vnode.tag  // 'ul'
    var attrs = vnode.attrs || {}
    var children = vnode.children || []
    if (!tag) {
        return null
    }

    // 创建真实的 DOM 元素
    var elem = document.createElement(tag)
    // 属性
    var attrName
    for (attrName in attrs) {
        if (attrs.hasOwnProperty(attrName)) {
            // 给 elem 添加属性
            elem.setAttribute(attrName, attrs[attrName])
        }
    }
    // 子元素
    children.forEach(function (childVnode) {
        // 给 elem 添加子元素
        elem.appendChild(createElement(childVnode))  // 递归
    })

    // 返回真实的 DOM 元素
    return elem
}
```

再看源版

```typescript
function createElm(vnode: VNode, insertedVnodeQueue: VNodeQueue): Node {
    let i: any, data = vnode.data;
    if (data !== undefined) {
        if (isDef(i = data.hook) && isDef(i = i.init)) {
            i(vnode);
            data = vnode.data;
        }
    }
    let children = vnode.children, sel = vnode.sel;
    if (sel === '!') {
        if (isUndef(vnode.text)) {
            vnode.text = '';
        }
        vnode.elm = api.createComment(vnode.text as string);
    } else if (sel !== undefined) {
        // Parse selector
        const hashIdx = sel.indexOf('#');
        const dotIdx = sel.indexOf('.', hashIdx);
        const hash = hashIdx > 0 ? hashIdx : sel.length;
        const dot = dotIdx > 0 ? dotIdx : sel.length;
        const tag = hashIdx !== -1 || dotIdx !== -1 ? sel.slice(0, Math.min(hash, dot)) : sel;
        const elm = vnode.elm = isDef(data) && isDef(i = (data as VNodeData).ns) ? api.createElementNS(i, tag)
            : api.createElement(tag);
        if (hash < dot) elm.setAttribute('id', sel.slice(hash + 1, dot));
        if (dotIdx > 0) elm.setAttribute('class', sel.slice(dot + 1).replace(/\./g, ' '));
        for (i = 0; i < cbs.create.length; ++i) cbs.create[i](emptyNode, vnode);
        if (is.array(children)) {
            for (i = 0; i < children.length; ++i) {
                const ch = children[i];
                if (ch != null) {
                    api.appendChild(elm, createElm(ch as VNode, insertedVnodeQueue));
                }
            }
        } else if (is.primitive(vnode.text)) {
            api.appendChild(elm, api.createTextNode(vnode.text));
        }
        i = (vnode.data as VNodeData).hook; // Reuse variable
        if (isDef(i)) {
            if (i.create) i.create(emptyNode, vnode);
            if (i.insert) insertedVnodeQueue.push(vnode);
        }
    } else {
        vnode.elm = api.createTextNode(vnode.text as string);
    }
    return vnode.elm;
}
```

上述函数其实工作很简单,就是根据 type 生成对应的 DOM，把 data 里定义的 各种属性设置到 DOM 上.

### Virtual DOM 的diff

#### vdom为什么使用diff算法

- DOM操作是昂贵的，尽量减少DOM操作
- 找出DOM必须更新的节点来更新，其他的不更新
- 找出的过程，就是diff算法的实现

Virtual DOM 的 diff才是整个Virtual DOM 中最难理解也最核心的部分,diff的目的就是比较新旧Virtual DOM Tree找出差异并更新.

可见diff是直接影响Virtual DOM 性能的关键部分.

要比较Virtual DOM Tree的差异,理论上的时间复杂度高达O(n^3),这是一个奇高无比的时间复杂度,很显然选择这种低效的算法是无法满足我们对程序性能的基本要求的.

好在我们实际开发中,很少会出现跨层级的DOM变更,通常情况下的DOM变更是同级的,因此在现代的各种Virtual DOM库都是只比较同级差异,在这种情况下我们的时间复杂度是O(n).

![diff](diff.png)

那么我们接下来需要实现一个函数,进行具体的diff运算,函数updateChildren的核心算法如下:

```js
function updateChildren(parentElm: Node,
    oldCh: Array<VNode>,
    newCh: Array<VNode>,
    insertedVnodeQueue: VNodeQueue) {
    let oldStartIdx = 0, newStartIdx = 0;
    let oldEndIdx = oldCh.length - 1;
    let oldStartVnode = oldCh[0];
    let oldEndVnode = oldCh[oldEndIdx];
    let newEndIdx = newCh.length - 1;
    let newStartVnode = newCh[0];
    let newEndVnode = newCh[newEndIdx];
    let oldKeyToIdx: any;
    let idxInOld: number;
    let elmToMove: VNode;
    let before: any;

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
        if (oldStartVnode == null) {
            oldStartVnode = oldCh[++oldStartIdx]; // Vnode might have been moved left
        } else if (oldEndVnode == null) {
            oldEndVnode = oldCh[--oldEndIdx];
        } else if (newStartVnode == null) {
            newStartVnode = newCh[++newStartIdx];
        } else if (newEndVnode == null) {
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newStartVnode)) {
            patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue);
            oldStartVnode = oldCh[++oldStartIdx];
            newStartVnode = newCh[++newStartIdx];
        } else if (sameVnode(oldEndVnode, newEndVnode)) {
            patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue);
            oldEndVnode = oldCh[--oldEndIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
            patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue);
            api.insertBefore(parentElm, oldStartVnode.elm as Node, api.nextSibling(oldEndVnode.elm as Node));
            oldStartVnode = oldCh[++oldStartIdx];
            newEndVnode = newCh[--newEndIdx];
        } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
            patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue);
            api.insertBefore(parentElm, oldEndVnode.elm as Node, oldStartVnode.elm as Node);
            oldEndVnode = oldCh[--oldEndIdx];
            newStartVnode = newCh[++newStartIdx];
        } else {
            if (oldKeyToIdx === undefined) {
                oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx);
            }
            idxInOld = oldKeyToIdx[newStartVnode.key as string];
            if (isUndef(idxInOld)) { // New element
                api.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm as Node);
                newStartVnode = newCh[++newStartIdx];
            } else {
                elmToMove = oldCh[idxInOld];
                if (elmToMove.sel !== newStartVnode.sel) {
                    api.insertBefore(parentElm, createElm(newStartVnode, insertedVnodeQueue), oldStartVnode.elm as Node);
                } else {
                    patchVnode(elmToMove, newStartVnode, insertedVnodeQueue);
                    oldCh[idxInOld] = undefined as any;
                    api.insertBefore(parentElm, (elmToMove.elm as Node), oldStartVnode.elm as Node);
                }
                newStartVnode = newCh[++newStartIdx];
            }
        }
    }
    if (oldStartIdx <= oldEndIdx || newStartIdx <= newEndIdx) {
        if (oldStartIdx > oldEndIdx) {
            before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].elm;
            addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
        } else {
            removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
        }
    }
}
```

我们可以假设有旧的Vnode数组和新的Vnode数组这两个数组,而且有四个变量充当指针分别指到两个数组的头尾.

重复下面的对比过程，直到两个数组中任一数组的头指针超过尾指针，循环结束

- 头头对比: 对比两个数组的头部，如果找到，把新节点patch到旧节点，头指针后移

- 尾尾对比: 对比两个数组的尾部，如果找到，把新节点patch到旧节点，尾指针前移

- 旧尾新头对比: 交叉对比，旧尾新头，如果找到，把新节点patch到旧节点，旧尾指针前移，新头指针后移

- 旧头新尾对比: 交叉对比，旧头新尾，如果找到，把新节点patch到旧节点，新尾指针前移，旧头指针后移

- 利用key对比: 用新指针对应节点的key去旧数组寻找对应的节点,这里分三种情况,当没有对应的key，那么创建新的节点,如果有key并且是相同的节点，把新节点patch到旧节点,如果有key但是不是相同的节点，则创建新节点

我们假设有新旧两个数组:

- 旧数组: [1, 2, 3, 4, 5]
- 新数组: [1, 4, 6, 1000, 100, 5]

初始化:
![diff-step-1.png](diff-step-1.png)

首先我们进行头头对比,新旧数组的头部都是1,因此将双方的头部指针后移.

头头对比:
![diff-step-2.png](diff-step-2.png)

我们继续头头对比,但是2 !== 4导致对比失败,我进入尾尾对比,5 === 5,那么尾部指针则可前移.

尾尾对比:
![diff-step-3.png](diff-step-3.png)

现在进入新的循环,头头对比2 !== 4,尾尾对比4 !== 100,此时进入交叉对比,先进行旧尾新头对比,即4 === 4,旧尾前移且新头后移.

旧尾新头对比：
![diff-step-4.png](diff-step-4.png)

接着再进入一个轮新的循环,头头对比2 !== 6,尾尾对比3 !== 100,交叉对比2 != 100 3 != 6,四种对比方式全部不符合,如果这个时候需要通过key去对比,然后将新头指针后移

全部不符合靠key对比:
![diff-step-5.png](diff-step-5.png)

继续重复上述对比的循环方式直至任一数组的头指针超过尾指针，循环结束.

![diff-step-6.png](diff-step-6.png)

在上述循环结束后,两个数组中可能存在未遍历完的情况:
循环结束后，

先对比旧数组的头尾指针，如果旧数组遍历完了（可能新数组没遍历完，有漏添加的问题），添加新数组中漏掉的节点

```js
if (oldStartIdx > oldEndIdx) {
    before = newCh[newEndIdx + 1] == null ? null : newCh[newEndIdx + 1].elm;
    addVnodes(parentElm, before, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
}
```

再对比新数组的头尾指针，如果新数组遍历完了（可能旧数组没遍历完，有漏删除的问题），删除旧数组中漏掉的节点

```js
else if(oldStartIdx > oldEndIdx) {
    removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
}
```

## Virtual DOM的优化

上一节我们的Virtual DOM实现是参考了snabbdom.js的实现,当然Vue.js也同样参考了snabbdom.js,我们省略了大量边缘状态和svg等相关的代码,仅仅实现了其核心部分.

snabbdom.js已经是社区内主流的Virtual DOM实现了,vue 2.0阶段与snabbdom.js一样都采用了上面讲解的「双端比较算法」,那么有没有一些优化方案可以使其更快?

其实，社区内有更快的算法，例如inferno.js就号称最快react-like框架(虽然inferno.js性能强悍的原因不仅仅是算法,但是其diff算法的确是目前最快的),而vue 3.0就会借鉴inferno.js的算法进行优化.

我们可以等到Vue 3.0发布后再一探究竟,具体的优化思想可以先参考diff 算法原理概述,其中一个核心的思想就是利用LIS(最长递增子序列)的思想做动态规划,找到最小的移动次数.

例如以下两个新旧数组,React的算法会把 a, b, c 移动到他们的相应的位置 + 1共三步操作,而inferno.js则是直接将d移动到最前端这一步操作.

```js
 * A: [a b c d]
 * B: [d a b c]
```

参考：

https://github.com/snabbdom/snabbdom

https://mp.weixin.qq.com/s/7VucLP5iFhEUIYQJJSmxeQ

https://juejin.im/post/5d7f32d5518825570327eff2
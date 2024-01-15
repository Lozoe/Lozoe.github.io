---
title: react面试题系列
date: 2024-01-15 19:52:43
categories: [Framework]
tags:
  - react
---

## 1. setState是同步还是异步？
v18之前：

React是希望setState表现为异步的，因为批量更新可以优化性能。因此在React能够管控到的地方，比如生命周期钩子和合成事件回调函数内，表现为异步。
在定时器和原生事件里，因为React管控不到，所以表现为同步。
在某些情况下，我们需要立即获取更新后的状态，这时可以使用第二个可选参数callback，在状态更新后立即执行回调函数来获取更新后的状态。例如：


```
this.setState({ counter: this.state.counter + 1 }, () => {
  console.log(this.state.counter); // 输出更新后的值
});

```
v18之后：

React18之后，默认所有的操作都放到批处理中，因此setState不管在那儿调用都是异步的

如果希望同步更新，可以使用flushSync这个API。

## 2. React组件通信方式有哪些？

- Props: 父组件可以通过 props 将数据传递给子组件。子组件可以通过 this.props 访问这些数据。
- Callback: 父组件可以通过回调函数将函数传递给子组件。子组件可以在适当的时候调用这些回调函数，以便与父组件通信。
- Context: 上下文是一种在组件树中共享数据的方法。通过 context，可以在组件树中传递数据，而不需要在每个级别显式地将 props 传递给所有组件。
- Redux: 复杂应用全局状态管理可以使用Redux、Mobx等状态管理库，项目里一般使用React Redux或者RTK工具包。
- Pub/Sub: 发布/订阅模式是一种通过事件来进行任意组件间通信的方法，和Vue里的事件总线原理一样。
- Hooks里也可以通过useReducer和useContext来实现全局组件通信。

## 3. 在项目中怎么使用Redux？

Redux是一个通用的JS库，一般在React项目里不会直接使用Redux，在项目中使用的一般是react-redux。

现在React官方推荐的是RTK工具包，使用起来更简洁，不用写很多样板代码，代码可读性更好。

当然也可以根据公司的需求，自研状态管理方案。


## 4. Redux中间件是什么？实现原理？

中间件的本质就是个函数，在Redux每次写数据的时候执行，用来实现一些通用的功能。

常见的中间件功能包括异步中间件、持久化中间件、log中间件。

Redux中间件的实现原理和Koa中间件、Axios拦截器类似，数组里面存函数，然后compose调用中间件函数，并传递参数给中间件。

## 5. 函数式组件和类组件有什么区别？

- 类组件：有props、state和生命周期，可以实现复杂UI和交互。
- 函数式组件：在Hooks之前，函数式组件就是纯渲染组件，接收props返回React元素。Hooks之后，函数式组件有了能表达交互的能力。
- 逻辑复用：类组件可以通过HOC和Mixin实现逻辑复用，函数式组件可以通过Hooks实现逻辑复用
## 6. React逻辑复用方式有哪些？

- Mixin：有很多缺点，已被弃用，可以不考虑。
- HOC(高阶组件)：高阶组件是一个函数，它接收一个组件作为参数并返回一个新的组件。高阶组件可以将一些通用的逻辑（如：数据获取、权限验证、错误处理等）封装到一个函数中，并将其作为高阶组件的参数传递给其他组件使用，HOC一般以withXxx命名，并可以结合装饰器优雅地使用。
- Render Props：通过在组件中传递一个函数作为prop，该函数将用于渲染组件的内容。这个函数可以接收组件需要的数据和方法，并返回React元素。
- Hooks：自定义Hooks，将通用逻辑封装到useXxx函数中，可以在多个组件内使用，常见的像数据请求、表单、防抖节流、拖拽等。

## xx. 使用Hooks有踩过哪些坑？

## 7. useRef有什么作用？
获取DOM。
存储上一次渲染的值，可以用useRef创建一个对象来存储setState前的旧值。


## 8. useMemo和useCallback有什么作用？

useMemo类似于Vue的计算属性。

```
import React, { useMemo } from 'react';

function ExpensiveComponent({ data }) {
  const expensiveResult = useMemo(() => {
    // 计算昂贵的结果
    return data.filter(item => item > 10);
  }, [data]);

  return <div>{expensiveResult}</div>;
}

```

useCallback：大多数人认为useCallback的作用是缓存函数的生成，但在实际应用中这种优化是微不足道的，useCallback真正的作用是在函数需要作为prop传递给子组件时，使用useCallback包裹可以避免子组件无谓的更新

## 9. React代码层面有哪些性能优化的方式？
React.memo()：可以缓存组件的渲染结果，避免不必要的重渲染。它接受一个函数组件，并返回一个新的组件，新组件将只在props发生变化时才重新渲染。
useMemo和useCallback。
shouldComponentUpdate：在类组件中，可以通过实现shouldComponentUpdate()方法来判断组件是否需要重新渲染。SCU接收两个参数：nextProps和nextState，我们可以在这个方法中比较当前props和state与下一个props和state的变化来决定是否需要重新渲染组件。
使用React.lazy()和Suspense进行组件懒加载。
对于大型应用，可以使用不可变数据的三方库比如Immer.js结合shouldComponentUpdate来做性能优化。

## 10. 什么是受控组件和非受控组件？
受控组件和非受控组件是针对表单的。

1. 受控组件：(类似于Vue的双向绑定)


React默认不是双向绑定的，也就是说当我们在输入框输入的时候，输入框绑定的值并不会自动变化。


通过给input绑定onChange事件，让React实现类似于Vue的双向绑定，这就叫受控组件。

```
import React, { useState } from 'react';

function ControlledComponent() {
  const [value, setValue] = useState('');

  const handleChange = (event) => {
    setValue(event.target.value);
  };

  return (
    <input type="text" value={value} onChange={handleChange} />
  );
}

```


2. 非受控组件：


非受控组件是让用户手动操作Dom来控制表单值。用ref来获取表单元素的值


非受控组件的好处是更自由，可以更方便地自行选择三方库来处理表单。

```
import React, { useRef } from 'react';

function UncontrolledComponent() {
  const inputRef = useRef(null);

  const handleClick = () => {
    console.log(inputRef.current.value);
  };

  return (
    <div>
      <input type="text" ref={inputRef} />
      <button onClick={handleClick}>获取值</button>
    </div>
  );
}

```
## 11. 什么是eject？
npm run eject命令可以将 create-react-app 创建的 React 项目里的配置文件暴露出来，以便我们自定义配置。
比如以下场景：

- 配置Less等预编译器语法。
- 使用PostCSS工具来做移动端自动适配。
- 搭建Typescript开发环境。
- Webpack优化。


## 12. Hooks实现原理？

1. memorizedState：Fiber节点上有个属性叫memorizedState，所有的Hooks都是围绕这个memorizedState来实现的，把要存的状态和函数队列存到这个属性上，然后按需求做增删改查就行了。
2. 链表存储状态：为了保证Hooks状态的序列，React采用链表来保存函数式组件的state，使用next属性来连接前后两个状态序列。

## 13. JSX和模板引擎有什么区别？

JSX：更加灵活，既可以写标签，也可以使用JS语法和表达式，在做复杂渲染时更得心应手。


模板引擎：更简单易上手，开发效率高，结合指令的可读性也比较好。


JSX太灵活就导致没法给编译器提供太多的优化线索，不好做静态优化，模板引擎可以在编译时做静态标记，性能更好。


JSX只是个编译工具，Vue经过一定的配置也可以使用。

## 14. 怎么理解Fiber和并发模式？

1. 为什么要设计并发模式？
在React的旧版本中，当组件状态发生变化时，React会将整个组件树进行递归遍历，生成新的虚拟DOM树，并与旧的虚拟DOM树进行比较，找出需要更新的部分，然后将这些部分更新到DOM中。这种遍历方式虽然简单，但是在组件树变得非常大、复杂的情况下，会导致渲染和更新性能下降，造成页面卡顿甚至无法响应用户操作的情况。为了解决这个问题，React引入了并发模式。

2. Fiber是什么？
- Fiber是一种数据结构，由VDOM转化生成。
- Fiber的思想是将组件树的遍历过程拆分成多个小的、可中断的任务，以实现更细粒度的控制和优化。
- 具体来说，Fiber将每个组件看作是一个执行单元，并将组件树转换成一棵Fiber树。每个Fiber节点都包含了组件的状态和一些额外的信息，例如优先级、副作用等。
- 在更新过程中，React会根据Fiber节点的优先级，将Fiber树转换成一个任务队列，然后按照优先级进行调度和执行。React还会利用浏览器提供的requestIdleCallback API来分配空闲时间，以避免阻塞渲染线程。
- 由于Fiber将组件树的遍历过程拆分成了多个小的、可中断的任务，因此React可以在需要更新的部分进行优化，从而提高渲染和更新的性能。例如，在执行更新任务时，React可以根据优先级调整任务的执行顺序，避免低优先级任务阻塞高优先级任务的执行，提高了应用程序的响应速度和性能。

React Fiber 是 React 库的内部重构，旨在改进 React 的渲染机制。它是在 React 16 版本中引入的一项重大变化。React Fiber 的目标是实现增量渲染，使 React 应用能够更好地处理大型和复杂的组件树，并提供更好的用户体验。

在 React 之前，React 使用了一种称为“栈调和”（Stack Reconciliation）的算法来处理组件的更新和渲染。这种算法在处理大型组件树时可能会导致性能问题，因为它是递归的，一旦开始处理组件树，就无法中断。这可能会导致长时间的 JavaScript 执行，从而阻塞用户界面的响应。

React Fiber 引入了一种新的渲染机制，它使用了一种称为“协作调和”（Concurrent Reconciliation）的算法。这种算法允许 React 在多个帧中分割工作，并根据优先级和时间片来调度和中断工作。这样，React 可以在多个帧之间分散工作，使用户界面保持响应，并允许其他高优先级的任务插入。

React Fiber 还引入了一种新的 API，称为“React Scheduler”，它允许开发人员控制更新的优先级，并将工作划分为不同的优先级。这使开发人员能够更好地控制应用程序的性能和响应性。

总之，React Fiber 是 React 库的一项内部重构，旨在改进渲染机制，实现增量渲染，并提供更好的性能和用户体验。它引入了协作调和算法和 React Scheduler API，使 React 应用能够更好地处理大型和复杂的组件树，并在多个帧之间分散工作，保持用户界面的响应*
## 15. 有了解过哪些类React框架，谈谈你对它们的看法？
Preact：可以理解为简易版React，但是和React有一样的API，性能比React还好，甚至也实现了并发模式。
Svelte：无虚拟Dom，依靠编译器和纯响应式的轻量级框架，然而性能却非常好。
SolidJS：和Svelte类似，但是SolidJS的语法更接近于React，Svelte的语法接近Vue。





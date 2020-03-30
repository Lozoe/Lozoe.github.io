---
title: react-redux源码分析
date: 2020-03-24 21:57:17
categories: [Framework]
---

## 前言

redux作为大型应用的状态管理工具，如果想配合react使用，需要借助react-redux。
redux主要完成两件事情：

- 负责应用的状态管理，保证单向数据流
- 当应用状态发生变化，触发监听器。

那么，如果想要将react和redux搭配使用，就需要react组件可以根据redux中所存储的状态（store）更新view。
并且可以改变store。其实react-redux主要就是完成了这两件事情。

第一，通过将store传入root组件的context，使子节点可以获取到state。
第二，通过store.subscribe订阅store的变化，更新组件。

另外还有对于性能的优化，减少不必要的渲染。

## react-redux使用

```js
import React from 'react';
import ReactDOM from 'react-dom';
import PropTypes from 'prop-types';
import { createStore } from 'redux';
import { Provider, connect } from 'react-redux';

const reducer = (state, action) => {
    if (action.type === 'add') {
        return {
            ...state,
            count: state.count + 1,
        }
    }
    return state;
};

const store = createStore(reducer, { count: 1 });

const mapStateToProps = (state) => {
    return ({
        count: state.count,
    });
};

const mapDispatchToProps = dispatch => ({
    add: () => dispatch({ type: 'add' }),
});

const mergeProps = (state, props) =>({
    countStr: `计数: ${state.count}`,
    ...props,
});

const options = {
    pure: true,
};

class App extends React.Component {
    render() {
        return (
            <div>
                <p>{this.props.countStr}</p>
                <button onClick={this.props.add}>点击+1</button>
            </div>
        )
    }
}

// @connect
const AppContainer = connect(
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    options,
)(App);

ReactDOM.render(
    <Provider store={store}>
        <AppContainer />
    </Provider>,
    document.getElementById('root')
);
```

Provider： 接收从redux而来的store，以供子组件使用。
connect： 高阶组件，当组件需要获取或者想要改变store的时候使用。可以接受四个参数：
    - mapStateToProps：取store数据，传递给组件。
    - mapDispatchToProps：改变store数据。
    - mergeProps：可以在其中对 mapStateToProps， mapDispatchToProps的结果进一步处理
    - options：一些配置项，例如 pure.当设置为true时，会避免不必要的渲染

## React-Redux的源码目录结构

├── components
│   ├── Provider.js
│   └── connectAdvanced.js
├── connect
│   ├── connect.js
│   ├── mapDispatchToProps.js
│   ├── mapStateToProps.js
│   ├── mergeProps.js
│   ├── selectorFactory.js
│   ├── verifySubselectors.js
│   └── wrapMapToProps.js
├── index.js
└── utils
├── PropTypes.js
├── Subscription.js
├── shallowEqual.js
├── verifyPlainObject.js
├── warning.js
└── wrapActionCreators.js

## 源码解读

基于react-redux 5.0.6

### 入口

首先来看一下index.js:

```js
import Provider, { createProvider } from './components/Provider'
import connectAdvanced from './components/connectAdvanced'
import connect from './connect/connect'

export { Provider, createProvider, connectAdvanced, connect }
```

对外提供的API有四个:Provider、createProvider,connectAdvanced,connect

### Provider部分

```js
class Provider extends Component {
    getChildContext() {
        return { [storeKey]: this[storeKey], [subscriptionKey]: null }
    }

    constructor(props, context) {
        super(props, context)
        this[storeKey] = props.store;
    }

    render() {
        return Children.only(this.props.children)
    }
}
```

Provider组件写了3个方法，getChildContext、constructor、render。目的是使用context向下传递store。将store注入整个React应用的某个入口组件，通常是应用的顶层组件。

constructor是构造方法，this.store = props.store中的this表示当前的组件。在构造函数定义this.store的作用是为了能够在getChildContext方法中读取到store。

getChildContext方法，翻译过来就是上下文。作用是返回store。

render渲染，返回一个react子元素。Children.only是react提供的方法，this.props.children表示的是只有一个root的元素。

### Connect部分

```js
export function createConnect({
    connectHOC = connectAdvanced,
    mapStateToPropsFactories = defaultMapStateToPropsFactories,
    mapDispatchToPropsFactories = defaultMapDispatchToPropsFactories,
    mergePropsFactories = defaultMergePropsFactories,
    selectorFactory = defaultSelectorFactory
} = {}) {
    return function connect(
        mapStateToProps,
        mapDispatchToProps,
        mergeProps,
        {
            pure = true,
            areStatesEqual = strictEqual,
            areOwnPropsEqual = shallowEqual,
            areStatePropsEqual = shallowEqual,
            areMergedPropsEqual = shallowEqual,
            ...extraOptions
        } = {}
    ) {
        // 封装了传入的mapStateToProps等函数，这里可以先理解为 initMapStateToProps = () => mapStateToProps 这种形式
        const initMapStateToProps = match(mapStateToProps, mapStateToPropsFactories, 'mapStateToProps')
        const initMapDispatchToProps = match(mapDispatchToProps, mapDispatchToPropsFactories, 'mapDispatchToProps')
        const initMergeProps = match(mergeProps, mergePropsFactories, 'mergeProps')

        // 选择器（selector）的作用就是计算mapStateToProps，mapDispatchToProps, ownProps（来自父组件的props）的结果，
        // 并将结果传给子组件props。这就是为什么你在mapStateToProps等三个函数中写的结果，子组件可以通过this.props拿到。
        // 选择器工厂函数（selectorFactory）作用就是创建selector
        return connectHOC(selectorFactory, {
            // used in error messages
            methodName: 'connect',

            // used to compute Connect's displayName from the wrapped component's displayName.
            getDisplayName: name => `Connect(${name})`,

            // if mapStateToProps is falsy, the Connect component doesn't subscribe to store state changes
            shouldHandleStateChanges: Boolean(mapStateToProps),

            // passed through to selectorFactory
            initMapStateToProps,
            initMapDispatchToProps,
            initMergeProps,
            pure,
            areStatesEqual,
            areOwnPropsEqual,
            areStatePropsEqual,
            areMergedPropsEqual,

            // any extra options args can override defaults of connect or connectAdvanced
            ...extraOptions
        })
    }
}

export default createConnect()
```

connect是对connectHOC(connectAdvanced)的封装，connectAdvanced使用了defaultSelectorFactory，来创建selector。
react-redux默认的selectorFactory中包含了很多性能优化的部分

#### connectHOC(connectAdvanced)

connectAdvanced位于src / components / connectAdvanced.js，说明作者把它看做更接近于组件相关，它是Connect的基础。

connectAdvanced的签名如下

```js
(selectorFactory, options) => WrappedComponent => HoistedComponent
```

用语言描述就是，connectAdvanced这个方法接受一个selectorFactory方法和一个配置对象，返回一个HOC方法，这个方法接收一个组件参数，并定义了一个叫Connect的组件，然后通过hoist-non-react-statics这个库，把WrappedComponent里面的非React的静态属性方法复制给Connect组件并返回。


#### selectorFactory部分

在解读selectorFactory代码之前，先说一下options.pure的作用。在开头的例子有一个配置项pure。作用是减少运算优化性能。当设置为false时，react-redux将不会优化，store.subscirbe事件触发，组件就会渲染，即使是没有用到的state更新，也会这样，举个例子

```js
// 大家都知道reducer的写法
return {
    ...state,
    count: state.count + 1,
}

// 必须返回一个新对象，那么如果我只是修改原来state比如
state.count += 1;
return state;
// 你会发现组件将不会渲染，其实store里的值是发生变化的。
// 这时如果你非想这么写， 然后又想重新渲染怎么办？
// 就是将pure设置为false，这样组件就会完全实时更新。
```

看下selectorFactory.js finalPropsSelectorFactory作为最终导出的代码

```js
export default function finalPropsSelectorFactory(dispatch, {
    initMapStateToProps,
    initMapDispatchToProps,
    initMergeProps,
    ...options
}) {
    const mapStateToProps = initMapStateToProps(dispatch, options)
    const mapDispatchToProps = initMapDispatchToProps(dispatch, options)
    const mergeProps = initMergeProps(dispatch, options)

    if (process.env.NODE_ENV !== 'production') {
        verifySubselectors(mapStateToProps, mapDispatchToProps, mergeProps, options.displayName)
    }

    const selectorFactory = options.pure
        ? pureFinalPropsSelectorFactory
        : impureFinalPropsSelectorFactory

    return selectorFactory(
        mapStateToProps,
        mapDispatchToProps,
        mergeProps,
        dispatch,
        options
    )
}
```

这里很好理解，是对selectorFactory的封装，根据options.pure的值，选取不同的SelectorFactory;

##### impureFinalPropsSelectorFactory

先来看看简单的impureFinalPropsSelectorFactory，无脑计算返回新值。

```js
export function impureFinalPropsSelectorFactory(
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    dispatch
) {
    return function impureFinalPropsSelector(state, ownProps) {
        return mergeProps(
            mapStateToProps(state, ownProps),
            mapDispatchToProps(dispatch, ownProps),
            ownProps
        )
    }
}
```

##### pureFinalPropsSelectorFactory

在需要的时候才执行mapStateToProps，mapDispatchToProps，mergeProps

```js
export function pureFinalPropsSelectorFactory(
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    dispatch,
    { areStatesEqual, areOwnPropsEqual, areStatePropsEqual }
) {
    let hasRunAtLeastOnce = false
    let state
    let ownProps
    let stateProps
    let dispatchProps
    let mergedProps

    function handleFirstCall(firstState, firstOwnProps) {
        state = firstState
        ownProps = firstOwnProps
        stateProps = mapStateToProps(state, ownProps)
        dispatchProps = mapDispatchToProps(dispatch, ownProps)
        mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
        hasRunAtLeastOnce = true
        return mergedProps
    }

    function handleNewPropsAndNewState() {
        stateProps = mapStateToProps(state, ownProps)

        if (mapDispatchToProps.dependsOnOwnProps)
            dispatchProps = mapDispatchToProps(dispatch, ownProps)

        mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
        return mergedProps
    }

    function handleNewProps() {
        if (mapStateToProps.dependsOnOwnProps)
            stateProps = mapStateToProps(state, ownProps)

        if (mapDispatchToProps.dependsOnOwnProps)
            dispatchProps = mapDispatchToProps(dispatch, ownProps)

        mergedProps = mergeProps(stateProps, dispatchProps, ownProps)
        return mergedProps
    }

    function handleNewState() {
        const nextStateProps = mapStateToProps(state, ownProps)
        const statePropsChanged = !areStatePropsEqual(nextStateProps, stateProps)
        stateProps = nextStateProps

        if (statePropsChanged)
            mergedProps = mergeProps(stateProps, dispatchProps, ownProps)

        return mergedProps
    }

    function handleSubsequentCalls(nextState, nextOwnProps) {
        const propsChanged = !areOwnPropsEqual(nextOwnProps, ownProps)
        const stateChanged = !areStatesEqual(nextState, state)
        state = nextState
        ownProps = nextOwnProps

        if (propsChanged && stateChanged) return handleNewPropsAndNewState()
        if (propsChanged) return handleNewProps()
        if (stateChanged) return handleNewState()
        return mergedProps
    }

    return function pureFinalPropsSelector(nextState, nextOwnProps) {
        return hasRunAtLeastOnce
            ? handleSubsequentCalls(nextState, nextOwnProps)
            : handleFirstCall(nextState, nextOwnProps)
    }
}
```

分两种情况：

- 当第一次运行时，执行 handleFirstCall，将三个map函数运行一遍，并将结果缓存下来
- 之后运行 handleSubsequentCalls，在其中将新的值和缓存的值做比较，如果变化，将重新求值并返回，如果没变化，返回缓存的旧值。

其中 mapFunction.dependsOnOwnProps 代表你传入的mapStateToProps是否使用了ownProps。
如果没有使用，那么props的变化将不会影响结果，换句话说对应的mapFunction将不会执行。
判断方法也很简单，就是获取function形参长度，如何获得呢？



https://juejin.im/post/5b8b5a60e51d4538c411ff12
https://github.com/reduxjs/react-redux/blob/v5.0.6/src/components/Provider.js
https://juejin.im/post/59eb25e651882549fc512bb7
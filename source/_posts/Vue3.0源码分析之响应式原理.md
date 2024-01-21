---
title: Vue3.0源码分析之响应式原理
date: 2024-01-07 22:35:24
categories: [Framework]
tags:
  - vue
  - 响应式原理
---

## Vue 3.0 的响应式

Vue 3.0 的响应式系统是独立的模块，可以完全脱离 Vue 使用，可以直接在 vue-next [packages/reactivity](https://github.com/vuejs/vue-next) 模块下调试。

<!-- more -->

### 首先认识 Proxy

简单调试办法
步骤：

1、clone 项目到本地`git clone https://github.com/vuejs/vue-next.git`
2、cd 到项目根目录 执行 `yarn`
3、根目录 执行 `yarn dev reactivity`
4、cd 到`packages/reactivity` 目录，可以看到 `dist/reactivity.global.js`大概 946 行代码，在此目录下创建 index.html

```js
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>
    <script src="./dist/reactivity.global.js"></script>
    <script>
        const { reactive, effect } = VueReactivity

        const origin = {
            count: 0
        }
        const state = reactive(origin)

        const fn = () => {
            const count = state.count
            console.log(`set count to ${count}`)
        }
        effect(fn)
    </script>
</body>

</html>
```

### 响应式整体思路

这里直接给结论，简单概括为

- 初始化阶段
- 依赖收集阶段
- 响应阶段

## 看下 vue3.0 reactivity 暴露了哪些？

通过在 repo 根目录执行 yarn build reactivity --types 可在 `temp/reactivity.api.md` 处生成 API 报告。(去看一眼代码)

## API Report File for "@vue/reactivity"

```ts
export function computed<T>(getter: ComputedGetter<T>): ComputedRef<T>;
export function computed<T>(
  options: WritableComputedOptions<T>
): WritableComputedRef<T>;
export type ComputedGetter<T> = (ctx?: any) => T;
export interface ComputedRef<T = any> extends WritableComputedRef<T> {
  // (undocumented)
  readonly value: T;
}
export type ComputedSetter<T> = (v: T) => void;

// Warning: (ae-forgotten-export) The symbol "CustomRefFactory" needs to be exported by the entry point index.d.ts
export function customRef<T>(factory: CustomRefFactory<T>): Ref<T>;

// Warning: (ae-forgotten-export) The symbol "DebuggerEventExtraInfo" needs to be exported by the entry point index.d.ts
export type DebuggerEvent = {
  effect: ReactiveEffect;
  target: object;
  type: TrackOpTypes | TriggerOpTypes;
  key: any;
} & DebuggerEventExtraInfo;

// Warning: (ae-forgotten-export) The symbol "Builtin" needs to be exported by the entry point index.d.ts
export type DeepReadonly<T> = T extends Builtin
  ? T
  : T extends Map<infer K, infer V>
  ? ReadonlyMap<DeepReadonly<K>, DeepReadonly<V>>
  : T extends ReadonlyMap<infer K, infer V>
  ? ReadonlyMap<DeepReadonly<K>, DeepReadonly<V>>
  : T extends WeakMap<infer K, infer V>
  ? WeakMap<DeepReadonly<K>, DeepReadonly<V>>
  : T extends Set<infer U>
  ? ReadonlySet<DeepReadonly<U>>
  : T extends ReadonlySet<infer U>
  ? ReadonlySet<DeepReadonly<U>>
  : T extends WeakSet<infer U>
  ? WeakSet<DeepReadonly<U>>
  : T extends Promise<infer U>
  ? Promise<DeepReadonly<U>>
  : T extends {}
  ? {
      readonly [K in keyof T]: DeepReadonly<T[K]>;
    }
  : Readonly<T>;
export function effect<T = any>(
  fn: () => T,
  options?: ReactiveEffectOptions
): ReactiveEffect<T>;
export function enableTracking(): void;
export function isProxy(value: unknown): boolean;
export function isReactive(value: unknown): boolean;
export function isReadonly(value: unknown): boolean;
export function isRef<T>(r: Ref<T> | unknown): r is Ref<T>;
export const ITERATE_KEY: unique symbol;
export function markRaw<T extends object>(value: T): T;
export function pauseTracking(): void;
export function proxyRefs<T extends object>(
  objectWithRefs: T
): ShallowUnwrapRef<T>;

// Warning: (ae-forgotten-export) The symbol "UnwrapNestedRefs" needs to be exported by the entry point index.d.ts
//
// @public
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>;
export interface ReactiveEffect<T = any> {
  // (undocumented)
  (): T;
  // (undocumented)
  active: boolean;
  // (undocumented)
  allowRecurse: boolean;
  // Warning: (ae-forgotten-export) The symbol "Dep" needs to be exported by the entry point index.d.ts
  //
  // (undocumented)
  deps: Array<Dep>;
  // (undocumented)
  id: number;
  // (undocumented)
  _isEffect: true;
  // (undocumented)
  options: ReactiveEffectOptions;
  // (undocumented)
  raw: () => T;
}
export interface ReactiveEffectOptions {
  // (undocumented)
  allowRecurse?: boolean;
  // (undocumented)
  lazy?: boolean;
  // (undocumented)
  onStop?: () => void;
  // (undocumented)
  onTrack?: (event: DebuggerEvent) => void;
  // (undocumented)
  onTrigger?: (event: DebuggerEvent) => void;
  // (undocumented)
  scheduler?: (job: ReactiveEffect) => void;
}
export const enum ReactiveFlags {
  // (undocumented)
  IS_REACTIVE = "__v_isReactive",
  // (undocumented)
  IS_READONLY = "__v_isReadonly",
  // (undocumented)
  RAW = "__v_raw",
  // (undocumented)
  SKIP = "__v_skip",
}

// @public
export function readonly<T extends object>(
  target: T
): DeepReadonly<UnwrapNestedRefs<T>>;
export interface Ref<T = any> {
  [RefSymbol]: true;
  // @internal (undocumented)
  _shallow?: boolean;
  // (undocumented)
  value: T;
}

// Warning: (ae-forgotten-export) The symbol "ToRef" needs to be exported by the entry point index.d.ts
export function ref<T extends object>(value: T): ToRef<T>;
export function ref<T>(value: T): Ref<UnwrapRef<T>>;
export function ref<T = any>(): Ref<T | undefined>;

// @public
export interface RefUnwrapBailTypes {}
export function resetTracking(): void;

// @public
export function shallowReactive<T extends object>(target: T): T;

// @public
export function shallowReadonly<T extends object>(
  target: T
): Readonly<{
  [K in keyof T]: UnwrapNestedRefs<T[K]>;
}>;
export function shallowRef<T extends object>(
  value: T
): T extends Ref ? T : Ref<T>;
export function shallowRef<T>(value: T): Ref<T>;
export function shallowRef<T = any>(): Ref<T | undefined>;
export type ShallowUnwrapRef<T> = {
  [K in keyof T]: T[K] extends Ref<infer V> ? V : T[K];
};
function stop_2(effect: ReactiveEffect): void;

export { stop_2 as stop };
export function toRaw<T>(observed: T): T;
export function toRef<T extends object, K extends keyof T>(
  object: T,
  key: K
): ToRef<T[K]>;
export type ToRefs<T = any> = {
  [K in keyof T]: T[K] extends Ref ? T[K] : Ref<UnwrapRef<T[K]>>;
};
export function toRefs<T extends object>(object: T): ToRefs<T>;
export function track(target: object, type: TrackOpTypes, key: unknown): void;
export const enum TrackOpTypes {
  // (undocumented)
  GET = "get",
  // (undocumented)
  HAS = "has",
  // (undocumented)
  ITERATE = "iterate",
}
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown,
  oldTarget?: Map<unknown, unknown> | Set<unknown>
): void;
export const enum TriggerOpTypes {
  // (undocumented)
  ADD = "add",
  // (undocumented)
  CLEAR = "clear",
  // (undocumented)
  DELETE = "delete",
  // (undocumented)
  SET = "set",
}
export function triggerRef(ref: Ref): void;
export function unref<T>(ref: T): T extends Ref<infer V> ? V : T;

// Warning: (ae-forgotten-export) The symbol "UnwrapRefSimple" needs to be exported by the entry point index.d.ts
export type UnwrapRef<T> = T extends Ref<infer V>
  ? UnwrapRefSimple<V>
  : UnwrapRefSimple<T>;
export interface WritableComputedOptions<T> {
  // (undocumented)
  get: ComputedGetter<T>;
  // (undocumented)
  set: ComputedSetter<T>;
}
export interface WritableComputedRef<T> extends Ref<T> {
  // (undocumented)
  readonly effect: ReactiveEffect<T>;
}
```

## 响应式源码分析

### 初始化阶段

初始化阶段核心其实就是 reactive 调用和 effect 调用
直接放核心代码：

#### reactive 函数代码片段

```js
export function reactive(target) {
  const observed = new Proxy(target, handler);
  return observed;
}
```

#### reactive 函数干了啥

在这里咱们先不管 handler（后面再详细说）, reactive 干了一件事儿，把 origin 对象转化成响应式的 Proxy 对象

#### effect 函数代码片段

```js
export function effect(fn) {
  // 构造一个 effect
  const effect = function effect(...args) {
    return run(effect, fn, args);
  };
  // 立即执行一次
  effect();
  return effect;
}
const effectStack = [];
export function run(effect, fn, args) {
  if (effectStack.indexOf(effect) === -1) {
    try {
      // 往池子里放入当前 effect
      effectStack.push(effect);
      // 立即执行一遍 fn()
      // fn() 执行过程会完成依赖收集，会用到 effect
      return fn(...args);
    } finally {
      // 完成依赖收集后从池子中扔掉这个 effect
      effectStack.pop();
    }
  }
}
```

当一个普通的函数 fn 被 函数 effect 包裹之后，就会变成一个响应式的 effect 函数，而 fn 也会被立即执行一次。
由于在 fn 里面有引用到 Proxy 对象的属性，所以这一步会触发对象的 getter，从而启动依赖收集。
除此之外，这个 effect 函数也会被压入一个名为”activeReactiveEffectStack“（此处为 effectStack）的栈中，供后续依赖收集的时候使用。

#### effect 函数干了啥

把函数 fn 作为一个响应式的 effect 函数

### 依赖收集阶段

#### 依赖收集触发时机

从图上其实可以看出这个阶段的触发时机，就是在 effect 被立即执行，其内部的 fn 触发了 Proxy 对象的 getter 的时候。简单来说，只要执行到类似 `state.count `的语句，就会触发 state 的 getter。

#### 依赖收集目的

建立一份”依赖收集表“，也就是图示的”targetMap"。

targetMap 是一个 WeakMap，其 key 值是当前的 Proxy 对象(state)代理前的对象(origin)，而 value 则是该对象所对应的 depsMap(我叫它观察者的集合)。

强行解释 depsMap: depsMap 是一个 Map，key 值为触发 getter 时的属性值（此处为 count），而 value 则是触发过该属性值所对应的各个 effect。举个栗子：
<img src="effect.png" alt="effect" width="500">
<img src="depsMap.png" alt="depsMap" width="500">

这样，「target => key => dep」 的对应关系就建立起来了，依赖收集也就完成了

```js
export function track(target, operationType, key) {
  const effect = effectStack[effectStack.length - 1];
  if (effect) {
    let depsMap = targetMap.get(target);
    if (depsMap === void 0) {
      targetMap.set(target, (depsMap = new Map()));
    }

    let dep = depsMap.get(key);
    if (dep === void 0) {
      depsMap.set(key, (dep = new Set()));
    }

    if (!dep.has(effect)) {
      dep.add(effect);
    }
  }
}
```

弄明白依赖收集表 targetMap 是非常重要的，因为这是整个响应式系统核心中的核心。

响应阶段

回顾上一章节的例子，我们得到了一个 { count: 0, age: 18 } 的 Proxy，并构造了三个 effect。在控制台上看看效果：
<img src="demo.png" alt="demo" width="200">

效果符合预期，那么它是怎么实现的呢？首先来看看这个阶段的原理图：
<img src="原理.png" alt="原理" width="500">

当修改对象的某个属性值的时候，会触发对应的 setter。

setter 里面的 trigger() 函数会从依赖收集表里找到当前属性对应的各个 dep，然后把它们推入到 effects 和 computedEffects（计算属性） 队列中，最后通过 scheduleRun() 挨个执行里面的 effect。

由于已经建立了依赖收集表，所以要找到属性所对应的 dep 也就轻而易举了，可以看看具体的代码实现：

```js
export function trigger(target, operationType, key) {
  // 取得对应的 depsMap
  const depsMap = targetMap.get(target);
  if (depsMap === void 0) {
    return;
  }
  // 取得对应的各个 dep
  const effects = new Set();
  if (key !== void 0) {
    const dep = depsMap.get(key);
    dep &&
      dep.forEach((effect) => {
        effects.add(effect);
      });
  }
  // 简化版 scheduleRun，挨个执行 effect
  effects.forEach((effect) => {
    effect();
  });
}
```

这里的代码没有处理诸如数组的 length 被修改的一些特殊情况，感兴趣的读者可以查看 vue-next 对应的源码，或者[这篇文章](https://juejin.cn/post/6844903957807169549?utm_source=gold_browser_extension#heading-2)，看看这些情况都是怎么处理的。

至此，响应式阶段完成。

## API 模拟实现

### reactive 实现

```js
/**
 *
 * @description 生成响应式对象
 * @param {any} target
 * @returns
 */
function reactive(target) {
  // 创建响应式对象
  return createReactiveObject(target);
}

/**
 *
 * @description 判断是不是object
 * @param {any} target
 * @returns {boolean}
 */
function isObject(target) {
  return typeof target === "object" && target !== null;
}

/**
 *
 * @description 创造响应式对象
 * @param {any} target
 * @returns
 */
function createReactiveObject(target) {
  // 判断target是不是对象,不是对象直接返回
  if (!isObject(target)) {
    return target;
  }

  // get set delete ...对象方法
  const handlers = {
    get(target, key, receiver) {
      // 取值
      let res = Reflect.get(target, key, receiver);
      return res;
    },
    set(target, key, value, receiver) {
      // 更改/新增属性
      let result = Reflect.set(target, key, value, receiver);
      return result;
    },
    deleteProperty(target, key) {
      // 删除属性
      const result = Reflect.deleteProperty(target, key);
      return result;
    },
  };
  // 开始代理
  observed = new Proxy(target, handlers);
  return observed;
}
let p = reactive({ name: "cangshudada" });
console.log(p.name); // 取值
p.name = "仓鼠大大"; // 设置
delete p.name; // 删除
```

但是可能存在这样的对象

```js
const person = {
  name: "cangshudada",
  age: 24,
  pets: {
    dog: {
      name: "guagua",
      age: 1,
    },
    cat: {
      name: "gugu",
      age: 2,
    },
  },
};
```

所以我们得继续实现`多层对象嵌套`情况下的代理：

```
get(target, key, receiver) {
    // 取值
    const res = Reflect.get(target, key, receiver);
    return isObject(res) ? reactive(res) : res; // 懒代理，只有当取值时再次做代理，vue2.0中一上来就会全部递归增加getter,setter
}
```

我们继续考虑`数组`的情况
Proxy 默认是可以支持数组，所以我们不需要像 Vue2.x 中一样对数组封装自己的方法并在其中来劫持监听数据改变，但是我们改变数组的时候仍然能够发现问题，那就是`数组的改变会触发两次set，分别是数组的长度变化以及索引值的变化`，接下来我们就需要屏蔽掉多次触发的问题。

```js
const toProxy = new WeakMap(); // 存放被代理过的对象
const toRaw = new WeakMap(); // 存放已经代理过的对象

/**
 *
 * @description 生成响应式对象
 * @param {any} target
 * @returns
 */
function reactive(target) {
    // 创建响应式对象
    return createReactiveObject(target);
}


/**
 *
 * @description 判断是不是object
 * @param {any} target
 * @returns {boolean}
 */
function isObject(target) {
    return typeof target === "object" && target !== null;
}


/**
 *
 * @description 判断对象中是否有该键
 * @param {object} target
 * @param {object} key
 * @returns {boolean}
 */
function hasOwn(target, key) {
    return target.hasOwnProperty(key);
}

/**
 *
 * @description 创造响应式对象
 * @param {any} target
 * @returns
 */
function createReactiveObject(target) {

    // 是否是对象
    if (!isObject(target)) {
        return target;
    }

    // 判断取到被代理的对象
    let observed = toProxy.get(target);

    if (observed) { // 判断是否被代理过
        return observed;
    }
    if (toRaw.has(target)) { // 判断重复代理的情况,如果重复代理
        return target;
    }

    const handlers = {
        get(target, key, receiver) {
            // 取值
            const res = Reflect.get(target, key, receiver);
            track(target, 'get', key)；//收集依赖
            return isObject(res) ? reactive(res) : res; // 懒代理，只有当取值时再次做代理，vue2.0中一上来就会全部递归增加getter,setter
        },
        set(target, key, value, receiver) {
            const oldValue = target[key];
            const hadKey = hasOwn(target, key);
            const result = Reflect.set(target, key, value, receiver);
            // 判断是否是新增还是修改的情况
            if (!hadKey) { //无key的情况则是新增
                trigger(target, 'add', key) // 触发依赖更新 - 增加
            } else if (oldValue !== value) { //防止数组重复操作修改索引或者length的时候多次触发set
                trigger(target, 'set', key) // 触发依赖更新 - 修改
            }
            return result;
        },
        deleteProperty(target, key) {
            trigger(target, 'delete', key)；// 触发依赖更新 - 删除
            const result = Reflect.deleteProperty(target, key);
            return result;
        }
    };

    // 开始代理
    observed = new Proxy(target, handlers);
    toProxy.set(target, observed);
    toRaw.set(observed, target); // 做映射表
    return observed;
}

// 对象的情况
const person = reactive({ name: 'cangshudada' });
console.log('person.name >>', person.name); // 获取
person.name = '仓鼠大大'; // 设置
delete person.name; // 删除
person.age = 12;//能够代理到直接在对象增加的键
person.age = 24

// 能够直接代理数组以及重复代理的情况
const ary = reactive([1, 2, 3, 4]);
ary.push(5)
const ary1 = reactive(ary); //此时重复代理会直接返回之前代理过的对象
```

### effect 实现

effect 也就是副作用的意思，这个方法默认会在调用的时候率先执行一次，之后如果数据有变化后则会再次触发此回调函数。

```js
const person = Vue.reactive({ name: "cangshudada" }); //person对象已经成为响应式数据
Vue.effect(() => {
  // effect方法会立即触发一次
  console.log(person.name);
});

person.name = "仓鼠大大"; // 当属性修改后会再次触发effect方法
```

我们先来实现`effect`函数

```js
/**
 *
 * @description effect函数
 * @param {function} fn 回调函数
 * @returns
 */
function effect(fn) {
  const effect = createReactiveEffect(fn); // 创建响应式的effect
  effect(); // 首先执行一次
  return effect;
}

// 存放响应式effect
const activeReactiveEffectStack = [];

/**
 *
 *
 * @param {function} fn 回调函数
 * @returns
 */
function createReactiveEffect(fn) {
  const effect = function () {
    // 响应式的effect
    return run(effect, fn);
  };
  return effect;
}

/**
 *
 * @param {function} effect 响应式的effect
 * @param {function} fn 回调函数
 * @returns
 */
function run(effect, fn) {
  try {
    activeReactiveEffectStack.push(effect);
    return fn(); // 先让fn执行,执行时会触发get方法，可以将effect存入对应的key属性
  } finally {
    activeReactiveEffectStack.pop(effect);
  }
}
```

当调用 fn()时可能会触发 get 方法，此时会触发上面 get 中调用的 track 函数

```js
const targetMap = new WeakMap();

function track(target, type, key) {
  // 查看是否有effect
  const effect =
    activeReactiveEffectStack[activeReactiveEffectStack.length - 1];
  if (effect) {
    const depsMap = targetMap.get(target);
    if (!depsMap) {
      // 如果不存在依赖数组对象则添加Map对象
      targetMap.set(target, (depsMap = new Map()));
    }
    const deps = depsMap.get(target);
    if (!deps) {
      // 如果deps不存在则增加Set数组
      depsMap.set(key, (deps = new Set()));
    }
    if (!deps.has(effect)) {
      // 如果deps中没有这个effect就将effect添加到依赖数组中
      deps.add(effect);
    }
  }
}
```

当更新属性时会触发 trigger 执行，并根据 key 值找到对应的存储集合中的 effect 依次执行

```js
function trigger(target, type, key) {
  const depsMap = targetMap.get(target);
  if (!depsMap) {
    return;
  }
  const deps = depsMap.get(key);
  if (deps) {
    deps.forEach((effect) => {
      effect();
    });
  }
}
```

这个时候其实还存在 length 的问题，比如我们在 effect 中监听数组的 length，这个时候因为我们上面在 set 函数中设置了 length 改变不触发 trigger 函数的机制，所以还需要在 trigger 中增加判断来兼容这种情况
```js
function trigger(target, type, key) {
  const depsMap = targetMap.get(target);
  if (!depsMap) {
    return;
  }
  const deps = depsMap.get(key);
  if (deps) {
    deps.forEach(effect => {
      deps();
    });
  }
  // 兼容处理当前更新类型是增加时，如果用到数组的length的effect应该也会被执行
  if (type === "add") {
    const lengthDeps = depsMap.get("length");
    if (lengthDeps) {
      lengthDeps.forEach(effect => {
        effect();
      });
    }
  }
}
```

### ref 实现
ref 可以将原始数据类型同样转换成响应式数据，这个时候需要通过`.value` 属性获取值
```js
/*
*
* @description 不同类型的数据响应式处理 如果是对象通过reactive函数进行数据绑定否则直接返回
*/
function convert(target) {
  return isObject(target) ? reactive(target) : target;
}

function ref(raw) {
  raw = convert(raw);
  const v = {
    _isRef: true, // 标识是ref类型
    get value() {
      track(v, "get", "");
      return raw;
    },
    set value(newVal) {
      raw = newVal;
      trigger(v,'set','');
    }
  };
  return v;
}
```

这个时候问题又来了，假如出现如下情况，则每次调用都得多加一个.value 就会非常麻烦，所以我们也得对这种情况做个兼容
```js
const name = ref('cangshudada');
const person = reactive({
c_Name: name
});
console.log(person.c_Name.value); // 每次调用 c.a 都得加上.value 比较麻烦
```
这个时候需要在 get 函数中兼容
```js
get(target, key, receiver) {
// 取值
const res = Reflect.get(target, key, receiver);
// 兼容 ref 的 value 情况 因为前面的判断所以 ref 不可能为对象 可以直接返回
if(res.\_isRef){
return res.value
}
track(target, 'get', key)；//收集依赖
return isObject(res) ? reactive(res) : res; // 懒代理
}
```

### computed 实现
之前版本的 computed 函数会缓存监听变量的值，只有当监听的变量值发生变化函数才会触发，在实际项目中用处非常大，如今 vue3.0 响应式数据机制重写，也导致了 computed 的重写，我们来看看在 vue3.0 computed 是如何实现的，首先我们来看看用法
```js
const person = reactive({name:'cangshudada'});
const _computed = computed(()=>{
  console.log('computed执行了')  
  return `${person.name} --- xixi`;
})
// 不取_computed.value值则回调函数不执行，除非监听对象改变则取n次只执行一次
console.log(_computed.value);// computed执行了 cangshudada --- xixi
console.log(_computed.value);// cangshudada --- xixi
person.name = '仓鼠大大';
console.log(_computed.value);// computed执行了 仓鼠大大 --- xixi
```

computed 代码
```js
function computed(fn){
  let dirty = true; // 第一次取值会触发
  const runner = effect(fn,{ // 标识这个effect是懒执行
    lazy:true, // 懒执行
    scheduler:()=>{ // 当依赖的属性变化了，调用此方法，而不是重新执行effect 依赖不更新则不更新dirty，进而不会触发runner()，缓存机制
      dirty = true;
    }
  });
  let value;
  return {
    _isRef:true,
    get value(){
      if(dirty){
        value = runner(); // 执行runner会继续收集依赖
        dirty = false;
      } 
      return value; // value没变化不会执行computed回调
    }
  }
}
```


修改 `effect` 函数 此处建议结合`effect实现`查看
```js
function effect(fn,options) {
  let effect = createReactiveEffect(fn, options);
  if(!options.lazy){ 
    effect();
  }
  return effect;
}

function createReactiveEffect(fn,options) {
  const effect = function() {
    return run(effect, fn);
  };
  effect.scheduler = options.scheduler;

  return effect;
}
```

在 `trigger` 时判断
```js
deps.forEach(effect => {
  if(effect.scheduler){ // 如果有scheduler 说明不需要执行effect
    effect.scheduler(); // 将dirty设置为true,下次获取值时变可以重新执行runner方法
  }else{
    effect(); // 否则正常执行effect即可
  }
});
```

```js
const person = reactive({name:'cangshudada'});
const \_computed = computed(()=>{
console.log('computed 执行了')  
 return `${person.name} --- xixi`;
})
// 不取\_computed.value 值则回调函数不执行，除非监听对象改变则取 n 次只执行一次
console.log(\_computed.value);
person.name = '仓鼠大大'; // 更改值 不会触发重新计算,但是会将 dirty 变成 true
console.log(\_computed.value); // 此时触发 get 函数进而调用 runner()重新调用计算方法
```
至此我们就将 Vue3.0 源码中的 reactivity 部分解析完毕了！了解了 vue 的数据绑定机制对于之后不管是面试还是后期的应用都有着很大的帮助，当然本篇文章只是对这部分进行了简要地解析，清楚了数据绑定这部分的逻辑与思想后再来读源码这部分相信各位会有更多的收获。

提问：

- 提问一：如何判定在什么情况下去收集该值的依赖？
- 提问二：如何判定在什么情况下去触发该值的依赖
- 提问三：该值的依赖收集后要存储在什么地方？
- 提问四：markRaw shallowRef shallowReactive toRefs

通过一个实例化类 Dep 来管理该值的更新机制，在该值被调用的时刻去收集该值的依赖，在该值变更的时刻去触发该值的所有依赖
<img src="dep.png" alt="dep" width="500">
其他参考链接：
https://segmentfault.com/a/1190000023465134
https://segmentfault.com/a/1190000020629159
https://www.infoq.cn/article/hzmbaolyeqanup0ycpzx
https://juejin.cn/post/6972350540210503693#heading-6

http://www.uigame.net/article/362987.html

https://juejin.cn/post/6972350540210503693
https://jishuin.proginn.com/p/763bfbd385e2

Vue2.0： https://juejin.cn/post/6857669921166491662
Composition Api vs Options API: https://blog.csdn.net/liuliuliuliumin123/article/details/113825310
compoted: https://www.cnblogs.com/xiaoheibanfe/p/14131508.html
computed: https://juejin.cn/post/6979055167228346376

vue3 更新机制的理解：https://juejin.cn/post/6912419591847313422

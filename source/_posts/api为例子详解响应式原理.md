---
layout: '入门:'
title: api为例子详解响应式原理
date: 2024-01-21 16:42:26
categories: [Framework]
tags:
  - vue
  - 响应式原理
---

## 需要解决什么问题(同vue2.0 老三样)
1. 数据劫持
2. 依赖收集
3. 派发更新
<!-- more -->
接下来咱们以reactive(其他比如ref computed都是以reactive为基础)为例子去看 在vue3.0里面是如何一步步完成响应式的。

### 第一步 数据劫持

vue3.0做数据代理就是通过 es6 Proxy这个api来完成的
Vue3.0 对于数据的响应式挟持，统一使用 composition API 来实现，对最典型的 reactive来讲解

#### 铺垫

依赖缓存的数据结构 (一对多对多的结构模式)

<img src="关系.png" alt="关系" width="500">

#### reactive

```js
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // 如果目标数据已经被 readonly() 封装过了，则直接返回，不对其进行响应式处理
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  // 通用createReactiveObject函数进行响应式处理
  return createReactiveObject(
    target, // 目标数据
    false, // 是否做只读操作
    mutableHandlers, // Object/Array 类型的代理处理器
    mutableCollectionHandlers // Map/Set/WeakMap/WeakSet 集合类型的代理处理器
  )
}
```

#### createReactiveObject

##### 前置函数或者依赖

这个先不看 createReactiveObject用到的时候再来看下

```js
export const reactiveMap = new WeakMap<Target, any>()
export const readonlyMap = new WeakMap<Target, any>()
const enum TargetType {
  INVALID = 0,
  COMMON = 1,
  COLLECTION = 2
}

function targetTypeMap(rawType: string) {
  switch (rawType) {
    case 'Object':
    case 'Array':
      return TargetType.COMMON
    case 'Map':
    case 'Set':
    case 'WeakMap':
    case 'WeakSet':
      return TargetType.COLLECTION
    default:
      return TargetType.INVALID
  }
}
```

##### createReactiveObject入口函数

```js
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  // 如果不是对象，直接抛出错误并返回
  if (!isObject(target)) {
    if (__DEV__) {
      console.warn(`value cannot be made reactive: ${String(target)}`)
    }
    return target
  }

  // 如果对象已经是响应式对象，则直接返回
  // 但如果是 readonly() 一个已经是响应式的数据则不返回，继续执行
  if (
    target[ReactiveFlags.RAW] &&
    !(isReadonly && target[ReactiveFlags.IS_REACTIVE])
  ) {
    return target
  }

  // 已经被代理的缓存（WeekMap），readonly 和 reactive 两种代理方式各有一个缓存
  const proxyMap = isReadonly ? readonlyMap : reactiveMap

  // 如果对象已经被代理过了，则直接从缓存中取出
  const existingProxy = proxyMap.get(target)
  if (existingProxy) {
    return existingProxy
  }

  // 判断目标对象是否是一些特殊的或者不需要劫持的对象，如果是则直接返回
  // 并获得其数据类型：Object/Array => TargeType.COMMON、Map/Set/WeakMap/WeakSet => TargeType.COLLECTION
  const targetType = getTargetType(target)
  if (targetType === TargetType.INVALID) {
    return target
  }
    
  // 创建 Proxy 代理，如果目标对象的类型是 Map/Set/WeakMap/WeakSet 则使用专门针对集合使用的代理处理器，反之用基本处理器
  const proxy = new Proxy(
    target,
    targetType === TargetType.COLLECTION ? collectionHandlers : baseHandlers
  )
  proxyMap.set(target, proxy) // 依赖的模子有了 具体再往后看
  return proxy
}
```

#### mutableHandlers(Object/Array 代理处理器)

注意看这里
有**依赖收集track**触发逻辑，在get、has、ownKeys拦截里面
有**惰性响应式**的实现，在get拦截里面
有**派发更新trigger**触发逻辑，在set、deleteProperty拦截里面

为了方便阅读，忽略了一些在 reactive 情境下的常量值判断，比如 readOnly，shadow

```js
export const mutableHandlers: ProxyHandler<object> = {
  get(target: Target, key: string | symbol, receiver: object) {
    // ...内部常量代理
    // ReactiveFlags.IS_REACTIVE = true
    // ReactiveFlags.IS_READONLY = false
    // ReactiveFlags.RAW = target
		
    // 目标对象是否是数组
    const targetIsArray = isArray(target);

    // 调用一些特定的数组方法时的特殊处理
    if (targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver);
    }

    // 获取值
    const res = Reflect.get(target, key, receiver);

    // 如果是一些原生内置 Symbol，或者不需要跟踪的值的直接返回
    if (
      isSymbol(key)
        ? builtInSymbols.has(key as symbol)
        : isNonTrackableKeys(key)
    ) {
      return res;
    }

    // 依赖收集
    track(target, TrackOpTypes.GET, key);

    // 如果对应值已经 Ref() 过，则根据当前是不是通过正常 key 值访问一个数组来决定要不要返回原生 ref，还是其 value
    if (isRef(res)) {
      const shouldUnwrap = !targetIsArray || !isIntegerKey(key);
      return shouldUnwrap ? res.value : res;
    }

    // 如果是对象，则挟持该对象（惰性响应式的出处）
    if (isObject(res)) {
      return reactive(res);
    }

    // 返回结果
    return res;
  },
  set(target: object, key: string | symbol, value: unknown, receiver: object): boolean {
    // 旧值
    const oldValue = (target as any)[key];
    
    // 新值去除可能存在的响应式
    value = toRaw(value);
    
    // 如果旧值是 Ref 值，则传递给 Ref 处理
    if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
      oldValue.value = value;
      return true;
    }

    // 有没有对应 key 值
    const hadKey =
      isArray(target) && isIntegerKey(key)
        ? Number(key) < target.length
        : hasOwn(target, key);
    
    // 设置值
    const result = Reflect.set(target, key, value, receiver);
    
    // 派发更新
    if (target === toRaw(receiver)) {
      if (!hadKey) {
        trigger(target, TriggerOpTypes.ADD, key, value);
      } else if (hasChanged(value, oldValue)) {
        trigger(target, TriggerOpTypes.SET, key, value, oldValue);
      }
    }
    return result;
  },
  deleteProperty(target: object, key: string | symbol): boolean {
    const hadKey = hasOwn(target, key);
    const oldValue = (target as any)[key];
    const result = Reflect.deleteProperty(target, key);
    // 派发更新
    if (result && hadKey) {
      trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue);
    }
    return result;
  },
  has(target: object, key: string | symbol): boolean {
    const result = Reflect.has(target, key);
    // 如果该值不是原生内部的 Symbol 值，则进行依赖收集
    if (!isSymbol(key) || !builtInSymbols.has(key)) {
      track(target, TrackOpTypes.HAS, key);
    }
    return result;
  },
  ownKeys(target: object): (string | number | symbol)[] {
  	// 依赖收集
    track(
      target,
      TrackOpTypes.ITERATE,
      isArray(target) ? "length" : ITERATE_KEY
    );
    return Reflect.ownKeys(target);
  },
};
```

#### collectionHandlers(Map/Set/WeakMap/WeakSet 代理处理器)

由于上述四个类型修改值都是通过函数修改的，所以代理函数只拦截 get 方法，用于拦截响应对象调用了哪个操作函数，通过key判断再进行具体的依赖收集或者派发更新

注意看这里，通过key作为代理函数
有**依赖收集track**触发逻辑，在get、size、has、ownKeys、forEach拦截里面
有**派发更新trigger**触发逻辑，在add、set、deleteEntry、clear拦截里面

```js

export const mutableCollectionHandlers: ProxyHandler<CollectionTypes> = {
  get: (
    target: CollectionTypes,
    key: string | symbol,
    receiver: CollectionTypes
  ) => {
    // ...内部常量代理
    // ReactiveFlags.IS_REACTIVE = true
    // ReactiveFlags.IS_READONLY = false
    // ReactiveFlags.RAW = target
		
    // 使用对应封装的函数，来进行处理
    return Reflect.get(
      hasOwn(mutableInstrumentations, key) && key in target
        ? mutableInstrumentations
        : target,
      key,
      receiver
    )
  }
}

// 函数代理
const mutableInstrumentations: Record<string, Function> = {
  get(this: MapTypes, key: unknown) {
    return get(this, key)
  },
  get size() {
    return size((this as unknown) as IterableCollections)
  },
  has,
  add,
  set,
  delete: deleteEntry,
  clear,
  forEach: createForEach(false, false)
}

// 获取值
function get(
  target: MapTypes,
  key: unknown,
  isReadonly = false,
  isShallow = false
) {
  // 原生对象
  target = (target as any)[ReactiveFlags.RAW]
  const rawTarget = toRaw(target)
  
  // 原生 key
  const rawKey = toRaw(key)
  
  // 如果是响应式的，则对响应式 key 进行依赖收集
  if (key !== rawKey) {
    track(rawTarget, TrackOpTypes.GET, key)
  }
    
  // 对原生 key 进行依赖收集
  track(rawTarget, TrackOpTypes.GET, rawKey)
    
  // 如果目标集合存在 has 函数，则再调用 has 进行依赖收集，因为 get()隐形依赖 has，并返回响应式 key
  const { has } = getProto(rawTarget)
  if (has.call(rawTarget, key)) {
    return toReactive(target.get(key))
  } else if (has.call(rawTarget, rawKey)) {
    return toReactive(target.get(rawKey))
  }
}

function size(target: IterableCollections, isReadonly = false) {
  // 依赖收集，key 值时 Vue 内部的 Symbol('iterate')
  target = (target as any)[ReactiveFlags.RAW]
  track(toRaw(target), TrackOpTypes.ITERATE, ITERATE_KEY)
  return Reflect.get(target, 'size', target)
}

function add(this: SetTypes, value: unknown) {
  value = toRaw(value)
  const target = toRaw(this)
  // 调用原生的 has
  const proto = getProto(target)
  const hadKey = proto.has.call(target, value)
  
  // 如果不存在，则添加，并派发更新
  if (!hadKey) {
    target.add(value)
    trigger(target, TriggerOpTypes.ADD, value, value)
  }
  return this
}

function set(this: MapTypes, key: unknown, value: unknown) {
  value = toRaw(value)
  const target = toRaw(this)
  const { has, get } = getProto(target)

  // 是否已经存在对应的 key，通过传入的 key 和 key 可能存在的真实 rawKey 去分别判断
  let hadKey = has.call(target, key)
  if (!hadKey) {
    key = toRaw(key)
    hadKey = has.call(target, key)
  }

  // 取出旧值，并设置
  const oldValue = get.call(target, key)
  target.set(key, value)
  
  // 如果是新增则触发新增的更新，反之触发设置的更新
  if (!hadKey) {
    trigger(target, TriggerOpTypes.ADD, key, value)
  } else if (hasChanged(value, oldValue)) {
    trigger(target, TriggerOpTypes.SET, key, value, oldValue)
  }
  return this
}

function deleteEntry(this: CollectionTypes, key: unknown) {
  const target = toRaw(this)
  const { has, get } = getProto(target)
  
  // 同 set，是否已经存在对应的 key，通过传入的 key 和 key 可能存在的真实 rawKey 去分别判断
  let hadKey = has.call(target, key)
  if (!hadKey) {
    key = toRaw(key)
    hadKey = has.call(target, key)
  }

  // 取出旧值，并删除
  const oldValue = get ? get.call(target, key) : undefined
  const result = target.delete(key)
  
  // 触发删除的更新
  if (hadKey) {
    trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue)
  }
  return result
}

function clear(this: IterableCollections) {
  const target = toRaw(this)
  const hadItems = target.size !== 0
  const result = target.clear()
  // 触发清空的更新
  if (hadItems) {
    trigger(target, TriggerOpTypes.CLEAR, undefined, undefined, undefined)
  }
  return result
}

function forEach(
  this: IterableCollections,
  callback: Function,
  thisArg?: unknown
) {
  const observed = this as any
  const target = observed[ReactiveFlags.RAW]
  const rawTarget = toRaw(target)
 	// 基于迭代器收集依赖
  track(rawTarget, TrackOpTypes.ITERATE, ITERATE_KEY)  
 	// 使其子集具备响应式
  return target.forEach((value: unknown, key: unknown) => {
    return callback.call(thisArg, toReactive(value), toReactive(key), observed)
  })
}
```

### 第二步 依赖收集

在刚刚数据劫持的逻辑里面可以看到很多 调用track，这个就是定义在effect模块里面，作为依赖收集的功能函数。
如何知道 响应对象 依赖了哪些 数据，这个问题进一步就是 响应对象用了哪些数据。

Vue 的大体思路是这样的，比如我一个函数 fnA，里使用了 data 里的 B 和 C。
想要知道 fnA 使用了 B 和 C，那我们干脆就直接运行一下 fnA，在 B 和 C 里面等待 fnA 的获取，然后建立两者的依赖。

在看完2.0源码分析 回顾Vue2.0 里有一些概念 Watcher，Dep，target。
- Watcher 就是指 fnA
- Dep 则是存在 B 的 setter 里面的一个对象，用于存放 Watcher 集合
- target 则是现在正在进行依赖收集的 Watcher

简单粗暴来讲
new Watcher(fnA) 
   => target 等于当前 Watcher并调用 fnA
   => fnA 获取 B 的值，B 会将 target 放到自己的 Dep 上 
   => B 更新了，通知自己 Dep 上的 Watcher 重新执行 fnA

Vue3.0 思路差不多，但是实现上大有不同，因为 Vue3.0 不再随意对数据进行侵入式修改或者挟持，所以 Vue3.0 单独拎出来了一个静态变量存储依赖关系，这个变量叫做 targetMap

同时引进了一个新概念 effect，它与 Vue2.0 的 Watcher 差不多，但是概念有些转换，从 监听者 变成了 副作用，指的是值 (对应依赖) 改变后会发生的副作用

#### targetMap数据类型

```js
type Dep = Set<ReactiveEffect>
type KeyToDepMap = Map<any, Dep>
const targetMap = new WeakMap<any, KeyToDepMap>()
```
`targetMap` 的 `key` 指向的是 `A` 和 `B` 所在的对象 `Data`
`targetMap` 的 `value` 指向的是`KeyToDepMap`
	`KeyToDepMap`的 `key` 指向的是被响应式数据的键值
    `KeyToDepMap`的 `value` 存放 `effect` 里面的 `Watcher` 集合

<img src="dep.png" alt="dep" width=400>

#### effect

effect 作用相当于 Vue2.0 里面的 Watcher，做的事情相对而言化繁为简
注意看这里
computed为何是**惰性**的答案在这里

第13行 立即执行副作用函数以触发依赖的getter 从而依赖收集
```js
const effectStack: ReactiveEffect[] = [] // 全局 effect 栈
let activeEffect: ReactiveEffect | undefined // 当前激活的 effect
```
```js
export function effect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions = EMPTY_OBJ
): ReactiveEffect<T> {
  // 如果传进来的函数已经是一个 effect 了，则取出其原生的函数进行处理
  if (isEffect(fn)) { fn = fn.raw }
  
  // 创建响应式副作用
  const effect = createReactiveEffect(fn, options)
  
  // 如果不是惰性的副作用，则直接运行并依赖收集，computed 就是惰性的
  if (!options.lazy) {
    effect()
  }
  return effect
}
```
#### createReactiveEffect

```js
function createReactiveEffect<T = any>(
  fn: () => T,
  options: ReactiveEffectOptions
): ReactiveEffect<T> {
  // 返回封装后的副作用函数
  const effect = function reactiveEffect(): unknown {
    // 副作用函数核心，稍后讲
  } as ReactiveEffect
  
  // 一些静态属性的定义
  effect.id = uid++
  effect.allowRecurse = !!options.allowRecurse // 是否允许递归
  effect._isEffect = true // 标识是一个 effect 函数
  effect.active = true // effect 自身的状态
  effect.raw = fn // 包装的原始函数
  effect.deps = [] // effect 对应的依赖，双向指针，依赖包含对 effect 的引用，effect 也包含对依赖的引用
  effect.options = options // effect 的相关配置
  return effect
}
```
#### reactiveEffect
18行关键点 真正触发副作用函数的地方
前置变量或者函数
`let shouldTrack = true // 是否允许依赖收集`
```js
function reactiveEffect(): unknown {
  // 如果副作用已经被暂停，则优先执行其调度器，再运行函数本体
  if (!effect.active) {
    return options.scheduler ? undefined : fn()
  }
  // 如果当前副作用未在运行的时候才进入
  if (!effectStack.includes(effect)) {
    // 先清除旧的依赖关系
    cleanup(effect)
    try {
      // 开启全局 shouldTrack，允许依赖收集
      enableTracking()
      // 压栈 加入运行中的副作用堆栈
      effectStack.push(effect)
      // 确认当前副作用，Vue2.0 里的 target
      activeEffect = effect
      // 执行函数 调用fn()时可能会触发get方法，此时会触发上面get中调用的track函数
      return fn() 
    } finally {
      // 出栈
      effectStack.pop()
      // 恢复 shouldTrack 开启之前的状态
      resetTracking()
      // 将当前副作用转交给上一个或者置空
      activeEffect = effectStack[effectStack.length - 1]
    }
  }
}
```

#### track

会在数据挟持的 get / has / ownKeys  中调用，这里前面已经敲重点了。
前置依赖
```js
export const enum TrackOpTypes {
  GET = 'get',
  HAS = 'has',
  ITERATE = 'iterate'
}
const effectStack: ReactiveEffect[] = [] // 全局 effect 栈
let activeEffect: ReactiveEffect | undefined // 当前激活的 effect
```

```js
export function track(target: object, type: TrackOpTypes, key: unknown) {
  // 如果当前没有在收集过程中，则退出
  if (!shouldTrack || activeEffect === undefined) {
    return
  }
  // 取出对象的 KeyToDepMap，如果没有则创建一个新的
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  
  // 取出对应 key 值的依赖集合，如果没有则创建一个新的
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  
  // 如果依赖中不存在当前的队列则添加进去，防止重复设置依赖
  if (!dep.has(activeEffect)) {
    // 双向依赖，保证新旧依赖的一致性
    dep.add(activeEffect)
    activeEffect.deps.push(dep)
  }
}
```

### 第三步 派发更新

只需要在值发生变动的时候，从依赖关系中取出对应的副作用集合，触发`effect`副作用函数即可。
我们可以在上面数据挟持中的 `set / deleteProperty`发现派发更新的函数 `trigger` 的调用。
前置依赖

```js
export const enum TriggerOpTypes {
  SET = 'set',
  ADD = 'add',
  DELETE = 'delete',
  CLEAR = 'clear'
}
```

#### trigger

```js
// 根据target为key值找到对应的存储集合中的effect依次执行，直接看到最后一句
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown,
  oldTarget?: Map<unknown, unknown> | Set<unknown>
) {
  // 响应值所在对象的 KeyToDepMap，如果没有则直接返回
  const depsMap = targetMap.get(target)
  if (!depsMap) {
    // never been tracked
    return
  }

  // 所需要触发的副作用集合
  const effects = new Set<ReactiveEffect>()
  // 添加到集合里的函数，可以在下面触发的时候再看
  const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
    // 传入一个副作用集合
    if (effectsToAdd) {
      // 遍历传入的副作用集合
      effectsToAdd.forEach(effect => {
        // 如果副作用不是当前正在执行的副作用（防止反复调用死循环），或者允许递归调用，则添加值即将触发的副作用集合中
        if (effect !== activeEffect || effect.allowRecurse) {
          effects.add(effect)
        }
      })
    }
  }
  // 这几个条件逻辑 就处理一件事儿 就是拿出需要执行的副作用函数
  if (type === TriggerOpTypes.CLEAR) {
  	// 如果对应的修改操作是，比如 new Set().clear()
    // 则将所有子值的副作用添加到副作用队列中
    depsMap.forEach(add)
  } else if (key === 'length' && isArray(target)) {
    // 如果修改的是数组的 length，意味新的长度后面的值都发生了变动，并将这些下标所对应的副作用加入到队列中
    depsMap.forEach((dep, key) => {
      if (key === 'length' || key >= (newValue as number)) {
        add(dep)
      }
    })
  } else {
    // 修改值，新增值，删除值
    // 将对应值的副作用添加至队列
    if (key !== void 0) {
      add(depsMap.get(key))
    }

    // 增删改对应需要触发的其他副作用（比如依赖于长度的副作用，依赖于迭代器的副作用）
    switch (type) {
      // 新增
      case TriggerOpTypes.ADD:
        // 新增意味着长度发生改变，触发对应迭代器和长度的副作用函数
        if (!isArray(target)) {
          add(depsMap.get(ITERATE_KEY))
          if (isMap(target)) {
            add(depsMap.get(MAP_KEY_ITERATE_KEY))
          }
        } else if (isIntegerKey(key)) {
          add(depsMap.get('length'))
        }
        break
      case TriggerOpTypes.DELETE:
        // 同上，由于数组的删除操作比较特殊所以没有出现
        if (!isArray(target)) {
          add(depsMap.get(ITERATE_KEY))
          if (isMap(target)) {
            add(depsMap.get(MAP_KEY_ITERATE_KEY))
          }
        }
        break
      case TriggerOpTypes.SET:
        // 依赖于迭代器（比如调用了 new Map().forEach() 等于变相依赖了 set）的函数
        if (isMap(target)) {
          add(depsMap.get(ITERATE_KEY))
        }
        break
    }
  }

  // 优先调用调度器否则调用副作用自身
  // 当属性值发生改变之后，会触发trigger函数进行派发更新，将所有依赖这个属性的effect函数循环遍历,使用run函数执行effect
  // 如果effect的参数中配置了scheduler，则就执行scheduler函数，而不是执行依赖的副作用函数
  // 当计算属性依赖的属性发生变化的时候，会执行包装getter函数的effect
  const run = (effect: ReactiveEffect) => {
    if (effect.options.scheduler) { // 如果有scheduler 说明不需要执行effect (computed懒执行)
      effect.options.scheduler(effect) // 将dirty设置为true,下次获取值时变可以重新执行runner方法
    } else {
      effect()
    }
  }
  effects.forEach(run) // 存储集合中的effect依次执行
}
```
到这里 是不是vue3.0响应式原理的面纱就揭开了~
了解了这些，再去看ref 和 computed 实现就简单了，实际上还是对reactive和effect的调用。去看看~

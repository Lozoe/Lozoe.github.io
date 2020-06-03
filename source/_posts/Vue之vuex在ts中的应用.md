---
title: Vue之vuex在ts中的应用
date: 2020-05-21 15:22:01
categories: [Framework]
tags:
---

之前在使用 Vuex 的时候，主要是依赖 state、getters、mutations 以及 actions，并且可以将它们模块化

## state 状态

```ts
import { Module, VuexModule } from 'vuex-module-decorators'

@Module
export default class Vehicle extends VuexModule {
    wheels = 2
}
// 来替代之前的
export default {
    state: {
        wheels: 2
    }
}
```

## Getter 与 class-style 中的 computed 类似，也是借助 getters 来实现的

```ts
import { Module, VuexModule } from 'vuex-module-decorators'

@Module
export default class Vehicle extends VuexModule {
    wheels = 2
    get axles() {
        return this.wheels / 2
    }
}
// 来代替之前的
export default {
    state: {
        wheels: 2
    },
    getters: {
        axles: (state) => state.wheels / 2
    }
}
```

## Mutation 用来修改 state，与 Vuex in js 一样，只能用来实现同步操作

```ts
import { Module, VuexModule, Mutation } from 'vuex-module-decorators'

@Module
export default class Vehicle extends VuexModule {
    wheels = 2

    @Mutation
    puncture(n: number) {
        this.wheels = this.wheels - n
    }
}
// 来替代之前的
export default {
    state: {
        wheels: 2
    },
    mutations: {
        puncture: (state, payload) => {
            state.wheels = state.wheels - payload
        }
    }
}
```

## actions 用来实现异步操作

```ts
// import { Module, VuexModule, Mutation } from 'vuex-module-decorators'

// @Module
// export default class Vehicle extends VuexModule {
//     wheels = 2

//     @Mutation
//     addWheel(payload: number) {
//         this.wheels = this.wheels + payload
//     }    
// }
const request = require('request')
export default {
    state: {
        wheels: 2
    },
    mutations: {
        addWheel: (state, payload) => {
            state.wheels = state.wheels + payload
        }
    },
    actions: {
        fetchNewWheels: async (context, payload) => {
            const wheels = await request.get(payload)
            context.commit('addWheel', wheels)
        }
    }
}

// 来替代之前的

const request = require('request')
export default {
    state: {
        wheels: 2
    },
    mutations: {
        addWheel: (state, payload) => {
            state.wheels = state.wheels + payload
        }
    },
    actions: {
        fetchNewWheels: async (context, payload) => {
            const wheels = await request.get(payload)
            context.commit('addWheel', wheels)
        }
    }
}
```

以上，我们简单实现了一个 Vuex 的 module，那么如何使用呢？将上述代码梳理一下，并配置到 Vuex 中，具体

```ts
import { Module, VuexModule } from 'vuex-module-decorators'

@Module({ name: 'Vehicle',  namespaced: true, stateFactory: true })
export default class Vehicle extends VuexModule {
    public wheels = 2;
    get axles() {
        return this.wheels / 2;
    }
    @Mutation
    public puncture(n: number): void {
        this.wheels = this.wheels - n;
    }
}
```

请注意 Module 的配置，我们需要将需要 namespaced: true，并且为该空间命名，并且引入到项目中

```ts
import Vue from 'vue'
import Vuex from 'vuex'
import Vehicle from './Vehicle'

Vue.use(Vuex)

export default new Vuex.Store({
    modules: {
        Vehicle,
    },
})
```

之后就是使用 vuex-class，来获取该命名空间下的状态以及方法

```tsx
import { Component, Vue } from 'vue-property-decorator';
import {
    Getter,
    Mutation,
    namespace,
} from 'vuex-class';

const Vehicle = namespace('Vehicle');

@Component
export default class HelloWorld extends Vue {
	// 引入 Vechicle 下的 Getters
    @Vehicle.Getter('axles') public axles: number | undefined;
    // 引入 Vechicle 下的 puncture
    @Vehicle.Mutation('puncture')
    public mutationPuncture!: (n: number) => void;

    private message: string = 'world';

    public render() {
        const { message, axles, mutationPuncture }: HelloWorld = this;

        return (
            <div onClick={ () => mutationPuncture(1) }>
                <h5 ref='quickEntry'>Hello {message} { axles }</h5>
            </div>
        );
    }
}
```

利用 namespace 可以很方便的获取到 Vuex 的 Vehicle 模块，再配合 Getter、Mutation 就可以完成引入

总体来说整个体验的过程很像之前写 Mobx & React in Typescript 的感觉，很相像。在实际项目中使用的话，会有 ts 带来的一些便利，但是也感觉总有些牵强，其他后续 3.0，看看能否与 ts 产生奇妙的化学反应吧。

---
title: vuex的使用方法示例
date: 2019-01-02 08:14:45
categories: [Framework]
tags:
    - vue
---

## 一、直接引入

<!-- more -->

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Vuex - Action</title>
    <script src="../../node_modules/vue/dist/vue.js"></script>
    <script src="../../node_modules/vuex/dist/vuex.min.js"></script>
</head>

<body>
    <div id="app">
        <h2>{{msg}}</h2>
        <a href="javascript:;" @click="add">点击</a>
        <counter></counter>
    </div>
    <script>
        const counter = {
            template: `
      <div>
        <div>点击数量:{{ count }}</div>
        <div>用户名:{{ userName }}</div>
      </div>
    `,
            computed: {
                count() {
                    return this.$store.state.count;
                },
                name() {
                    return this.$store.state.name;
                },
                userName() {
                    return this.$store.getters.userName;
                }
            }
        };
        const store = new Vuex.Store({
            state: {
                count: 10,
                name: 'jack'
            },
            getters: {
                userName(state) {
                    return state.name + ',Hello';
                }
            },
            mutations: {
                increment(state, num) {
                    state.count = num;
                },
                updateName(state, userName) {
                    state.name = userName;
                }
            },
            actions: {
                incrementAction(context, num) {
                    context.commit("increment", num);
                }
            }
        });
        new Vue({
            el: "#app",
            store,
            data: {
                msg: "Vuex的使用"
            },
            components: {
                counter
            },
            methods: {
                add() {
                    this.$store.dispatch("incrementAction", 5);
                }
            }
        })
    </script>
</body>

</html>
```

## 二、Vue.use

// store/index.js

```js
import Vue from 'vue'
import Vuex from 'vuex'

import getters from './getters'
import actions from './actions'
import mutations from './mutations'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {
        a: 1,
    },
    getters,
    actions,
    mutations
})
```

// store/mutation-types.js

```js
export const UPDATEA = 'UPDATEA';
```

// store/actions.js

```js
import * as Types from './mutations-types'

export default {
    updateA ({ commit }, a) {
        commit(Types.UPDATEA, a)
    }
}
```

// store/mutations.js

```js
import * as Types from './mutations-types'

export default {
    [Types.UPDATEA] (state, a) {
        state.a = a;
    }
}

```

// store/getters.js

```js
    export default {
        doublea(state) {
            return state.a * 2;
        }
    }

```

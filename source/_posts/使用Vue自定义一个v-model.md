---
title: 使用Vue2.0自定义一个v-model
date: 2020-10-13 15:14:31
tags:
---

## Vue2.0

1. 组件代码

```html
<template>
    <div class="custom-input">
        <input :value="value" @input="$_handleChange" />
    </div>
</template>
<script>
export default {
    props: {
        value: {
        type: String,
        default: ''
        }
    },
    methods: {
        $_handleChange(e) {
        this.$emit('input', e.target.value)
        }
    }
}
</script>
```

2. 在代码中使用组件

```vue
<template>
    <custom-input v-model="value"></custom-input>
</template>
<script>
    export default {
        data() {
            return {
                value: ''
            }
        }
    }
</script>
```

在Vue2.0中我们通过为组件设置名为value属性同时触发名为input的事件来实现的v-model，当然也可以通过model来修改属性名和事件名

## 使用Vue3.0自定义一个v-model示例


1. 组件代码

```vue
<template>
  <div class="custom-input">
    <input :value="value" @input="_handleChangeValue" />
  </div>
</template>
<script>
export default {
    props: {
        value: {
        type: String,
        default: ""
        }
    },
    name: "CustomInput",
    setup(props, { emit }) {
        function _handleChangeValue(e) {
            // vue3.0 是通过emit事件名为 update:modelValue来更新v-model的
            emit("update:value", e.target.value);
        }
        return {
            _handleChangeValue
        };
    }
};
</script>
```

2. 在代码中使用组件

```vue
<template>
  <!--在使用v-model需要指定modelValue-->
  <custom-input v-model:value="state.inputValue"></custom-input>
</template>
<script>
import { reactive } from "vue";
import CustomInput from "../components/custom-input";
export default {
    name: "Home",
    components: {
        CustomInput
    },
    setup() {
        const state = reactive({
            inputValue: ""
        });
        return {
            state
        };
    }
};
</script>
```

到了Vue3.0中，因为一个组件支持多个v-model，所以v-model的实现方式有了新的改变。首先我们不需要使用固定的属性名和事件名了，在上例中因为是input输入框，属性名我们依然使用的是value，但是也可以是其他任何的比如name,data,val等等，而在值发生变化后对外暴露的事件名变成了update:value，即update:属性名。而在调用组件的地方也就使用了v-model:属性名来区分不同的v-model。

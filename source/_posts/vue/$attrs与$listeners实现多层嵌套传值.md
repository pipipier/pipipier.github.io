---
title: 使用$attrs与$listeners实现组件多层嵌套传值
date: 2019-11-12 16:44:32
categories:
 vue
---

**父子组件通信有多种方式，此文只讨论 `v-bind` 和 `v-on` 两种**

假设有3个嵌套组件： A -> B -> C

## v-bind

当 A 组件需要往 B 组件传值时，可以通过以下两种方式：

```html
<!-- A组件 -->
<template>
    <B p1="p1" p2="p2" p3="p3" p4="p5" />
</template>

<script>
import B from './B.vue'
export default {
    components: { 
        B
    },
}
</script>
```

```html
<!-- A组件 -->
<template>
    <B v-bind="bProps" />
</template>

<script>
import B from './B.vue'
export default {
    components: { 
        B
    },
    data() {
        return {
            bProps: {
                p1: 'p1',
                p2: 'p2',
                p3: 'p3',
                p4: 'p4',
            }
        }
    },
}
</script>
```

这两种方式的传值是一模一样的。

## vm.$attrs

先看一下官方定义：

> `vm.$attrs` 包含了父作用域中不作为 `prop` 被识别 (且获取) 的特性绑定 (`class` 和 `style` 除外)。当一个组件没有声明任何 `prop` 时，这里会包含所有父作用域的绑定 (`class` 和 `style` 除外)，并且可以通过 `v-bind="$attrs"` 传入内部组件——在创建高级别的组件时非常有用。

啥意思呢？ 翻译成人话就是说： 假如 A 组件向B组件传了 p1, p2, p3, p4 四个参数，但是 B 组件props只接受了 p1, p2 两个参数，那么剩下的两个 p3, p4 会被 `$attrs` 识别。

```html
<!-- B组件 -->
<template>
    <div></div>
</template>

<script>
export default {
    props: {
        p1: String,
        p2: String,
    },
    created() {
        console.log(this.$props) // {p1: "p1", p2: "p2"}
        console.log(this.$attrs) // {p3: "p3", p4: "p4"}
    }
}
</script>
```

现在应该已经知道了 `$attrs` 中包含的是没有被props接受的参数，并且 `$attrs` 是一个对象，既然是对象，就可以通过 `v-bind` 向子组件传值。

```html
<!-- B组件 -->
<template>
    <C v-bind="$attrs" />
    <!-- 相等于 -->
    <!-- <C v-bind="{ p3: 'p3', p4: 'p4' }" /> -->
</template>

<script>
import C from './C.vue'
export default {
    components: { 
        C
    },
    props: {
        p1: String,
        p2: String,
    },
    created() {
        console.log(this.$props) // {p1: "p1", p2: "p2"}
        console.log(this.$attrs) // {p3: "p3", p4: "p4"}
    }
}
</script>
```

这样的话在 C 组件中就可以获得 p3, p4 两个参数啦！

```html
<!-- C组件 -->
<template>
    <div></div>
</template>

<script>
export default {
    props: {
        p3: String,
        p4: String,
    },
    created() {
        console.log(this.$props) // {p3: "p3", p4: "p4"}
    }
}
</script>
```

## vw.$attrs 优先级

如果在 B 组件中，通过 v-bind 绑定的普通参数，和 `v-bind="$attrs"` 包含的参数有冲突会怎么办呢？如下所示：

```html
<!-- A组件 -->
<template>
    <B p1="p1" p2="p2" p3="p3" p4="p5" />
</template>

<script>
import B from './B.vue'
export default {
    components: { 
        B
    },
}
</script>
```

```html
<!-- B组件 -->
<template>
    <C v-bind="$attrs" p3="B-p3"/>
    <!-- <C v-bind="{ p3: 'A-p3', p4: 'p4' }" /> -->
</template>

<script>
import C from './C.vue'
export default {
    components: { 
        C
    },
    props: {
        p1: String,
        p2: String,
    },
    created() {
        console.log(this.$props) // {p1: "p1", p2: "p2"}
        console.log(this.$attrs) // {p3: "A-p3", p4: "p4"}
    }
}
</script>
```

```html
<!-- C组件 -->
<template>
    <div></div>
</template>

<script>
export default {
    props: {
        p3: String,
        p4: String,
    },
    created() {
        console.log(this.$props) // {p3: "B-p3", p4: "p4"}
    }
}
</script>
```

在 B 组件向 C 组件传递了 p3 属性， 并且 `$attrs` 中也包含了 p3 属性， 那么 B 组件传递的 p3 会覆盖掉 `$attrs` 中的 p3

## vm.$listeners

>包含了父作用域中的 (不含 `.native` 修饰器的) `v-on` 事件监听器。它可以通过 `v-on="$listeners"` 传入内部组件——在创建更高层次的组件时非常有用。

```html
<!-- C组件 -->
<template>
    <button @click="click">click</button>
</template>

<script>
export default {
    methods: {
        click() {
            this.$emit('e1')
            this.$emit('e2')
            this.$emit('e3')
            this.$emit('e4')
        }
    }
}
</script>
```

```html
<!-- B组件 -->
<template>
    <C @e1="e1" @e2="e2" />
</template>

<script>
import C from './C.vue'
export default {
    components: { 
        C
    },
    methods: {
        e1() {
            console.log('B --> e1')
        },
        e2() {
            console.log('B --> e2')
        }
    }
}
</script>
```

```javascript
// 点击按钮后运行结果
// B --> e1
// B --> e2
```

在上面的代码中，点击按钮之后 C 组件抛出 e1, e2, e3, e4 4个事件，但是 B 组件只接受了 e1, e2 两个事件, 所以会输出出 e1, e2, 剩下的 e3, e4 可以通过 `v-on="$listeners"` 传递给父组件

```html
<!-- B组件 -->
<template>
    <C @e1="e1" @e2="e2" v-on="$listeners"/>
</template>

<script>
import C from './C.vue'
export default {
    components: { 
        C
    },
    methods: {
        e1() {
            console.log('B --> e1')
        },
        e2() {
            console.log('B --> e2')
        }
    }
}
</script>
```

```html
<!-- A组件 -->
<template>
    <B @e3="e3" @e4="e4"/>
</template>

<script>
import B from './B.vue'
export default {
    components: { 
        B
    },
    methods: {
        e3() {
            console.log('A --> e3')
        },
        e4() {
            console.log('A --> e4')
        }
    }
}
</script>
```

```javascript
// 点击按钮后运行结果
// B --> e1
// B --> e2
// A --> e3
// A --> e4
```

## vw.$listeners 优先级

如果在 B 组件中，主动触发的事件和 `$listeners` 包含的事件有冲突会怎么办呢？如下所示：

```html
<!-- C组件 -->
<template>
    <button @click="click">click</button>
</template>

<script>
export default {
    methods: {
        click() {
            this.$emit('e1')
            this.$emit('e2')
            this.$emit('e3', '由C组件触发')
            this.$emit('e4')
        }
    }
}
</script>
```

```html
<!-- B组件 -->
<template>
    <C @e1="e1" @e2="e2" v-on="$listeners"/>
</template>

<script>
import C from './C.vue'
export default {
    components: { 
        C
    },
    methods: {
        e1() {
            console.log('B --> e1')
        },
        e2() {
            console.log('B --> e2')
            this.$emit('e3', '由B组件触发')
        }
    }
}
</script>
```

```html
<!-- A组件 -->
<template>
    <B @e3="e3" @e4="e4"/>
</template>

<script>
import B from './B.vue'
export default {
    components: { 
        B
    },
    methods: {
        e3(e) {
            console.log('A --> e3', e)
        },
        e4() {
            console.log('A --> e4')
        }
    }
}
</script>
```

```javascript
// 点击按钮后运行结果
// B --> e1
// B --> e2
// A --> e3 由B组件触发
// A --> e3 由C组件触发
// A --> e4
```

在 B 组件中主动触发了 e3 事件，并且 `$listeners` 中也包含了 e3 事件，这种情况会先执行 B 组件中主动触发的事件，后执行 `$listeners` 中包含的事件！


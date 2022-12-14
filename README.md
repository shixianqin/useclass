# Use Class

Develop [Vue 3](https://vuejs.org/) based applications using a composable
native [class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) API

English | [简体中文](https://github.com/shixianqin/useclass/blob/main/README-zh.md)

## Why?

+ Clearer, more aggregated, and more flexible code organization capabilities
+ Natural logical code namespace
+ No `.value`

## Installation

npm:

```
npm i useclass
```

yarn:

```
yarn add useclass
```

## Example

```vue

<template>
  <div>
    <p>Count: {{ main.count }}</p>
    <p>Double count: {{ main.doubleCount }}</p>
    <div>
      <button @click="main.increase()">increase</button>
      <button @click="main.decrease()">decrease</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { watch, onMounted } from 'vue'
import { useClass } from 'useclass'

const main = useClass(class {
  // reactivity data
  count = 0

  setup () {
    // Use watch in scope
    watch(() => this.count, () => {
      console.log('count was changed')
    })

    // Scoped use of component lifecycle hooks
    onMounted(() => {
      console.log('mounted')
    })
  }

  // Using computed properties
  get doubleCount () {
    return this.count * 2
  }

  increase () {
    this.count++
  }

  decrease () {
    this.count--
  }
})

// Other logical code blocks
// const other = useClass(class { /* ... */ })

</script>
```

## ⚠️ Precautions

### template

When binding events in the template, the function cannot omit `()`, which will cause the scope of `this` to be incorrect

```vue

<!-- ❌ Error example, the scope of `this` is incorrect -->
<button @click="main.increase">increase</button>

<!-- ✅ Correct example -->
<button @click="main.increase()">increase</button>
```

### constructor

In the `constructor` method of the class, only default values can be set on the instance, no other operations are
allowed, because the `this` at this time
Not a proxy object, not reactive. The initialization logic should be moved to the `setup` method

```ts

import { watch, onMounted } from '@vue'
import { useClass } from 'useclass'

const main = useClass(class {
  count = 0
  name: string

  constructor () {
    // ✅ Allowed, default value can be set on the instance
    this.name = 'i am the default'

    /**
     * Error example
     */
    // ❌ Not allowed
    this.myMethod();

    // ❌ Not allowed, `this` is not a proxy object, watch will never fire, please move to the `setup` method
    watch(() => this.count, () => {})

    // ❌ Not allowed, `this` is not a proxy object, the view will not refresh, please move to the `setup` method
    onMounted(() => {
      this.count++
    })
  }

  // `setup` method will be called automatically inside `useClass`
  setup () {
    /**
     * Correct example
     */
    // ✅ Allowed
    this.myMethod();

    // ✅ Allowed
    watch(() => this.count, () => {})

    // ✅ Allowed
    onMounted(() => {
      this.count++
    })
  }

  myMethod () {
    // custom method
  }
})
```

## Vue.js official related documentation reference

+ [Composition API FAQ](https://vuejs.org/guide/extras/composition-api-faq.html)
+ [Reactivity Transform](https://vuejs.org/guide/extras/reactivity-transform.html)

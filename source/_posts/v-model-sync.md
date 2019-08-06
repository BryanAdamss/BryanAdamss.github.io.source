---
title: v-model-sync
tags:
  - vue
categories:
  - 前端
date: 2019-02-22 21:34:48
---

# v-model、.sync 的异同

## v-model

- 实现双向数据绑定(数据 model 改变会自动反映到视图 view 上，视图 view 的数据变化也会同步到数据 model 中)，一般用在表单的双向数据绑定
  约定俗成的用在表单相关组件上
- v-model 会忽略所有表单元素的 value、checked、selected 特性的初始值而总是将 Vue 实例的数据作为数据来源。在调试工具中直接修改这 3 个特性值都不会生效；
- 语法糖:默认形况下会绑定表单的 value 以及监听 input 事件；可通过 model 选项，配置 v-model 的触发机制

```javascript
<input v-model="searchText">
// 等价于：
<input
  v-bind:value="searchText"
  v-on:input="searchText = $event.target.value"
>
// 用在组件上时
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event" // $event为组件内部派发出来的值
></custom-input>
// custom-input组件必须接受value 以及在 派发input事件；即v-model默认情况下无论是用在原生表单组件还是自定义组件上，都必须绑定value以及监听input事件
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
// 自定义v-model的触发机制
Vue.component('base-checkbox', {
  model: { // 将v-model的机制调整为 绑定checked和监听change事件
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```

## .sync 修饰符

- 主要用来实现父子组件间数据的双向传递
- 单向数据传递为:props down,events up
- vue 中是不推荐使用双向数据传递的，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源，会带来维护上的问题
- 但在特殊情况下，有一定实用价值，所以 vue 在 2.3 版本中又添加回来了
- 本质还是个语法糖

```javascript
// 在没有.sync时，vue推荐使用下面方式实现子组件修改父组件的数据
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
Vue.component('text-document', {
  props: {
   title : String
  },
  template: `
    <div v-text="title" @click="$emit('update:title','我是新title')">
    </div>
  `
})
// 为了简化上述流程，推出了.sync修饰符
<text-document
  v-bind:title.sync="doc.title" // 无需再监听update:title事件,但子组件仍需要派发对应自定义事件
></text-document>
Vue.component('text-document', {
  props: {
   title : String
  },
  template: `
    <div v-text="title" @click="$emit('update:title','我是新title')">
    </div>
  `
})
```

## 二者异同

- v-model 默认情况下为 `value+input` 的组合；.sync 为任意组合
- v-model 常用在表单中；.sync 主要用在父子组件通信中，子组件需要修改父组件数据时
- 可以将 v-model 理解成.sync 的一种特殊情况，二者底层的实现机制类似；v-model 做了数据传递+修改的工作；.sync 更多的是数据传递，修改需要交给用户自己
- 可以参考  [https://blog.csdn.net/Qin_Shuo/article/details/82693919](https://blog.csdn.net/Qin_Shuo/article/details/82693919)

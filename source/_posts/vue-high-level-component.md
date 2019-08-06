---
title: vue-high-level-component
tags:
  - vue
categories:
  - 前端
date: 2019-02-22 21:24:37
---

# Vue inheritAttrs、`$attrs`、`$listeners`、provide、inject、slot、slotScope

## 不使用 vuex、eventBus 在高层级组件中传递数据

- 组件调用关系
  - A->B->C （A 调用 B，B 调用 C）
- 基本概念
  - `inheritAttrs`
    - 默认情况下父作用域中的**不被子组件认作为 props 的**特性绑定 (attribute bindings) 将会“回退”且作为普通的 HTML 特性应用在子组件的根元素上。
    - 通过设置 B 组件的 inheritAttrs 为 false，可屏蔽掉这种默认行为。B 组件仍可通过`$attrs` 访问这些`不被认作 props`的特性
    - 例子 1
      - 例如 A 组件传递了 a,b,c,d 四个参数给 B 组件，但 B 组件实际通过 props 只接收了 a、b 两个个，那么两外两个 c、d 将做为普通 html 特性应用到子组件的根元素上。
      - 通过将 B 的`inheritAttrs`设置为 false，则可屏蔽这种行为(c、d 将不会做为普通 html 特性应用到子组件的根元素上，但 B 组件仍能接收 a、b)。B 组件仍可通过 `this.$attrs` 访问到这些不被认作 props 的特性 c、d。
  - `$attrs`
    - 包含了父作用域中不被当前组件认作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。
    - 场景如例子 1
      - B 组件设置了`inheritAttrs`为 false 后，仍可通过`this.$attrs`访问到那些不被认作 props 的特性 c、d。
      - B 组件可在调用 C 组件时，通过在 C 组件上编写`v-bind="$attrs"`将这些参数传递到 C 组件中。(**这样就可以避免在 B 组件调用 C 组件时，需要在 B 组件的 data 中事先声明 C 组件需要的参数，让 B 组件尽可能的保持 Dry**)
      - 通过上述流程其实就完成了 A 组件的参数传递到了更深层次的 C 组件中。
  - `$listeners`
    - 包含了父作用域中的 (不含 .native 修饰器的) 挂载在当前组件上的 `v-on` 事件监听器。
    - 在 B 组件中访问`$listeners`，将会得到 A 组件写在 B 组件上的所有监听器。
    - 例子 2
      - A 调用 B 组件，A 在 B 组件上通过@监听了`click、enter`两个事件，则在 B 组件访问`$listeners`将会得到`click、enter`这两个事件监听器，并能直接调用他们。
      - B 组件调用 C 组件时，可通过在 C 组件上编写`v-on="$listeners"`将这些事件监听器传递到 C 组件中。
      - 通过上述流程就实现了 A 组件监听 C 组件触发的事件效果**这样就避免了 B 组件需要监听 C 组件的事件，然后做一层转发，进而传递给 A 组件**。
  - `provide/inject`
    - 以允许一个**祖先组件**向其**所有子孙后代**注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效
      - 可以理解为一个**加强版的 props**
    - props 只能完成父向子传递数据，无法完成更深层级组件间的数据传递。
    - `provide/inject`二者通常配套使用
    - `provide/inject`**提供的数据是不可响应的，并且是单向的（祖先->子孙）。**
    - 例子 3
    ```javascript
    // 父级A组件提供 'foo'
    var Provider = {
      provide: {
        foo: 'bar'
      }
      // ...
    }
    // 子孙组件B和C都可以注入 'foo'
    var Child = {
      inject: ['foo'],
      created() {
        console.log(this.foo) // => "bar"
      }
      // ...
    }
    // 设置inject默认值
    const Child = {
      inject: {
        foo: { default: 'foo' }
      }
    }
    // 如果它需要从一个不同名字的属性注入，则使用 from 来表示其源属性
    const Child = {
      inject: {
        foo: {
          from: 'bar',
          default: 'foo'
        }
      }
    }
    // 与 prop 的默认值类似，你需要对非原始值使用一个工厂方法
    const Child = {
      inject: {
        foo: {
          from: 'bar',
          default: () => [1, 2, 3]
        }
      }
    }
    ```

## 在深层级组件中传递 slot

- slot 元素可通过设置 name 来标识其为一个具名插槽，**其本身还可以设置一个 slot 特性用来表示它将应用到子组件的哪个插槽中**。
- 利用上述特性就可以实现在深层级组件中传递 slot
- 例子 4
  - A->B->C （A 调用 B，B 调用 C）
  - C 中有一个名为 `cHd` 的具名插槽
  - B 中有一个名为 `bHd` 的具名插槽，并设置了 `slot` 特性为 `cHd`
  - A 在调用 B 组件时，传入了一个 `h1` 并设置 `slot` 特性为 `bHd`
  - 最终此 `h1` 将会传递到 C 组件的 `cHd` 插槽中
  - ![demo](code.png)

## slotScope

- 可参考[https://www.jianshu.com/p/0645bc9033a5](https://www.jianshu.com/p/0645bc9033a5)、[https://segmentfault.com/a/1190000015884505](https://segmentfault.com/a/1190000015884505)
- slot-scope 是应用在组件封装者不知道调用者如何使用组件的情况下，开放了一个接口给调用者让其 DIY（自定义样式、业务逻辑）

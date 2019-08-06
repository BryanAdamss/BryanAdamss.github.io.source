---
title: vue-analysis-note
tags:
  - vue
categories:
  - 前端
date: 2019-01-20 16:57:33
---

> 学习黄奕老师的《vue 源码分析》视频的相关笔记
> 仓库地址：[https://github.com/BryanAdamss/vue-for-analysis](https://github.com/BryanAdamss/vue-for-analysis)

# Vue-analysis-note

## 版本

- `vue 2.5.17-beta.0`

## Flow

### 静态类型检查器

#### 检查方式
- 类型推断
  - 根据调用的方法，推断入参的类型
- 类型注释
  - 主动添加入参及返回值的类型
- 在 vue 中的配置
  - 通过根目录下的`.flowconfig`进行相关配置
    - `[libs] \n flow`字段指名了`flow 自定义类型定义文件的目录`为根目录下的`flow`文件夹
    ```
    flow
      |--- compiler.js      # 编译相关
      |--- component.js     # 组件数据结构
      |--- global-api.js    # Global API 结构
      |--- modules.js       # 第三方库定义
      |--- options.js       # 选项相关
      |--- ssr.js           # 服务端渲染相关
      |--- vnode.js         # virtual node 相关
    ```

## 目录结构

### src 目录结构

- `vue` 源码存放在 `vue 项目`的 `src` 目录下
  - `src` 目录结构如下
  ```
  src
  |--- compiler           # 编译相关代码
  |--- core               # 核心代码
  |--- platforms          # 平台相关代码(web/weex)
  |--- server             # 服务端渲染代码
  |--- sfc                # .vue单文件编译成js对象代码
  |--- shared             # 辅助方法及常量
  ```

## 源码构建

### 基于 rollup 构建

#### 构建命令
```json
  // package.json
  "scripts":{
    ...,

    "build": "node scripts/build.js", // 构建web平台
    "build:ssr": "npm run build -- web-runtime-cjs,web-server-renderer", // 构建成服务端渲染
    "build:weex": "npm run build -- weex",// 构建成weex平台
    ...
  }
```

#### 生成目录
```
  dist
  |--- vue.common.js
  |--- vue.esm.browser.js
  |--- vue.esm.js
  |--- vue.js
  |--- vue.min.js
  |--- vue.runtime.common.js
  |--- vue.runtime.esm.js
  |--- vue.runtime.js
  |--- vue.runtime.min.js
```

#### 目录结构解释
| version                       |        UMD         |       CommonJS        |          ES Module |
| ----------------------------- | :----------------: | :-------------------: | -----------------: |
| **Full**                      |       vue.js       |     vue.common.js     |         vue.esm.js |
| **Runtime-only**              |   vue.runtime.js   | vue.runtime.common.js | vue.runtime.esm.js |
| **Full (production)**         |     vue.min.js     |           -           |                  - |
| **Runtime-only (production)** | vue.runtime.min.js |           -           |                  - |

#### 说明
   - `Full`: 包含`Compiler + Runtime`
   - `Compiler`: 负责将`template`字符串编译成`vue render函数`
   - `Runtime`: 负责创建vue实例、渲染、打补丁等
   - `UMD`: 可直接使用`<script>`标签引用的版本；`unpkg CDN`上[https://unpkg.com/vue](https://unpkg.com/vue)的默认为`compiler+runtime`版本(`vue.js`)
   - `CommonJS`: 供`browserify、webpack1`使用
   - `ES Module`: 供`webpack2、rollup`使用
   - `CommonJS`及`ES Module`版本未提供压缩版本，需要通过打包工具自行压缩

#### Runtime + Compiler vs. Runtime-only
- `Runtime + Compiler`
  - 如果需要使用vue的`template`字段，则需要使用此版本；会在运行时将`template`字符串编译成`render`函数
  - 并配置相应webpack别名
  ``` js
  module.exports = {
    // ...
    resolve: {
      alias: {
        'vue$': 'vue/dist/vue.esm.js' // 'vue/dist/vue.common.js' for webpack 1
      }
    }
  }
  ````
- `Runtime-only`
  - 当使用`vue-loader`或者`vueify`时，则可以直接使用此版本，因为`*.vue 文件中的tempalte`已经在编译阶段自动编译成渲染函数了，所以并不需要`compiler`，仅需要`runtime`就可以了
  - 此版本体积比`Runtime + Compiler`版本体积<~30%

### 入口文件

- web平台的Full版本入口位置
  - `src/platform/web/entry-runtime-with-compiler.js`
  - `vue最初被定义的位置`-`src/core/instance/index.js`
  ```javascript
  // src/platform/web/entry-runtime-with-compiler.js
  /* @flow */

  import config from 'core/config'
  import { warn, cached } from 'core/util/index'
  import { mark, measure } from 'core/util/perf'

  import Vue from './runtime/index'
  ...

  // src/platform/web/runtime/index.js
  /* @flow */

  import Vue from 'core/index'
  import config from 'core/config'
  import { extend, noop } from 'shared/util'
  ...

  // src/core/index.js
  import Vue from './instance/index'
  import { initGlobalAPI } from './global-api/index'
  ...

  // src/core/instance/index.js
  import { initMixin } from './init'
  import { stateMixin } from './state'
  import { renderMixin } from './render'
  import { eventsMixin } from './events'
  import { lifecycleMixin } from './lifecycle'
  import { warn } from '../util/index'

  /* vue 在此处被定义 */
  function Vue(options) {
    if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
      warn('Vue is a constructor and should be called with the `new` keyword')
    }
    this._init(options)
  }

  initMixin(Vue)
  stateMixin(Vue)
  eventsMixin(Vue)
  lifecycleMixin(Vue)
  renderMixin(Vue)

  export default Vue
  ```
  
## 数据驱动

### new Vue时发生了什么

#### 初始化基本思路
1. 调用new Vue(options)
2. 调用通过`initMixin(Vue)`方法导入的`_init`方法
3. `_init`内部调用了相关方法，做了一些初始化操作

#### 初始化时如何代理data的?
1. `_init`方法内部调用`initState(vm)`方法
2. `initState(vm)`方法中调用了`initData(vm)`方法
3. `initData(vm)`方法判断了`vm.$options.data`的类型(函数或其它)并根据data类型，将获取到的`data值`赋值给`vm._data`
4. 遍历检查`vm._data`中是否有和`props、methods`重名的`key`(因为`props、methods、data`最终都要挂载到`vm`上，所以不能重名)，未重名时，判断下`key`是否是vue的保留字(vue中将以`$和_`开头的字段视为vue保留字,这也是声明data时，data中的key不能以`$和_`开头的原因)
5. 不是保留字，则调用`proxy(vm,'_data',key)`方法将`vm.data[key]`代理到`vm._data[key]`上(这也说明了，vue中data申明的数据本质上保存在vm._data上的)

```javascript
// src/core/instance/index.js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

// 定义vue类
function Vue(options) {
  // 非生产环境必须通过new 来生成vue 实例
  if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }

  // 初始化，并传入参数
  this._init(options) // 此方法在initMixin中定义
}

initMixin(Vue) // 导入_init方法
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

// src/core/instance/init.js
...
export function initMixin(Vue: Class<Component>) {
  // 在Vue原型上定义_init方法
  Vue.prototype._init = function(options?: Object) {
    ...
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    
    initState(vm) // 初始化组件自身状态(data中定义的数据)

    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')

    ...
  }
}

// src/core/instance/state.js
...

const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}
// 将target[key]代理到target[sourceKey][key]上，主要实现将vm.data[key]代理到vm._data[key]上的操作
export function proxy(target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter() {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter(val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}

export function initState(vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm) // 初始化data
  } else {
    observe((vm._data = {}), true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}

function initData(vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function' ? getData(data, vm) : data || {} // 将$options中的data取出并赋值给vm._data
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' &&
      warn(
        'data functions should return an object:\n' +
          'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
        vm
      )
  }
  // proxy data on instance
  // 遍历并检查_data的key是否和methods及props重名
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(`Method "${key}" has already been defined as a data property.`, vm)
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' &&
        warn(
          `The data property "${key}" is already declared as a prop. ` +
            `Use prop default value instead.`,
          vm
        )
    } else if (!isReserved(key)) {
      // 若key不是vue的保留字($及_开头的默认都是vue保留字，所以vue中申明状态时，不能以$及_开头)
      // 代理将vm[key]代理到vm[_data][key]上
      proxy(vm, `_data`, key)
    }
  }
  // observe data;观测数据
  observe(data, true /* asRootData */)
}

```

### Vue实例挂载的实现

#### 基本思路
1. 通过`$mount`方法实现实例挂载
2. 不同版本的vue实现的挂载方式不同，不过都是在`runtime`版本的`公共$mount`基础上，修改以满足不同版本的定制化挂载需求
3. 无论什么版本，vue最终都是需要一个`render`函数，若是`compiler`版本，会检查是否有用户编写的`render`，否则将`template`编译成`render`函数供后续流程使用

#### 带compiler版本的$mount调用顺序
1.缓存`公共$mount`方法，供`修改后的$mount方法`调用
2.修改原型上的`$mount`方法
3.若没有指定`render`函数，则通过`template`字段获取模板，再调用`compileToFunctions`方法生成一个`render`方法绑定到vm.$options上
4.调用`公共$mount`方法
5.`公共$mount`方法内部调用了`mountComponent`方法
6.`mountComponent`方法内部首先检查了`vm.$options.render`函数的存在性，不存在，则将创建`空vnode`的`createEmptyVNode`方法赋值给`vm.$options.render`
7.在`mountComponent`方法内部定义了一个`updateComponent`方法，此方法主要调用了`vm._update`方法，此方法接收一个`vm._render`返回的`VNode`
8.`mountComponent`方法内生成一个`渲染watcher实例`,`渲染watcher`实例内部会调用传入的`updateComponent`方法进行Vue实例挂载

```javascript
// src/platforms/web/entry-runtime-with-compiler.js
...

const mount = Vue.prototype.$mount // 缓存了原型上的$mount方法，然后基于compiler版本做了相应修改；原始$mount方法是在./runtime/index.js中定义的
Vue.prototype.$mount = function(
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el) // query返回一个dom元素

  // el 不能是body、html元素(因为挂载会替换对应元素)
  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' &&
      warn(
        `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
      )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          // template是一个id选择器，则取其innerHTML
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        // template传入的是一个dom对象,取其innerHTML
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      // 未指定template字段，但el存在，则获取el的outerHTML(innerHTML+el标签自身)
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      // 编译，生成render函数
      const { render, staticRenderFns } = compileToFunctions(
        template,
        {
          shouldDecodeNewlines,
          shouldDecodeNewlinesForHref,
          delimiters: options.delimiters,
          comments: options.comments
        },
        this
      )

      // 赋值到options上
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)// 调用缓存的公共$mount方法
}
...

// src/platforms/web/runtime/index.js
...
// 适用于每个平台的公共$mount方法
Vue.prototype.$mount = function(
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)// 调用lifecycle中的mountComponent
}
...

// src/core/instance/lifecycle.js
export function mountComponent(
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el // 将el保存到$el上
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if (
        (vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el ||
        el
      ) {
        // 在使用runtime-only版本时，没有写render函数，而是使用了template，此时会报一个警告(使用单文件时，虽然也有template模块，但其在vue-loader时，就已经将其编译成render函数了)
        warn(
          'You are using the runtime-only build of Vue where the template ' +
            'compiler is not available. Either pre-compile the templates into ' +
            'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        // 既没写template也没写render函数
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }

  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    // 定义updateComponent函数，内部调用vm._update方法，第一个参数为vm._render返回的一个Vnode
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  // 实例化一个渲染watcher，其将立即调用一次updateComponent
  new Watcher(
    vm,
    updateComponent,
    noop,
    {
      before() {
        if (vm._isMounted) {
          callHook(vm, 'beforeUpdate')
        }
      }
    },
    true /* isRenderWatcher 标识此watcher是一个渲染watcher，见Watcher类*/
  )
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

#### vm._render方法

- `updateComponent`方法中主要调用了`vm._update`方法，`vm._update`方法则接收一个`vm._render`方法返回的Vnode
- `vm._render`方法定义在`src/instance/render.js`中
- `vm._render`方法会取出`vm.$options.render`方法并执行。
- `vm.$options.render`将在`vm._renderProxy`上下文中执行，并传入一个`vm.$createElement`方法，最终返回一个`vnode`
- `vm._renderProxy`在`src/core/instance/init.js的initProxy方法`中定义，其会根据浏览器是否支持`Proxy`来返回一个`Proxy实例`还是`vm`自身,开发环境中是一个`Proxy实例`，在生产环境中就是`vm`自身
- `vm.$createElement`在`initRender`方法中被定义，`initRender`方法会根据`render`函数来源(vue编译模板而来还是用户编写的)不同生成不同版本的`createElement`版本
```javascript
// src/core/instance/lifecycle.js
export function mountComponent(
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
 ...

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
  ...
  } else {
    // 定义updateComponent函数，内部调用vm._update方法，第一个参数为vm._render返回的一个Vnode；vm._render方法定义在src/instance/render.js中
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }
}

// src/core/instance/render.js
...
// 此方法将在new Vue的过程中被执行，在vm上挂载不同版本的createElement方法
export function initRender(vm: Component) {
  ...
  // bind the createElement fn to this instance
  // so that we get proper render context inside it.
  // args order: tag, data, children, normalizationType, alwaysNormalize
  // internal version is used by render functions compiled from templates
  // 最后一个标志量是alwaysNormalize（是否总是规范化）,为false 标识此方法是给编译生成的render函数使用
  vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
  // normalization is always applied for the public version, used in
  // user-written render functions.
  // 为true 标识此方法是给用户编写的render函数使用
  vm.$createElement = (a, b, c, d) => createElement(vm, a, b, c, d, true)
  ...
}

export function renderMixin(Vue: Class<Component>) {
  ...
  Vue.prototype._render = function(): VNode {
    const vm: Component = this
    // 从$options中拿到用户编写的或者vue生成的render函数
    const { render, _parentVnode } = vm.$options
    ...

    // render self
    let vnode
    try {
      // 调用render方法；vm._renderProxy在src/instance/init.js中定义；vm._renderProxy在生产环境下就是vm，开发环境下是一个proxy对象;      vm.$createElement在initRender方法中被定义
      vnode = render.call(vm._renderProxy, vm.$createElement)
    } catch (e) {
      ...
    }
    // return empty vnode in case the render function errored out
    // 创建的vnode并不是VNode实例
    if (!(vnode instanceof VNode)) {
      // vnode为一个数组，则表示我们模板有多个根节点
      if (process.env.NODE_ENV !== 'production' && Array.isArray(vnode)) {
        warn(
          'Multiple root nodes returned from render function. Render function ' +
            'should return a single root node.',
          vm
        )
      }
      // 创建一个空vnode
      vnode = createEmptyVNode()
    }
    // set parent
    vnode.parent = _parentVnode
    // 返回一个vnode
    return vnode
  }
}

```

### VirtualDOM

#### 出现原因
- 现有DOM因为历史原因，结构很复杂，每个节点上包含了很多非必要信息(字段)，操作起来性能消耗很大。
- 为了解决这一问题，人们想出在`DOM`层上再添加一个速度更多，性能消耗小的层，需要操作`DOM`时，先操作新层，新层在合适的时机通过一系列算法计算出最小更新范围，再将这些更新应用到`DOM`层
- 简单理解就像计算机读写硬盘上的文件一样，如果直接在硬盘上读写文件，速度会很慢。于是人们在硬盘和系统之间加了一层(内存)，内存速度快，系统可以先在内存中对文件进行一系列操作，最后在合适的时机再将文件重新写入硬盘。这样就避免了频繁的操作硬盘了。

#### VirtualDOM基本思路
- `VirtualDOM`利用结构简单的`原生js对象(VNode)`去描述一个`DOM节点`，只保留了必要的字段，这样节点就变得简单了，操作起来速度更快
- 通过关联相关的`VNode`组成一个`VNode tree`
- 通过相应渲染方法将`VNode tree`渲染成真正的`DOM tree`
- 需要更新`DOM tree`时，会通过`diff`算法比较新旧两棵`VNode tree`，找出差异，得出最小更新范围，通过打补丁的方式(`patch`)将这些差异更新真正的`DOM tree`上
- 通过上述过程，一来简化了节点大小，操作速度提升。二来，降低了变更`DOM tree`的频率，性能更好。
- `vue`的`VirtualDOM`实现参考了`snabbdom`
  
#### VNode定义
- 定义在`src/core/vdom/vnode.js`中
- 仅保留了一些必要属性
```javascript
/* @flow */

export default class VNode {
  tag: string | void // 标签名
  data: VNodeData | void // VNode数据
  children: ?Array<VNode> // 子节点
  text: string | void // 文本
  elm: Node | void
  ns: string | void
  context: Component | void // rendered in this component's scope
  key: string | number | void
  componentOptions: VNodeComponentOptions | void
  componentInstance: Component | void // component instance
  parent: VNode | void // component placeholder node

  // strictly internal
  raw: boolean // contains raw HTML? (server only)
  isStatic: boolean // hoisted static node
  isRootInsert: boolean // necessary for enter transition check
  isComment: boolean // empty comment placeholder?
  isCloned: boolean // is a cloned node?
  isOnce: boolean // is a v-once node?
  asyncFactory: Function | void // async component factory function
  asyncMeta: Object | void
  isAsyncPlaceholder: boolean
  ssrContext: Object | void
  fnContext: Component | void // real context vm for functional nodes
  fnOptions: ?ComponentOptions // for SSR caching
  fnScopeId: ?string // functional scope id support

  constructor(
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child(): Component | void {
    return this.componentInstance
  }
}

export const createEmptyVNode = (text: string = '') => {
  const node = new VNode()
  node.text = text
  node.isComment = true
  return node
}

export function createTextVNode(val: string | number) {
  return new VNode(undefined, undefined, undefined, String(val))
}

// optimized shallow clone
// used for static nodes and slot nodes because they may be reused across
// multiple renders, cloning them avoids errors when DOM manipulations rely
// on their elm reference.
export function cloneVNode(vnode: VNode): VNode {
  const cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  )
  cloned.ns = vnode.ns
  cloned.isStatic = vnode.isStatic
  cloned.key = vnode.key
  cloned.isComment = vnode.isComment
  cloned.fnContext = vnode.fnContext
  cloned.fnOptions = vnode.fnOptions
  cloned.fnScopeId = vnode.fnScopeId
  cloned.asyncMeta = vnode.asyncMeta
  cloned.isCloned = true
  return cloned
}
```

#### VNode的创建
- 通过`vm._render`方法生成，内部调用`vm.$createElement`

### createElement

#### 定义
- `vm.$createElement`在内部调用了`createElement`方法
- `createElement`方法定义在`src/core/vdom/create-element.js`中
- `createElement`方法内部重设了`normalizationType`并调用了`_createElement`方法
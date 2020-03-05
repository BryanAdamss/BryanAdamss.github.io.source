---
title: hashchange-beforeunload-popstate
tags:
  - 前端
  - history
categories:
  - 其他
date: 2020-03-05 11:39:25
---

# hashchange、beforeunload、popstate

## hashchange 事件

### 触发时机

- 在页面 hash 片段(`即 URL 中#和#后面的部分`)改变时触发
- hash（#）是 URL 的锚点，代表的是网页中的一个位置，单单改变#后的部分，浏览器只会滚动到相应位置，不会重新加载网页，也就是说 **#是用来指导浏览器动作的，对服务器端完全无用，HTTP 请求中也不会不包括#；同时每一次改变#后的部分，都会在浏览器的访问历史中增加一个记录，使用”后退”按钮，就可以回到上一个位置**。(hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中。它是用来指导浏览器动作的，对服务器端完全无用，因此，改变 hash 不会重新加载页面)

### 小结

- `hash` 改变时，会触发`hashchange`事件，**并会在全局历史记录中添加一条记录**

## beforeunload 事件

### 触发时机

- **仅在页面文档被卸载时触发**
    - 如果处理函数为 `Event 对象`的`returnValue`属性赋值`非空字符串`，浏览器会弹出一个对话框，来询问用户是否确定要离开当前页面
    - 有些浏览器会将返回的字符串展示在弹框里，但有些其他浏览器只展示它们自定义的信息。没有赋值时，该事件不做任何响应
```javascript
window.addEventListener("beforeunload", function (e) {
  var confirmationMessage = "离开提示信息";
  (e || window.event).returnValue = confirmationMessage;     // Gecko and Trident
  return confirmationMessage;                                // Gecko and WebKit});
```

### 相关场景

- 刷新
    - 刷新时，原有页面文档会被卸载，所以会触发；不会触发 popstate 事件
- 关闭
    - 页面关闭，原有页面文档也会被卸载，所以也会触发；不会触发 popstate 事件
- hashchange 时
    - hash 改变，原有页面文档不会被卸载，所以不会触发；会触发 popstate 事件
- `pushState、replaceState` 时
    - 页面不会被卸载，所以不会触发`beforeunload`；不会触发 `popstate` 事件
- 返回、前进
    - 当返回、前进操作引起页面卸载时，会触发；引起卸载时，不会触发 `popstate` 事件，没有引起卸载时，会触发 `popstate` 事件

### 小技巧

- 判断除关闭以外的操作是否会触发`beforeunload`的技巧(适用于 `chrome`)
    - 进行操作时，查看当前浏览器 tab 上是否出现正在加载资源的标识

### 注意

- 此事件的处理函数中，会忽略对`window.alert(), window.confirm(), 和 window.prompt()`的调用
- 如果页面**没有发生交互，浏览器可能不会展示弹窗，甚至可能即使发生交互了也直接不显示**，并且会提示如下错误信息；[相关链接](https://developer.mozilla.org/zh-CN/docs/Web/Events/beforeunload)
```javascript
     [Intervention] Blocked attempt to show a 'beforeunload' confirmation panel for a frame that never had a user gesture since its load
```

### 小结

- beforeunload 仅在页面文档被卸载时触发
- 可通过页面是否重新加载资源，来判断操作是否会触发 beforeunload
- 即使触发了，也可能会被浏览器拦截

## History API

### 介绍

- 从 HTML5 开始——提供了对 history 栈中内容的进行操作的 API；
- 由于安全原因，浏览器不允许脚本读取历史记录的地址，但是允许在地址之间导航

### 查看当前窗口历史记录栈长度

- 历史记录栈类型
    - 当前窗口历史记录栈
        - `由window.history维护`
    - 全局历史记录栈
        - chrome 中通过`ctrl+h`打开
```javascript
window.history.length
```

### 前进、后退、跳到指定位置

```javascript
// 在history中向后跳转；这和用户点击浏览器回退按钮的效果相同
window.history.back()
// 向前跳转（如同用户点击了前进按钮
window.history.forward()
// 向后移动一个页面 (等同于调用 back())
window.history.go(-1)
// 向前移动一个页面, 等同于调用了 forward()
window.history.go(1)
```

### `window.history.go(0)`

- 作用等同于刷新
- `go(0)` vs `reload`的区别
    - `reload`会重新拉取数据，`go(0)`直接从缓存中获取数据
```javascript
// 直接读取缓存数据，不会从服务器端获取数据，也不会增加新的历史记录
window.history.go(0)
// 是会从服务端获取数据的
window.location.reload()
```

### 添加、修改记录

- 使用`pushState`、`replaceState`方法
    - 前者添加一条新记录、后者修改当前记录

#### `pushState、replaceState`参数

- `pushState、replaceState`支持传入 3 个参数，依次为`stateObj`、`title`、`url`
- `stateObj`就是一个普通对象，与对应的历史记录条目绑定；
    - stateObj 有大小限制
        - **stateObj 序列化后必须<=640k**
    - 每次，当用户路由到对应历史记录条目，则可以通过`history.state或者popstate事件`获取到对应的`stateObj`
```javascript
// 获取当前历史记录的状态对象
history.state
// 通过popstate事件获取stateObj
window.onpopstate = e => {
  console.log(e.state)
}
```
- `title`是一个可省略的标题
    - 建议即使不用时，也传递个空字符串
- `url`定义了新的历史 URL 记录
    - **必须是同域 url**
    - 可以是相对 url，也可以是绝对 url
```javascript
// 当前页面地址www.a.com/foo/bar.html
// 相对路径
history.pushState({ test: 1 }, 'test', './test.html') // 页面地址变为www.a.com/foo/test.html
// 绝对路径
history.pushState({ test2: 2 }, 'test2', '/test2.html') // 页面地址变为www.a.com/test2.html
// 非同域
history.pushState({ test3: 3 }, 'test3', 'http://www.baidu.com') // 跨域，报错
```

#### 注意

- **使用 pushState、replaceState 添加历史记录时，不会立即加载相应页面，甚至都不会检查新的历史记录地址是否真的存在**
    - 它仅仅是在**当前历史记录栈中**新增了一条记录，并不会立即加载相应页面
        - **这样就实现了不刷新页面改变 URL**
    - 只在刷新或先跳转到其他域页面，再返回的场景下，才会触发加载页面，否则都是无刷新的
- **pushState()、replaceState()  绝对不会触发  hashchange  事件，即使新的 URL 与旧的 URL 仅哈希不同也是如此。**

#### pushState、replaceState 和改变 hash 来实现无刷新改变 URL 的区别

- 前者可以修改同源下的任意路径；后者**只能修改 hash 部分**
- 前者即使**在添加的 `url` 跟之前完全相同时**，也会被添加到历史记录栈中，并会触发 `popstate` 事件；后者**只有在新 hash 不同时**，才会添加新的历史记录，并触发`hashchange`，否则无法触发`hashchange`
- 前者支持关联任意的数据到历史记录项上；后者只能把相关数据转成字符串添加到 hash 上，变相绑定相关数据

## popstate 事件

### 介绍

- 每当处于激活状态的历史记录条目发生变化时，会触发`popstate`事件
    - 更准确的定义：每当**同一个文档**的浏览历史（即 `history` 对象）出现变化时，就会触发 `popstate` 事件。当历史记录变化，引**起文档变化(reload)时，popstate 不会触发**(`A popstate event is dispatched to the window each time the active history entry changes between two history entries for the same document`)

### 触发场景

- 调用`pushState、replaceState`时，**不会触发`popstate`事件**
- 只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用`History.back()、History.forward()、History.go()`方法导致历史记录条目变化时才会触发
- 该事件只针对同一个文档，如果浏览历史的切换，**导致加载不同的文档，该事件也不会触发**。
- **当网页加载时,各浏览器对`popstate事件`是否触发有不同的表现,`Chrome` 和 `Safari`会触发`popstate`事件, 而`Firefox`不会.（有待验证）**
- `hash`改变时，也会触发`popstate事件`

### 注意

- 如果当前处于激活状态的历史记录条目是由`history.pushState()方法`创建,或者由`history.replaceState()方法`修改过的, 则`popstate事件对象的state属性包含了当前这个历史记录条目的state对象的一个拷贝`
- 即便进入了那些非`pushState`和`replaceState`方法作用过的(比如http://example.com/example.html)没有 state 对象关联的那些网页, popstate 事件也仍然会被触发

### demos

```javascript
// 绑定事件处理函数.
window.onpopstate = function(event) {
  alert(
    'location: ' + document.location + ', state: ' + JSON.stringify(event.state)
  )
}
// 添加并激活一个历史记录条目 http://example.com/example.html?page=1,条目索引为1
history.pushState({ page: 1 }, 'title 1', '?page=1')
// 添加并激活一个历史记录条目 http://example.com/example.html?page=2,条目索引为2
history.pushState({ page: 2 }, 'title 2', '?page=2')
// 修改当前激活的历史记录条目 http://ex..?page=2 变为 http://ex..?page=3,条目索引为3
history.replaceState({ page: 3 }, 'title 3', '?page=3')
// 弹出 "location: http://example.com/example.html?page=1, state: {"page":1}"
history.back()
// 弹出 "location: http://example.com/example.html, state: null
history.back()
// 弹出 "location: http://example.com/example.html?page=3, state: {"page":3}
history.go(2)
```

### 小结

- `pushState、replaceState时`不会触发`popstate`事件
- 加载不同的文档时，不会触发
- `hash`改变时，会触发

## vue-router 中的 hash、history 模式

### 设置 mode 为 hash，并不一定用的就是 hash

- 在新版本`vue-router`中，如果浏览器支持`History API`，`vue-router`会优先使用`History`相关 API（`pushState、replaceState`）来`模拟hash`，即使用户设置`mode`为`hash`；只有在浏览器不支持`History`时，才会使用`hashchange`

### 验证

- `vue-router` 版本 `3.0.6`，设置`mode`为 `hash`；`chrome` 版本 `78`
- 结论
    - 调用 `this.$router.push` 方法跳转新路由时，`没有触发popstate事件`
    - 如果使用的`hashchange`，则`hash`改变时，肯定会触发`popstate事件`
- 原因
    - 正常认知下，设置了`hash模式`，`vue-router`内部肯定会用到`hashchange事件`，`$router.push`时，`hash改变`，肯定会触发`hashchange和popstate事件`，但实际上，两个事件都没有触发；原因是`vue-router`检测到浏览器支持`history`，强制使用了`history`的`pushState、replaceState`来模拟`hash`(URL 表现形式上仍然会出现#号)，而`pushState、replaceState`调用时，不会触发`popstate事件`
- 源码
    - 在 `HashHistory` 类的 `setupListeners` 方法中，是根据`supportsPushState`来确定是绑定`popstate`还是`hashchange`事件
    - ![router.png](router.png)
    - `supportsPushState` 的实现，主要判断了一些特定浏览器，直接返回 `false`，其他的通过特性检测来判断；所以，其并没有根据传入的 `mode` 来选择监听的事件类型，而是根据浏览器是否支持来确定监听事件的类型
    - ![router2.png](router2.png)

## 触发时机总结

| 操作                                       | hashchange | popstate | beforeunload | beforeRouteLeave |
| ------------------------------------------ | ---------- | -------- | ------------ | ---------------- |
| 点击刷新                                   | N          | N        | Y            | N                |
| 点击主页                                   | N          | N        | Y            | N                |
| 点击关闭 tab                               | N          | N        | Y            | N                |
| 手动修改 URL，引起页面加载                 | N          | N        | Y            | N                |
| 手动修改 hash                              | Y          | Y        | N            | Y                |
| `history.pushState、history.replaceState`  | N          | N        | N            | Y                |
| `$router.push`(调用了 `history.pushState`) | N          | N        | N            | Y                |
| `$router.back`(调用了 `history.back`)      | Y          | Y        | N            | Y                |

## 应用

### vue 单页离开时，给予保存提示

- 监听`beforeunload`处理页面刷新、关闭
- 监听组件内的`beforeRouteLeave`，处理页面内路由跳转
- `vue`中可抽取成一个单独的`mixins`，供组件复用
- 如果希望独立`vue`使用，考虑使用`popstate`替代`beforeRouteLeave`
```javascript
/**
 * @author GuangHui
 * @description 页面离开提示 mixins
 */
export const mixinsName = 'LEAVE_TIPS_MIXINS'
export const getName = name => `${mixinsName}_${String(name)}`

export default {
  data() {
    return {
      [getName('tips')]: '系统可能不会保存您所做的更改'
    }
  },
  created() {
    this[getName('init')]()
  },
  activated() {
    this[getName('init')]()
  },
  beforeRouteLeave(to, from, next) {
    this[getName('handleBeforeRouteLeave')](to, from, next)
  },
  methods: {
    /**
     * 处理beforeRouteLeave
     *
     * @param {*} to
     * @param {*} from
     * @param {*} next
     */
    [getName('handleBeforeRouteLeave')](to, from, next) {
      if (this[getName('showTips')]()) {
        next()
      } else {
        next(false)
      }
    },
    /**
     * 展示默认tips
     *
     * @returns confrim返回值
     */
    [getName('showTips')]() {
      return window.confirm(this[getName('tips')])
    },
    /**
     * 初始化
     *
     */
    [getName('init')]() {
      /* eslint-disable-next-line */
      this[getName('handleBeforeUnloadBinded')] = this[
        getName('handleBeforeUnload')
      ].bind(this)
      this[getName('bindEvents')]()
      this.$once('hook:deactivated', this[getName('cleanEvents')])
      this.$once('hook:beforeDestroy', this[getName('cleanEvents')])
    },
    /**
     * 处理beforeunload事件
     *
     * @param {*} e 事件对象
     */
    [getName('handleBeforeUnload')](e) {
      // https://developer.mozilla.org/zh-CN/docs/Web/Events/beforeunload
      ;(e || window.event).returnValue = this[getName('tips')] // Gecko and Trident
      return this[getName('tips')] // Gecko and WebKit
    },
    /**
     * 绑定事件
     *
     */
    [getName('bindEvents')]() {
      window.addEventListener(
        'beforeunload',
        this[getName('handleBeforeUnloadBinded')],
        false
      )
    },
    /**
     * 清除事件
     *
     */
    [getName('cleanEvents')]() {
      window.removeEventListener(
        'beforeunload',
        this[getName('handleBeforeUnloadBinded')],
        false
      )
    }
  }
}
```

## 参考

- https://www.cnblogs.com/wancheng7/p/8542544.html
- https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onpopstate



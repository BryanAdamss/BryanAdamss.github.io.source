---
title: chrome-devtools
tags:
  - 调试
  - Chrome
  - DevTools
categories:
  - 其他
date: 2018-09-14 23:22:21
---

> 记录 chrome 调试工具的一些常用技巧
> 没有时间详细整理，想到什么记录什么

# Chrome Devtools 常用技巧

## 网络请求相关

### 拷贝网络请求地址

- 选择`NetWork`Tab
- `右键`一个网络请求
- 选择`copy`
- 选择`copy link address`即可拷贝某一请求实际地址

- 注意
  - 如果直接双击一个网络请求会打开一个新标签页，然后会请求对应网络地址，如果有装`JSONView`可以直接查看对应结果

### 拷贝网络请求的 response

- 选择`NetWork`Tab
- `右键`一个网络请求
- 选择`copy`
- 选择`copy response`即可

- 注意
  - 选择`copy response`拷贝出的 response 是没有经过格式化的(没有缩进),可以通过类似`json.cn`等在线网站进行美化，或者使用`JSON.stringify(obj,null,4)`或者在`console`中使用`copy`方法进行美化，如下面例子

```javascript
copy({"most_visited": [], "history_on": false, "recent": []})
// * 将得到下面美化后的结果
{
  "most_visited": [],
  "history_on": false,
  "recent": []
}
```

### 将网络请求转换为 fetch 形式

- 选择`NetWork`Tab
- `右键`一个网络请求
- 选择`copy`
- 选择`copy as fetch`即可，devtools 会将请求的相关参数都拼装好并放到一个 fetch 中，可以直接调用

```javascript
fetch('https://developers.google.com/profile/userhistory', {
  credentials: 'include',
  headers: {},
  referrer: 'https://developers.google.com/web/tools/chrome-devtools/sources',
  referrerPolicy: 'no-referrer-when-downgrade',
  body: null,
  method: 'GET',
  mode: 'cors'
})
```

### 如何判断网站是否开启 gzip

- 比较资源大小

  - 在`NetWork` Tab 的左上角选择`Use large request rows`
    ![](gzip.png)
  - 选中后，重新加载页面
    ![](gzip2.png)
  - 在`size`列会出现两个资源大小，上面是实际下载资源的大小，下面的是原始资源大小，如果二者大小几乎一致，则可以判定网站未开启`gzip`
  - 下面为开启后的
    ![](gzip3.png)

- 其它方法

  - 查看网络请求的`response header`的`Content-Encoding`是否为`gzip`

- 注意
  - 在开启`gzip`时，需要忽略图片格式，因为针对图片开启 gzip 不但不会缩减大小，反倒会增大图片资源的传输大小，适得其反

## 命令相关

### CommandMenu

- 和很多工具一样，`devtools` 也有命令菜单，可以通过`ctrl+shift+p`唤出
- 可以通过输入命令的形式使用 `devtools` 或者打开特定菜单
- 常用`command`
  - `ctrl+p`->直接输入，则查找文件
  - `ctrl+p`->输入`?`->查看帮助
  - `ctrl+p`->输入`:`->跳转特定行
  - `ctrl+p`->输入`@`->跳转到特定符号处(symbol)
  - `ctrl+p`->输入`!`->运行 snippet
  - `ctrl+p`->输入`>`->命令菜单

### 创建所有页面都能使用的 snippet(代码片段)

- 例如有些网站没有引入`jQuery`，此时可以创建一个在任何页面都能使用的插入`jQuery`的代码片段
- 使用`ctrl+shift+p`打开 CommandMenu，输入`Create new snippet`
- 拷贝下面的代码，`ctrl+s`保存并`ctrl+enter`执行
- 你会发现在页面的`head`标签下已经引入一个 jQuery 了

```javascript
let script = document.createElement('script')
script.src = 'https://code.jquery.com/jquery-3.2.1.min.js'
script.crossOrigin = 'anonymous'
script.integrity = 'sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4='
document.head.appendChild(script)
```

### 快速执行一个已经创建的 snippet

- 使用`ctrl+p`，输入`!`，即会列出所有 snippet，选择一个执行即可

### 格式化源码

- 有时源码会进行压缩或者格式不对，需要对其进行格式化
  - 在`source` Tab 下打开的`js、html、css`文件，在右下角会展示一个`Format`按钮(大概长这样`{}`)，点击即可格式化当前代码
    ![](format.png)
  - 格式化后
    ![](format2.png)

## console 相关

### 将 DOM 元素格式化为 JavaScript 对象

- 使用`console.dir(dom元素)`
  ![](console_dom.png)

### 使用`$0`

- 在`Element`Tab 中选择一个元素，然后使用`$0`可以引用这个 DOM 元素
- 例如下面打印出 DOM 元素

```javascript
// * 打印选中的DOM元素
console.log($0)

// * 如果需要转成JS对象，可以使用dir
console.dir($0)
```

- 在`vue-devtools`中有类似变量，可以使用`$vm0`引用选中组件

### 在 console.log 中使用 css 样式

- 使用`%c`

```javascript
// * 使用%c
console.log('%c我将是蓝色的%c我是绿色的', 'color: blue; ', 'color:green;')
```

![](console_color.png)

### 快速清除 console 面板内容

- 使用`ctrl+l`或者`clear()`

## 查看 js、css 中的无用代码

### 使用 coverage

- `ctrl+shift+p`输入`Show Coverage`并选择之
- 会列出`css`、`js`中没有生效的占比
  ![](coverage.png)
- 点击其中一条，会打开对应文件，左侧红色代表未生效的代码行，绿色代表已经执行过的代码行
  ![](coverage2.png)

### 未完待续

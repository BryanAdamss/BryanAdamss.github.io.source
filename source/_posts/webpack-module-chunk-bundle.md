---
title: webpack-module-chunk-bundle
tags:
  - 工程化
  - webpack
categories:
  - 前端
date: 2020-03-05 11:58:26
---

# module、chunk、bundle


## module
- 对于`webpack`来说，所有的资源(`.js`、`.css`、`.png`)都是module
- `.js`文件`webpack`原生可以处理，其他类型文件，需要通过`loader`进行转换，然后交给`webpack`处理
- `webpack`配置中，可以通过`module`字段，对不同`module`使用不同的`rule`处理
```json
module: {
rules: [
        {
            test: /\.css$/,
            use: [
                    { loader: "style-loader" },
                    { loader: "css-loader" } 
                ] 
        }, ... 
    ] 
},

```

## entry-point
- 生成依赖图(`dependency graph`)的入口；
- `webpack`只能以`.js`文件做为生成依赖图的入口；`parcel`能使用其他模块(`.html`)做为入口，生成依赖图

## chunk
- 是`webpack内部处理时`的概念；
- 一个`chunk`是对依赖图的部分(子图)进行封装的结果`Chunk the class is the encapsulation for parts of your dependency graph`
- 可以说一个`chunk`是一堆`module`的集合
- 可以通过多个`entry-point`来生成一个`chunk`

### chunk分类
- `entry chunk`
    - 包含webpack `runtime code`并且是最先执行的chunk
- `initial chunk`
    - 包含`同步`加载进来的module且`不包含runtime code`的`chunk`
    - 在`entry chunk`执行后再执行
- `normal chunk`
    -  使用`require.ensure`、`System.import`、`import()`异步加载进来的`module`，会被放到`normal chunk`中
    - 一般`normal chunk`(异步chunk)，会单独生成一个`bundle`，其名称一般通过`webpackChunkName魔法注释`和`output.chunkFilename`来指定
        - 在`webpackChunkName魔法注释`中指定的`name`可以在`output.chunkFilename`中使用`[name]`取到

## bundle
- 最终输出的`chunk`在用户端，被称之为`bundle`；
- 正常情况下，可以将`chunk`和`bundle`当成一个概念，只不过是不同时间的同一个东西
    - `chunk和bundle的区别`，可以理解为`chunk`是过程中的代码块(给`webpack`处理的)，`bundle`是结果的代码块(展现给用户的)
- 一般一个`chunk`对应一个`bundle`，只有在配置了`sourcemap`时，才会出现一个`chunk对应多个bundle`的情况；


## 例子
```js
// other.js
import(/*webpackChunkName: "lodash */ 'lodash').then(_ => {
  console.log(_)
})


// webpack.config.js
entry:{
    main:['./src/main.js','./src/test.js'],
    other:['./src.other.js']
},
output:{
    path:path.resolve(__dirname,'./dist'),
    filename:"[name].bundle.js", // 正常生成的bundle名称
    chunkFileName:"[name].async.bundle.js" // normalChunk的名称，也即最终生成的异步bundle的名称
}
```

- `main.js`中引入了`gloabl.css`；
- `other.js`中异步导入了`lodash`并使用了`webpackChunkName魔法注释`

- `module`
    - `main.js、test.js、other.js、global.css`都是module
- `entry-point`
    - `main.js、test.js、other.js`
- `chunk`
    - `main`
    - `other`
- `bundle`
    - `main.bundle.js`
    - `other.bundle.js`
    - `lodash.async.bundle.js`

- `entry`的两个`key`指定了两个`chunk(bundle)`，名称分别为`mian`、`other`，最终会输出两个`bundle`；
- `main.js、test.js、other.js`都是`entry-point`
- 因为没有抽取`css`，所以没单独生成一个`css文件`，`global.css的内容`是`内联`在`main.bundle.js`中的

## loader
- `webpack loader` 是 `webpack` 为了处理各种类型文件的一个中间层，`webpack` 本质上就是一个 `node 模块`，它不能处理` js` 以外的文件，那么 `loader` 就帮助`webpack` 做了一层转换，将所有文件都转成字符串，你可以对字符串进行任意操作/修改，然后返回给 `webpack` 一个包含这个字符串的对象，让 `webpack` 进行后面的处理。如果把 `webpack` 当成一个垃圾工厂的话，那么 `loader` 就是这个工厂的垃圾分类

## plugin
- 如果把 `webpack` 当成一个垃圾工厂，`loader` 就是垃圾分类，将所有垃圾整理好交给 `webpack`。`plugin` 就是如何去处理这些垃圾。

## 核心流程
- `webpack` 启动后会从 `entry-point` 开始递归解析 `entry-point` 依赖的所有 `module`。
-  每找到一个 `module`， 就会根据配置的 `loader` 去找出对应的转换规则，对 `module` 进行转换后，再解析出当前 `module 依赖的 module`。 
-  这些模块会以 `entry`字段配置的`key(chunk name)为单位`进行`分组`，一个 `entry 和其所有依赖的 module` 被分到一个组也就是一个 `chunk`。
-  最后 `webpack` 会把`所有 chunk` 转换成文件输出为`bundle`； 
-  在整个流程中 `webpack` 会在恰当的时机执行 `plugin` 里定义的逻辑，在`plugin`执行逻辑中，可能会再抽取其他`bundle`(例如从`js bundle`中再单独抽取出一个`css bundle`文件)，但这不属于`webpack`主逻辑，属于`plugin`附加的能力
-  正常情况下，`webpack`是一个`js entry-point`对应一个`js chunk`对应生成一个`js bundle`(除非配置了`sourcemap`，会单独再生成一个`sourcemap bundle`)；
-  `plugin`属于增强`webpack`功能，`loader`给`webpack`提供`非js`文件的转换能力；


## 相关项目
- [webpack-chain源码分析](https://github.com/BryanAdamss/webpack-chain-source)
- [使用webpack-chain编写webpack的实践](https://github.com/BryanAdamss/webpack4-template-use-webpack-chain)
- [《深入浅出Webpack》学习demos](https://github.com/BryanAdamss/webpack-dive-into-demos)

## 参考
[https://github.com/webpack/webpack.js.org/issues/970](https://github.com/webpack/webpack.js.org/issues/970)
[https://juejin.im/post/5cede821f265da1bbd4b5630](https://juejin.im/post/5cede821f265da1bbd4b5630)
[https://juejin.im/post/5d2b300de51d45775b419c76](https://juejin.im/post/5d2b300de51d45775b419c76)
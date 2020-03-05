---
title: webpack-hash-chunkhash-contenthash
tags:
  - 工程化
  - webpack
categories:
  - 前端
date: 2020-03-05 11:59:38
---

# hash、chunkhash、contenthash

## hash
- hash是跟项目构建挂钩的，只要项目中没有文件发生变化，hash就不会改变
```js

const path = require('path')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const resolve = _path => path.resolve(__dirname, _path)
module.exports = {
  entry: {
    main: resolve('./main.js'),
    other: resolve('./other.js')
  },
  output: {
    path: resolve('./dist'),
    filename: '[name].[hash].bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        // 调用extract从已有的loader生成一个新的loader
        use: ExtractTextPlugin.extract({
          // 转换 .css 文件需要使用的 Loader
          use: [
            {
              loader: 'css-loader',
              options: {
                url: false
              }
            }
          ]
        })
      }
    ]
  },
  plugins: [
    // 实例化插件
    new ExtractTextPlugin({
      // 从 .js 文件中提取出来的 .css 文件的名称
      filename: `[name].[hash].css`
    })
  ]
}
```
![hash.png](hash.png)
- 只要项目中没有文件发生变化，hash值就不会改变；反之，当有任意一个文件发生改变，hash就会发生改变
- 这样会造成所有文件的hash都会发生变化

## chunkhash
- 跟`chunk`挂钩,只要`chunk`中的`module`没有发生变化，`chunkhash`就不会变化
- 每个`chunk`有自己`独立的chunkhash`，属于相同`chunk`的`module`，`chunkhash`也一样
```js

const path = require('path')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const resolve = _path => path.resolve(__dirname, _path)
module.exports = {
  entry: {
    main: resolve('./main.js'),
    other: resolve('./other.js')
  },
  output: {
    path: resolve('./dist'),
    filename: '[name].[chunkhash].bundle.js'  // 修改为chunkhash
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        // 调用extract从已有的loader生成一个新的loader
        use: ExtractTextPlugin.extract({
          // 转换 .css 文件需要使用的 Loader
          use: [
            {
              loader: 'css-loader',
              options: {
                url: false
              }
            }
          ]
        })
      }
    ]
  },
  plugins: [
    // 实例化插件
    new ExtractTextPlugin({
      // 从 .js 文件中提取出来的 .css 文件的名称
      filename: `[name].[chunkhash].css`  // 修改为chunkhash
    })
  ]
}
```
- 修改`main.js`中代码，发现只有`main chunk`相关的`chunkhash`发生变化了，`other chunk`的没有发生变化
![chunkhash.png](chunkhash.png)
- 当一个`.js`文件引用了一个`.css`文件，当`js`文件变化时，`css`文件的`chunkhash`也会发生变化，导致缓存失效，这时需要`contenthash`

## contenthash
```js

const path = require('path')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const resolve = _path => path.resolve(__dirname, _path)
module.exports = {
  entry: {
    main: resolve('./main.js'),
    other: resolve('./other.js')
  },
  output: {
    path: resolve('./dist'),
    filename: '[name].[chunkhash].bundle.js'  // 此处仍为chunkhash
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        // 调用extract从已有的loader生成一个新的loader
        use: ExtractTextPlugin.extract({
          // 转换 .css 文件需要使用的 Loader
          use: [
            {
              loader: 'css-loader',
              options: {
                url: false
              }
            }
          ]
        })
      }
    ]
  },
  plugins: [
    // 实例化插件
    new ExtractTextPlugin({
      // 从 .js 文件中提取出来的 .css 文件的名称
      filename: `[name].[contenthash].css`  // 修改为contenthash
    })
  ]
}
```
- 仅仅将`css`的`chunkhash`改为`contenthash`；
- 修改`main.js`不会导致`.css`样式的`contenthash`发生变化
![contenthash.png](contenthash.png)
- 修改`.css`文件，也仅仅是`.css`文件的`contenthash`变化，不会影响`main.js`的`chunkhash`
![contenthash2.png](contenthash2.png)
- `vue-cli 2.0`也是使用相同策略(`chunk`使用`chunkhash`，`css`文件使用`contenthash`)
![contenthash3.png](contenthash3.png)


## 相关项目
- [webpack-chain源码分析](https://github.com/BryanAdamss/webpack-chain-source)
- [使用webpack-chain编写webpack的实践](https://github.com/BryanAdamss/webpack4-template-use-webpack-chain)
- [《深入浅出Webpack》学习demos](https://github.com/BryanAdamss/webpack-dive-into-demos)

## 总结
- `css、jpg、font`等`asset文件`可考虑使用`contenthash`
- 其余文件使用`chunkhash`
---
title: webpack
tags:
  - webpack
  - 构建
categories:
  - 前端
date: 2017-12-21 14:59:17
---

> webpack官方指南学习笔记


# webpack官方指南学习笔记

## 起步
- `packagejson.js`必须是纯正的json文件，用双引号，结尾不能有逗号
- `npm init -y`会默认全选`yes`来初始化`package.json`
- npm中可以直接使用模块名来引用本地安装的包，而不需要写出完整路径
    - `./node_modules/.bin/webpack src/index.js dist/bundle.js`可替换为`webpack src/index.js dist/bundle.js`
- `webpack`命令默认会查找`webpack.config.js`配置文件。可以通过`--config`来使用其他配置文件打包
- 可以使用`npm run 命令名`的形式来运用`package.json`中`scripts`声明的自定义命令
    ```javascript
     ...
      "scripts": {
        "build": "webpack"
      },
    
    // 使用
    npm run build
    ```

## 管理资源
- 加载css
    - 在webpack所有资源都可以看成模块，css也不例外
    - 通过`css-loader`可以在js文件中使用`import './css/style.css'`的形式导入样式
    - 通过`style-loader`可以将样式文件提取出来生成一个`<style>`标签插入到`<head>`中，如果是单页面应用，则可以减少请求(当前也可以分离样式到单独的文件中)
    ```javascript
    // nodejs获取路径
    const path = require('path');
    
    module.exports = {
        entry: './src/index.js', // 入口
        output: { // 出口
            filename: 'bundle.js',
            path: path.resolve(__dirname, 'dist'), //__dirname是nodejs中的全局变量用于获取当前文件的完整绝对路径
        },
        module: {
            rules: [{ // 针对收集到的.css结尾的依赖，先使用css-loader，再使用style-loader
                test: /\.css$/,
                use: ['style-loader', 'css-loader'],// 同一个依赖依次使用多个loader时要按照从右往左的顺序
            }]
        }
    };
    ```
- 加载图片
    - 通过`file-loader`可以在js文件中使用`import Img from './img/icon.jpg'`形式来使用图片，并`Img`变量包含该图像在处理后的最终url；当使用了`css-loader`后，css中的`url("./img/icon/jpg")`路径也会被替换；当使用`html-loader`后，html中的`<img src="./my-image.png" />`路径也会被替换
    ```javascript
    import Cat from './img/cat.jpg';
    
    function component() {
        ...
        var image = new Image();
        image.src = Cat;
        image.onload = function() {
            element.appendChild(image);
        };
    
        return element;
    }
    ```    
- 加载字体
    - 加载字体的策略和加载图片的策略基本一致，可以通过`file-loader`实现
- 加载CSV、XML
    - 可以通过对应的loader来处理

## 管理输出
- 生成模板
    - 使用`html-webpack-plugin`插件，可以动态生成html文件，并自动添加上生成的包
    ```javascript
       plugins: [
         new HtmlWebpackPlugin({
           title: '我是标题'
         })
       ],
    ```
- 清理文件夹
    - 使用`clean-webpack-plugin`插件，来清理指定文件夹
```javascript
    plugins: [
        new CleanWebpackPlugin(['dist']),// 指定要清理的文件夹
        new HtmlWebpackPlugin({
            title: 'Output Management'
        })
    ],
```
- 使用`webpack`自带插件时，需要先`require('webpack')`

## 开发
- sourceMap
    - 可以将编译后的代码映射回原始源代码
    - 通过配置webpack的`devtool`
        - `devtool: 'inline-source-map',`
- 自动编译
    - webpack watch模式
        - 使用`webpack --watch`观测文件的变动，有变动，则自动编译，可以将`webpack --watch`写入到`package.json`的`scripts`中；缺点是无法自动刷新浏览器，重新编译后需要手动刷新浏览器
        ```javascript
        "scripts": {
              "test": "echo \"Error: no test specified\" && exit 1",
              "watch": "webpack --watch",
              "build": "webpack"
            },
        ```
    - 使用`webpack-dev-server`
        - 包含一个简单web服务器，并能实现自动刷新
        - 需配置`devServer`
        ```javascript
           devServer: {
             contentBase: './dist' // 将dist文件夹下的文件作为可访问文件
           },
        ```
        - 使用`webpack-dev-server --open`启动，可将其写到`package.json`的`scripts`中
    - 使用`webpack-dev-middleware`
        - 它是一个中间件容器，可以将webpack处理后的文件发布到另外一个服务器中，`webpack-dev-server`中也使用了它，可以配合`express`来使用

## 模块热替换MHR
- 可以在不刷新浏览器的情况下替换模块，如css、图片、js。重要的是在替换js模块时还能保留js模块运行时的相关环境
- 使用
    - `webpack-dev-server`中可以通过配置`devServer`的`hot`为`true`，并使用`webpack`自带的` HotModuleReplacementPlugin`插件来实现

## Tree Shaking
- 通过`uglifyjs-webpack-plugin`插件实现，此插件自带Tree Shaking功能，能剔除掉未用到的模块
- 使用Tree Shaking功能必须使用ES2015的模块语法

## 生产环境构建
- 为了区分开发环境和生产环境，可以提取出一个公用配置文件并分别配置不同的配置文件。
- 不同配置文件的合并需要使用到`webpack-merge`工具
- 可以使用`webpack`自带的`DefinePlugin`插件指定环境
```javascript
     new webpack.DefinePlugin({
       'process.env': {
         'NODE_ENV': JSON.stringify('production'),// 定义的NODE_ENV可以被/src下的所有文件访问到
       }
     })
```

## 代码分离
- 分离代码的方法
    - 配置不同的入口起点，手动分离代码
    ```javascript
      entry: {
        index: './src/index.js',
        another: './src/another-module.js'
      },
    ```
    - 使用`CommonsChunkPlugin`去重和分离块
        - 不同模块A、B，可能会引用同一个模块C,打包后的A、B会默认包含C，使用`webpack`自带的`CommonsChunkPlugin`插件可以提取相同代码到同一个包中
        ```javascript
             new webpack.optimize.CommonsChunkPlugin({
               name: 'common' // 指定公共 bundle 的名称。
             })
        ```
        - 可以使用`ExtractTextPlugin`将css从主程序中分离出来
        - `promise-loader`用于分离代码和延迟加载生成的 bundle，主要用promise
    - 动态导入
        - 使用`import()`函数，并设置`chunkFilename`

## 懒加载
- 在需要某个模块的时候使用`import()`动态导入
- 注意当调用 ES6 模块的 import() 方法（引入模块）时，必须指向模块的 .default 值，因为它才是 promise 被处理后返回的实际的 module 对象。

## 缓存
- `[hash]`可以给文件打上统一的项目指纹
- `[chunkhash]`可以给文件打上唯一的文件指纹

## 创建Library
- 可以通过指定`externals`来外部化一些依赖

## Shimming
- 可以使用`ProvidePlugin`在合适时机加载一些全局变量。

## Typescript
- 可以使用`ts-loader`来编译typescript

## 构建性能
- 后期补

## 环境变量
- 后期补

## 管理依赖
- 后期补

## Public Path
- 后期补

## 集成
- 后期补
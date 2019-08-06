---
title: webpack-multi-page
tags:
  - webpack
categories:
  - 前端
date: 2018-01-02 09:38:03
---

> 最近在看webpack的配置，自己尝试写了个多页面的配置文件。
> 在此做个记录，方便日后查找。
> 详细代码可以查看(https://github.com/BryanAdamss/WebpackTemplate)

# Webpack多页面配置

## 想实现的功能
- 开发环境、生产环境配置分离
- 开发环境
    - HMR
    - 自动生成HTML文件
    - source map
- 生产环境
    - 提取css、sass
    - 提取公共模块
    - 压缩代码
    - hash 缓存

## 与SPA的不同
- 单页面应用在入口处引入所有的js文件
- 多页面应用需要在每个页面中引入公共的js文件和自身的js文件(公共代码、多入口)

## 项目结构
```
|---config/
|       config.js
│       webpack.config.base.js
│       webpack.config.dev.js
│       webpack.config.prod.js
|---dist/
|---node_modules/
|---src/
|       css/
|       html/
|       img/
|       sass/
|       vendors/
|       a.js
|       b.js
|---.babelrc
|---package.json
|---postcss.config.js
```
- 为了方便管理，将配置文件拆分为4个并全部放置在config文件夹下
- `dist`文件夹主要放置打包后的文件
- `src`主要是源文件和一些第三方组件
- `.babelrc`为babel配置文件
- `postcss.config.js`为postcss的配置文件

## package.json
```javascript
{
  "name": "WebpackTemplate",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack  --progress --color --config ./config/webpack.config.prod.js",
    "dev": "webpack-dev-server --progress --color --open --config ./config/webpack.config.dev.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "auto-prefixer": "^0.4.2",
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.1",
    "clean-webpack-plugin": "^0.1.17",
    "css-loader": "^0.28.7",
    "extract-text-webpack-plugin": "^3.0.2",
    "file-loader": "^1.1.6",
    "html-webpack-plugin": "^2.30.1",
    "node-sass": "^4.7.2",
    "postcss-loader": "^2.0.9",
    "sass-loader": "^6.0.6",
    "style-loader": "^0.19.1",
    "uglifyjs-webpack-plugin": "^1.1.5",
    "url-loader": "^0.6.2",
    "webpack": "^3.10.0",
    "webpack-dev-server": "^2.9.7",
    "webpack-merge": "^4.1.1"
  },
  "dependencies": {
    "babel-polyfill": "^6.26.0"
  }
}
```
- `post-css-loader`、`auto-prefixer`配合完成前缀修复工作
- `babel-core`、`babel-preset-env`、`babel-polyfill`是babel编译es6所必须的文件；`babel-loader`是用来处理es6的；`babel-polyfill`是垫片文件
- `clean-webpack-plugin`是清除文件夹的
- `css-loader`、`style-loader`是加载样式的
- `extract-text-webpack-plugin`用来提取css到一个单独文件的
- `file-loader`可以用来加载图片字体等资源
- `url-loader`作用类似`file-loader`，不同的是可以将低于某大小的文件直接转成dataUrl插入到页面中
- `html-webpack-plugin`可根据模板自动生成html页面，并插入需要的js文件
- `node-sass`、`sass-loader`配合完成编译sass(scss)的工作
- `uglifyjs-webpack-plugin`用来压缩js文件
- `webpack-dev-server`生产环境用的静态服务器
- `webpack-merge`用来合并不同的配置文件

## 配置文件
- `config.js`主要放置了一些全局配置文件，如各种路径...
```javascript
// 全局配置，比如 HTML 文件的路径、publicPath 等

const path = require('path');

// __dirname是当前文件所在目录，process.cwd()是node当前工作的目录，即package.json所在目录

const PROJECT_PATH = process.cwd(); // 项目目录

const config = {
    PROJECT_PATH, // 项目目录
    CONFIG_PATH: path.join(__dirname), // 配置文件目录
    SRC_PATH: path.join(PROJECT_PATH, './src/'), // 源文件目录
    BUILD_PATH: path.join(PROJECT_PATH, './dist/'), // 打包目录
    PUBLIC_PATH: '/assets/', // 静态文件存放目录
    HTML_PATH: path.join(PROJECT_PATH, './src/html/'),
    VENDORS_PATH: path.join(PROJECT_PATH, './src/vendors/'), // vendors目录
    NODE_MODULES_PATH: path.join(PROJECT_PATH, './node_modules/'), // node_modules目录
    ignorePages: ['test'], // 没有入口js文件的html
};

console.log('\n/-----相关路径-----/\n');
console.log(config);
console.log('\n/-----相关路径-----/\n');

module.exports = config;
```
- `webpack.config.base.js`是基础配置文件，包含了一些通用配置
```javascript
// 基础配置文件，包含了不同环境通用配置

const path = require('path'); // nodejs路径模块，用于读取路径
const fs = require('fs'); // nodejs文件模块，用于读取文件

const config = require('./config.js'); // 获取配置

const HTMLWebpackPlugin = require('html-webpack-plugin'); // 用于生成html

// 获取html文件名，用于生成入口
const getFileNameList = (path) => {
    let fileList = [];
    let dirList = fs.readdirSync(path);
    dirList.forEach(item => {
        if (item.indexOf('html') > -1) {
            fileList.push(item.split('.')[0]);
        }
    });
    return fileList;
};

let htmlDirs = getFileNameList(config.HTML_PATH);

let HTMLPlugins = []; // 保存HTMLWebpackPlugin实例
let Entries = {}; // 保存入口列表

// 生成HTMLWebpackPlugin实例和入口列表
htmlDirs.forEach((page) => {
    let htmlConfig = {
        filename: `${page}.html`,
        template: path.join(config.HTML_PATH, `./${page}.html`) // 模板文件
    };

    let found = config.ignorePages.findIndex((val) => {
        return val === page;
    });

    if (found === -1) { // 有入口js文件的html，添加本页的入口js和公用js，并将入口js写入Entries中
        htmlConfig.chunks = [page, 'commons'];
        Entries[page] = `./src/${page}.js`;
    } else { // 没有入口js文件，chunk为空
        htmlConfig.chunks = [];
    }

    const htmlPlugin = new HTMLWebpackPlugin(htmlConfig);
    HTMLPlugins.push(htmlPlugin);
});

module.exports = {
    context: config.PROJECT_PATH, // 入口、插件路径会基于context查找
    entry: Entries,
    output: {
        path: config.BUILD_PATH, // 打包路径，本地物理路径
    },
    module: {
        rules: [{
            test: /\.(woff|woff2|eot|ttf|otf)$/,
            include: [config.SRC_PATH],
            exclude: [config.VENDORS_PATH], // 忽略第三方的任何代码
            use: [{ // 导入字体文件，并最打包到output.path+ options.name对应的路径中
                loader: 'file-loader',
                options: {
                    name: 'fonts/[name].[ext]'
                }
            }]
        }, {
            test: /\.(png|jpg|gif|svg)$/,
            include: [config.SRC_PATH],
            exclude: [config.VENDORS_PATH],
            use: [{ // 图片文件小于8k时编译成dataUrl直接嵌入页面，超过8k回退使用file-loader
                loader: 'url-loader',
                options: {
                    limit: 8192, // 8k
                    name: 'img/[name].[ext]', // 回退使用file-loader时的名称
                    fallback: 'file-loader', // 当超过8192byte时，会回退使用file-loader
                }
            }]
        }, {
            test: /\.js$/,
            include: [config.SRC_PATH],
            exclude: [config.VENDORS_PATH, config.NODE_MODULES_PATH],
            use: ['babel-loader']
        }]
    },
    plugins: [
        ...HTMLPlugins, // 扩展运算符生成所有HTMLPlugins
    ]
};
```
    - 自动生成html文件是通过getFileNameList函数来实现的，主要利用node的fs模块，读取`src/html/`下的所有直接子文件，并通过后缀结合`config.js中ignore`进行筛选过滤，返回需要生成的html的名字数组，遍历此数组生成相关入口文件路径和`HTMLWebpackPlugin`实例
- `webpack.config.dev.js`开发环境配置文件
```javascript
// 开发环境配置文件

const webpackBase = require('./webpack.config.base.js'); // 引入基础配置
const config = require('./config.js'); // 引入配置

const webpack = require('webpack'); // 用于引用官方插件
const webpackMerge = require('webpack-merge'); // 用于合并配置文件

const webpackDev = { // 开发配置文件
    output: {
        filename: 'js/[name].[hash:8].bundle.js', // 开发环境用hash
    },
    devtool: 'cheap-module-eval-source-map', // 开发环境设置sourceMap，生产环境不使用
    devServer: { // 启动devServer，不会在本地生成文件，所有文件会编译在内存中(读取速度快)
        contentBase: './dist/', // 这个目录下的内容可被访问
        overlay: true, // 错误信息直接显示在浏览器窗口中
        inline: true, // 实时重载的脚本被插入到你的包(bundle)中，并且构建消息将会出现在浏览器控制台
        hot: true, // 配合webpack.NamedModulesPlugin、webpack.HotModuleReplacementPlugin完成MHR
        // publicPath: config.PUBLIC_PATH, // 静态资源存放位置，根目录的assets文件夹，确保publicPath总是以斜杠(/)开头和结尾。可以设置为CDN地址。这个选项类似url-prefix
        host: "0.0.0.0", // 设置为0.0.0.0并配合useLocalIp可以局域网访问
        useLocalIp: true, // 使用本机IP打开devServer，而不是localhost
        // proxy: {// 可以通过proxy代理其他服务器的api
        //     "/api": "http://localhost:3000"
        // }
    },
    module: {
        rules: [{
            test: /\.css$/, // 开发环境不提取css
            include: [config.SRC_PATH],
            exclude: [config.VENDORS_PATH],
            use: ['style-loader', 'css-loader', 'postcss-loader']
        }, {
            test: /\.scss$/, // 开发环境不提取css
            include: [config.SRC_PATH],
            exclude: [config.VENDORS_PATH],
            use: ['style-loader', 'css-loader', 'postcss-loader', 'sass-loader']
        }]
    },
    plugins: [
        new webpack.NamedModulesPlugin(), // 开发环境用于标识模块id
        new webpack.HotModuleReplacementPlugin(), // 热替换插件
    ]
};


module.exports = webpackMerge(webpackBase, webpackDev);
```
- `webpack.config.prod.js`生产环境配置文件
```javascript
// 生产环境配置文件

const webpackBase = require('./webpack.config.base.js'); // 引入基础配置
const config = require('./config.js'); // 引入配置

const webpack = require('webpack'); // 用于引用官方插件
const webpackMerge = require('webpack-merge'); // 用于合并配置文件
const CleanWebpackPlugin = require('clean-webpack-plugin'); // 用于清除文件夹
const UglifyJSPlugin = require('uglifyjs-webpack-plugin'); // 用于压缩文件


const ExtractTextWebpackPlugin = require('extract-text-webpack-plugin'); // 提取css，提取多个来源时，需要实例化多个，并用extract方法
const cssExtracter = new ExtractTextWebpackPlugin({
    filename: './css/[name]-css.[contenthash:8].css', // 直接导入的css文件，提取时添加-css标识
    allChunks: true, // 从所有的chunk中提取，当有CommonsChunkPlugin时，必须为true
});
const sassExtracter = new ExtractTextWebpackPlugin({
    filename: './css/[name]-sass.[contenthash:8].css', // 直接导入的sass文件，提取时添加-sass标识
    allChunks: true,
});

const webpackProd = { // 生产配置文件
    output: {
        filename: 'js/[name].[chunkhash:8].bundle.js', // 生产环境用chunkhash
    },
    module: {
        rules: [{
                test: /\.css$/, // 生产环境提取css
                include: [config.SRC_PATH],
                exclude: [config.VENDORS_PATH],
                use: cssExtracter.extract({
                    fallback: 'style-loader',
                    use: [{
                        loader: 'css-loader',
                        options: {
                            minimize: true //css压缩
                        }
                    }, 'postcss-loader']
                })
            },
            {
                test: /\.scss$/, // 生产环境提取css
                include: [config.SRC_PATH],
                exclude: [config.VENDORS_PATH],
                use: sassExtracter.extract({
                    fallback: 'style-loader',
                    use: [{
                        loader: 'css-loader',
                        options: {
                            minimize: true //css压缩
                        }
                    }, 'postcss-loader', 'sass-loader']
                })
            }
        ]
    },
    plugins: [
        cssExtracter,
        sassExtracter,
        new webpack.DefinePlugin({ // 指定为生产环境，进而让一些library可以做一些优化
            'process.env.NODE_ENV': JSON.stringify('production')
        }),
        new webpack.HashedModuleIdsPlugin(), // 生产环境用于标识模块id
        new CleanWebpackPlugin(['./dist/'], {
            root: config.PROJECT_PATH, // 默认为__dirname，所以需要调整
        }),
        new webpack.optimize.CommonsChunkPlugin({ // 抽取公共chunk
            name: 'commons', // 指定公共 bundle 的名称。HTMLWebpackPlugin才能识别
            filename: 'js/commons.[chunkhash:8].bundle.js'
        }),
        new UglifyJSPlugin(),
    ]
};


module.exports = webpackMerge(webpackBase, webpackProd);
```
- `.babelrc`为babel配置文件
```javascript
{
    "presets": [
        ["env",
        {
            "targets":
            {
                "browsers": ["android>=4.0", "ios>=7.0", "ie>=8", "> 1% in CN"]
            },
            "useBuiltIns": "usage",
            "modules": false,
            "loose": true
        }]
    ]
}
```
    - 使用`babel-preset-env`预设，它包含了babel-preset-es2015, babel-preset-es2016,babel-preset-es2017，并可设置`targets`让其自动为目标浏览器进行`polyfill`和代码转换工作；
    - `"useBuiltIns": "usage"`使用自带的`polyfill`即`babel-polyfill`，并只在使用了某个目标浏览器不支持的es6语法时自动`import`相关垫片；[https://github.com/babel/babel/tree/master/packages/babel-preset-env](https://github.com/babel/babel/tree/master/packages/babel-preset-env)
    ```javascript
    //入口文件
    a.js
    var a = new Promise();
    
    b.js
    var b = new Map();
    
    //Out (如果目标浏览器不支持)
    import "core-js/modules/es6.promise";
    var a = new Promise();
    
    import "core-js/modules/es6.map";
    var b = new Map();
    
    //Out (如果目标浏览器支持)
    var a = new Promise();
    var b = new Map();
    ```
    - `"modules": false`不将ES6的模块转为其他模块类型(AMD、UMD...)
    - `"loose":true`使用更简单 ES5 代码来兼容老浏览器
- `postcss.config.js`postcss配置文件
```javascript
module.exports = {
    plugins: {
        'autoprefixer': {
            browsers: ["android>=4.0", "ios>=7.0", "ie>=8", "> 1% in CN"],
            //是否美化属性值 默认：true 
            cascade: true,
            //是否去掉不必要的前缀 默认：true 
            remove: true
        }
    }
};
```

## 总结
- webpack已经成为前端必须会的东西，网上也有很多现成的配置文件可以直接使用，但是想配一个适合自己的还是需要花费很多时间的，因为官方文档暴露出的细节太少，必须查阅大量资料、阅读相关源码再加上大量的实践才能基本了解。

## 参考
> https://www.jianshu.com/p/2cc4a1078953
> https://zhuanlan.zhihu.com/p/29161762
> https://segmentfault.com/a/1190000006843916
> https://doc.webpack-china.org/api/
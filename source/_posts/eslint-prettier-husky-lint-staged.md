---
title: eslint-prettier-husky-lint-staged
tags:
  - eslint
  - prettier
  - husky
  - lint-staged
categories:
  - 前端
date: 2019-03-17 00:48:14
---

# ESLint + Prettier + Husky + lint-staged 打造代码检查工作流

> 记录搭建代码检查工作流的相关问题

## 基本介绍

- ESLint
  - 代码检查工具
  - 特点
    - 代码静态检查
      - 提前发现低级语法错误
    - 代码风格检查
      - 检查代码风格
  - 关于`eslint`的配置可以查阅我之前的文章，有详细解释
- Prettier
  - 代码格式化工具
  - 特点
    - 有一套自己的格式化规范
    - 基本支持前端涉及的所有语言
      - `js、jsx、css、html、md等等`
- ESLint、Prettier 二者异同
  - `eslint`还是侧重于代码静态检查、`prettier`侧重于代码格式化
    - `eslint`使用`--fix`也可以根据特定代码风格进行一些格式化工作；
- Husky
  - 在`git`中提供`precommit、prepush`钩子，可在其中执行一些脚本
- lint-staged
  - 可针对添加到`git 暂存区`中不同格式的文件，分别执行不同的脚本

## 工具介绍

### ESLint 相关

- 通过`npm install eslint`安装的
  - 可以用来执行`eslint 命令`
    - 如`eslint --fix`、`eslitn --ext .js,.vue src`
  - 通过`webpack`中的`eslint-loader`可以实现保存时自动修复
- `VSCode`中的扩展`ESLint`
  - 它会读取项目中的`eslint 配置文件`，然后启动一个`lint服务`，在`vscode`中智能提示不符合项目`eslint 配置`的代码
  - 通过配置`eslint.autoFixOnSave`，可以实现保存时，根据配置文件规则，自动修复一些代码问题（大部分都是一些代码风格相关的问题）
- `eslint-loader`
  - 在`webpack`中配合`eslint`使用的`loader`
    - 通过配置可实现在每次打包时，自动 lint；如用 vue-cli2.x 初始化的项目中下面代码

```bash
module:{
    rules:[
        {
            test: /\.(js|vue)$/,
            loader: 'eslint-loader',
            enforce: 'pre', // 遇到js、vue文件时，会第一个调用此loader；
            include: [resolve('src'), resolve('test')],
            options: {
            formatter: require('eslint-friendly-formatter'),
            emitWarning: !config.dev.showEslintErrorsInOverlay
        }
    ]
}
```

- `eslint-friendly-formatter`
  - 可以输出更友好的错误提示；如包含违反规则的文件地址和规则解释网址

### Prettier 相关

- 通过`npm install prettier`安装的`prettier cli`
  - 可以调用`prettier`的一些命令；如`prettier --wirte test.js`来实现`test.js`的格式化
    - 更多命令可查看`https://prettier.io/docs/en/cli.html`
  - 会读取项目中的`.pretterrc`配置文件
- `VSCode`中的扩展`Prettier`
  - 本质是内部调用`pretter-cli`
  - 此扩展内部内置了`prettier`格式化规则，不过只暴露了极少的配置选项。
  - 配置读取优先级
    - 会优先读取`.prettierrc`文件
    - 不存在则读取`.editorconfig`文件
    - 其次读取插件自身配置选项
  - 可以配合`ESLint、TSLint`使用
    - 有`prettier.eslintIntegration`选项
      - 设置为`true`后，内部会使用`eslint-prettier`替换`prettier-cli`
        - `eslint-pretter`
          - 会将格式化的代码传递给`eslint --fix`
          - Code ➡️ prettier ➡️ eslint --fix ➡️ Formatted Code

### vetur

- `VSCode`中的`vue`扩展，开发`vue`必备
  - 自带`lint`功能(需要配合`eslint`插件使用)
  - 自带`format`功能
    - 内置`prettier`
      - 若本地安装了`prettier`，则会使用本地`prettier`
        - 内置`prettier`也会读取`.prettierrc`文件进行格式化工作
      - 可以针对`.vue`文件中的不同部分设置不同的`formatter`
        - 可参考[https://vuejs.github.io/vetur/formatting.html#settings](https://vuejs.github.io/vetur/formatting.html#settings)

### ESLint+Prettier 相关

- `eslint-config-prettier`
  - 关闭`eslint`中与`prettier`冲突的`rules`
- `eslint-plugin-prettier`
  - 会对比格式化前后的代码，将不一致的地方标记为错误，在 eslint 中抛出

### VSCode 的设置

- `editer.autoFormatOnSave`
  - 编辑器的自动格式化，会调用一个格式化器
  - 它是总开关，若为 false,所有格式化工具将不能在保存时自动格式化

## 目标

- 代码的检查和格式化工作，在项目内调用命令即可完成，无需依赖编辑器配置；
- 针对使用`vue-cli`创建的项目
  - 针对`js、vue`文件使用`vue-cli`官方推荐的的规则来检查（我们团队习惯使用`plugin:vue/recommended`）
  - 其它类型文件不需要`lint`，直接使用`prettier`来做格式化工作

## 直接启用 vue-cli 内部集成的 eslint

- `vue-cli`创建的项目内部已经集成了`eslint`，并配置好了相关配置；我们只需要安装相关依赖，并在配置中启用即可
  - 开启后，在开发环境就可在`webpack`每次`bundle`时自动`lint`

```javascript
// build/webpack.base.conf.js

'use strict'
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')
function resolve(dir) {
return path.join(__dirname, '..', dir)
}
const createLintingRule = () => ({
    test: /\.(js|vue)$/,
    loader: 'eslint-loader',
    enforce: 'pre',
    include: [resolve('src'), resolve('test')],
    options: {
        formatter: require('eslint-friendly-formatter'),
        emitWarning: !config.dev.showEslintErrorsInOverlay
    }
})
...

// config/index.js

module.exports = {
    dev:{
        ...
        useEslint:true // 开启即可
    }
```

- 优缺点
  - 优点
    - 这是最快捷、最快速的办法
  - 缺点
    - `webpack`每次`bundle`时，都会进行全局`lint`，效率较低；
      - 可以通过配置`cache`有所缓解
    - `eslint`仅针对`.js、.vue`实现了检查和格式化，但其它文件如`.md`的格式化并未实现

## ESLint + Prettier 配合使用

### ESLint + Prettier 如何配合使用

- ESLint 侧重代码静态检查
- Prettier 侧重代码风格格式化
- 何不将二者结合起来呢？
- `Prettier`官方文档介绍了 3 种将`Prettier`结合`ESLint`使用的方法
  - [https://prettier.io/docs/en/eslint.html](https://prettier.io/docs/en/eslint.html)

1. 使用`eslint-plugin-prettier`
   - 会对比格式化前后的代码，将不一致的地方标记为错误，在 eslint 中抛出

```bash
// .eslintrc
{
    "plugins": ["prettier"],
    "rules": {
        "prettier/prettier": "error"
    }
}
```

2. 使用`eslint-config-prettier`
   - 关闭`eslint`中与`prettier`冲突的`rules`

```bash
// .eslintrc
{
    "extends": ["prettier"]
}
```

3. 使用`"plugin:prettier/recommended"`
   - 结合了 1、2 方法；推荐使用

```bash
// .eslintrc
{
    "extends": ["plugin:prettier/recommended"]
}
```

- 针对非`vue`项目，这么配置就可以在`eslint --fix`时使用`eslint`做代码检查，使用`prettier`做格式化了
- 二者结合使用时，可能会遇到二者格式化相关规则冲突的问题，我们在`vue`项目中就遇到了一些问题

### 结合使用时遇到的问题

- 我们团队内部习惯使用`plugin:vue/recommended`和`standard`规则
- 我们尝试将其同`Prettier`格式化相关规则结合一起

```bash
{
    "extends": ["plugin:vue/recommended","plugin:prettier/recommended","standard"]
}
```

- 我们发现保存自动格式化时`"plugin:vue/recommended","plugin:prettier/recommended"`这两个规则会产生很多冲突
  - 例如，二者会因为传递给组件的`props`是否需要在同一行还是另起一行产生冲突。最后导致代码错误。
    - 解决方法
      - 关闭其中一种规则
      - 不使用`plugin:vue/recommended`，改用约束性较低的`plugin:vue/essential`规则
  - 我们发现我们还是更倾向于`plugin:vue/recommended`中定义的规则，所以我们决定在`vue`文件中不使用`"plugin:prettier/recommended"`规则来做格式化；在其他文件中调用`prettier --write`来格式化

```bash
{
    "extends": ["plugin:vue/recommended","standard"]
}
```

- 无论是直接启用`vue-cli`配置还是结合`prettier`使用，在我们项目中都有一些问题
  - 前者可以在`bundle`时自动`lint`，但没有使用`prettier`
  - 后者虽使用了`prettier`，却无法自动调用

### 使用 husky+lint-staged 改进

- 关闭`vue-cli`自带的`bundle`时`lint`的配置；在`git commit`之前自动做`lint检查`和`prettier格式化`
- `husky和lint-staged`的作用上面已经做过介绍，不做赘述，直接展示配置

```bash
// package.json
"husky": {
    "hooks": {
            "pre-commit": "lint-staged"
     }
},
"lint-staged": {
    "*.{js,vue}": [// 针对js、vue文件使用eslint --fix来做检查和格式化
        "eslint --fix",
        "git add"
    ],
    "*.{json,css,scss,less,sass,md,html,flow,ts,tsd}": [ // 针对非js、vue文件使用pretter格式化
        "prettier --write",
        "git add"
    ]
}
```

- 这样基本实现了，团队内部提交的代码风格一致，完成了预定目标
- 但我们团队内部使用的都是`vscode`，大家不想在提交代码时再做检查和格式化；希望能在开发、保存时就能发现错误、格式化

## VSCode 中实现自动提示错误，保存时自动格式化

- 在上一步配置的基础上，再借助`VSCode的ESLint、Prettier、Vetur`扩展就可以完成此目标
  - 三个扩展的作用可以看文章前面的介绍
- 我们希望针对`js、vue`文件，在保存时，自动发现低级错误并格式化；其它格式文件在保存时可以自动格式化

1. 首先安装`ESLint、Prettier、Vetur、Setting Sync`扩展
2. 配置`ESLint`扩展，开启保存时自动修复

```javascript
// settings.json
// * ------------------
// * eslint相关
// * ------------------
"eslint.enable": true,
"eslint.alwaysShowStatus": true, // 始终展示eslint状态
"eslint.validate": [
"javascript",
"javascriptreact",
    {
    "language": "vue",
    "autoFix": true // 自动修复.vue文件的错误和完成格式化工作
    }
],
"eslint.autoFixOnSave": true // 保存时自动修复
```

3. 开启`vscode`的保存时自动格式化选项

```javascript
// settings.json
// 保存时自动格式化。需要借助格式化器；
// 若此项为false，则所有格式化器（pretter插件）不能在保存时自动格式化；
// 通过npm包安装的格式化器如prettier、ESLint不受此项限制。
"editor.formatOnSave": true,
```

4. 在`vue`文件中禁用`pretter`扩展

```javascript
"prettier.disableLanguages": ["vue"],
```

5. 在`vue`文件中禁用`vetur`的格式化选项

```javascript
"vetur.format.defaultFormatter.html": "none",
"vetur.format.defaultFormatter.js": "none",
```

- 完整`settings.json`配置如下

```javascript
{
// * ------------------
// * 格式化相关
// * 主要思路：
// * .js和.vue文件使用eslint+eslint-plugin-vue配合autoFix来做代码检查和格式化；
// * .vue文件中禁用prettier和vetur对template和script部分的格式化
// * 其他类型文件如.css、.sass、.md等都使用prettier插件来格式化
// * ------------------
// 保存时自动格式化。需要借助格式化器；
// 若此项为false，则所有格式化器（pretter插件）不能在保存时自动格式化；
// 通过npm包安装的格式化器如prettier、ESLint不受此项限制。
"editor.formatOnSave": true,
// prettier插件配置
// 读取配置的优先级如下
// .prettierrc文件 > .editorconfig > prettier插件选项
"prettier.tabWidth": 2,
"prettier.singleQuote": true,
"prettier.semi": false,
// 在vue中使用eslint+eslint-plugin-vue并配合autoFix来做格式化，所以在vue中禁用prettier插件
"prettier.disableLanguages": ["vue"],
// vetur配置
// vetur中自带prettier格式化器，其也会读取项目中的.prettierrc配置文件
// 在.vue文件中将其禁用，使用eslint+eslint-plugin-vue配合autoFix来格式化；仅使用vetur中的prettier来格式化vue文件中的css部分
"vetur.format.defaultFormatter.html": "none",
"vetur.format.defaultFormatter.js": "none",
// * ------------------
// * eslint相关
// * ------------------
"eslint.enable": true,
"eslint.alwaysShowStatus": true, // 始终展示eslint状态
"eslint.validate": [
"javascript",
"javascriptreact",
{
"language": "vue",
"autoFix": true // 自动修复.vue文件的错误和完成格式化工作
}
],
"eslint.autoFixOnSave": true // 保存时自动修复
}
```

- 问题
  - 上述配置是建立在使用了`vue-cli`创建项目，并且项目中存在`eslint`的配置文件时，如果项目中没有`eslint`配置文件，那么`.vue`文件将无法格式化。
    - 解决方法
      - 注释掉在`.vue`文件禁用`prettier`相关设置

```javascript
// "prettier.disableLanguages": ["vue"],
// "vetur.format.defaultFormatter.html": "none",
// "vetur.format.defaultFormatter.js": "none",
```

- PS: `VSCode`设置的共享，可以借助`Settings Sync`扩展来实现，具体方法可 google

## 参考

- > [https://nice.lovejade.cn/zh/article/beautify-vue-by-eslint-and-prettier.html#pre-commit-hook-约束代码提交](https://nice.lovejade.cn/zh/article/beautify-vue-by-eslint-and-prettier.html#pre-commit-hook-%E7%BA%A6%E6%9D%9F%E4%BB%A3%E7%A0%81%E6%8F%90%E4%BA%A4)

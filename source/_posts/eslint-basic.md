---
title: eslint-basic
tags:
  - eslint
categories:
  - 前端
date: 2019-03-12 18:54:53
---
# eslint 配置记录

> 最近项目中需要集成eslint，学习后特在此做个记录

## 介绍
- 官方介绍
    - ESLint 是一个开源的 JavaScript 代码检查工具，由 Nicholas C. Zakas 于2013年6月创建。代码检查是一种静态的分析，常用于寻找有问题的模式或者代码，并且不依赖于具体的编码风格。对大多数编程语言来说都会有代码检查，一般来说编译程序会内置检查工具。
    - 总结
        - 代码静态检查
            - 提前发现低级错误；如声明了变量却没有使用、使用了`==`
        - 代码风格检查（此能力较弱）
            - 团带整体编码风格的检查；如单双引号、分号等的使用

## 安装

### 安装及初始化
```bash
npm i -D eslint // 安装为开发依赖

npx eslint --init // 会以交互形式生成eslint配置文件
```

## 配置

### 配置文件格式
- eslint的配置文件可以有多种格式
    - `.eslintrc`(废弃)
    - `.eslintrc.js`
    - `.eslintrc.json`
    - `.eslintrc.yml`
    - 还可以在`package.json`中声明配置
- PS：经常能看到配置文件以`rc`结尾，其实这是约定俗成的配置文件的命名方法，是`run control运行控制`的意思

### 优先级
- 既然有这么文件格式，那同时出现时，肯定有个优先级
- `.eslintrc.js > .eslintrc.yaml > .eslintrc.yml > .eslintrc.json > .eslintrc > package.json`
    - `package.json`优先级最低，`.eslintrc..js`最高，所以可以直接使用`.eslintrc.js`
- eslint还支持在文件中通过注释的形式来声明配置选项，如`/*eslint-disable*/`
    - 这种方式的优先级要高于配置文件的形式

## 配置

### 配置信息分类
- 环境相关
    - 需要lint的脚本运行环境，每种环境都有一组特定的预定义全局变量。；eslint在检查时，就不会将一些全局变量检测为为未定义而报错
- 全局变量
    - 需要跳过eslint未定义检查的全局变量；如`jQuery、Zepto`这些未在环境中预置但又在脚本执行时访问的额外的全局变量
- 规则相关
    - 配置启用哪些规则及错误级别

### 规则错误级别

| 字符串表示 | 数字表示 | 说明                                                             |
| ---------- | -------- | ---------------------------------------------------------------- |
| "off"      | 0        | 关闭当前规则检查                                                 |
| "warn"     | 1        | 启用当前规则检查，不符合当前规则，会报一个警告（不影响现有代码） |
| "error"    | 2        | 启用当前规则检查，不符合当前规则，会报一个错误（中断后续流程）   |

### 配置项说明

- `root`
    - 告知eslint在查找配置文件时，到此文件结束，不再往上级目录查找
- `parser`
    - 指定eslint解析器，默认为Espree；
    - 只有在使用了eslint无法解析的babel特性时，才需要替换为babel-eslint；babel-eslint会提供一份能被eslint正常解析的js文件，eslint基于此再做静态检查
    - 正常情况下，都会使用babel-eslint
- `parserOptions`
    - 解析器选项
    ```javascript
    parserOptions: {
        ecmaVersion: 6, // ES的版本，默认为5
        sourceType: 'module', // 指定源码类型，script | module，默认为script。
        ecmaFeatures: {
        // 你想使用的额外的语言特性
        experimentalObjectRestSpread: true, // 启用扩展运算符
        jsx: true, // 启用jsx语法
        globalReturn: true, // 允许return在全局使用
        impliedStrict: true // 启用严格校验模式
    }
    ```
- `env`
    - 设置源码运行的环境，每种环境都有一组特定的预定义全局变量
    - eslint在检查时，就不会将一些全局变量检测为为未定义而报错
    -  环境变量是非互斥的，可以添加多个
    -  具体可查询这里：https://cn.eslint.org/docs/user-guide/configuring#specifying-environments
-  `globals`
    - 额外使用的全局变量
- `extends`
    - 一个配置文件可以从基础配置中继承已启用的规则，extends字段就是用来配置继承的规则
    - extends 属性值可以是：
        - 在配置中指定的一个字符串
        - 字符串数组：**每个配置继承它前面的配置，出现重复规则时，如extends:[1,2,3] 配置2会覆盖1，3会覆盖2，类似Object.assign**
    - eslint内置规则默认情况下都是禁用的
        - 可以通过`eslint:recommended`或者`eslint:all`启用
    - 还可使用第三方的规则，如eslint-config-standard、eslint-config-airbnb等等
        - 在指定时，可忽略前面个的slint-config-
        - 如下面的standard，全名为eslint-config-standard
    - 若使用插件中的规则
        - 可使用 `plugin:插件name/规则名` 的格式来指定
- `rules`
    - 在extends基础上额外附加的规则，其优先级高于extends的规则，会覆盖他们
- `plugins`
    - 针对一些特殊情况，需要使用插件
    - 插件能赋予eslint一些其他能力也会包含一些规则
- `settings`
    - 指明需要在不同配置文件中共享的数据
- 示例
```javascript
module.exports = {
    root: true, // 告知eslint在查找配置文件时，到此文件结束，不再往上级目录查找
    // 指定eslint解析器，默认为Espree；
    // 只有在使用了eslint无法解析的babel特性时，才需要替换为babel-eslint；babel-eslint会提供一份能被eslint正常解析的js文件，eslint基于此再做静态检查
    // 正常情况下，会使用babel-eslint
    parser: 'babel-eslint',
    // 解析器选项
    parserOptions: {
        ecmaVersion: 6, // ES的版本，默认为5
        sourceType: 'module', // 指定源码类型，script | module，默认为script。
        ecmaFeatures: {
            // 你想使用的额外的语言特性
            experimentalObjectRestSpread: true, // 启用扩展运算符
            jsx: true, // 启用jsx语法
            globalReturn: true, // 允许return在全局使用
            impliedStrict: true // 启用严格校验模式
        }
    },
    env: {
        // 设置源码运行的环境，每种环境都有一组特定的预定义全局变量；
        // eslint在检查时，就不会将一些全局变量检测为为未定义而报错
        // 环境变量是非互斥的，可以添加多个
        // 具体可查询这里：https://cn.eslint.org/docs/user-guide/configuring#specifying-environments
        browser: true, // 添加浏览器环境下的全局变量 如 window、document等
        es6: true // 添加es6的全局变量
    },
    globals: {
        // 额外使用的全局变量
        Zepto: true // true代表不能重写此变量
    },
    // 一个配置文件可以从基础配置中继承已启用的规则，extends字段就是用来配置继承的规则
    // extends 属性值可以是：
    // 在配置中指定的一个字符串
    // 字符串数组：每个配置继承它前面的配置，出现重复规则时，如[1,2,3] 配置2会覆盖1，3会覆盖2，类似Object.assign
    // eslint内置规则默认情况下都是禁用的，可以通过`eslint:recommended`或者`eslint:all`启用
    // 还可使用第三方的规则，如eslint-config-standard、eslint-config-airbnb等等
    // 在指定时，可忽略前面个的slint-config-
    // 如下面的standard，全名为eslint-config-standard
    // 若使用插件中的规则
    // 可使用 plugin:插件name/规则名 的格式来指定
    extends: [
        'plugin:vue/recommended', // 使用eslint-plugin-vue中的recommended规则
        'standard'
    ],
    // 在extends基础上额外附加的规则，其优先级高于extends的规则，会覆盖他们
    rules: {
        // 关闭不允许同一个var声明多个变量
        'one-var': 'off'
    },
    // 针对一些特殊情况，需要使用插件
    // 插件能赋予eslint一些其他能力也会包含一些规则
    plugins: ['vue'],
    // 指明需要在不同配置文件中共享的数据
    settings: {
        sharedData: 'Hello'
    }
}
```

## 行内规则

### 常用行内规则
- `eslint-disable`
    - 当前文件下面所有代码禁用eslint检查
    - 在不希望使用eslint的文件顶部添加
```javascript
/* eslint-disable */

alert('foo');

/* eslint-enable */
```
- `eslint-disable-next-line`
    - 下面一行代码，禁用eslint检查
```javascript
/* eslint-disable-next-line */
alert('foo');
```

## 忽略文件

### .eslintignore
- 使用`.eslintignore`文件声明需要忽略的文件及目录
    - `.eslintignore`中使用`glob`模式表明哪些路径应该忽略检测
```javascript
//  .eslintignore
**/*.js
```

## eslint-cli

### 常用操作
- 生成配置文件
```bash
eslint --init // 以交互形式生成.eslintrc文件
```
- 修复错误
    - 绝大部分修复的是格式化的错误，如空白符、分号等
```bash
eslint --fix file.js // 修复file.js中的格式化错误
```
- 指定额外的配置文件
```bash
eslint -c myconfig.js file.js // 指定配置并lint当前目录下的file.js
```
- 指定环境变量
```bash
eslint --env browser,node file.js // 指定环境变量
```
- 指定扩展
    - `--ext` 只有在参数为目录时，才生效。如果你使用`glob 模式`或`文件名`，`--ext` 将被忽略
```bash
eslint --ext .js,.vue src // lint src目录下后缀为js、vue的文件
```
- 缓存
    - 存储处理过的文件的信息以便只对有改变的文件进行操作
    - 缓存默认被存储在 `.eslintcache`中
```bash
eslint "src/**/*.js" --cache // 只对改变的文件进行lint
```
- 彩色输出
```bash
eslint --color file.js
```


## 参考
> [https://juejin.im/post/5bab946cf265da0ae92a75ca](https://juejin.im/post/5bab946cf265da0ae92a75ca)


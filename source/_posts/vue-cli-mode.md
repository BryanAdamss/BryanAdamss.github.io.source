---
title: vue-cli-mode
tags:
  - vue-cli
  - webpack
categories:
  - 前端
date: 2020-06-22 16:12:13
---

# vue-cli4 mode、NODE_ENV、webpack mode

- 最近使用`vue-cli4`初始化了一个项目，对其中`mode、NODE_ENV、webpack mode`的关系有点迷
- 所以花了点时间整理了下

## vue-cli4 环境变量

### 声明环境变量

- `vue-cli`可通过`.env`前缀的环境文件来声明环境变量

```sh
# .env
Foo=Bar
VUE_APP_TITLE=标题
```

- 被载入的变量将会对`vue-cli-service`的所有命令、插件和依赖可用

### 客户端可以使用的环境变量

- 在客户端(`src/目录下`)可以使用`process.env.XXX`来引用环境变量，主要有下面的变量可以引用
  - 环境文件中以`VUE_APP_`开头的环境变量
  - `NODE_ENV` ， 会是  `development`、`production`  或  `test`  中的一个
  - `BASE_URL`，会和  `vue.config.js`  中的  `publicPath`  选项相符

## vue-cli mode

### 默认 mode

- `mode`可以理解为`vue-cli`运行的一个环境，一个标志量；内部的插件依赖此变量来决定是否启用
- `vue-cli mode`默认只会有`development`、`production`、`test`三种取值
- `development`
  - 会在调用`vue-cli-service serve`命令时，`mode`被设置为`development`
- `production`
  - 会在调用`vue-cli-service build`  和` vue-cli-service test:e2e`命令时，`mode`被设置为`production`
- `test`
  - 会在调用`vue-cli-service test:unit`命令时，`mode`被设置为`test`
- 每个模式都会将`NODE_ENV`  的值设置为模式的名称——比如在 `development` 模式下  `NODE_ENV`  的值会被设置为`development`

### 覆盖、自定义 mode

- 可以使用`vue-cli-service xxx --mode custom`来覆盖原有`mode`(自定义 mode)

## vue-cli mode 和环境文件的关系

- 当指定了`mode`时(无论是命令默认值还是`--mode` 自定义的)
- 会去项目根目录下找到`.env.[mode]`文件，读取并设置其中的环境变量
- 如果`mode`的值是除`development、production、test`以外的值，则需要在环境文件中手动指定`NODE_ENV`值，这样`vue-cli`内部的插件才知道需要工作在哪种环境下
- 例子
- `vue-cli-service build --mode staging`，使用`--mode`覆盖了默认的`production`模式，则需要在`.env.staging`环境文件中手动指定`NODE_ENV`

```sh
# .env.staging
NODE_ENV=production
VUE_APP_TITLE=My App (staging)

```

- `vue-cli mode`和`NODE_ENV`及`webpack mode`三者间是如何互动，下面会解释

## vue-cli mode、NODE_ENV、webpack mode 三者的关系

### webpack mode

- webpack 从 4 开始，支持`mode`选项，根据`mode`自动开启一些插件和优化
- `mode`的可选值有
  - `development`
    - 告知`webpack`启用  `NamedChunksPlugin`  和  `NamedModulesPlugin`
  - `production`
    - 告知`webpack`启用  `FlagDependencyUsagePlugin`、 `FlagIncludedChunksPlugin` 、`ModuleConcatenationPlugin`、`NoEmitOnErrorsPlugin`、` OccurrenceOrderPlugin`、`SideEffectsFlagPlugin`、`UglifyJsPlugin`
    - 启用压缩、优化
- `vue-cli`内部会使用到`webpack`，所以也需要设置`webpack mode`

### 三者关系

- 首先在`vue-cli`中`NODE_ENV`默认被设置为`development`，除非`vue-cli mode`为`production`或者`test`
- 其次，环境文件中的`NODE_ENV`具有更高的优先级，即如果环境文件中也有设置`NODE_ENV`，则会覆盖上一步的`NODE_ENV`
- 最后`webpack mode`是根据上一步的`NODE_ENV`值来确定的，只要值不是`production`，则`webpack`都会运行在`development`模式

### 例子 1

- `vue-cli-service serve`
  - `mode`初始为`development`，`NODE_ENV`初始为`development`
- `.env.development`

```sh
NODE_ENV=production # 被覆盖为production
```

- `NODE_ENV`最终为`production`，所以`webpack mode`最终为`production`，会启用压缩、优化等行为

### 例子 2

- `vue-cli-service build --mode staging`
  - 通过`cli`参数覆盖默认`mode`，此时`mode`为`staging`
- `.env.staging`

```sh
# 未设置NODE_ENV

VUE_APP_TITLE=标题
```

- 由于使用了自定义`mode`、环境文件中又没有设置`NODE_ENV`，导致`NODE_ENV`为默认值`development`、所以`webpack mode`为`development`，并不会启用压缩等行为

### 例子 3

- `vue-cli-service build --mode staging`
  - 通过`cli`参数覆盖默认`mode`，此时`mode`为`staging`
- `.env.staging`

```sh
NODE_ENV=production # 设置NODE_ENV
VUE_APP_TITLE=标题
```

- 环境文件中设置了`NODE_ENV=production`，所以`webpack mode`为`production`

### 例子 4

- `vue-cli-service build --mode staging`
  - 通过`cli`参数覆盖默认`mode`，此时`mode`为`staging`
- `.env.staging`

```sh
NODE_ENV=staging # 设置NODE_ENV为非production、development、test的值
VUE_APP_TITLE=标题
```

- 环境文件中设置了`NODE_ENV=staging`，所以`webpack mode`会运行在默认`development`环境下

## 参考

- [https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-service/lib/Service.js#L111](https://github.com/vuejs/vue-cli/blob/dev/packages/%40vue/cli-service/lib/Service.js#L111)
- [https://github.com/vuejs/vue-cli/issues/4699](https://github.com/vuejs/vue-cli/issues/4699)
- [https://github.com/vuejs/vue-cli/issues/4743](https://github.com/vuejs/vue-cli/issues/4743)

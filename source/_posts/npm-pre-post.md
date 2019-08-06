---
title: npm-pre-post
tags:
  - npm
  - pre
  - post
  - git
categories:
  - 前端
date: 2019-03-17 00:45:17
---

# npm script 的 pre、post 钩子及在 pre commit 时进行 lint

## pre、post 钩子

### 介绍

- `npm run`为每条命令提供了`pre-`和`post-`两个钩子（hook）。
- 以`npm run lint`为例，执行这条命令之前，`npm`会先查看有没有定义`prelint`和`postlint`两个钩子，如果有的话，就会先执行`npm run prelint`，然后执行`npm run lint`，最后执行`npm run postlint`
- 可以在这两个钩子中来完成一些其它事情，如在执行`test`前先执行`lint`操作

### 示例

```bash
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "lint": "eslint --ext .js,.vue src",

     "test": "karma start --log-leve=error karma.config.js --single-run=true",
     "pretest": "npm run lint",
     "posttest": "echo 'Finished running tests'"
}
```

- 执行`npm run test`时会先执行`npm run pretest`再执行`npm run test`最后执行`npm run posttest`
- 中间任意一环节报错，就会中断后续执行
- 不能在`pre脚本`之前`再加pre`，即`prepretest脚本`不起作用。
- 即使`npm`可以自动运行`pre`和`post`脚本，也可以手动执行它们

### 缺陷

- `pre、post`钩子只能在`npm run`时生效，没法在其他场景使用；
- 如需要在`git commit`时先`lint`代码

### 改进

- 使用`husky`

## husky

### 介绍

- `husky` 可以在`git commit`和`git push`时执行自定义脚本
- 可以利用这个特性在`commit`时做`lint`工作

### 示例

```bash
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "lint": "eslint --ext .js,.vue src",
},
"husky": {
    "hooks": {
        "pre-commit": "npm run lint" // 在commit前执行npm run lint
    }
}
```

### 注意

- 老版本`husky`的钩子是直接定义在`scripts`中的，最新版本将其转移到`husky`字段下了
- 老版本`husky`的钩子名称一般为`precommit、prepush`，现在统一改成了`pre-commit、pre-push`

### 缺点

- 每次针对所有文件进行`lint`，我们希望仅针对修改的或者说是`staged`的文件进行`lint` -`husky`配合`lint-staged`使用

## lint-staged

### 介绍

- `husky`可以让我们在`git commit`和`git push`时执行一段脚本
- `lint-staged`则可以针对`add`进`git 暂存区`的文件进行`lint`操作

### 使用

- 首先安装
  - `npm i -D lint-staged husky`
- 定义`lint-staged`相关脚本
  - 可针对不同类型文件执行不同脚本，参考示例

### 示例

```bash
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "lint": "eslint --ext .js,.vue src",
},

"lint-staged": {
    "*.{js}": [ // 针对git 暂存区中的文件先执行eslint --fix，没有报错时，再add进暂存区
            "eslint --fix",
            "git add"
        ]
}
,
"husky": {
    "hooks": {
        "pre-commit": "lint-staged" // 在commit前执行lint-staged任务
    }
}
```

## 参考

> [http://javascript.ruanyifeng.com/nodejs/npm.html#toc15](http://javascript.ruanyifeng.com/nodejs/npm.html#toc15)

---
title: npm-publish-flow
tags:
  - npm
  - package
  - publish
categories:
  - 其他
date: 2019-10-16 21:46:34
---

# npm 发包流程总结

## 注册npm账号
- [https://www.npmjs.com/signup](https://www.npmjs.com/signup)

## package.json中指明必要字段
- `name、version`是必须的，缺失将无法发布
- `main`指向入口文件，一般为`commonjs、esm`格式
- `module`指向`esm`格式入口文件
- `unpkg`字段
    - 发布到`npmjs.com`中的包会自动同步到`unpkg.com`上
    - 通过`unpkg`字段指明`unpkg.com`应该如何查找入口文件
    - 一般为`umd`格式
- `repository`
```javascript
"repository": {
    "type": "git",
    "url": "git+https://github.com/BryanAdamss/drawing-board.git"
  }
```
- 参考[https://juejin.im/post/5b231f6ff265da595f0d2540](https://juejin.im/post/5b231f6ff265da595f0d2540)

## 检查包名是否重名
- 重名时，将报403错误，提示没有权限提交包(你没有权限修改`别人`的包)
    - `403 Forbidden - PUT http://registry.npmjs.org/react-native-app-info - You do not have permission to publish "react-native-app-info". Are you logged in as the correct user?`
- npm会使用`package.json`中的`name`字段做为包名
- `https://www.npmjs.com/`中搜索包名，检查是否重名
- 有重名，则需要修改包名
    -  修改为`scope package`即`@scope/package-name`
        -  这种在`npm publish`时会被自动认定为私有仓库，私有仓库是需要付费的
        -  需要在`npm publish`时追加`--access public`指明包为公开访问
            -  `npm publish --access=public`
            -  注意的是这种形式的包名跟 npm 账户有对应关系，不能随便填写
                -  需要在`init`时指明`scope`
                    -  `npm init --scope=@user-name`
    -  直接修改为一个不会重名的包名
-  参考[https://github.com/XXHolic/segment/issues/28](https://github.com/XXHolic/segment/issues/28)

## 检查仓库源
- 国内使用一般都会切到淘宝源(https://registry.npm.taobao.org/)上，在发包时，需要切到npm官方源(https://registry.npmjs.org/)
- `npm config ls`查看当前源
    - 若源不正确，需要使用`npm config set registry https://registry.npmjs.org/`切换到官方源

## 登录
- 使用`npm login或npm adduser`命令
- 然后输入账号、密码登录
- 登录成功会提示
    - `foo@bar.com Logged in as username on http://registry.npmjs.org/.`
    - 可通过提示信息，判断源是否真确

## 发包publish
- 登录后，直接`npm publish`即可
    - 记得`publish`前要先构建一波

## 使用stand-version做发包版本控制
```javascript
// package.json
 "scripts": {    
    "release": "standard-version",
  }


// 初次发布版本
npm run release --first-release
// 添加版本信息和指定发布版本等级
npm run release -m "Commit message" -r minor

// 确认发布，npm publish 发布到 npm
git push --follow-tags origin master && npm publish
```
- 参考[https://segmentfault.com/a/1190000015376832](https://segmentfault.com/a/1190000015376832)

## 常见错误
 - 无法login
        - 源不对
- 403
    - 包重名
        - 修改包名
    - 没有发包权限
        - 联系管理员开通权限
    - 包版本重复
        - 更新发包版本
    - 源不对
        - 切换源
- 发布成功后，使用淘宝源无法下载最新包
    - 淘宝源同步问题
        - 等待淘宝源同步后再下载
        - 切换到官方源下载
- 参考[https://www.jianshu.com/p/5ea8e50d628e](https://www.jianshu.com/p/5ea8e50d628e)

## 总结
- 注册账号
- 检查`package.json`必要字段
- 检查重名
- 设置官方源
- 登录
- 发布
- 重设淘宝源

## demo
```javascript

{
  "name": "@bryanadamss/drawing-board",
  "version": "1.0.1",
  "description": "canvas绘图板，提供绘制、撤销、旋转、下载等功能",
  "main": "dist/drawing-board.es.js",
  "module": "dist/drawing-board.es.js",
  "unpkg": "dist/drawing-board.umd.js",
  "scripts": {
    "setNpmRegistry": "npm config set registry https://registry.npmjs.org/",
    "setTaoBaoRegistry": "npm config set registry https://registry.npm.taobao.org/",
    "prepublishOnly": "npm run build && npm run setNpmRegistry",// 发布前构建、设置npm源
    "postpublish": "npm run setTaoBaoRegistry",// 发布后，重设为淘宝源
    "build": "rollup --config rollup.config.js",
    "release": "standard-version",
    "release:first": "npm run release --first-release",
    "publishToNpm": "git push --follow-tags origin master && npm publish --access=public"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/BryanAdamss/drawing-board.git"
  },
  "keywords": [
    "canvas",
    "drawing",
    "paint"
  ],
  "author": "GuangHui",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.5.5",
    "@babel/plugin-proposal-class-properties": "^7.5.5",
    "@babel/preset-env": "^7.5.5",
    "core-js": "^3.2.1",
    "rollup": "^1.20.3",
    "rollup-plugin-babel": "^4.3.3",
    "rollup-plugin-babel-minify": "^9.0.0",
    "rollup-plugin-commonjs": "^10.1.0",
    "rollup-plugin-node-resolve": "^5.2.0",
    "standard-version": "^7.0.0"
  },
  "bugs": {
    "url": "https://github.com/BryanAdamss/drawing-board/issues"
  },
  "homepage": "https://github.com/BryanAdamss/drawing-board#readme",
  "dependencies": {}
}
```

## 项目链接
- [https://github.com/BryanAdamss/drawing-board](https://github.com/BryanAdamss/drawing-board)








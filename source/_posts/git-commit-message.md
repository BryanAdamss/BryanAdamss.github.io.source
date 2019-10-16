---
title: git-commit-message
tags:
  - git
  - commit message
categories:
  - 其他
date: 2019-10-16 21:48:55
---

# git提交规范Commit Message

## 规范
- 社区推荐
    - [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
    - 此提交风格最初用angular，后来独立开发成一个规范
    - 很多开源项目使用此提交风格，如vue

## Conventional Commits介绍
```bash
<type>([optional scope]): <description>
// 空一行
[optional body]
// 空一行
[optional footer(s)]
```
- 包含header、body、footer
    - header必填
    - body、footer可省略
- header包含以下几个部分
    - type
        - 提交类型
        - 必填
        - 有以下枚举值
            - feat
                - 新功能、新特性(feature)
            - fix
                - 修补bug
            - docs
                - 文档(documentation)
            - style
                - 格式(不影响代码运行的变动)
            - refactor
                - 重构(即不是新增功能，也不是修改bug的代码变动)
            - test
                - 增加测试
            - chore
                - 构建过程或辅助工具的变动
    - scope
        - 此次commit影响的范围
        - 非必填
        - 一般是名词
    - description
        - commit的简短描述
- body
    - 此次commit的详细描述
    - 使用第一人称
    - 说明改动的动机，改动前后的对比
- footer
    - 适用于以下情况
        - 存在不兼容变动时
            - footer部分要以`BREAKING CHANGE`开头
        - 关闭issue
            - 在footer部分指明关闭哪个issue
            
## 工具
- `commitizen`cli
    - 类似postcss、babel是一个通用工具，通过插件支持各种风格的互动提交
- `cz-conventional-changelog`包
    - 一个commitizen adapter，用以支持`Conventional Commits`提交风格
- `@commitlint/cli @commitlint/config-conventional husky`
    - 这3个包是用来在提交时，对提交message做lint用

## commitizen安装

- 本地安装commitizen
```
npm i -D commitizen
```
- 因为本地安装commitizen，所以需要使用下面命令(npm>=5.2)初始化安装cz-conventional-changelog
```
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```
- npm<5.2版本，可以使用下面命令来安装
```
./node_modules/.bin/commitizen init cz-conventional-changelog --save-dev --save-exact
```
- 若全局安装了commitizen，则在项目根目录中使用下面命令安装cz-conventional-changelog
```
commitizen init cz-conventional-changelog --save-dev --save-exact
```
- 经过以上步骤后，整个项目就是Commitizen-friendly
    - 会自动在package.json中添加下面字段，用于告诉commitizen，我们希望贡献者使用哪个adapter(cz-conventional-changelog)来提交commit
```
"config": {
    "commitizen": {
      "path": "path/to/cz-conventional-changelog"
    }
  }
```
- package.json中添加以下script
```
  "scripts": {
    "commit": "git-cz"
  }
```
- `npm run commit`将会出现交互界面
    - `vs code` 用户也可以用`vscode-commitizen`插件来提交
    
 ## commit-lint安装
 - 安装
 ```
 npm i -D @commitlint/cli @commitlint/config-conventional husky
 ```
 - 配置commitlint
 ```
 // 项目根目录新建 commitlint.config.js包含以下内容
 
 module.exports = {
  extends: ['@commitlint/config-conventional']
} 
```
 - 配置package.json
 ```
 {
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  },
}
```
 - 这样在每次提交commit时都会针对提交风格做lint
 
 ## 使用stand-version做发包版本控制
 - 自动生成CHANGELOG
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
git push --follow-tags origin master
```
- 参考[https://segmentfault.com/a/1190000015376832](https://segmentfault.com/a/1190000015376832)
 
 
 ## 项目地址
 - [vue-awesome-template](https://github.com/BryanAdamss/vue-awesome-template)

## 参考
- [https://github.com/commitizen/cz-cli](https://github.com/commitizen/cz-cli)
- [http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)
- [https://segmentfault.com/a/1190000016503661](https://segmentfault.com/a/1190000016503661)
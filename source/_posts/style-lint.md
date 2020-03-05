---
title: style-lint
tags:
  - stylelint
categories:
  - 前端
date: 2020-03-05 11:46:28
---

#  配置stylelint并在保存时自动修复

## 安装

- vscode安装插件`stylelint-plus`，其支持`autoFixOnSave`，其它插件不支持
- 插件关键配置
```json

// * ------------------
  // * stylelint相关
  // * 主要思路：
  // * 使用vscode插件stylelint-plus插件，并开启autoFixOnSave
  // * 由于stylelint-plus内部自带stylelint，项目中可不需要再安装stylelint包
  // * stylelint-plus机制和vscode eslint插件机制类似，会读取项目中的stylelint配置文件
  // * 若项目无配置文件，则使用自带的默认配置
  // * 项目配置中若声明了plugins，则需要使用npm 安装对应的plugins包
  // * 由于vscode自带css、less、scss的校验，为避免重复检查，可选择关闭vscode的校验
  // * ------------------
  "stylelint.autoFixOnSave": true,
  "css.validate": false,
  "less.validate": false,
  "scss.validate": false,
```

## package.json
- 添加以下包

```json
    "stylelint": "^12.0.0",
    "stylelint-config-rational-order": "^0.1.2",
    "stylelint-order": "^3.1.1",
    "stylelint-selector-bem-pattern": "^2.1.0"
```

## .stylelintrc
- 参照以下文章，使用如下配置，可解决vsocde保存时报：[Error: severity property of a stylelint warning must be either 'error' or 'warning', but it was 'ignore' (string). at stylelintWarningToVscodeDiagnostic.](https://github.com/constverum/stylelint-config-rational-order/issues/16)
- https://github.com/constverum/stylelint-config-rational-order/issues/13#issuecomment-522421878
- https://github.com/constverum/stylelint-config-rational-order/issues/10

```json
{
// 不再使用"extends"继承stylelint-config-rational-order配置，而是只使用stylelint-config-rational-order/plugin
  "plugins": [
    "stylelint-order", // 显示添加stylelint-order
    "stylelint-config-rational-order/plugin", // 添加stylelint-config-rational-order/plugin
    "stylelint-selector-bem-pattern" // 规定必须需要suit规范的class名；其内部使用了https://github.com/postcss/postcss-bem-linter插件，不再默认用suit 规范(postcss-bem-linter插件默认使用suit css规范)，所以使用suit规范，需要手动指定preset
  ],
  "rules": {
    "order/properties-order": [], // 参照issue #10 ，添加此配置，并且此项最好在rules首位
    "plugin/rational-order": [
      true,
      {
        "border-in-box-model": false, // 将border放到视觉部分，而不是盒模型
        "empty-line-between-groups": true, // 不同分组间，添加一个空行     
      }
    ],
    "plugin/selector-bem-pattern": {
      "preset": "suit", // 使用suit 预设
      "presetOptions": {
        "namespace": "c" // 指明命名空间，组件名必须以c-开头
      },
      // 指名src/**/*.vue都是component，其<style>中的类名必须是c-文件名
      "implicitComponents": [
        "src/**/*.vue"
      ],
      // 指名ssrc/sass/helpers/*.scss都是utils，类名必须u-开头
      "implicitUtilities": [
        "src/sass/helpers/*.scss"
      ]
    },
  }
}
```
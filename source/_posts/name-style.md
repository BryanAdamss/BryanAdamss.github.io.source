---
title: name-style
tags:
  - 命名
categories:
  - 其他
date: 2018-11-07 19:04:47
---

> 工作中，经常因命名规范而头疼，所以特意整理了部分规范，就便后期查找

# 文件及文件夹命名规范

## 原则

- 命名尽量简短，不要产生多个单词；尽量采用约定俗称的名称；
- 需要重点标识的可采用特殊标识去；

## 规范

- 文件夹

  - 多个单词时：统一采用全小写+连字符；如 `source-code`
    http://www.ruanyifeng.com/blog/2017/02/filename-should-be-lowercase.html
  - 特殊情况需要特别标识的可以采用帕斯卡命名方式，如 vue 组件的文件夹 SyncClass

- 文件

  - 统一采用全小写+连字符形式：`auto-height.js`

    - 特殊情况需要特别标识的可以采用帕斯卡命名方式，如 `vue` 组件 `SyncClass.vue`
    - 特殊文件可采用全大写形式，如 `README`
    - 文件内部内容命名
      - js 变量
        - 驼峰 `isLoading`
      - js 常量
        - 大写+下划线 `ERROR_TEXT`
      - sass 变量
        - $开头，小写+连字符 `$color-red`
      - vue
        - 组件 name
          - 帕斯卡 `name:'ErrorCorrection'`
        - 组件调用
          - 帕斯卡 `<ErrorCorrection></ErrorCorrection>`

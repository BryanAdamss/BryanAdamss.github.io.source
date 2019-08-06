---
title: postman-guide
tags:
  - postman
categories:
  - 其他
date: 2018-02-24 09:34:28
---

> 最近在看postman文档(https://www.getpostman.com/docs/)
> 记录一些常用操作

# postman 常用操作

## 介绍
- postman是一个调试、管理接口的神器。
- 登不登录都可以使用，登录后可以同步数据。

## 界面
- ![](preview.png)
- 主要分两部分
    - 左侧为侧边栏，包含历史记录(History)、集合(Collections)
    - 右侧为request builder
- 历史记录Tab页
    - 主要记录了之前操作的一些request；可以通过`clear all`来清空；以及上面的`filter`来筛选
    - 每个请求左侧都会标识出请求的类型(post、get...)
    - ![](side.png)
- 集合Tab页
    - 主要用来展示所有的项目
    - 一般一个集合对应着一个项目，如`ShangCheng`就对应着一个项目；
    - 黄色星星代表置顶项目
    - 项目可以按照字母或者时间来排序
    - 可以通过右键->`edit`对当前`collection`进行详细编辑，如添加描述、脚本等
    - ![](collections.png)
- request builder
    - 主要用来构建请求
    - 看下图，一共有5个红框，依次为
        - 第一个框右侧为环境相关选项，下拉框可以选择当前request发送的环境(在环境中会定义一些变量)，比如可以为请求定义开发环境和生产环境，二者请求的url不同；眼睛图标，可以查看当前环境下定义的一些变量的具体值；再右侧齿轮是环境的管理
        - 第二个框主要有当前请求的一些简单描述(注意在postman中可以使用markdown语法来写描述)；右侧`examples`可以用来保存当前request的example，供其他人查看请求可能返回的情况
        - 第三个框中最左侧可以选择请求的类型(get、post..)，再右侧是url地址(`path`是定义的一个变量)，`params`是添加查询字符串的按钮，点击后可以通过key-value的形式添加查询字符串；再右侧是发送请求(可通过`ctrl+enter`来快速发送)以及保存当前请求
        - 第四个框是request相关的，分别有
            - 授权
            - request header(通过key-value形式添加)
            - request body(会根据选择的请求类型，自动置灰，如选择get请求，request body会不可用)
                - 可以在`descriptions`给value添加描述
            - pre-request script(请求发送之前会执行的脚本，js编写)
            - tests(请求返回后会执行的script)
            - 右侧cookie是cookie管理
            - 右侧code是生成不同语言发送当前请求代码段的按钮，生成后可以直接拷贝使用
        - 最后一个框是response相关的
            - body展示的是response body
                - 可以通过pretty、raw(源码)、preview(传会图片时会有用)以不同形式来查看返回的值
            - cookie是后台返回的cookie
            - header是response header
            - test results是展示测试脚本执行的结果
    - ![](builder.png)

## 变量
- 主要用来复用
    - 常用的变量有环境变量、集合变量、全局变量
        - 环境变量仅在对应对应环境中起作用
        - 集合变量仅在当前集合中起作用
        - 全局变量则是在任何地方都起作用
        - 同名变量冲突时，作用范围小的生效，如一个变量同时在上述三个中都有定义，则环境变量中定义的生效
- 使用
    - 使用`双大括号包裹使用`
    - 如上文看到的`双大括号path`就是一个变量

## 传递参数
- 实际场景：在登录后，保存token，在后续的请求中带上token；这就涉及到如何保存，如何传递token了
- 解决:可以在登录接口的`test`中编写代码，将token做为一个变量保存到全局或者环境变量(需要先创建一个环境)中，在其他接口中直接使用此变量即可
- ![](saveVar.png)
- ![](useVar.png)

## 总结
> https://www.getpostman.com/docs/
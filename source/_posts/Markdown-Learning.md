---
title: Markdown-Learning
tags:
  - Markdown
categories:
  - 其他
date: 2017-03-23 11:21:16
---

# Markdown-Github Flavored Markdown常用操作

Markdown拥有多种语法风格
* 标准风格-不支持表格
* 扩展风格-支持表格
* github风格-Github Flavored Markdown它在标准风格上做了很多改进，如对表格的支持，针对不同编程语言实现代码高亮等

因为经常使用github，所以选择了Github Flavored Markdown风格。

Hexo搭建的博客也是使用github风格来解析markdown的。

win上支持Github Flavored Markdown风格的编辑器我常用的有:markdown pad2和typora。
typora最让我心动的是支持快捷键创建表格，非常的方便。


## 标题
``` 
# 一级标题
## 二级标题，二级标题自带下划线
### 三级标题
...
###### 六级标题
```

## 粗体斜体
```
**两个星号为粗体**
*一个星号为斜体*
**粗中带 _斜_**
*内部换行用<br>,第二行*
```

## 引用
```
> 这样引用
```

## 无序列表
```
- 无序
  - 我前面有2个空格，我能缩进
      - 无序
- 无序
- 无序
```

## 有序列表
```
1. 第一行
    1. 我前面有2个空格
2. 第二行
3. 第三行
```

## 任务列表
```
- [x] 我代表选中
- [ ] 我没选中
- [ ] 我没选中
```

## 图片与链接
```
[链接名](链接地址)
![图片alt](图片地址)
```

## 代码段
```
使用`包裹的区域会形成代码段,区段内的 &、< 和 > 都会被自动的转换成HTML实体，一般用在行内
```

## 代码块
```
使用三个`会产生格式化好的代码块,而 &、< 和 > 也一样会自动转成HTML实体，一般用于一大段代码
```

## 代码块高亮
```javascript
function show(){
    console.log("我是带语法高亮的代码块，在三个`后添加上语言类型即可高亮");
}
```

## 表格
```
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
```

Markdown中表格生成较为麻烦，建议使用编辑器快速生成,如typora中使用ctrl+t

## 分割线
```
    ***或--- 会产生分割线
```

还有很多东西因为没用到，所以不做介绍。
更多的可以参考这
> https://help.github.com/categories/writing-on-github/
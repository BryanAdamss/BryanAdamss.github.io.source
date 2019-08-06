---
title: Graphviz
tags:
  - Graphviz
  - 绘图
categories:
  - 其他
date: 2017-03-24 16:55:14
---

# Graphviz的安装以及基本使用

## 介绍
Graphviz可通过代码的方式生成图形

## 安装
win下，可在官网[http://http://www.graphviz.org/Download_windows.php](http://http://www.graphviz.org/Download_windows.php)下载，安装好后，手动将bin文件夹添加到环境变量即可。
cmd 下键入`dot -version`,能出现Graphviz相关信息，则表示安装成功

## 生成图片
```
dot 源文件 -T 图片格式 -o 输出文件
dot input.dot -T png -o output.png
```
可利用sublimeText的编译系统，实现图片实时预览
> 具体可参考这篇文章
[https://zhuanlan.zhihu.com/p/22820399](https://zhuanlan.zhihu.com/p/22820399)

新建`*.dot`文件，然后编写相应代码，再编译就能生成图

## 基本语法
```
图类型 图名{
    //其他
}
```

### 无向图
```dot
// 无向图用--表示节点之间的关系
graph graphname {
    a -- b--e; 
    b -- c;
    b -- d;
    d -- a;
}
```

### 有向图
```dot
// 有向图用a->b表示从a节点指向b节点
digraph graphname{
	a->b
	b->c
	a->c
}
```

### 定义一类节点
```dot
digraph graphname{
    T [label="Teacher",fontcolor="red"] //定义节点T，并给予属性
    P [label="Pupil"]  //定义节点P，并给予属性

    T->P [label="Instructions", fontcolor=darkgreen] //定义边T->P，并给予属性 
}
```

### 设置属性(样式)
```dot
graph G {
    // 设置当前图和子图的属性
    fontname="Microsoft JhengHei";
    fontsize=20;
    label="图";
    fontcolor=blue;
    //设置当前大括号范围内所有节点和边的属性，包含子图里面节点和边，类css中标签选择器
    node[fontname="Microsoft JhengHei",fontsize=16];
    edge[fontname="Microsoft JhengHei",fontsize=16];
    // 可针对某一类节点设置属性，类css中class选择器
    "黑海"[fontcolor="pink",style ="filled",fillcolor = "black"];

    "黑海" -- "亚速海";
    "黑海" -- "博斯普鲁斯海峡";
    "达达尼尔海峡" -- "爱琴海";

    // 子图，用subgraph声明，并图名字前缀必须是cluster_否则识别失败；子图和父图的类型必须一致，父图是无向则子图也必须是无向，不能是有向
    subgraph cluster_T {
    	// 设置子图的label属性，它的颜色继承父图的fontcolor=blue
        label="黑海海峡";
        "达达尼尔海峡" -- "马尔马拉海" -- "博斯普鲁斯海峡";
    }

    subgraph cluster_M {
        label="黑海海峡";
        // 一对多，空格分隔
        "中部地中海" -- {
            "爱琴海" "爱奥尼亚海" "西西里海峡"
        };
        // 一对多，并设置每对都有一个label标签说明，并把字体颜色设置为red，线条颜色设置为yellow
        "西部地中海" -- {
            "西西里海峡" "第勒尼安海" "利古里亚海" "伊比利海" "阿尔沃兰海"
        }[label="标签说明",fontcolor="red",color="yellow"];
        "爱奥尼亚海" -- "亚得里亚海";
        "阿尔沃兰海" -- "直布罗陀海峡";
    }
}
```
最终生成的
![](http://i.imgur.com/bx6evhH.png)

### 中文乱码
保证.dot文件是以UTF-8编码
通过设置fontname为中文字体来解决

### 参考链接
> https://zhuanlan.zhihu.com/p/21993254
> https://zhuanlan.zhihu.com/p/22820399
> http://blog.csdn.net/xiajian2010/article/details/23748557
> http://www.tuicool.com/articles/vy2Ajyu
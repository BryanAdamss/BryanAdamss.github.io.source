---
title: bfc
tags:
  - bfc
categories:
  - 前端
date: 2020-03-05 12:21:04
---

# BFC

## 定位方案(positioning scheme)

### 布局的核心

- html 中的元素是一个一个的盒子
- 布局的核心是如何将一系列将盒子放在`x-y-z坐标系`的合适位置上
- `x、y`的坐标其实是由`定位方案(positioning scheme)`来决定的；

### 定位方案是什么

- 定位方案是 css 布局的一个最高层抽象，确定了定位方案后，其可以进一步被一些特殊的`布局模式(layout modes)`修改，例如`display:table，display:inline-table`；
- 哪怕最新的布局模式`flexbox`、`grid`都属于某一定位方案(`normal flow`)中

### 定位方案的分类

- css2.1 中确定了三种定位方案
  - `normal flow`
    - 常说的文档流或普通流定位
    - 按照次序依次定位每个盒子
    - 其包含了以下`格式化上下文(formatting context)`
      - `block formatting contexts`
      - `inline formatting contexts`
      - `relative formatting contexts`
      - `flex formatting contexts` css3 新增
      - `grid formatting contexts` css3 新增
  - `floats`
    - 浮动定位
    - 将盒子从普通流中单独拎出来，将其放到外层盒子的某一边
    - 其与`normal flow`互动，很多网格框架都使用了它
  - `absolute positioning`
    - 绝对定位
    - 按照绝对位置来定位盒子，其位置根据盒子的包含元素所建立的绝对坐标系来计算，因此绝对定位元素有可能会覆盖其他元素
    - 主要用来处理`absolute`和`fixed`元素相对于`normal flow`的定位
- **所有的元素除非通过设置`float`和`position`属性从`normal flow`中移除，默认都属于`normal flow`。**

### 布局需要关心的点

- 其实布局需要关心两个点
  - 元素盒子的尺寸和对齐方式；
    - 这一般由`display、width、height、margin`等来控制
  - 特定父元素中的子元素(直接子元素)间如何相对彼此放置
    - 这一般由`formatting context`来控制

## 格式化上下文(formatting context)

### 定义

- 它是一种渲染规则，决定了其子元素将如何定位，以及和其他元素的关系和相互作用
  - 通俗理解是它是一种规则，用来确定页面如何布局(排版)

### 作用

- 控制特定父元素中的子元素(直接子元素)间如何相对彼此放置

### 创建者

- 父元素
  - 一个父元素的子元素的相对位置是通过父元素为子元素建立的`formatting context`控制的

### 作用对象

- **创建者**的**直接**子元素

### 常见 fc

- `block formatting contexts`
- `inline formatting contexts`
- `relative formatting contexts`
- `flex formatting contexts` css3 新增
- `grid formatting contexts` css3 新增

## BFC

### 规则定义

- 其是一个独立区域，内部的元素不会影响到`BFC`外部
  - BFC 就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素，反之亦然
- 处于同一个`BFC`的直接子盒子将依次自顶向下垂直排列
- 两个相邻的盒子之间的距离由由`margin`属性确定
- 相邻`block-level盒子`的`垂直的margin`可能发生重叠`margin collapsing(外边距重叠、边界折叠)`https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing
  - 外边距折叠只会发生在属于同一 BFC 的块级元素之间。
- 每个盒子的左外边缘触碰到`containing block`的左边缘，`rtl`时右边缘会相触
- `BFC`将包含**创建它的元素**(一般为父盒子)内部的所有内容
  - 创建 bfc 元素的高度将会计算内部所有内容，包括内部浮动的元素
- 浮动定位和清除浮动时只会应用于同一个 BFC 内的元素
  - 浮动不会影响其它 BFC 中元素的布局，而清除浮动只能清除同一 BFC 中在它前面的元素的浮动。

### 创建 BFC

- 下列方式将会创建一个新的 BFC
  - 根元素(<html>)
    - html 自带 BFC，所以 html 内部盒子默认都是垂直排列的
  - 浮动元素（元素的  float  不是  none）
  - 绝对定位元素（元素的  position  为  absolute  或  fixed）
  - 行内块元素（元素的  display  为  inline-block）
  - 表格单元格（元素的  display 为  table-cell，HTML 表格单元格默认为该值）
  - 表格标题（元素的  display  为  table-caption，HTML 表格标题默认为该值）
  - 匿名表格单元格元素（元素的  display 为  table、table-row、 table-row-group、table-header-group、table-footer-group（分别是 HTML table、row、tbody、thead、tfoot 的默认属性）或  inline-table）
  - overflow  值不为  visible  的块元素
  - display  值为  flow-root  的元素
  - contain  值为  layout、content 或  paint  的元素
  - 弹性元素（display 为  flex  或  inline-flex 元素的直接子元素）
  - 网格元素（display 为  grid  或  inline-grid  元素的直接子元素）
  - 多列容器（元素的  column-count  或  column-width  不为  auto，包括  column-count  为  1）
  - column-span  为  all  的元素始终会创建一个新的 BFC，即使该元素没有包裹在一个多列容器中（标准变更，Chrome bug）。

### BFC 的应用

#### 让浮动内容和周围的内容等高(类似 clear 清除浮动效果)

- 一般会使用清除浮动，这里也可以利用 bfc 特性来实现
- 原理：创建 BFC 的元素将包含其内部所有内容

```html
<style>
  .box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
  }

  .float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border: 1px solid black;
    padding: 10px;
  }
</style>

<div class="box">
  <div class="float">I am a floated box!</div>
  <p>I am content inside the container.</p>
</div>
```

- 可给`.box`元素添加`overflow:auto;`来创建一个`bfc`，进而利用创建`bfc`的元素必须包含其内部所有元素特性

#### 避免外边距重叠

- 创建新的 BFC 避免两个相邻  <div>  之间的   外边距重叠   问题
- 原理：外边距重叠只会发生在属于同一 BFC 的块级元素之间，通过创建新的 BFC 来避免重叠

```html
<style>
  .blue,
  .red-inner {
    height: 50px;
    margin: 10px 0;
  }

  .blue {
    background: blue;
  }

  .red-outer {
    overflow: hidden;
     /* 创建bfc，避免折叠*/
  background: red;
  }
</style>

<div class="blue"></div>
<div class="red-outer">
  <div class="red-inner">red inner</div>
</div>
```

#### display:flow-root 无副作用创建 BFC

- 使用`display:flow-root`无副作用创建 BFC

```html
<style>
  .box {
    background-color: rgb(224, 206, 247);
    border: 5px solid rebeccapurple;
    display: flow-root; /* 创建bfc */
  }

  .float {
    float: left;
    width: 200px;
    height: 150px;
    background-color: white;
    border: 1px solid black;
    padding: 10px;
  }
</style>

<div class="box">
  <div class="float">I am a floated box!</div>
  <p>I am content inside the container.</p>
</div>
```

## 总结

- 网页中每个元素都是一个盒子
- 布局就是将盒子放到合适的位置上
- 浏览器准备了几种大的定位方案(normal flow、float、positioning)用来确定盒子布局的大方法
- 确定了大的定位方案后，就需要考虑如何将盒子放到准确的位置上
- 放到准确的位置上需要考虑两点
- 1.盒子自身的尺寸，这一般由盒子类型(display)、width、height 来决定
- 2.盒子的位置(盒子同其他盒子的关系)，一般由 formatting context 决定
- 常见 fc 有 bfc、ifc、rfc、ffc、gfc
- 在不同场景下会创建不同的 fc 来确定其内部直接子盒子同其他直接子盒子的位置关系
- fc 一般由父元素创建，作用于直接子元素
- 最常见的是 bfc
- bfc 是一个独立布局区域，其内外部不会相互影响
- bfc 中的盒子，会垂直排列，间距由 margin 决定
- bfc 中块级盒子的垂直边距会发生重叠
- 外边距重叠只会出现同一个 bfc 中的块级盒子
- 所以创建一个新的 bfc 能避免外边距重叠
- 使用 clear 清除浮动也只会作用于同一个 bfc 中的元素
- 创建 bfc 的元素的会包含其内部所有元素，包括内部浮动元素
- 所以可以利用创建 bfc，来实现类似 clear 清除浮动效果(使内部有 float 盒子的元素高度包含 float 盒子)
- 创建 bfc 有很多方式，无副作用的是使用 display:flow-root

## 参考

- https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Visual_formatting_model
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flow_Layout/在Flow中和Flow之外
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing
- https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context
- http://book.mixu.net/css/index.html
- https://zhuanlan.zhihu.com/p/23829153
- https://zhuanlan.zhihu.com/p/23856688

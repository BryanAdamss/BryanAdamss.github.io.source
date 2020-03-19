---
title: stacking-context
tags:
  - z-index
  - stacking context
categories:
  - 前端
date: 2020-03-07 16:02:04
---

# stacking context

## z 轴

- 网页其实是一个三维的，拥有`x、y、z`三个轴
- 如下图所示；
- ![zb.png](zb.png)
- z 轴正方向正对着屏幕；
- 当 html 中的元素发生层叠时，就需要确定元素哪个在上(离用户近)，哪个在下(离用户远)；
- 而决定元素在 z 轴顺序的因素有
  - 元素的类型(float, block-level, inline-level, absolutely positioned)
  - 元素在当前`stacking context`中的`z-index`值

## stacking context

### 定义

- `stacking context`层叠上下文是一个抽象概念，借用`mdn`的定义

```
我们假定用户正面向（浏览器）视窗或网页，而 HTML 元素沿着其相对于用户的一条虚构的 z 轴排开，层叠上下文就是对这些 HTML 元素的一个三维构想。众 HTML 元素基于其元素属性按照优先级顺序占据这个空间。
```

- 我们不用关注他的定义，只需要知道它和元素在 z 轴的展示有一定关系就可以了
- 层叠上下文可以理解为层叠发生的环境，所有的层叠都是发生在层叠上下文环境中的
- 层叠上下文是可以嵌套的
- 层叠上下文是元素发生层叠的环境

### 生成

- 发生层叠，必须有层叠上下文
- 以下途径会生成层叠上下文
  - 文档根元素（<html>）
    - 根层叠上下文
  - position  值为  absolute（绝对定位）或   relative（相对定位），并且  z-index  值不为  auto  的元素；
    - 设置了 absolute 或者 relative，并且 z-index 不为 auto
  - position  值为  fixed（固定定位）或  sticky（粘滞定位）的元素；
  - flex (flexbox) 容器的子元素，且  z-index  值不为  auto
    - flex 容器中的 z-index 不为 auto 的 item 元素会创建一个新的层叠上下文
  - grid (grid) 容器的子元素，且  z-index  值不为  auto
    - grid 容器中的 z-index 不为 auto 的 item 元素会创建一个新的层叠上下文
  - opacity  属性值小于  1  的元素
  - mix-blend-mode  属性值不为  normal  的元素
  - transform、filter、perspective 、clip-path、mask、mask-image、mask-border 任意值不为 none 的元素都会创建一个新的层叠上下文
  - isolation  属性值为  isolate  的元素
  - webkit-overflow-scrolling  属性值为  touch  的元素
  - will-change  值设定了任一属性而该属性在 non-initial 值时会创建层叠上下文的元素
  - contain  属性值为  layout、paint  或包含它们其中之一的合成值（比如  contain: strict、contain: content）的元素
- 注意
  - <html>元素会自动创建一个根层叠上下文
      - 所以每个盒子都会属于一个层叠上下文，不存在某个元素找不到所属的层叠上下文
  - 层叠上下文是可以相互嵌套的
  - 每个层叠上下文都是完全独立于它的兄弟元素，处理层叠时只考虑其子元素
  - 当元素的后代元素发生层叠后，该元素将被作为整体在父级层叠上下文中按顺序进行层叠

## stacking level

- 层叠等级(stacking level)是一个概念，用来描述在同一个层叠上下文中元素在 z 轴的位置，可以这样说同一个层叠上下文中层叠等级越大，越靠近用户
- 层叠等级不等同于`z-index`，层叠等级是一个抽象概念，`z-index`在特殊情况下会影响元素的层叠等级
- 层叠等级描述的是元素在上下文中位置

## z-index

- z-index 规定了，发生层叠时，处于同一个层叠上下文的定位过的元素(position 不为 static)层叠规则，z-index 大的在上面
- 注意
  - 所有元素(无论是否设置了 position 属性)都有 z-index,默认 z-index 为 auto；但只有被定位过，z-index 才会生效
  - 当元素的 position 不为 static 时，代表元素被定位过，此时 z-index 才会发挥作用，才会对同一层叠上下文中的层叠顺序产生影响(z-index 大的定位过元素在上面，小的在下面)
  - 未定位过的元素即使设置了很大的 z-index，在层叠时也会被当作 auto 处理
  - 当 z-index 发挥作用时，其值为 auto 则在比较层叠顺序时可视为 0

## stacking order(重点)

- 层叠顺序是一种规则，规定了元素在 z 轴上按什么样的规则层叠(哪个在上，哪个在下)
- 在同一个层叠上下文中，按下面顺序，在 z 轴上由下到上渲染
  - 产生层叠上下文元素的 background、border
  - z-index 为负值的定位过盒子
  - 块级盒子
  - float 盒子
  - inline/inline-block 盒子
  - z-index 为 auto 或 0 的定位过盒子、其他不依赖 z-index 产生层叠上下文的盒子(例如 opacity<1 的盒子)
    - 当二者冲突时，遵循“后来居上”的原则，在 DOM 流中处于后面的元素会覆盖前面的元素
  - z-index 为正数值的定位过盒子
- 两大原则
  - 谁大谁上：当具有明显的层叠水平标示的时候，如识别的 z-indx 值，在同一个层叠上下文领域，层叠水平值大的那一个覆盖小的那一个。
  - 后来居上：当元素的层叠水平一致、层叠顺序相同的时候，在 DOM 流中处于后面的元素会覆盖前面的元素。
- ![order.png](order.png)

## 如何比较

- 给定任意两个元素，其显示区域会发生重叠，如何确定覆盖关系
- 如果两个元素处于相同的层叠上下文中，可直接参照`stacking order`图中的层叠顺序确定覆盖关系
- 若处于不同的层叠上下文中，则需要根据 DOM 树依次向上找到父元素，比较父元素的层叠顺序，当有共同的层叠上下文时，然后按照`stacking order`中的层叠顺序确定覆盖关系
- 举例，确定 child1、child2 的覆盖关系

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Document</title>
    <style>
      #box1  {
          background: #f00;
          width: 100px;
          height: 100px;
          position: relative;
          z-index: 1;
      }
      #box2  {
          background: #00f;
          width: 100px;
          height: 100px;
          position: relative;
          z-index: 1;
          margin-top: -50px;
          margin-left: 50px;
      }
      p  {
          border: 1px dashed #0f0;
          color: #fff;
      }
      #child1  {
          position: relative;
          z-index: 2;
      }
      #child2  {
          position: relative;
          z-index: 1;
          background-color: #000;
      }
    </style>
  </head>
  <body>
    <div id="box1">
      <p id="child1">测试文案 测试文案测试文案测试文案测试文案</p>
    </div>
    <div id="box2">
      <p id="child2">2222222</p>
    </div>
  </body>
</html>
```

- 单看 chid1、child2，前者 z-index 为 2，后者为 1，应该前者覆盖后者，但其实是后者覆盖了前者；
  - 因为 box1、box2 分别创建了一个层叠上下文，所以 child1、child2 处于不同的层叠上下文无法比较；
  - 所以向上找到其父元素，比较父元素的层叠关系，box1、box2 都处于 html 创建的层叠上下文中，可以比较；
  - 由于二者都是定位过的盒子，所以 z-index 生效，根据层叠顺序图，发现二者处于同一个层叠上下文且层叠顺序一样，所以适用后来居上的原则，在 DOM 树中靠后的 box2 覆盖在 box1 之上，进而 child2 覆盖在 child1 上
  - ![example1.png](example1.png)
- 再看，如果将 box2 的 z-index 去掉，child1、child2 的覆盖关系又如何
  - 先看结论
  - ![example2.png](example2.png)
  - 此时 child1 处于 box1 创建的层叠上下文中，而 child2 由于 box2 去掉了 z-index，box2 没有产生层叠上下文，所以 child2 处于 html 创建的根层叠上下文中，此时二者上下文不同，无法比较，必须找到拥有相同层叠上下文的祖先元素进行比较；box1、box2、child2 都处于根层叠上下文中，三者可以比较，由于三者都为定位过的盒子，直接比较 z-index，由于 child2、box1 的 z-index 相同，所以处于 DOM 树后面的 child2 覆盖在 box1 上了，box2 没有 z-index 属性默认为 auto ,在同一层叠上下文中 auto 可当做 0 对待，所以三者 z-index 顺序为 child2>box1>box2，child2 在最上面，box1 在中(因为 child1 处于 box1 的层叠上下文中，所以 child1 的层叠顺序跟 box1 一致)，box2 在最下面
- 总结
  - 无论需要比较的两个元素处于 DOM 树的什么位置，只要向上找到到共同的层叠上下文(一定能找到，因为最终都有一个共同的层叠上下文-根层叠上下文)，然后根据层叠顺序图就可以得出覆盖关系

## 工具

- 手动寻找层叠上下文比较麻烦，可以借助`z-context`工具来快速找到层叠上下文
- chrome 插件`z-context`，可以轻松查看元素是否生成了层叠上下文、生成原因、父层叠上下文是哪个元素创建的
- ![tool.png](tool.png)

## 参考

- https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/
- https://juejin.im/post/5b876f86518825431079ddd6
- http://book.mixu.net/css/3-additional.html
- https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/z-index

---
title: line-height
tags:
  - css
  - line-height
categories:
  - 前端
date: 2020-03-05 16:35:09
---

# line-height

## 英文字体四线

1. 基线:创建字体的基准之线,是小写字母 x 的下边缘;
2. 中线:小写字母 x 的上边缘；
3. 顶线:文字的最上边缘
4. 底线:文字的最下边缘

综合 1 和 2 可发现小写 x 对于英文字母的设计非常重要，所以设计字体一般会先设计出 x，再设计其他文字

## content area

**每一个**文字都有一个"内容区域 content area"的矩形盒子将文字包裹在其中;**"内容区域"的高度就是 font-size;**

## 盒子

1. 每一行文字都会产生一个行框(line boxes);
2. 每个行框(line boxes)中又包含一个个的行内框(inline boxes);行内框(inline boxes)又有显性行内框和匿名行内框之分，显性行内框一般由显性的行内标签 span、em 等包裹一个或多个文字后产生，没有行内标签包裹的一个或多个文字一般会产生匿名行内框
3. **在没有其他因素（padding）影响的时候，行内框(inline boxes)高度等于内容区域(content area)高度**
4. ![1.png](1.png)
   ![2.png](2.png)
   ![3.png](3.png)
   ![4.png](4.png)
   ![5.png](5.png)
   ![6.png](6.png)
   ![7.png](7.png)
   ![8.png](8.png)
   ![9.png](9.png)
   ![10.png](10.png)
   ![11.png](11.png)
   ![12.png](12.png)
   ![13.png](13.png)

## 高度

1. 行高(line-height):文本的高度即行内框(inline box)的高度,不是 line box 的高度,因为每个单独的 inline 元素都可以设置 line-height
2. 行(间)距:每行文本的间距;一般为文字"底线"到下一行文字的"顶线"之间的垂直距离或者为"顶线"到上一行"底线"间的垂直距离；行距=行高-文字内容区域的高度(一般为字体大小);**行距=line-height - font-size**
3. 半行(间)距:一般为文字"顶线"到行框(line boxes)  的最上部或"底线"到行框的最下部;**大小为行距的一半**,半行(间)距  =行距/2=(line-height - font-size )/2;**会添加到内容区域(content area)的上下部；**

## 行框(line boxes)特性

1. **行框总是以文字的水平中线进行上下对称分布的**(利用此特性可实现文本的垂直居中)
2. 行框高度是由**内部行高最大的 inline box 决定的**
3. ![line-boxes.png](line-boxes.png)、看 test1 的结果，此时 line boxes 的高度为 0，但是它是以文字的水平中垂线对称分布的。这一重要的特性可以用来实现文字或图片的垂直居中对齐。(通过 test1 也说明 line-height 可以小于 font-size)

## 行高(line-height)的取值特点

body 设置 line-height:120%、120px、1.2 区别(假设 font-size 为 16px)

1. 百分比  ：body 的行高通过计算得出应该是 16x120%=19.2px;子元素会继承 body 计算后的 line-height 数值,而不是直接继承声明值(120%)再和自身 font-size 做计算得出自己的 line-height
2. 长度值:body 自身的 line-height 为 120px,子元素也会继承这个值 120px
3. 值(缩放因子):body 本身计算后为 16x1.2=19.2;子元素不会继承计算后的值，而是继承这个声明值 1.2,然后再和自身的 font-size 做计算，得出自身的 line-height.所以此方法比较好.
4. ![l1.png](l1.png)
   ![l2.png](l2.png)
   ![l3.png](l3.png)
   ![l4.png](l4.png)
   ![l5.png](l5.png)
   ![l6.png](l6.png)
   ![l7.png](l7.png)
   ![l8.png](l8.png)
   ![l9.png](l9.png)
   ![l10.png](l10.png)
   ![l11.png](l11.png)
   ![l12.png](l12.png)
   ![l13.png](l13.png)

## 参考

- https://www.cnblogs.com/fengzheng126/archive/2012/05/18/2507632.html
- https://www.zhangxinxu.com/wordpress/2009/11/css%e8%a1%8c%e9%ab%98line-height%e7%9a%84%e4%b8%80%e4%ba%9b%e6%b7%b1%e5%85%a5%e7%90%86%e8%a7%a3%e5%8f%8a%e5%ba%94%e7%94%a8/

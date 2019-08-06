---
title: break-word、word-wrap、white-space
tags:
  - css
categories:
  - 前端
date: 2017-03-10 17:04:18
---

# white-space

white-space的定义是用来设置如何处理元素中的空白。这里的空白指的是空格、tab制表符。

默认情况下，html中连续出现的多个空格会被合并成一个空格，Tab也会被替换成一个空格。回车换行(br换行不在内)会被忽略并将其替换成一个空格。当在容器剩余空间不足以容纳一个单词时，浏览器会在单词**结束处**自动换行。(默认情况下，是无法在一个单词内进行自动换行的，只能在结尾处换行。)

|    值     | 是否合并空白符(空格、tab) | 是否忽略回车换行 | 是否允许自动换行 |
| :------: | :-------------: | :------: | :------: |
|  normal  |       合并        |    忽略    |    允许    |
|  nowrap  |       合并        |    忽略    |   不允许    |
|   pre    |       保留        |    保留    |   不允许    |
| pre-wrap |       保留        |    保留    |    允许    |
| pre-line |       合并        |    保留    |    允许    |

# word-wrap

word-wrap的定义是用来说明当一个不能被分开的字符串太长而不能填充其包裹盒时，为防止其溢出，浏览器是否允许这样的单词中断换行。既指明是否允许浏览器在单词**内**进行自动换行

|     值      |                  解释                   |
| :--------: | :-----------------------------------: |
|   normal   |               在单词结束处换行                |
| break-word | 如果行内没有多余空间容纳该单词到行尾，则会强制将单词截断，在单词内进行换行 |

# word-break

word-break指定了怎样在单词内断行

|     值     |                 解释                 |
| :-------: | :--------------------------------: |
|  normal   |               默认换行规则               |
| break-all | 对于non-CJK (中文/日文/韩文) 文本，可在任意字符间断行。 |
| keep-all  | CJK 文本不断行。 Non-CJK 文本表现同 `normal`。 |

**在white-space、word-wrap、word-break都为normal值时，既默认情况下时。一行的剩余空间不足以容纳一个单词时，浏览器会将这个单词挪到下一行显示。挪后如果这个单词比容器还长，则这个单词会直接溢出，因为默认情况下，浏览器是无法在单词内进行换行的。中文会一行空间不足以容纳一个字时，在字后进行换行**

![](http://images.cnblogs.com/cnblogs_com/2050/201208/201208101725521060.png)

此时如果设置word-wrap:break-word，则会将这个长单词进行截断，从截断处进行换行。



![](http://images.cnblogs.com/cnblogs_com/2050/201208/201208101725587335.png)

可以发现第一行仍然有一点空间没有利用，此时就需要用到word-break:break-all;

可以说word-break:break-all是word-wrap:break-word的升级版本，它不会在剩余空间不够的时候将长单词挪到下一行，它将单词放在原位，并在容器边界处直接将这个长单词进行截断，然后换行。

![](http://images.cnblogs.com/cnblogs_com/2050/201208/201208101726046184.png)

将这三个属性，组合使用会怎么样

当设置了white-space:nowrap;时，word-wrap:break-word;和word-break:break-all;都将失效。文本将会强制在一行内显示

当同时设置word-wrap:break-word;和word-break:break-all时，word-break:break-all的效果会生效

# word-spacing

规定英文**单词**之间的间距

# letter-spacing

规定英文**字符**之间的间距

总结:

默认情况下，当一行的剩余空间不足以容纳某一单词时，浏览器会将此单词整体挪到下一行显示。此时，若这个单词超长(长度超出容器的宽度)，则此单词会直接溢出(此时上一行会留下一段空白)。

word-wrap:指明是否允许在长单词中换行，当设置其属性为break-word，则会把超过容器长度的单词进行截断，并换行(上一行留下的空白并不会被清除)

word-break:当其设置break-all时，它是word-wrap:break-word的升级版，它能解决上一行留白问题。它会让单词先在当前行显示，当单词某个字符到达容器边界时，会直接在此字符出进行截断，并换行。这样就最大限度的利用了空间。

white-space:指定处理空白符的方式，比较有用的属性为nowrap，设置文本不换行。一般配合其它css实现文本过长省略号

word-spacing:规定英文单词之间的间距

letter-spacing:规定英文单词字符之间的间距

>   图片引用在自http://www.cnblogs.com/2050/archive/2012/08/10/2632256.html
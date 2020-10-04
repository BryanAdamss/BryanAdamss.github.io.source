---
title: flex-space-algorithm
tags:
  - flex
  - 布局
categories:
  - 前端
date: 2020-03-19 22:36:58
---

# flex-basis、flex-grow、flex-shrink

- `flex-basis、flex-grow、flex-shrink`这三个属性决定了`flex item`的展示尺寸
- 要想了解三者是如何决定`flex item`的尺寸前，先要了解一下`max-content`、`min-content`、`正负向自由空间`概念
- 注意以下的讨论都是在主轴为横轴基础上讨论的

## min-content、max-content

- 这两个都是用来设置`width`的
- 通过下面的例子，我们就知道他们是什么含义

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .wp {
        width: 400px;
        border: 1px solid red;
      }
      .box {
        border: 1px dotted blue;
      }
      .box1 {
        width: min-content;
      }
      .box2 {
        width: max-content;
      }
    </style>
  </head>
  <body>
    <div class="wp">
      <p class="box box1">
        Lorem ipsum dolor sit amet consectetur, adipisicing elit. Officia
        voluptatibus quos nihil dolores
      </p>
      <p class="box box2">
        Lorem ipsum dolor sit amet consectetur, adipisicing elit. Officia
        voluptatibus quos nihil dolores
      </p>
    </div>
  </body>
</html>
```

- mix-max-contet.png
  - ![mix-max-content.png](mix-max-content.png)
- 可以看到`box1`的`width`设置了`min-content`后，它抓住了一切机会来换行，直到出现一个最大的无法换行的单词才停止，就如截图中的`voluptatibus`单词
  - 也就是说`min-content`的宽度是由`最长不可换行单词`决定的
- 而设置了`max-contet`的`box2`却有截然相反的表现，它尽可能的不换行，就跟我们设置了`white-space:no-wrap;`的表现很类似，即使外层`.wp`设置了固定尺寸，它也会冲破限制，溢出了
  - `max-content`确定的宽度是`内容正常展示时`的`最大宽度`
- `min-content`、`max-content`在`flex`布局中有关键作用，下面会做讲解

## 正负向自由空间

### 正向自由空间 positive free space

- 正向自由空间描述的是`flex`布局时，`flex container`剩余的空间
- positive-free-space.png
  - ![positive-free-space.png](positive-free-space.png)
- 在`flex`布局中，当`flex container`宽为`500px`，内部有三个`100px`初始宽度的`flex item`，那么剩下的`200px`就是正向自由空间，这`200px`的宽度后续将交给`flex-grow`按照一定的策略来分配

### 负向自由空间 negative free space

- 负向自由空间其实是一个和正向类似的概念
- 它描述的是`所有flex item宽度和超出flex container`的部分
- negative-free-space.png
  - ![negative-free-space.png](negative-free-space.png)
- `flex container`仍然是`500px`，但三个`flex item`初始宽度变为`200px`了，他们宽度总和超出`flex container 100px`，这个`100px`就为负向自由空间，这`100px`的宽度后续将交给`flex-shrink`按照一定策略来缩减(消化)，让其三者宽度总和不超过`500px`

### 注意

- 正负向自由空间都是`flex-grow`、`flex-shrink`参与空间分配前的概念，当这两个属性参与空间分配后，其实这个空间就可以视为不存在了

## flex-basis

- `flex-basis`是`flex`布局的基础
- `flex-basis`决定了`flex item`在主轴上的`初始尺寸`
  - 初始尺寸用来计算正、负向空间
  - 正、负空间决定了`flex-grow`和`flex-shrink`哪个生效

### 取值

- `auto`
  - **默认值**
  - 如果设置了`width`，则使用`width`值做为初始尺寸，否则使用`max-content(最大内容宽度)`做为初始尺寸
- `content`
  - 不管有没有设置`width`，都使用我的`max-content(最大内容宽度)`做为初始尺寸
  - 存在兼容问题
  - flex-basis-content.png
    - ![flex-basis-content.png](flex-basis-content.png)
- `非0数值`
  - `10px`
  - `10%`
  - `10em`
  - 将对应数值的计算值做初始尺寸
- `0`
  - 初始尺寸为 0，代表当前`flex item`不参与正、负空间计算

### 例子

- 由于`content`存在兼容问题，所以没举`content`的例子

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .wp {
        width: 400px;
        border: 1px solid red;
        display: flex;
        flex-direction: row;
      }
      .box {
        border: 1px dotted blue;
        /* 为了保持一致，让flex item 尺寸不放大也不缩小，就用初始尺寸 */
        flex-grow: 0;
        flex-shrink: 0;
      }

      .box+.box{
        margin-left:10px;
      }

      .box:nth-child(1) {
        /* 未设置width，所以使用max-content */
        flex-basis: auto;
      }

      .box:nth-child(2) {
        /* auto，并width有值，则取width对应值 */
        flex-basis: auto;
        width: 40px;
      }

      .box:nth-child(3) {
        /* 20px  */
        flex-basis: 20px;
      }

      .box:nth-child(4) {
        /* 30% */
        flex-basis: 30%;
      }

      .box:nth-child(5) {
        /* 0 */
        /* 最后宽度为min-content */
        flex-basis: 0;
      }

    </style>
  </head>
  <body>
    <div class="wp">
      <p class="box">1 123</p>
      <p class="box">22</p>
      <p class="box">333 3</p>
      <p class="box">4444</p>
      <p class="box">55555 11</p>
    </div>
  </body>
</html>
```

- 第一个盒子
  - example-1.png
    - ![example-1.png](example-1.png)
  - `flex-basis`为`auto`，又没设置`width`,所以最终尺寸为`max-content`
- 第二个盒子
  - example-2.png
    - ![example-2.png](example-2.png)
  - 设置`auto`又设置了`width`，所以取`width`宽度`40px`
    - 注意这里虽然`min-content`>`width`，但是最终尺寸仍然是`width:40px`(导致产生溢出)
    - 所以当`flex-basis`指定`auto`同时又设置`width`时，最终尺寸将不受`min-content`影响
- 第三个盒子
  - example-3.png
    - ![example-3.png](example-3.png)
  - 虽然设置了`flex-basis:20px;`，但由于`min-content`为`30px`，所以最终的大小为`30px`
- 第四个盒子
  - example-4.png
    - ![example-4.png](example-4.png)
  - `30%`为`120px`，而且`min-content`<`120px`，所以最终尺寸为`120px`
- 第五个盒子
  - example-5.png
    - ![example-5.png](example-5.png)
  - `flex-basis`为`0`，不参与空间计算，所以最终尺寸为`min-content`

### width、flex-basis、min-width、max-width 关系

- 记住公式
- 优先级`flex-basis`>`width`>`min|max-content`
  - 最终展示的大小会受`max|min-width`限制
  - 同时设置上述属性并且`flex-basis`不为`auto`时
    - `flex-basis`会覆盖(忽略)`width`值进行布局，`flex-item`最终展现的尺寸会受`min|max-width`限制
- 具体可看下面例子

```

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .wp {
        border: 1px solid red;
        display: flex;
        flex-direction: row;
      }
      .box {
        border: 1px dotted blue;
        flex-grow: 0;
        flex-shrink: 0;
      }
      .box + .box {
        margin-left: 10px;
      }
      .box:nth-child(1) {
        flex-basis: auto;
        /* flex-basis为auto，所以取width为初始尺寸 */
        width: 200px;
        /* 最终展示受max-width限制 */
        max-width: 100px;
      }
      .box:nth-child(2) {
        flex-basis: 200px;
        /* flex-basis非auto，所以width被忽略 */
        width: 400px;
      }
      .box:nth-child(3) {
        flex-basis: 200px;
        /* flex-basis非auto，所以width被忽略 */
        width: 400px;
        /* 最终展示受max-width限制 */
        max-width: 100px;
      }
    </style>
  </head>
  <body>
    <div class="wp">
      <p class="box">1 123 zcjviodf ajdiof jaoisdf asdjifo</p>
      <p class="box">1 123 zcjviodf ajdiof jaoisdf asdjifo</p>
      <p class="box">1 123 zcjviodf ajdiof jaoisdf asdjifo</p>
    </div>
  </body>
</html>
```

- 第一个盒子
  - space-1.png
    - ![space-1.png](space-1.png)
  - `flex-basis`为`auto`，所以`width:200px`生效，但最终的展示大小受`max-width:100px`限制，所以最终`100px`
- 第二个盒子
  - space-2.png
    - ![space-2.png](space-2.png)
  - `flex-basis`不为`auto`，所以`width`直接被忽略，所以最终尺寸为`flex-basis`指定的`200px`
- 第三个盒子
  - space-3.png
    - ![space-3.png](space-3.png)
  - `flex-basis`不为`auto`，所以`width`被忽略，展示尺寸为`flex-basis`指定的`200px`，最终的尺寸受`max-width:100px`限制，所以最后为`100px`

### flex-basis 和 box-sizing

- 注意`flex-basis`会受`box-sizing`影响
  - `flex-basis`指定的尺寸，其实是`box-sizing`设置的盒子尺寸
- 例子

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .wp {
        display: flex;
      }
      .box {
        flex-basis: 100px;
        border: 10px solid red;
        padding: 20px;
      }
      .box + .box {
        margin-left: 10px;
      }
      .box:nth-child(1) {
        box-sizing: border-box;
        /* flex-basis指定的是border-box的尺寸 */
      }
      .box:nth-child(2) {
        /* flex-basis指定的是content-box的尺寸 */
        box-sizing: content-box;
      }
    </style>
  </head>
  <body>
    <div class="wp">
      <div class="box">1</div>
      <div class="box">2</div>
    </div>
  </body>
</html>
```

- 可以看到同样的`padding`、`border`最后得出的尺寸不一样
- 第一个盒子
  - `flex-basis`指定的是`border-box`的尺寸
  - box-sizing-1.png
    - ![box-sizing-1.png](box-sizing-1.png)
- 第二个盒子
  - `flex-basis`指定的是`content-box`的尺寸
  - box-sizing-2.png
    - ![box-sizing-2.png](box-sizing-2.png)

### 小结

- `flex-basis`决定了`flex-item`的初始尺寸
- `flex-basis`设置的是`box-sizing`指定的盒子尺寸
- 而初始尺寸又决定了正负向空间，进而影响`flex-grow`、`flex-shrink`的表现
- 当`flex-basis`为`auto`时，取`width`值为初始尺寸
- 否则用`flex-basis`指定的值做为初始尺寸
- 最终展示的尺寸会受`min|max-width`限制

## flex-grow

- 正向空间的分配
  - 存在正向空间时，则需要将空间按策略分配给每个`flex-item`
  - 也就是按照一个放大策略将剩余空间分配给每个`flex-item`
  - 决定如何放大的关键属性就是`flex-grow`
- 默认值为`0`，即不放大

### 放大算法

- `flex-grow`只能取正值
- 它代表的是一个拉伸(放大)因子
  - 可以理解为一个比例(权重)，代表有多少正向空间的分配权利
- 算法公式
  - `最终尺寸=初始尺寸 + 分配比例*正向空间大小`
  - `分配比例` = `当前flex-item的拉伸因子`/`所有flex-item的拉伸因子之和`
    - 注意
      - 所有`flex-item`的拉伸因子之和 **最小值为 1**，即**如果拉伸因子之和小于 1，也会取 1 参与计算**
      - 这会导致出现正向空间没有被全部分配的场景，具体见第二个例子
  - 比如正向空间为 `x`，三个元素的 `flex-grow` 分别为 `a，b，c`。设 `sum 为 a + b + c`。那么三个元素将得到的正向空间分别是  `x * a / sum`, `x * b / sum`, `x * c / sum`，最终尺寸为初始尺寸加上+获得的正向空间

#### 例子 1，拉伸因子之和>=1

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .c-Wp {
        width: 500px;
        border: 1px solid red;
        display: flex;
      }
      .c-Box {
        outline: 1px dotted blue;
        /* 不缩放 */
        flex-shrink: 0;
      }
      .c-Box:nth-child(1) {
        /* 正向空间我分配 1/6 */
        flex-grow: 1;
        flex-basis: 100px;
      }
      .c-Box:nth-child(2) {
        /* 正向空间我分配 2/6 */
        flex-grow: 2;
        flex-basis: 150px;
      }
      .c-Box:nth-child(3) {
        /* 正向空间我分配 3/6 */
        flex-grow: 3;
        flex-basis: 100px;
      }
    </style>
  </head>
  <body>
    <div class="c-Wp">
      <div class="c-Box">one jiojio</div>
      <div class="c-Box">two jlkzjclkvz</div>
      <div class="c-Box">three cjioajdisof</div>
    </div>
  </body>
</html>


```

- 可以看到`flex container`尺寸为`500px`
- 三个`flex item`的初始尺寸分别为`100px`、`150px`、`100px`，所以剩余`500-350=150`的正向空间
- 三个`flex item`分别设置了`1、2、3`的拉伸因子(权重)
- 我们记`sum=1 + 2 + 3 = 6`为权重总和，所以每个`flex-item`的分配比例也就可以计算出来
- 第一个盒子
  - `1/sum`即`1/6`，即`150 * 1/6 = 25`
  - 所以最终尺寸为`100(初始尺寸) + 25(正向空间)=125`
  - grow-over-1.png
    - ![grow-over-1.png](grow-over-1.png)
- 第二个盒子
  - `2/sum`即`2/6`，即`150 * 1/6 = 50`
  - 所以最终尺寸为`150(初始尺寸) + 50(正向空间)=200`
  - grow-over-2.png
    - ![grow-over-2.png](grow-over-2.png)
- 第一个盒子
  - `3/sum`即`3/6`，即`150 * 1/6 = 75`
  - 所以最终尺寸为`100(初始尺寸) + 75(正向空间)=175`
  - grow-over-3.png
    - ![grow-over-3.png](grow-over-3.png)

#### 例子 2，拉伸因子之和<1

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .c-Wp {
        width: 500px;
        border: 1px solid red;
        display: flex;
      }
      .c-Box {
        outline: 1px dotted blue;
        /* 不缩放 */
        flex-shrink: 0;
      }
      .c-Box:nth-child(1) {
        /* 正向空间我分配 0.1/1 */
        flex-grow: 0.1;
        flex-basis: 100px;
      }
      .c-Box:nth-child(2) {
        /* 正向空间我分配 0.2/1 */
        flex-grow: 0.2;
        flex-basis: 150px;
      }
      .c-Box:nth-child(3) {
        /* 正向空间我分配 0.3/1 */
        flex-grow: 0.3;
        flex-basis: 100px;
      }
    </style>
  </head>
  <body>
    <div class="c-Wp">
      <div class="c-Box">one jiojio</div>
      <div class="c-Box">two jlkzjclkvz</div>
      <div class="c-Box">three cjioajdisof</div>
    </div>
  </body>
</html>
```

- 可以看到`sum= 0.1 + 0.2 + 0.3 = 0.6 < 1`，所以最终取`1`参与计算
- 第一个盒子
  - 最终尺寸:`100(初始尺寸) +0.1/1 * 150 = 115`
  - grow-less-1.png
    - ![grow-less-1.png](grow-less-1.png)
- 第二个盒子
  - 最终尺寸:`150(初始尺寸) +0.2/1 * 150 = 180`
  - grow-less-2.png
    - ![grow-less-2.png](grow-less-2.png)
- 第三个盒子
  - 最终尺寸:`100(初始尺寸) +0.3/1 * 150 = 145`
  - grow-less-3.png
    - ![grow-less-3.png](grow-less-3.png)
- 可以看到最后还有`500 -115 - 180 - 145 = 60`的尺寸没有分配

#### 等分布局

- 如何实现自适应等分布局?
  - 所有`flex item`的`flex-basis设置为0`和`flex-grow设置为相同值(总和>1)`即可
- 原理
  - `flex-basis:0`代表初始尺寸为 0，当所有`flex item`的初始尺寸都为 0，那计算出来的正向空间就为`flex container`的尺寸
  - `flex-grow`设置为相同值并且总和>1，代表平均分配所有正向空间，由于初始尺寸为 0，所有`flex item`得到的正向空间又相同，所以就实现了等分布局
- 例子

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .c-Wp {
        width: 500px;
        border: 1px solid red;
        display: flex;
      }
      .c-Box {
        outline: 1px dotted blue;
        /* 不缩放 */
        flex-shrink: 0;

        flex-basis: 0;
        flex-grow: 1;
      }
    </style>
  </head>
  <body>
    <div class="c-Wp">
      <div class="c-Box">one jiojio</div>
      <div class="c-Box">two jlkzjclkvz</div>
      <div class="c-Box">three cjioajdisof</div>
    </div>
  </body>
</html>
```

- average.png
  - ![average.png](average.png)

## flex-shrink

- 负向空间的分配
  - 存在负向空间时，需要按照缩小算法收缩每个`flex-item`
- 默认值为`0`，即不缩小

## 缩小算法

- 和放大算法不同，缩小算法在计算每个`flex item`的收缩大小时，不仅需要考虑收缩因子(`flex-shrink指定的值`)，还需要考虑`item的初始尺寸`
- 具体算法
  - 总权重 TW = 每个(`item`的`flex-shrink` \* `item`的初始尺寸)之和
  - 当前`item`需要缩小的尺寸= 负向空间 _ ((当前 item 的`flex-shrink` _ 初始尺寸) / 总权重 TW)
  - 最终尺寸 = 当前`item初始尺寸` + 需要缩小尺寸 (因为负向空间尺寸为负值，所以此处为加号)

## 例子 1，缩小因子之和>=1

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .c-Wp {
        width: 500px;
        border: 1px solid red;
        display: flex;
      }
      .c-Box {
        outline: 1px dotted blue;
        flex-grow: 0;
      }
      .c-Box:nth-child(1) {
        flex-shrink: 1;
        flex-basis: 150px;
      }
      .c-Box:nth-child(2) {
        flex-shrink: 2;
        flex-basis: 200px;
      }
      .c-Box:nth-child(3) {
        flex-shrink: 3;
        flex-basis: 300px;
      }
    </style>
  </head>
  <body>
    <div class="c-Wp">
      <div class="c-Box">1</div>
      <div class="c-Box">2</div>
      <div class="c-Box">3</div>
    </div>
  </body>
</html>
```

- 负向空间 = `500 - 150 - 200 - 300 = -150`
- 计算总权重 TW = `1 * 150 + 2 * 200 + 3 * 300 = 1450`
- 第一个盒子
  - 最终尺寸= `150 + (-150负向空间 * ((1*150) / 1450)) = 134.5`
  - shrink-over-1.png
    - ![shrink-over-1.png](shrink-over-1.png)
- 第二个盒子
  - 最终尺寸= `200 + (-150负向空间 * ((2*200) / 1450)) = 158.6`
  - shrink-over-2.png
    - ![shrink-over-2.png](shrink-over-2.png)
- 第三个盒子
  - 最终尺寸= `300 + (-150负向空间 * ((3*300) / 1450)) = 206.9`
  - shrink-over-3.png
    - ![shrink-over-3.png](shrink-over-3.png)

### 例子 2，缩小因子之和<1

- 当缩放因子之和<1 时，只有缩放因子之和\* 负向空间的空间会参与收缩
  - 例如 , 缩放因子分别为`0.1、0.2、0.3`，总和<1，负向空间的尺寸为 150，那么只有 90 的空间会被缩小
- 例子(将例子 1 的缩放因子设置为缩小十倍)

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      * {
        box-sizing: border-box;
      }
      .c-Wp {
        width: 500px;
        border: 1px solid red;
        display: flex;
      }
      .c-Box {
        outline: 1px dotted blue;
        flex-grow: 0;
      }
      .c-Box:nth-child(1) {
        flex-shrink: 0.1;
        flex-basis: 150px;
      }
      .c-Box:nth-child(2) {
        flex-shrink: 0.2;
        flex-basis: 200px;
      }
      .c-Box:nth-child(3) {
        flex-shrink: 0.3;
        flex-basis: 300px;
      }
    </style>
  </head>
  <body>
    <div class="c-Wp">
      <div class="c-Box">1</div>
      <div class="c-Box">2</div>
      <div class="c-Box">3</div>
    </div>
  </body>
</html>

```

- 负向空间 = `500 - 150 - 200 - 300 = -150`
- 计算总权重 TW = `0.1* 150 + 0.2 * 200 + 0.3 * 300 = 145`
- 第一个盒子
  - 最终尺寸= `150 + (-150负向空间 * (0.1 + 0.2 + 0.3) * ((0.1*150) / 145)) = 140.69`
  - shrink-less-1.png
    - ![shrink-less-1.png](shrink-less-1.png)
- 第二个盒子
  - 最终尺寸= `200 + (-150负向空间 * (0.1 + 0.2 + 0.3) * ((0.2*200) / 145)) = 175.17`
  - shrink-less-2.png
    - ![shrink-less-2.png](shrink-less-2.png)
- 第三个盒子
  - 最终尺寸= `300 + (-150负向空间 * (0.1 + 0.2 + 0.3) * ((0.3*300) / 145)) = 244.14`
  - shrink-less-3.png
    - ![shrink-less-3.png](shrink-less-3.png)
- 可以看到还有`60px`溢出了(没有被收缩)

### 小结

- 放大和缩小的算法不太一样，缩小算法需要考虑初始尺寸
- 总结公式如下
- sumary.png
  - ![sumary.png](sumary.png)

## 简写属性

### flex

- `flex`是上述三者的简写
  - 其默认值为`flex: 0 1 auto;`
  - 三个值依次代表`flex-grow`、`flex-shrink`、`flex-basis`
  - 0 代表不放大
  - 1 代表不缩小
  - auto 代表使用 width 值做为初始尺寸
- 由于默认值为`0 1 auto`，所以在某些特定场景，会出现问题，所以建议在任何场景都显示指明`flex`的三个属性值

### 常用缩写

- `flex:1`
  - 等同`flex: 1 1 0;`
- `flex:0`
  - 等同`flex: 0 1 0;`
- `flex:auto`
  - 等同`flex: 1 1 auto;`

## flex 应用

### 移动端常见，头尾固定，中间局部滚动布局

```
<template>
  <div class="c-BaseLayoutVertical">
    <div
      v-if="this.$slots.header"
      class="c-BaseLayoutVertical-hd"
    >
      <slot name="header" />
    </div>
    <div class="c-BaseLayoutVertical-bd">
      <slot />
    </div>
    <div
      v-if="this.$slots.footer"
      class="c-BaseLayoutVertical-ft"
    >
      <slot name="footer" />
    </div>
  </div>
</template>
<script>
/**
 * * 常见垂直上中下三行布局
 */
export default {
  name: 'BaseLayoutVertical'
}
</script>
<style scoped>
/**
 * ! c-BaseLayoutVertical节点的父节点需要有高度
 */
.c-BaseLayoutVertical {
  display: flex;
  flex-direction: column;
  height: 100%;
}
.c-BaseLayoutVertical-hd {
  flex: 0 0 auto;
}
.c-BaseLayoutVertical-bd {
  flex: 1 1 auto;
  height: 100%;
  overflow: auto;
  -webkit-overflow-scrolling: touch;
  /* 优化滚动性能 */
  will-change: scroll-position;
  position: relative;
  z-index: 1;
}
.c-BaseLayoutVertical-ft {
  flex: 0 0 auto;
}
</style>
```

- apply.png
  - ![apply.png](apply.png)
- 同理水平方向，我们也可以实现常见的`双飞翼、圣杯布局`
- 具体可参考下面连接
  - [https://github.com/BryanAdamss/vue-awesome-template/blob/master/src/views/LayoutTest/LayoutTest.vue](https://github.com/BryanAdamss/vue-awesome-template/blob/master/src/views/LayoutTest/LayoutTest.vue)

## 总结

### 确定 flex item 尺寸的步骤

- 首先确定初始尺寸
  - `flex-basis、width`决定
    - `flex-basis为auto`时，取`width值`，若未显示指定`width`值，则取`max-content`
    - 非`auto`时，直接取对应值的计算值为初始尺寸
- 确定空间类型
  - 初始尺寸之和 >`flex container`尺寸时存在负向空间
    - 负向空间大小 = 初始尺寸之和 - `flex container`尺寸
  - 初始尺寸之和 < `flex container`尺寸时存在正向空间
    - 正向空间大小 = `flex container`尺寸 - 初始尺寸之和
- 根据空间类型决定使用`flex-grow`、`flex-shrink`策略来分配空间，得到展示大小
  - 正向空间直接按拉伸因子(`flex-grow申明的`)所占比例来分配
  - 负向空间收缩不光要考虑收缩因子，还需要考虑到初始尺寸
- 展示大小受`min|max-width|height`的限制

## 参考

- https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-basis
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Controlling_Ratios_of_Flex_Items_Along_the_Main_Ax
- https://www.zhangxinxu.com/wordpress/2019/12/css-flex-basis/
- https://www.zhangxinxu.com/wordpress/2019/12/css-flex-deep/
- https://github.com/xieranmaya/blog/issues/9
- https://www.cnblogs.com/yunqishequ/p/10006872.html

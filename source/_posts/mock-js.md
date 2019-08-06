---
title: mock-js
tags:
  - mock
  - 模拟数据
categories:
  - 前端
date: 2017-03-24 10:52:19
---

# Mock.js的基本用法

## 如何使用

```javascript

<script type="text/javascript" src="js/jquery-2.2.4.min.js"></script>
<!-- 引入Mock.js -->
<script type="text/javascript" src="js/mock-min.js"></script>

<script>
    // 根据数据模板生成模拟数据。
    Mock.mock('http://test.cn', {
        "userName": "@cname",
        "sex|1": ["男", "女"],
        "avator": Mock.Random.image('100x100', '#894FC4', '#FFF', 'png', '头像')
    });
    // 模拟请求
    $.ajax({
        url: 'http://test.cn',
        dataType: 'json'
    }).done(function(data, status, xhr) {
        // 请求成功，do something
        console.log(data);
    });
</script>

```

## 语法规范

1. **数据模板**定义规范（Data Template Definition，DTD）
2. **数据占位符**定义规范（Data Placeholder Definition，DPD） 

### 数据模板定义语法

```
'name|rule':value
name:属性名
rule:生成规则
vale:属性值
```

#### 注意
- 属性名(name)和生成规则(value)之间要用`|`分隔
- 生成规则(rule)不是必须的
- **最终生成值的类型和初始值由属性值(value)确定**
- 属性值(value)中可以包含数据占位符(`@占位符`)
- 生成规则有7种格式
    - 'name|min-max': value
    - 'name|count': value
    - 'name|min-max.dmin-dmax': value
    - 'name|min-max.dcount': value
    - 'name|count.dmin-dmax': value
    - 'name|count.dcount': value
    - 'name|+step': value
- **生成规则的具体含义还要配合属性值(value)的类型来确定**

#### 规则
1. 属性值(value)是String
    1. 'name |min-max':stringValue
        - 通过重复stringValue，生成一个字符串。重复次数在[min,max]区间取值(次数大于等于min，小于等于max)
    2. 'name |count':stringValue
        - 通过重复stringValue字符串count次，生成一个字符串。
2. 属性值(value)是Number
    1. 'name|+1': numberValue
        - 属性值自动加 1，初始值为numberValue。
    2. 'name|min-max':numberValue
        - 生成一个[min,max]之间的整数，此时属性值numberValue只是用来确定类型。
    3. 'name|min-max.dmin-dmax': numberValue
        - 生成一个整数部分在[min,max]间取值,小数保留的位数在[dmin,dmax]间取值的浮点数。同理此时属性值(value)也只是用来确定最终返回的数据的类型。
3. 属性值是Boolean
    1. 'name|1': booleanValue
        - 随机生成一个布尔值，值为 booleanValue 的概率是 1/2，值为 !booleanValue 的概率同样是 1/2。
    2. 'name|min-max': booleanValue
        - 随机生成一个布尔值，值为 booleanValue 的概率是 min / (min + max)，值为 !booleanValue 的概率是 max / (min + max)。此处属性值必须是Boolean类型，若为Number,则意义不同，见2
        - "test|1-2":true
            - 生成一个布尔值，为true的概率为1/3，为false概率为2/3
4. 属性值是对象 Object
    1. 'name|count': object
        - 返回的对象只包含从属性值object中随机选取的count个属性。
    2. 'name|min-max': object
        - 返回的对象只包含从属性值object中随机选取的min到max个属性。
5. 属性值是数组 Array
    1. 'name|1': array
        - 从属性值 array 中随机选取 1 个元素，作为最终值。
    2. 'name|+1': array
        - 从属性值 array 中顺序选取 1 个元素，作为最终值。
    3. 'name|min-max': array
        - 通过重复属性值array的值生成一个新数组，重复次数大于等于 min，小于等于 max。
    4. 'name|count': array
        - 通过重复属性值array的值生成一个新数组，重复次数为 count。 
6. 属性值是函数 Function
    1. 'name': function
        - 执行函数 function，取其返回值作为最终的属性值，函数的上下文为属性 'name' 所在的对象。
7. 属性值是正则表达式 RegExp
    1. 'name': regexp
        - 根据正则表达式 regexp 反向生成可以匹配它的字符串。用于生成自定义格式的字符串。

### 数据占位符定义语法

Mock中的占位符和Sass中的placeholder很像。可以直接用在数据模板的属性值中。
Mock中提供了很多预先定义的占位符，当然你也可以自定义占位符。

基本调用格式
```
@占位符
@占位符(参数 [, 参数])

或者Mock.Random.占位符(参数 [, 参数])
```

#### 注意：
- 用@来标识其后的字符串是占位符。
- 占位符引用的是Mock.Random中的方法。
- 通过 Mock.Random.extend() 来扩展自定义占位符。
- 占位符也可以引用数据模板中的属性。
- 占位符会优先引用数据模板中的属性。
- 占位符支持相对路径和绝对路径。

#### 预定义的占位符
- Basic         提供一些基础占位符，如布尔值、整数、自然数、字符串等
- Date          提供日期相关的占位符
- Image         提供图片相关的占位符
- Color         提供色值相关的占位符
- Text          提供文本相关的占位符
- Name          提供英文、中文名称相关的占位符
- Web           提供了url、IP、protocol等相关的占位符
- Address       提供省份、城市等地域相关信息的占位符
- Helper        提供一些常用工具如字母转换大小写等相关占位符
- Miscellaneous 提供了guid、身份证、自增等相关占位符号

#### 自定义扩展符
```javascript
Mock.Random.extend({
    constellation: function(date) {
        var constellations = ['白羊座', '金牛座', '双子座', '巨蟹座', '狮子座', '处女座', '天秤座', '天蝎座', '射手座', '摩羯座', '水瓶座', '双鱼座']
        return this.pick(constellations)
    }
})
Mock.Random.constellation()
// => "水瓶座"
Mock.mock('@CONSTELLATION')
// => "天蝎座"
Mock.mock({
    constellation: '@CONSTELLATION'
})
// => { constellation: "射手座" }
```

更多API请参考
> https://github.com/nuysoft/Mock/wiki'

更多实例请参考
> http://mockjs.com/examples.html

可以一边看API、一边看实例，很容易就能上手
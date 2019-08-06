---
title: Maintainable-Javascript
tags:
  - js
categories:
  - 前端
date: 2017-06-30 10:39:31
---

> 本文为阅读《编写可维护的Javascript》的笔记，记录了一些个人认为重要的点，带有一定个人理解，并未深入展开，如需详细了解可阅读原书籍。
> 这本书，是从可维护性的角度出发，介绍了如何编写可维护性的js代码，读完，还是有收获的，特别是第二部分的编程实践，很基础，但也很实用。综合来看，还是一本不错的书籍，值得一看。
> PS:本书主要从可维护性的角度出发，有些写法并不一定是最优解，因人而异，取其精华，去其糟粕。
> 本人github上有很多本人学习前端时保存的demo，都带有注释，适合新手入门。
> 如果对大家有帮助。望star~https://github.com/BryanAdamss/SourceSave


# 《编写可维护的Javascript》笔记

## 编程风格

### 基本格式化

- 代码缩进
    - 使用4空格代替tab；不同编辑器对于tab的解释不一样，有的是2空格长度，有的是4空格长度；
- 语句结尾
    - 总是使用分号`;`结尾
- 行的长度
    - 单行不超过80个字符
- 空行
    - 使用空行分隔语义不同的代码段
- 命名
    - 驼峰命名法
    - 变量
        - 名词开头->`count`、`myName`
    - 函数
        - 动词开头->(can、has、is、get、set)`isEnabled`、`getName`
    - 构造函数
        - 首字母大写
        ```javascript
        function Person(name){
            this.name=name;
        }
        ```
    - 常量
        - 全大写，下划线区分
        ```javascript
        var MAX_COUNT=10,
            URL='https://github.com/BryanAdamss/SourceSave';
        ```
- 直接量
    - 字符串->单双引号皆可，不过个人推荐用单引号，因为在拼接html字符串时很方便)
    - 数字->不省略小数点前后的数字
    - null->当做对象占位符使用
    - undefined->已声明但没有赋值的变量会获得此值
    - 对象直接量
        ```javascript
        // 不好的写法
        var book=new Object();
        book.title='Javascript';
        // 好的写法
        var book={
            title:'Javascript'
        };
        ```
    - 数组直接量
        ```javascript
        // 不好的写法
        var colors=new Array('red','green');
        // 好的写法
        var colors=['red','green'];
        ```

### 注释
- 只在需要注释的时候才添加注释->只在需要让代码变得更清晰的时候添加注释
    - 逻辑复杂难于理解的代码
    - 可能被误认为错误的代码

### 语句和表达式
- switch语句
    - js中的switch不同于其他语言，switch的条件和case从句可以是任意类型值，其他语言必须是原始值或者常量
- with语句->不要使用
- 循环
    - for->在初始化中缓存遍历次数
    ```javascript
    for(var i=0,len=arr.length;i<len;i++){
        doSth();
    }
    ```
    - for-in->配合hasOwnProperty过滤非实例属性/方法
    ```javascript
    for(var prop in testObj){
        if(testObj.hasOwnProperty(prop)){
            console.log('属性名为:'+prop);
            console.log('属性名对应的属性值为:'+testObj[prop]);
        }
    }
    ```
    - forEach->针对数组用forEach
    - 总结:对象(除数组)用for-in，数组用forEach，其他用for

### 变量、函数和运算符
- 变量声明
    - 单var声明
    ```javascript
    var a=3,
        b=4,
        c=5;
    ```
    - 将局部变量的定义做为函数内第一条语句
    ```javascript
    function getName(){
        var a=3,
            b=4,
            c=5;
    }
    ``` 
- 函数声明
    - 先声明再使用
    - 函数内声明函数时，可将函数声明放在变量生命之后
    ```javascript
    function getName(){
        var a=3,
            b=4,
            c=5;
        function getOtherName(){
            doSth();
        }
        
        getOtherName();
    }
    ```
- 立即调用函数
    - 使用圆括号包裹
    ```javascript
    var a=(function(){
        doSth();
    })();
    ```
- 严格模式->只在局部使用
- 相等->使用===
- eval->避免使用
- 原始包装类型->避免使用原始包装类型构造函数

## 编程实践

### UI层的松耦合
- 将javascript从css中抽离
    - 禁用css表达式
    ```css
    /*不好的写法*/
    .box{
        width:expression(document.body.offsetWidth+"px");
    }
    ```
- 将css从javascript中抽离
    - 用js控制样式类，而不是直接操纵样式
    - 当需要控制元素位置时，可直接用js操纵样式(top,left...)
- 将javascript从HTML中抽离
    - 不要在html标签上用onclick=...，改用事件addEventListener
- 将HTML从javascript中抽离
    - 使用客户端模板引擎，例如handlebars
- 

### 避免使用全局变量
- 全局变量带来的问题
    - 命名冲突
    - 代码脆弱性
    - 难以测试
- 意外的全局变量
    - 未声明直接赋值了
    ```javascript
    function (){
        var a=3;// 局部变量
        var b;// 局部变量
        c=3;// 全局变量
    }
    ```
    - 如何避免
        - 总是使用var来声明变量，即时是声明全局变量
- 单全局变量
    - 只声明一个全局变量，所有功能全挂载到这个全局变量上
- 模块
    - 规范CommonJs、AMD、CMD
    - 对应实现NodeJs、RequireJs、SeaJs
- 零全局变量
    - 使用立即函数包裹
    ```javascript
    (function(win){
        // doSth
    })(window);
    ```

### 事件处理
- 不好的写法
    ```javascript
    function handleClick(event){
        var popup=document.getElementById('popup');
        popup.style.left=event.clientX+'px';
        popup.style.top=event.clientY+'px';
        popup.className='reveal';
    }
    ele.addEventListener('click',handleClick,false);
    ```
- 事件处理规则1
    - 隔离应用逻辑->将应用(业务)逻辑从事件处理程序中抽离
    ```javascript
    var MyApp={
        handleClick:function(event){
            this.showPopup(event);
        },
        showPopup:function(event){
            var popup=document.getElementById('popup');
            popup.style.left=event.clientX+'px';
            popup.style.top=event.clientY+'px';
            popup.className='reveal';
        }
    };
    ele.addEventListener('click',function(event){
        MyApp.handleClick(event);
    },false);
    ```
    - 不要分发事件对象->只传需要的信息
        - 应用逻辑不应当依赖于event对象来正确完成功能
            - 将event对象做为参数并不能告诉你event的哪些属性是有用的
            - 测试时，需要重建event对象
        - 最佳实践
            - 让事件处理程序使用event对象来处理事件，然后拿到需要的数据传给应用逻辑
            ```javascript
                var MyApp={
                    handleClick:function(event){
                        this.showPopup(event.clientX,event.clientY);// 只传需要的信息
                    },
                    showPopup:function(x,y){
                        var popup=document.getElementById('popup');
                        popup.style.left=x+'px';
                        popup.style.top=y+'px';
                        popup.className='reveal';
                    }
                };
                ele.addEventListener('click',function(event){
                    MyApp.handleClick(event);
                },false);
            ```
            - 让事件处理程序成为接触到event对象的唯一的函数，事件处理程序应当在进入应用逻辑之前针对event对象执行任何必要的操作。包括阻止默认事件和冒泡。
            ```javascript
                var MyApp={
                    handleClick:function(event){// 在事件处理程序中针对event进行必要的处理
                        event.preventDefault();
                        event.stopPropagation();
                        this.showPopup(event.clientX,event.clientY);// 只传需要的信息
                    },
                    showPopup:function(x,y){
                        var popup=document.getElementById('popup');
                        popup.style.left=x+'px';
                        popup.style.top=y+'px';
                        popup.className='reveal';
                    }
                };
                ele.addEventListener('click',function(event){
                    MyApp.handleClick(event);
                },false);
            ```

### 避免空比较
- 检测原始值的类型
    - 字符串、数字、布尔、undefined->typeof
    ```javascript
    typeof 'a';// 'string'
    typeof 3;// 'number'
    typeof true;// 'boolean'
    typeof undefined;// 'undefined'
    ```
    - null->一般不用于类型检测，除非null是一种可预期的页面时可用===和!==来判断是否为null值
    ```javascript
    var ele=document.getElementById('my-div');
    if(ele!==null){// 如果DOM元素不存在，则ele就为null，此时null是一个可预期的值，所以可以===或!==来判断
        // doSth    
    }
    ```
- 检测引用值的类型
    - 使用value instanceof constructor
    ```javascript
    if(value instance Data){
        // doSth
    }
    if(value instance RegExp){
        // doSth
    }
    if(value instance Object){   
        // doSth    
    }
    ```
    - 但函数、数组不能用instanceof来判断，因为存在跨帧问题(cross-frame)
- 检测函数(判断某一引用值是否是函数(是否是函数类型))
    - 使用typeof
    ```javascript
    function myFn(){}
    typeof myFn;// 'function'
    ```
    - 使用typeof检测IE8及以下DOM元素的方法时，会返回'object';退而求其次会使用in来判断；因为DOM明确定义，了解到对象成员如果存在则意味着它是一个方法
    ```javascript
    if('querySelectorAll' in document){
        var imgs=document.querySelectorAll('img');
    }
    ```
- 检测数组(判断某一引用值是否是数组(是否是数组类型))
    - 使用ES5的isArray
    - 不支持的则使用Object.prototype.toString.call(value)
    ```javascript
    function isArray(value){
        if(typeof Array.isArray==='function'){
            return Array.isArray(value);
        }else{
            return Object.prototype.toString.call(value)==='[object Array]';
        }
    }
    ```
- 检测属性/方法存在性
    - 使用prop in obj
    - 检测属性/方法是否为实例属性->obj.hasOwnProperty('prop')
    - IE8及以下判断是否为实例属性->需先判断hasOwnProperty的存在性

#### 总结
- 判断数据类型
    - 原始值
        - 字符串、数字、布尔、undefined->typeof
            - 如`if(typeof 'test'==='string'){...}`
        - null->只有在null是一个可预期的值时，才用来比较，使用===，!==
    - 引用值
        - 自定义、非函数、非数组对象->使用obj instanceof constructor
            - 如`obj instanceof Data`
        - 函数/方法
            - 非DOM对象的方法/函数->typeof
                - 如`typeof myFn==='function'`
            - DOM对象的方法->无法使用typeof，只能通过in判断它存在，然后直接使用
        - 数组
            - 支持isArray->`Array.isArray(value)`
            - 不支持->`Object.prototype.toString.call(value)==='[object Array]'`
- 判断属性/方法存在性
    - 一般属性/方法->统一使用in
    - 实例属性存在性->统一使用hasOwnProperty
        - IE8及以下判断实例属性存在性->先判断hasOwnProperty的存在性，再调用hasOwnProperty

### 将配置数据从代码中抽离
- 配置数据
    - URL
    - 展现给用户的字符串
    - 重复的值
    - 设置(每页的配置项)
    - 任何可能发生变更的值
- 抽离
    - 将配置数据抽离成一个对象
    ```javascript
    var config={
        MSG_INVALID_VALUE:'不合法的值',
        URL:'https://github.com/BryanAdamss/SourceSave'
    };
    ```
    - 将配置数据抽离成一个对象，并放在一个单独的文件中

### 抛出自定义错误
- 如果没有通过try-catch语句捕获，抛出任何值都将引发一个错误。如直接`throw 'message'`，会引发一个错误
- 何时抛出错误
    - 抛出错误最佳的地方是在工具函数中，如addClass()函数，它是通用脚本的一部分，会在很多地方使用。->在javascript类库中使用
- 错误类型
    - Error、EvalError、RangeError、ReferenceError、SyntaxError、TypeError、URIError

### 不是你的对象不要动
- 不要修改原生对象以及一些类库对象
- 原则：将已经存在的js对象当做工具函数库那样使用
    - 不覆盖方法
        - 不覆盖原对象的方法
    - 不新增方法
        - 不在不属于你的对象上添加方法
    - 不删除方法
        - 不要删除一个不是你的对象上的方法
- 更好方法
    - 继承原对象，在其基础上扩充
- 阻止修改(锁定后，将无法解锁)
    - 防止扩展->无法新增属性和方法，可删除
        - Object.preventExtension(obj);
        - Object.isExtensible();
    - 密封对象->已存在的属性、方法无法被删除，可修改
        - Object.seal(obj);
        - Object.isSealed();
    - 冻结对象->防止扩展+密封，无法删除，无法修改
        - Object.freeze(obj);
        - Object.isFrozen();

### 浏览器嗅探
- UA检测
    - 缺点
        - UA可以被修改
        - 浏览器为了兼容性，都会包含其他浏览器的UA字符串
- 特性检测->根据功能(特性)来检测
    - 不要进行特性推断->不要根据一个特性的是否存在去推断另一个特性是否存在
    - 不要进行浏览器推断->不要根据一个特性的是否存在去推断是某种浏览器
- 优先级:特性检测>UA检测

## 自动化
文章第三部分介绍的是前端自动化方面的知识，但用的是Ant(需要JAVA环境)，由于现在用gulp的比较多，所以这一块就只是大概扫了一下。
- 流程
    - 构建->验证->合并、加工->精简、压缩->文档化->自动化测试->集成

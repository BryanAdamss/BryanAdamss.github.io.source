---
title: type-existence
tags:
  - 类型检测
  - 存在性
categories:
  - 前端
date: 2017-08-04 16:08:31
---

> 本来在《编写可维护性的javascript》中已经做过总结，但最近在使用上又出现一点问题，所以推翻以前的单独写篇关于类型检测和存在性检测的文章

# 类型检测和存在性检测

## 必备知识点

- 哪些值会被转成false
    - ""、0、NaN、false、null、undefined会在期待布尔值的地方被转成false;
    - 非上面提及的值都会被转成true；
    - 注意空对象(没有任何属性/方法的对象)也会视为true
    ```javascript
    var a={};
    if(a){
        console.log(true);// true
    }
    ```
- 声明和赋值
    - 未声明(更未赋值)的变量
        - 直接使用，会报错
        ```javascipt
        console.log(b);// Uncaught ReferenceError: b is not defined
        ```
        - 如果通过typeof b来使用，则不会报错；因为typeof存在一个特殊的安全防范机制；
        
    - 已声明未赋值的变量
        - 会有默认值undefined
        ```javascript
        var a;
        console.log(a===undefined);// true
        console.log(typeof a);// 'undefined'
        ```
    - 注意:当未声明的变量使用typeof检测时，并不会报错，而且返回`'undefined'`；因为typeof存在一个特殊的安全防范机制；
    ```javascript
        console.log(typeof b);// 'undefined'，并没有报错
    ```
    - 总结
        - **未声明和已声明未赋值的变量使用typeof检测时，都会返回`'undefined'`**
- **访问对象上不存在的属性/方法时，并不会报错，而是返回一个`undefined`**
```javascript
var obj={
    a:3
};
console.log(obj.b);// undefined
```

- 类型检测->(判断值的类型)
    - 首先变量是没有类型的，类型本质指的是变量持有的值的类型，一般说的变量类型，实际指的是变量持有的值的类型
    - 判断类型主要用来，检测输入的参数是否为想要的类型
    ```javascript
    function test(fn){
        if(typeof fn==='function'){
            // xxxxx
        }
    }
    ```
    - 一般值
        - string、number、boolean、undefined->typeof来判断
        - null一般不用做类型检测，只有在变量是一个可预期的null值时，才用来判断
        ```javascript
        var obj=null;
        if(obj===null){
            // xxx
        }
        ```
    - 引用值
        - 自定义、非数组、非函数->使用obj instanceof constructor
        ```javascript
        function People(name){
            this.name=name;
        }
        var p=new People();
        console.log(p instanceof People);// true
        
        var date=new Date();
        console.log(date instanceof Date);// true
        ```
        - 函数->typeof
        ```javascript
        function fn(){}
        console.log(typeof fn==='function');// 'function'
        ```
        - 数组
            - es5的isArray
            ```javascript
            var arr=[];
            console.log(Array.isArray(arr));// true
            ```
            - Object.prototype.call(arr);
            ```javascript
            var arr=[];
            console.log(Object.prototype.toString.call(arr)==='[object Array]');
            ```
- 存在性
    - 常用检测存在性的不足
    ```javascript
    var obj={
        b:0
    };
    if(b){}// 如果b存在，则xxx；当b为"",0,NaN,false,null,undefined时，就无法检测；
    同理b&&b()也会出现类似问题，所以只有在明确知道要检测的值不会是"",0,NaN,false,null,undefined中的一种时才能用
    ```
    - 变量是否存在(是否已经声明)
        - 全局变量的存在性
        ```javascript
        console.log('a' in window);// false;判断变量a在全局环境下是否声明
        ```
        - 局部变量的存在性
            - 局部变量无法用in判断，只能退而求其次用typeof，typeof无法准确判断出是未声明还是已声明未赋值，如下
            ```javascript
            var a;
            console.log(typeof a==='undefined');// true;a已经声明但未赋值 
            console.log(typeof b==='undefined');// true;b没有声明
            ```
    - 对象的属性是否存在
        - 一般属性->统一使用in
        - 实例属性存在性->统一使用hasOwnProperty
            - IE8及以下判断实例属性存在性->先判断hasOwnProperty的存在性，再调用hasOwnProperty
       ```javascript
        var obj_a={
            test:'测试'
        };
        console.log('test' in obj_a);// true
        console.log('toString' in obj_a);// true，能检测到原型链上的方法
        console.log(obj_a.hasOwnProperty('toString'));//false,obj_a并没有实例属性(方法)`toString`，`toString`存在于其原型对象上，hasOwnProperty无法检测到
        ```
             
            


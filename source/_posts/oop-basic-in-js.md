---
title: oop-basic-in-js
tags:
  - oop
  - 面向对象
  - 创建对象
  - 继承
categories:
  - 前端
date: 2018-04-27 10:23:58
---

> 最近在复习基础知识，特对js面向对象相关知识做个记录

# js中的面向对象编程基础知识

## 创建对象

### 字面量模式
- 最简单也是最常用创建简单对象的方式
```javascript
var personCgh={
    name:'cgh',
    age:'18',
    sex:'boy',
    sayName:function(){
        alert(this.name);
    }
};
```
- 缺点
    - 如果需要创建大量类似的对象，会产生大量重复代码

### 工厂模式
- 抽象(封装)创建具体对象的过程
```javascript
function createPersonFactory(name,age,sex){
    var o=new Object();
    o.name=name;
    o.age=age;
    o.sex=sex;
    o.sayName=function(){
        alert(this.name);
    };
    return o;
}

var personCgh=createPersonFactory('cgh',18,'boy');
```
- 解决了字面量模式创建多个类似对象的问题
- 缺点
    - 无法知道一个对象的类型

### 构造函数模式
- 通过创建自定义构造函数并通过new来创建实例
```javascript
function Person(name,age,sex){
    this.name=name;
    this.age=age;
    this.sex=sex;
    this.sayName=function(){
        alert(this.name);
    };
}

var personCgh=new Person('cgh',18,'boy');

personCgh instanceof Person; // true
personCgh instanceof Object; // true
```
- 缺点
    - 每个方法都需要在实例上重新创建一遍

### 原型模式
- 每个函数在创建时都会自动生成一个原型(`prototype`)属性，指向函数的原型对象;原型对象会自动获得一个`constructor`属性指向`prototype`属性所在的函数；
- 实例对象、构造函数、原型对象三者关系
    - 构造函数被创建时，会有一个`prototype`属性指向原型对象
    - 原型对象通过`constructor`反向指回构造函数
    - 构造函数通过实例化过程(new过程)创建实例对象
    - 实例对象通过内部`[[prototype]]`或者`__proto__`属性(不可直接访问)指向原型对象
```javascript
function Person(){}

Person.prototype={
    constructor:Person,// 因为完全重写了prototype对象，所以必须定义constructor指向
    name:'cgh',
    age:18,
    sex:'boy',
    friends:['a','b'],
    sayName:function(){
        alert(this.name)
    }
};

var personCgh=new Person();
var personYg=new Person();

personCgh.friends.push('c');

personCgh.friends; // ['a','b','c']
personYg.friends; // ['a','b','c']

personCgh.name === personYg.name; // true
```
- 缺点
    - 会存在共享引用值的问题

### 组合使用构造函数模式和原型模式
- 使用构造函数创建实例属性，用原型模式创建共享的方法和属性
- 常用模式
```javascript
function Person(name,age,job){
    this.name=name;
    this.age=age;
    this.job=job;
    this.friends=['a','b'];
}

Person.prototype={
    constructor:Person,
    sayName:function(){
        alert(this.name);
    }
};

var personCgh=new Person('cgh',18,'boy');
var personYg=new Person('yg',14,'boy');

personCgh.friends.push('c'); // ['a','b','c']
personYg.friends; // ['a','b']
```

### new的过程
- new Fn操作符主要完成了下面几件事情
    - 创建一个新的空对象`tempObj`，空对象的默认原型对象(`__proto__`)为`Object.prototype`
    - 设置空对象的`__proto__`指向构造函数的原型对象；
    - 调用构造函数，构造函数的`this`指向`tempObj`；
    - 如果构造函数返回的是一个`非null`的引用类型的对象，则用此对象替代`tempObj`成为new操作的返回对象
    - new会将返回对象的`__proto__`指向构造函数的`.prototype`对象
```javascript
function Test(){
    this.name='test';
}

var test=new Test();
// new Test实际执行的伪代码
...
{
    var temp={};// 创建临时对象
    temp.__proto__=Test.prototype;// 改变__ptoto__指向
    var ret=Test.call(tempObj);
    return ret!==null && (typeof ret === 'object'|| typeof ret === 'function') ? ret : temp ;// 如果构造函数调用后返回的是非null的引用类型，则用其替代temp做为new操作符的返回值
}
```

## 继承

### 使用原型链实现继承
- 让一个类(构造函数，基类)的原型对象指向另一个类(构造函数，父类)的实例即可
```javascript
function SuperType(){
    this.property=true;
}

SuperType.prototype.getSuperValue=function(){
    return this.property;
};

function SubType(){
    this.subProperty=false;
}

// 继承父类 基类的原型对象指向父类的一个实例
SubType.prototype=new SuperType();

SubType.prototype.getSubVaule=function(){
    return this.subProperty;
};

var instance=new SubType();
instance.getSuperValue();// true
```
- 如果基类要覆盖父类方法一定要在继承之后再覆盖
- 缺点
    - 父类上定义的引用类型值将会被共享
    - 无法在不影响所有对象实例的基础上，给父类的构造函数传递参数

### 借用构造函数实现继承
- 通过在子类的构造函数中调用父类的构造函数来实现继承
```javascript
function SuperType(name){
    this.colors=['red','blue']
    this.name=name;
}

function SubType(){
    SuperType.call(this);
}

var instance1=new SubType('cgh');
instance1.colors.push('green');

var instance2=new SubType('yg');

instance1.colors;// ['red','blue','green']
instance2.colors;// ['red','blue']
```
- 缺点
    - 方法都将在构造函数中定义，无法实现方法的复用

### 组合继承
- 借用构造函数+原型链
- 使用原型链实现对原型属性、方法的继承，使用借用构造函数实现实例属性的继承
```javascript
function SuperType(name){
    this.name=name;
    this.colors=['red','blue'];
}

SuperType.prototype.sayName=function(){
    alert(this.name);
};

function SubType(name,age){
    // 继承属性
    SuperType.call(this,name);
    
    this.age=age;
}

// 继承方法
SubType.prototype=new SuperType();
SubType.prototype.constructor=SubType;

SubType.prototype.sayAge=function(){
    alert(this.age);
};

var instance1=new SubType('cgh',20);
instance1.colors.push('green');
instance1.colors;// ['red','blue','green'];
instance1.sayName();// 'cgh'
instance1.sayAge();// 20

var instance2=new SubType('yg',18);
instance1.colors.push('pink');
instance1.colors;// ['red','blue','pink'];
instance1.sayName();// 'yg'
instance1.sayAge();// 18
```

### 原型继承
- 通过改变对象内部的`__proto__`指针来实现
- 不需要显示的创建构造函数
- 思路直接以一个对象为原型，从其克隆出一个新对象
```javascript
function object(o){
    function F(){}
    F.prototype=o;// 指定原型对象
    return new F();// 通过new 调用将返回对象的__proto__指向F.prototype即o   
}

// 本质是将返回对象的__proto__指向了传入对象
var returnObj=object(testObj);
returnObj.__proto__===testObj;//true
```
- es5对这种方式做了标准化。直接使用`Object.create()`来实现
```javascript
var a={
        name:'cgh',
        sayName:function(){
            alert(this.name);
        }
    };

var b=Object.create(a);
b.name='yg';
b.sayName();// yg
```

### 实现继承的本质

- `js`中实现继承的关键在`__proto__`这个内部指针
- 首先基础知识
    - js中万物都是对象，没有实际的类(可以靠模拟实现类的效果)、函数、构造函数(本质就是普通函数)、原型对象(`xxx.prototype`)，这些都是对象
    - 每个对象都有`__proto__`这个内部指针
    - 每个函数都有`.prototype`这个属性
    - 所有函数都是对象；所以函数既有`__proto__`也有`.prototype`
    - 内部通过`__proto__`来构建原型链实现类似继承效果，即方法属性的查找通过`__proto__`链来一层层查找;(类似作用域链的回溯查找)
    - 所有原型链最后端都是`Object.prototype`，`Object.prototype`这个对象的`__proto__`则指向了`null`
    - `__proto__`和`fn.prototype`不一样，前者才是实现继承的关键，后者只是为了模拟传统类继承而衍生出来方便表达的属性。可以当作一个普通对象对待(但它又有不同，它默认自带`constructor`指回函数)
```javascript
var o = {a: 1};
// o 这个对象继承了Object.prototype上面的所有属性
// o 自身没有名为 hasOwnProperty 的属性
// hasOwnProperty 是 Object.prototype 的属性
// 因此 o 继承了 Object.prototype 的 hasOwnProperty
// Object.prototype 的原型为 null
// 原型链如下:
// o ---> Object.prototype ---> null

// o.__proto___===Object.prototype;// true
// Object.prototype.__proto__===null;// true


var a = ["yo", "whadup", "?"];
// 数组都继承于 Array.prototype 
// (Array.prototype 中包含 indexOf, forEach等方法)
// 原型链如下:
// a ---> Array.prototype ---> Object.prototype ---> null

// a.__proto__===Array.prototype;// true
// Array.prototype.__proto__===Object.prototype;// true
// Object.prototype.__proto__===null;// true

function f(){
  return 2;
}
// 函数都继承于Function.prototype
// (Function.prototype 中包含 call, bind等方法)
// 原型链如下:
// f ---> Function.prototype ---> Object.prototype ---> null

// f.__proto__===Function.prototype;// true
// Function.prototype.__proto__===Object.prototype;// true
// Object.prototype.__proto__===null;// true
```
- js中万物都是对象，不一定非要模拟传统语言先有类(模板)再创建实例的形式来实现继承效果呢？为什么不能直接用一个对象做为模板，直接克隆出另一个类似对象呢?js内部通过`__proto_`，来实现两个对象之间的关联(委托)关系，进而实现类似继承的效果。
    - 原型继承和`Object.create()`都是类似思想，直接通过建立两个对象间的委托关系，实现继承效果
```javascript
var a={
        name:'cgh',
        sayName:function(){
            alert(this.name);
        }
    };

var b=Object.create(a);
b.name='yg';
b.sayName();// yg
```
- `__proto__`是一个内部属性，es6对其做了规范可以通过`Object.getPrototypeOf`和对应的`Object.setPrototypeOf`来操作`__proto__`

> 参考链接
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
> https://www.zhihu.com/question/34183746
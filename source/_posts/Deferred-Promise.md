---
title: Deferred_Promise
tags:
  - 异步
  - Promise
categories:
  - 前端
date: 2017-09-07 20:56:31
---

> 在最近项目中经常会遇到异步处理的相关问题，在查阅相关资料后，特在此做一篇笔记。

# 使用Deferred、Promise解决jQuery中异步相关问题

## 问题
1. ABC是3个异步请求，现在要求C在AB三个异步请求都成功返回的情况下再执行。这种就比较麻烦，可以尝试设置请求完成状态变量，当AB的请求完成变量都true时再请求C；如果不只3个请求，这种方法就会很糟糕。
2. ABC是3个异步请求，现在要求ABC3个请求按顺序依次执行，A->B->C。这种用传统方法可能就需要用回调嵌套的方法来实现

---
以上两种情况是在异步中经常遇到的，用传统方法编写，会导致嵌套层次过多，不仅影响可读性，还不易于维护。为了解决这种问题，CommonJs组织制定了异步编程规范[Promises/A](http://wiki.commonjs.org/wiki/Promises/A)。这个规范有很多实现，如when.js、ES6的Promise等。
今天就借助jQuery的Deferred、Promise对象来做个简单了解。


## Promise状态

### Promise对象存在3种状态
- pending(未完成状态)
- resolved(肯定状态)
- rejected(否定状态)

### 这三种状态的转换关系
- pending->resolved
- pending->rejected
- pending->pending

### 当转换到resolved或者rejected状态时，状态是无法再发生变化，即下面的状态转换都是不可行的
- resolved->rejected
- resolved->pending
- rejected->resolved
- rejected->pending


## 创建一个Promise对象

### 在jQuery中Deferred可以理解为Promise的加强版，先不做区分，可以将Deferred当成就是Promise，后面会介绍二者区别。
```javascript
var dfr=$.Deferred();// 创建一个Deferred对象(就是Promise对象)
console.log(dfr.state());// 获取当前状态,pending
dfr.resolve();// 将Deferred对象状态改变为resolved
console.log(dfr.state());// resolved

var dfr2=$.Deferred();
console.log(dfr2.state());// 获取当前状态,pending
dfr.reject();// 将Deferred对象状态改变为rejected
console.log(dfr2.state());// rejected
```

### 状态的作用
- 通过上面的例子，我们可以知道，可以人为的改变Deferred对象的状态。状态不一样有什么用呢？我们可以根据不同的状态进行不同的操作(添加不同的回调函数)。

## 给Promise对象添加回调

### 添加回调，并触发
```javascript
var dfr=$.Deferred();// 创建Deferred对象
dfr
.done(function(){// Deferred对象状态变为resolved时的回调
    alert('成功');
})
.fail(function(){// Deferred对象状态变为reject时的回调
    alert('失败');
})
.progress(function(){// Deferred对象状态为pending时的回调
    alert('进行中...');
});
dfr.notify(); // 触发Deferred对象pending状态的回调
dfr.resolve();// 触发Deferred对象resolved状态的回调
```
- 通过done()、fail()、progress()给Deferred对象的不同状态分别添加了回调，并通过notify()、resolve触发了响应的回调

### 传递数据
- 通过done()、fail()、progress()触发Deferred对象的回调时，可传递一些数据(任何类型)给回调函数
```javascript
var dfr2=$.Deferred();// 创建Deferred对象
dfr2
.done(function(msg){// Deferred对象状态变为resolved时的回调
    alert(msg+'成功');
})
.fail(function(msg){// Deferred对象状态变为reject时的回调
    alert(msg+'失败');
})
.progress(function(msg){// Deferred对象状态为pending时的回调
    alert(msg+'进行中...');
});
dfr2.notify('dfr2'); // dfr2进行中...
dfr2.reject('dfr2');// dfr2失败
```

### 链式调用
- done()、fail()、progress()会返回调用者对象Deferred对象，因此可以进行无限的链式调用；可以在done()后再添加done()、fail()、progress()，他们会在对应状态被激活时，依次按照添加顺序调用。
```javascript
var dfr=$.Deferred();
dfr
.done(function(){ // 回调1
    alert('成功1');
})
.fail(function(){
    alert('失败');
})
.progress(function(){
    alert('进行中...');
})
.done(function(){// 回调2
    alert('成功2');
});

dfr.resolve();// 成功1->成功2
```

### deferred.always()
- 通过deferred.always()添加的回调，无论状态是resolved还是rejected都会在最后被调用
```javascript
var dfr=$.Deferred();
dfr
.done(function(){// 
    alert('成功1');
})
.fail(function(){
    alert('失败');
})
.progress(function(){
    alert('进行中...');
})
.always(function(){
    alert('我总会被执行');
});

dfr.resolve();// 成功1->我总会被执行
```

### Deferred对象使用方式
```javascript
var dfr = $.Deferred();// 创建一个Deferred对象
var task = function(dtd) {
    setTimeout(function() {
        console.log('timeOut');
        dtd.resolve(); // 异步任务结束，手动resolve
    }, 3000);
    return dtd;// 返回Deferred对象，供$.when()使用
};
$.when(task(dfr)).done(function() {
    alert('success');// 3s后弹出
});
```
- 上面例子由于dfr是全局对象，并且包含改变状态的方法resolve、reject，所以可以在外部提前终止任务
```javascript
var dfr = $.Deferred();// 创建一个Deferred对象
var task = function(dtd) {
    setTimeout(function() {
        console.log('timeOut');
        dtd.resolve(); // 异步任务结束，手动resolve
    }, 3000);
    return dtd;// 返回Deferred对象，供$.when()使用
};
$.when(task(dfr)).done(function() {
    alert('success');// 立即弹出
});
dfr.resolve();// 外部resolve后会立即执行done
```
- 防止外部终止，可以将全局的dfr放到函数内部
```javascript
var task = function() {
    var dfr = $.Deferred();// 创建一个Deferred对象
    setTimeout(function() {
        console.log('timeOut');
        dfr.resolve(); // 异步任务结束，手动resolve
    }, 3000);
    return dfr;// 返回Deferred对象，供$.when()使用
};
$.when(task()).done(function() {
    alert('success');// 立即弹出
});
dfr.resolve();// 无法调用
```

## jQuery中Deferred和Promise的区别
- Deferred对象可以理解为Promise对象的加强版。
- Deferred对象包含改变状态的方法，如dfr.resolve()、dfr.reject()、dfr.notify()
- Promise对象则不包含以上方法；
- **要想改变状态必须在Deferred对象上调用相关方法，Promise对象没有相关方法。**
- 通过deferred.promise()可以将Deferred对象转换为Promise对象

## 在ajax中使用Promise

### ajax和Promise的关系
- 在jQuery1.5之前$.ajax()返回的是一个jqXHR对象，1.5之后返回的是一个类Promise对象，它在原先的jqXHR对象基础上又添加一些Promise方法，因此我们能在$.ajax()之后链式调用Promise相关方法；
- 注意返回的是一个类Promise对象，因此它不包含改变状态的相关方法；
- **改变相关状态由ajax内部完成，无需手动调用相关方法(也无法调用)；**
```javascript
// 老的ajax写法  
$.ajax({  
　　url: "a.html",  
　　success: function(){  
　　　　alert("成功");  
　　},  
　　error:function(){  
　　　　alert("错误");  
　　}  
});  
  
// 使用promise后的写法  
$.ajax("test.html")  
 .done(function(){})  
 .fail(function(){})  
 .done(function(){})  
 .fail(function(){});
```

### 解决问题1
- 问题1要求C在AB都执行完后再执行。即A&&B->C；这时候就需要使用jQuery提供的$.when()函数。$.when()返回一个Promise对象。所以可以调用done、fail、progress等函数
```javascript
$.when($.ajax(url1),$.ajax(url2))
.done(function(){
    console.log('url1、url2都请求成功');
    $.ajax(url3)
})
.fail(function(){
    console.log('url1、url2有一个或者两个没请求成功');
});
```
- $.when()实现了多个ajax请求完成后再执行某些操作；即实现了A&&B->C的效果

### 解决问题2
- 问题2的要求是ABC3个异步请求顺序执行。传统写法可能是
```javascript
$.ajax({
    url:'a.json',
    success:function(){
        $.ajax({
            url:'b.json',
            success:function(){
                $.ajax({
                url:'c.json',
                success:function(){
                    console.log('gg');   
            }    
        }
    }
});
```
- 可读性很差，还不方便维护。为解决问题2需要使用到jQuery提供的Deferred.then()方法；
- then方法可以传入3个回调，分别是resolved、rejected、pending状态的回调；
```javascript
function success(data)  
{  
    alert("success data = " + data);  
}  
  
function fail(data)  
{  
    alert("fail data = " + data);  
}  
  
function progress(data)  
{  
    alert("progress data = " + data);  
}  
  
var deferred = $.Deferred();  
  
// 一起注册回调  
deferred.then(success, fail, progress);  
  
// 分别注册回调  
deferred.done(success);  
deferred.fail(fail);  
deferred.progress(progress);  
  
deferred.notify("10%");  
deferred.resolve("ok"); 
```

- **其实在执行then方法后将返回一个新的Promise对象**
    - **可以在后面无限级联调用相关Promise方法.then().then().done().fail()....**
    - 这就意味着在then后就无法在返回对象(返回的是Promise对象)上手动改变状态了。
    - **必须在原先的Deferred对象上调用方法改变状态**
```javascript
function success(data)  
{  
    alert("success data = " + data);  
}  
  
function fail(data)  
{  
    alert("fail data = " + data);  
}  
  
function progress(data)  
{  
    alert("progress data = " + data);  
} 
var dfr=$.Deferred();
var pro=dfr.then(success,fail,progress);
console.log(dfr===pro);// false

// 没有改变状态的方法
console.log('resolve' in pro); // false
console.log('reject' in pro); // false
console.log('notify' in pro); // false

// 只能在原先的Deferred对象调用相关方法
dfr.resolve('resolved'); // success data = resolved
```

- 其实then()中传入的不是回调函数，官方说法又叫做过滤函数；前面说过Deferred对象在调用改变状态方法时，可以传递数据，其实通过then注册的回调可以对数据进行过滤，然后通过return将数据传递给下一个回调函数(done、fail、progress)，如果下一个回调函数是通过then注册的，则可以继续对数据进行过滤，并传递给下一个对应状态的回调函数；
- 我们知道deferred.resolve()、deferred.reject()、deferred.notify()可以指定参数值，这个参数会传递给相应状态下的回调函数。
- **如果我们使用的是done()、fail()、progress()注册的回调函数，那么某个状态下的所有回调函数得到的都是相同参数**。
- 不是通过then注册的回调函数，无法对数据过滤并通过return传递给下一个回调，他们得到的都是相同值，可看下面例子
```javascript
var dfr = $.Deferred();
dfr
.done(function(type) {
    console.log(type);// resolved
    return type + 'first';
})
.done(function(type) {
    console.log(type);// resolved
    return type + 'last';
})
.done(function(type) {
    console.log(type);// resolved
});
dfr.resolve('resolved');
```
- 但是如果我们**使用了then()注册回调函数，那么第一回调函数的返回值将作为第二个回调函数的参数，同样的第二个函数的返回值是第三个回调函数的参数**。
```javascript
var deferred = $.Deferred();  
  
// then()返回的是一个新Promise对象  
//then注册的回调函数的返回值将作为这个新Promise的参数  
var then_ret = deferred.then(function(data){  
    alert("data="+data);//5  对数据进行过滤
    return 2 * data;  // 并通过return 传递给下一个done
});
  
alert(then_ret == deferred);//false  
  
then_ret.done(function(data){  
    alert("data="+data);//10  
});  
  
deferred.resolve( 5 );  
```
- 如果仔细观察，会发现在上面例子中，我们返回的是普通值，如果我们返回的是Deferred或者Promise对象，它会将返回的Deferred、Promise对象的状态和返回值传递给下一个回调函数，做为其触发依据和参数。可以用这种方法解决问题2
```javascript
var promise1 = $.ajax(url1);  
var promise2 = promise1.then(function(data){
    return $.ajax(url2, { "data": data });// 返回一个promise，它的状态将决定触发promise2.then中的哪个回调，它的返回值将传递给对应的回调函数
});  
var promise3 = promise2.then(function(data){
    return $.ajax(url3);// 返回一个promise，它的状态将决定触发promise3中的哪个回调，它的返回值将传递给对应的回调函数
});  
promise3.done(function(data){  
    console.log(data);
});  
```
- 这样其实我们可以得到一个范式，处理有依赖关系的异步请求时，可以.then().then().done().fail()，通过then中的回调(过滤)函数，对数据进行加工，最后交给不是通过then注册的done或者fail来进行最后处理；done其实就预示着对传过来的数据不进行加工了；

## 总结
- jQuery中的Deferred、Promise对象主要用来解决异步任务中嵌套问题
- Deferred可以理解为Promise对象的加强版
    - Deferred对象拥有方法resolve、reject、notify来手动改变状态
    - Promise对象无法手动改变状态
    - deferred.promise()可以将一个Deferred对象转换成Promise对象
    - jQuery中异步任务返回的都是Promise对象或者类Promise对象(ajax返回的)，它们都无法手动改变状态，它们状态的改变是jQuery在内部自动完成的
***    
- $.Deferred()返回一个Deferred对象
- deferred.done、deferred.fail、deferred.progress用来定义Deferred对象状态对应的回调函数
- deferred.always()来用定义无论成功还是失败都会调用回调函数
- deferred.resolve()、deferred.reject()手动改变Deferred对象的状态
    - 改变状态时，可以传递数据给回调函数
        - deferred.resolve('msg')
    - 防止改变状态方法在异步任务外调用
        - 可将Deferred对象定义为异步任务内的局部变量
        - 可以使用deferred.promise()转换成Promise对象
- deferred.notify()用来触发deferred.progress定义的回调函数，实际可以用来完成进度条效果
- deferred.then()会返回一个新的promise对象
    - then中定义的回调函数可以理解为过滤函数，可对resolve、reject中传递的数据进行加工、过滤，然后通过return传递给下一个回调函数
        - 如果return的是Deferred或者Promise对象，它会将返回的Deferred、Promise对象的状态和返回值传递给下一个回调函数，做为其触发依据和参数。
***
- A&&B->C类型异步任务可以使用$.when()来解决；见上面例子
    - 范式
    ```javascript
    $.when($.ajax(url1),$.ajax(url2))
    .done(function(){
        $.ajax(url3);
    }).fail(function(){
        console.log('出错');
    );
    ```
- A->B->C类型异步任务可以使用Promise对象的then()来解决；见上面例子
    - 范式
    ```javascript
    $.ajax(url1)
    .then(function(url1Data){
        return $.ajax(url2);
    })
    .then(function(url2Data){
        return $.ajax(url3);
    })
    .done(function(url3Data){
        // 最终成功处理
    })
    .fail(function(url3Data){
        // 最终失败处理
    });
    ```
---
title: how-does-timer-work
tags:
  - timer
categories:
  - 前端
date: 2017-07-31 10:55:32
---

> 最近看了些关于js中定时器原理解析的文章，所以特在此做一个记录
> 本文带有个人理解，若有错误，望指正。 

# JS中的定时器(setTimeout)是如何工作的?

## 定时器
 
js中的setTimeout主要用来完成一些超时调用的任务，可以指定函数在未来的某个时间执行。
```javascript
    setTimeout(function(){
        console.log('hi');
    },500);
```
理论情况下，'hi'会在500ms后被打印出来。

考虑下面这种情况
```javascript
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },1000);
    console.log(3);
```
最后输出的顺序是`1,3,2`，你可能在想因为2被延迟了1000ms，所以最后输出的。

再看下面的
```javascript
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },0);
    console.log(3);
```
这次我们将延迟的时间从1000调成了0，这次应该输出`1,2,3`了吧，但实际上最后的输出结果还是`1,3,2`
这是为什么呢？要解释清楚这个就必须了解setTimeout的工作原理了。

## 工作原理

js是单线程的，它同一时间它只能干一件事情。那你可能会问为什么不多弄几个线程，这样多管齐下，不是执行效率更高了吗？当时js的用途(交互、操作dom)决定了它只能是单线程的，如果多线程，就会存在多线程同步的问题。我在一个线程中删除了节点a，另一个线程在节点a上添加了一些内容，这样就会导致冲突，将一个简单的问题负责化了，所以最终js是单线程的。

解释下下面代码的执行过程
```javascript
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },1000);
    console.log(3);
```

首先js中存在一个callstack(调用栈)的东西，它会将函数/方法压入(push)到栈中，并依次出栈(pop)执行。

默认上面代码外围有个main函数
1. main入栈
2. console.log(1)入栈
3. console.log(1)出栈并调用打印出1
4. setTimeout入栈
5. 发现setTimeout是个延迟执行，出栈时，将需要延迟执行的回调函数交给浏览器的timer模块，timer模块负责观察延迟执行的回调函数是否到达触发条件，此时call stack会继续将后面的方法压入栈中
6. console.log(3)入栈
7. console.log(3)出栈并调用打印出3
8. main出栈
9. timer模块观察到延迟执行的函数到达触发条件后，将延迟执行的回调函数推入任务队列(task queue)中
10. 当调用栈处于空闲状态时，它会将任务队列中的第一个任务压入callstack中，并调用，并一直重复这个过程直到任务队列为空。这个过程称为event loop

上面的setTimeout的延迟是1000，为0的时候其实也是一样的，只不过在timer模块中，它会立即到达触发条件，并被推入任务队列中，等待call stack空闲时，再压入到call stack中并调用。

上面是关于setTimeout延迟函数的调用过程，其实js中的事件、ajax的执行流程也一样(其实你会发现他们有个共同点，都有回调函数)。只不过setTimeout有一个具体的延迟时间，延迟时间到达了触发。事件是在用户进行某种操作后(点击)，立即将回调函数推入任务队列中，call stack空闲时取第一个并执行。ajax则是在返回数据后(满足触发条件)，将回调推入任务队列中，call stack空闲时取第一个并执行。

***

其实js的任务(代码)可以分为同步任务和异步任务(事件、ajax、setTimeout)，异步任务的回调一定是在所有同步任务都执行完了以后再被调用;
```javascript
    console.log(1);
    setTimeout(function(){
        console.log(2);
    },0);
    console.log(3);
```
如上面，即使setTimeout的延迟时间为0，它的回调函数也没有直接被调用，而是等到console.log(3)执行完，call stack为空时，再被调用执行的。
所以setTimeour(fn,0)常用来在所有同步任务执行完后，尽可能早的执行；

再看下面的代码
```javascript
    var req = new XMLHttpRequest();
    req.open('GET', url);    
    req.onload = function (){};    
    req.onerror = function (){};    
    req.send();
```
和
```javascript
    var req = new XMLHttpRequest();
    req.open('GET', url);
    req.send();
    req.onload = function (){};    
    req.onerror = function (){};   
```
二者效果一样。

onload和onerror的位置无关紧要，不用担心先send了,load和error不会触发。因为load和error事件都属于异步任务(事件)，他们的回调函数一定是在所有同步任务完成后再被调用的。

***

**总结**：

通过上面可以发现，js中的异步任务(事件、ajax、setTimeout)，是需要call stack、浏览器中的对应模块(DOM Binding、network、timer)、task queue三者配合来完成异步任务；
- call stack负责压入待执行的方法/函数，遇到异步任务时，会将异步任务交给对应模块处理；
- 浏览器对应模块负责判断异步任务是否满足触发条件，若满足触发条件，则将异步任务的回调推入task queue中
- task queue负责保存所有已经满足触发条件可以压入call stack中执行的异步任务回调。
- 当call stack空闲时，会将task queue中的第一个回调压入call stack中并执行，并一直循环这一过程直到task queue为空；->event loop

所有异步任务的回调一定是在所有同步任务都执行完了后再被调用

setTimeout(fn,0)无论写在哪，它的作用都是在所有同步任务执行完后，尽可能早的执行fn

参考链接
> http://www.alloyteam.com/2015/10/turning-to-javascript-series-from-settimeout-said-the-event-loop-model/
> http://www.ruanyifeng.com/blog/2014/10/event-loop.html
> https://vimeo.com/96425312
> http://latentflip.com/loupe/
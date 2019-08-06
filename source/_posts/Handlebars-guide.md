---
title: Handlebars-guide
tags:
  - Handlebars
categories:
  - 前端
date: 2017-03-17 18:48:09
---

#HandleBars备忘

## 表达式

	{{xxx}}

	{{{xxxx}}}因为双大括号默认会进行HTML转义,将<转换为&lt;,通过三括号可以避免

## Helper

### 默认Helper

	{{#if a}}如果a为真执行这个{{/if}}

    {{#else}}否则执行这个{{/else}}

    {{#unless a}}当a为假的时候执行这个{{/unless}}

    {{#each obj}}遍历obj的每个属性{{/each}}
    	#each内可以用{{@index}}获取当前遍历的索引值，用{{@key}}获取当前属性的属性名，用{{this}}可以获当前属性的值
    	如何获取父对象
    		可以通过{{../父对象某属性}}来获取父对象的某个属性，{{@../index}}获取父对象当前的所引值
    		
    {{#with obj}}xxx{{/with}}类似js中with，可改变作作用域，在each中也可通过#with 父对象名 来访问父对象
    
    {{lookup xxx}}一般用来按照索引来找兄弟变量对应的值{{lookup ../父对象某属性 @index}}，查找父对象在当前索引下值

### 自定义Helper

#### 行级Helper

	语法{{helperName [普通值参数][hash值参数]}}

	{{customHelper "My Text" class="my-class" visible=true counter=4}}

	Handlebars.registerHelper('customHelper', function() {
			console.log(arguments[0]);//==>"My Text"
			console.log(arguments[1].hash);//==>{class:"my-class",visible:true,conter:4}
	});	

#### 块级Helper

	语法{{#helperName context [普通值参数][hash值参数]}}xxxxxxxxx{{/helperName}}

	{{#customHelper nav "normalValue" class="my-class" visible=true counter=4}}
		{{if name}}
			aaaaaa
		{{else}}
			bbbbbbb
		{{/if}}
	{{/customHelper}}
	
	Handlebars.registerHelper('customHelper', function(context,options) {
		console.log(arguments[0]);//==>context
		console.log(arguments[1]);//==>normalValue
		console.log(arguments[2]);//==>options
		//说明在registerHelper内部,第二个参数匿名函数,只会存在3个实参，第一个为当前使用的上下文，第二个如果有则为一个普通值参数，另外一个就是封装了函数相关信息的options对象，它有hash(封装好一个或多个键值对参数)、fn(传入一个上下文，并在此上下文中执行customHelper模板中的{{#if}}到{{else}}中的模板)、inverse(同fn相反执行{{else}}到{{/if}}之间的)、name、data等属性。
	});

> 参考:http://cnodejs.org/topic/56a2e8b1cd415452622eed2d
> http://www.cnblogs.com/iyangyuan/archive/2013/12/12/3471227.html
---
title: Angualr-guide
tags:
  - MVC
  - framework
  - Angular
categories:
  - 前端
date: 2017-06-07 09:19:44
---


> 本文为自己学习angular的笔记，带有个人理解。
> 学习的版本为1.2.32
> 对应的源码在https://github.com/BryanAdamss/SourceSave/tree/master/AngularJs
> 本仓库还包含大量本人前端练习用的demo，希望对大家有帮助。望star~


# Angular学习笔记

## Angular适用场景
-  适合：大量CRUD(增删改查)操作的场景，如后台管理系统
-  不适合：游戏、大量UI操作的场景

## DataBinding
- 双向：view、model之间会相互同步数据

## Controller
- 在controller中做
    - 设置$scope的初始状态
    - 为$scope添加一些行为
- 不要在controller中做
    - 手动操作DOM：controller中应该仅包含业务逻辑；如果在controller中添加表现相关逻辑，会严重影响测试；DOM操作应该封装在directives中；另外AngularJs中也封装了一些常用DOM操作指令，如ng-show等。
    - 格式化输入： Use AngularJS form controls instead.
	- 过滤输出： Use AngularJS filters instead.
	- 传递数据或状态： Use AngularJS services instead.
	- 管理其他组件的周期：例如在controller中创建services

## Services
- 特点
	- lazy实例化：只有当某个services被依赖时，它才会被实例化
	- 单例：每个依赖services的component都会得到由service factory产生的service单实例的一个引用（service都是单例的，只要创建了一个Service，那么程序都在使用这唯一的Service)
	- 以$开头的，都是内置服务，eg：$http
	- service可以被用来传递数据、实现代码复用
- 创建并使用：通过service工厂函数来创建（factory函数）
``` javascript
  angular.module('myServiceModule', []). controller('MyController', ['$scope', 'notify', function($scope, notifyInstance) {//3.这里notyfy被依赖，所以立马被实例化并将2处的匿名函数赋值给了notifyInstance
   $scope.callNotify = function(msg) {
     notifyInstance (msg);
   };
 }])
.factory('notify', ['$window', function(win) {// 1.这里注册了一个notify的服务，而且还依赖另一个内置服务$winodw，注意这里还只是注册一个notify的构造函数，并没有创建notify的实例
   var msgs = [];
   return function(msg) {// 2.return里的是当服务被加载(依赖时)返回的实例对象或函数
     msgs.push(msg);
     if (msgs.length === 3) {
       win.alert(msgs.join('\n'));
       msgs = [];
     }
   };
 }]);
```

## Scope
- 特点
    - 它只是一个普通的js对象
    - 它指向了应用的model
	- 它是表达式的执行环境context
	- 它拥有和DOM一样的树形结构
	- 它能监视表达式
	- 它能传播事件
	- 它提供了$watch来观察模型的变化
	- 它提供了$apply来传播模型的变化
	- 它可以嵌套以限制对应用程序组件属性的访问，同时提供对共享模型属性的访问。
	- 它是controller和view间的胶水
- 层次结构（类似DOM的树形结构）
	- 每个angularApp都有一个根scope，$rootScope，$rootScope有一个或多个子scope
	- 查找某属性时，会像js作用域一样，逐层向上找
	- angularJs会在每个绑定了scope元素的class上添加ng-scope
	- directive可以创建scope
- 获取DOM元素上绑定的scope
    - 可以通过angular.element(dom元素).scope()
    - 在chrome中，也可以通过 angular.element($0).scope()或者在选中一个dom元素后直接在控制台中$scope，就能得到相应dom上的$scope
	- 可以通过安装 AngularJS Batarang插件来查看
- 事件传播
	- $emit(eventName) 向上传播事件
	- $broadcast(eventName) 向下传播事件
- 生命周期： Creation-> Watcher registration-> Model mutation-> Mutation observation-> Scope destruction

## DependencyInjection(DI)
- 使用
    - services、directives、filter、animation可以将"services"、"value"型组件作为依赖注入
    ```javascript
    angular.module('myModule', []).factory('serviceId', ['depService', function(depService) {
      // ...
    }])
    .directive('directiveName', ['depService', function(depService) {
      // ...
    }])
    .filter('filterName', ['depService', function(depService) {
      // ...
    }]);
    ```
    - controller可以将"services"、"value"型组件作为依赖注入，但他们还可以注入一些特殊的依赖如$scope
    ```javascript
    someModule.controller('MyController', ['$scope', 'dep1', 'dep2', function($scope, dep1, dep2) {
    ...
    $scope.aMethod = function() {
    ...
    }
    ...
    }]);
    ```
    - 为module提供run、config方法时，可以使用DI
        - config接收一个函数，函数可以注入"provider"、"constant"型组件；不可将"services"、"value"型注入到config中
        - run接收一个函数，函数可以注入"services"、"value"以及"constant"(常数)型组件；不可将"providers"型注入到run中
        ```javascript
        angular.module('myModule', []).config(['depProvider', function(depProvider) {
        // ...
        }]).run(['depService', function(depService) {
        // ...
        }]);
        ```
- 依赖声明
    - 行内数组声明(推荐、最优)
    ```javascript
    someModule.controller('MyController', ['$scope', 'greeter', function($scope, greeter) {
    // ...
    }]);
    ```
    - 使用$inject(可让controller通过js压缩)
    ```javascript
    var MyController = function($scope, greeter) {
    // ...
    }
    MyController.$inject = ['$scope', 'greeter'];
    someModule.controller('MyController', MyController);
    ```
    - 隐式声明依赖(压缩时，会出错)->尽量避免用此方法
    ```javascript
    someModule.controller('MyController', function($scope, greeter) {
    // ...
    });
    ```
    - 通过在ng-app指令所在html元素上添加 ng-strict-di指令，以限制隐式声明的使用(若使用隐式声明 ，会报错)

## Templates
- angularJS中的template是由html以及angularJs声明的元素及特性组成。angularJs通过controller组合model中的信息和模板以呈现动态的view给用户
- templates中可以使用
	- directive
	- {% raw %}{{}}{% endraw %}
	- filter
	- form controller
    ```html
    <html ng-app>
     <!-- Body tag augmented with ngController directive  -->
     <body ng-controller="MyController">
       <input ng-model="foo" value="bar">
       <!-- Button tag with ngClick directive, and
              string expression 'buttonText'
              wrapped in "{{ }}" markup -->
       <button ng-click="changeFoo()">{{buttonText}}</button>
       <script src="angular.js"></script>
     </body>
    </html>
    ```
    - 在复杂的app中,可以将不同的模板放在单独的html中，然后通过ng-view来引用

## Expressions
- 主要用在插值绑定(interpolation bindings)中，但也可以直接用在指令中；如ng-click="functionExpression()"；以下都是合法的
	- 1+2
	- a+b
	- user.name
	- items[index]
- 和Js的表达式的异同
	- context：js中的context一般是全局变量window；angular中表达式的context则是scope
	- js中尝试计算未定义的属性，会报错；angular则不会直接报错，而是转为undefined或null
	- 在angular表达式中可以用filter在展示数据前进行数据格式化
	- 在angular的表达式中没有条件控制相关语句；如if、for等，但三元操作符中可以用
	- 在angular的表达式中不能有函数声明，及时在ng-init指令中
	- 在angular的表达式中不能创建正则表达式
	- 在angular的表达式中不能通过new创建对象
	- 在angular的表达式中不能使用位运算、void、逗号等操作符
	- 如果想解析angularJs表达式，不要用eval，使用$eval
	- 总结：如果想使用复杂js代码，可以将其封装在controller中，然后在view中调用。不推荐直接在表达式中书写大量代码；
- $event
    - 在执行ng-click、ng-focus等指令时，会在表达式范围内将$event暴露出来，$event是类似jquery Event的对象
- one-time绑定(数据只绑定一次)
    - 优势：只绑定一次，可减少监视次数
    - 使用：在变量前添加双冒号；类似{% raw %}{{::name}}{% endraw %}
    - 何时( 当表达式被设定后，就不会被改变时 )
        - 用在插值文本和特性时
        ```javascript
        <div name="attr: {{::color}}">text: {{::name | uppercase}}</div>
        ```
        - 当用directive双向绑定数据并且参数不会改变时
        ```javascript
        someModule.directive('someDirective', function() {
          return {
            scope: {
              name: '=',
              color: '@'
            },
            template: '{{name}}: {{color}}'
          };
        });
        
        <div some-directive name="::myName" color="My color is {{::myColor}}"></div>
        ```
        - 指令中包含表达式时
        ```javascript
        <ul>
          <li ng-repeat="item in ::items | orderBy:'name'">{{item.name}};</li>
        </ul>
        ```

## Interpolation
- 针对布尔attr，如disabled、required、selected、checked、readOnly、open，不要使用原生的，使用ng-disabled、ng-required...
- 使用ng-attr-xxx绑定任意特性，如ng-attr-cx；若为驼峰形式，则用下划线代替，如viewBox，则使用ng-attr-view_box

## Filters
- 在view中的语法
    - 正常语法
    ```javascript
    {{ expression | filterName }}
    ```
    - Filter Chain->用上一个filter的输出作为下一个filter的输入
    ```javascript
    {{ expression | filter1Name | filter2Name | ... }}
    ```
    - 带参数
    ```javascript
    {{ expression | filterName:argument1:argument2:... }}
    如 {{ 1234 | number:2 }}
    ```
- 当filter用在controller、services、directives上时，需采用<filterName>Filter形式
    - 需要在controller中使用number过滤器时
    ```javascript
    angular.module('numberFilterExample', [])
    .controller('ExampleController', ['numberFilter', function(numFilter) {
      // ....
    }]);
    ```
- 创建自定义filter
    - 使用filter函数
    ```javascript
    angular.module('myReverseFilterApp', []).filter('reverse', function() {
      return function(input) {// return 一个函数
        input = input || '';
        var out = '';
        for (var i = 0; i < input.length; i++) {
          out = input.charAt(i) + out;
        }
        return out;
      };
    })
    // html中直接
    {{ data | reverse}}
    ```
- 常用filter
    - date日期格式-> {% raw %}{{ now | date:'yyyy-MM-dd hh:mm:ss a' }}{% endraw %}
    - currency货币格式化
    - fiter对数组、字符串、对象等进行筛选显示
    ```javascript
    $scope.city = [{
            id: "001",
            name: "上海"
        }, {     
            id: "002",
            name: "北京"
    }];
    // view
     {{city}}  
     {{city|filter:'上海'}}// 默认筛选出所有value值为'上海'的object
     {{city|filter:{name:'北京'} }}// 筛选出name为北京的object
    ```

    ![](filter.png)

    - orderBy排序
        - {% raw %}{{city |orderBy:'id'}}{% endraw %} 默认正序
        - {% raw %}{{city |orderBy:'-id'}}{% endraw %} 反序
    - json
        - 将对象解析成json，主要用来调试

## Forms
- angularJs对表单域做了增强，添加了很多功能
- 使用ng-model就可以将表单的值和model进行双向绑定
- 使用novalidate屏蔽浏览器原生验证
- angualrJs会添加一些class类，来标识验证的状态，根据这些验证状态class类，来写不同的样式
	- ng-valid：model验证通过
	- ng-invalid：model未验证通过
	- ng-valid-[ruleName]：ruleName的验证规则已通过
	- ng-invalid-[ruleName ]：ruleName的验证规则未通过
	- ng-pristine：这个表单域还没有交互过(未修改过)
	- ng-dirty：这个表单域已经交互过(修改过)
	- ng-touched：这个表单域 失去焦点
	- ng-untouched：这个表单域未失去焦点
	- ng-pending：异步验证还未完成
- 可以根据表单验证的一些状态，来辅助添加帮助信息
- 通过ng-model-options来设置一些属性
    - ng-model-options="{ updateOn: 'blur' }" 在blur时更新model
- 延时更新model
    - ng-model-options="{ debounce: 500 }"
    - ng-model-options="{ updateOn: 'default blur', debounce: { default: 500, blur: 0 } }"
- 可通过编写directive来创建自己的验证规则、表单域
- 可以修改内置的验证规则
- 相关状态
	- 字段错误信息->formName.fieldName.$error->验证通过的规则会显示false，未通过的显示true
	- 字段无效信息->formName.fieldName.$invalid
	- 字段有效信息->formName.fieldName.$valid
	- 字段是否更改->formName.fieldName.$dirty
	- 字段是否未更改->formName.fieldName.$pristine
- $scope.formName.$setPristine->将表单恢复到最初状态，class、$dirty等都被恢复

## Directives
- 匹配
    ```html
    <div ng-controller="Controller"><!-- 下面这几种形式，都将input和model中name绑定起来了，绑定这个指令就匹配上了 -->
      Hello <input ng-model='name'> <hr/><!-- 建议形式 -->
      <span ng-bind="name"></span> <br/><!-- 建议形式 --> 
      <span ng:bind="name"></span> <br/>
      <span ng_bind="name"></span> <br/>
      <span data-ng-bind="name"></span> <br/>
      <span x-ng-bind="name"></span> <br/>
    </div>
    ```
- ng-attr-xxx->所有ng-attr开头的特性，最后都会转化到原生的特性上，建议不要在原生的特性上绑定值，都用ng-attr开头(因为某些原生特性和ng配合的不好)
    ```html
    <svg>
      <circle ng-attr-cx="{{cx}}"></circle><!-- 将{{cx}}绑定到了原生的cx上 -->
    </svg>
    ```
- 指令类型
    - A（attribute）、E（element）、M（comment）、C（class）;M和C不常用，如果需要兼容IE8，建议全部用A
    ```html
    <my-dir></my-dir>
    <span my-dir="exp"></span>
    <!-- directive: my-dir exp -->
    <span class="my-dir: exp;"></span>
    ```
    - 注意：由于历史原因，浏览器在解析html标签和标签attribute时，会自动忽略大小写，统一使用小写形式；这就导致了，用驼峰形式定义的html标签（`<myTag>`）和特性会被转换为全小写（`<mytag>`）；那么用驼峰形式定义的指令在匹配E和A时，就找不到（无法匹配），为了解决这问题，ng会在定义时用的驼峰形式directive("myTag",xxx)转换成my-tag，这样在html中my-tag形式的标签和特性就会被匹配到。如果定义时没用驼峰形式(全小写)，则不会转换，直接匹配。驼峰形式的指令名在匹配M和C时不会存在转换->总结:在定义指令时如果用了驼峰形式匹配EA，则在html中使用时就要转换成短横线连接的形式如myTag转换成my-tag->最佳实践：ng中指令若用驼峰则html中用短横线连接；
- 创建指令
    - module.directive(directiveName,fn);
    ```javascript
    angular.module('docsSimpleDirective', []).controller('Controller', ['$scope', function($scope) {
      $scope.customer = {
        name: 'Naomi',
        address: '1600 Amphitheatre'
      };}])
    .directive('myCustomer', function() {
      return {// 尽量都返回一个object，不要只返回一个函数
        template: 'Name: {{customer.name}} Address: {{customer.address}}'    // 除非你的模板很小，否则使用templateUrl来引入外部单独的模板html文件
      };
    });
    ```
    - templateUrl->除非你的模板很小，否则使用templateUrl来引入外部单独的模板html文件
        - 当replace为true时，tpl文件内容必须被包裹在一个标签内，也即tpl文件只能有一个根标签；即不能存在有文本未被标签包裹，也不能存在多个根标签；因为替换的时候ng找不到一个唯一的节点做为替换节点，所以必须得有一个最外层的根节点；template也存在同样情况->最佳实践，任何情况下，都让模板文件包裹在一个根标签中，这样也方便文件的组织管理
        - 可以在模板中使用$scope中的变量
        - templateUrl中可以指定type=text/ng-template的script为模板，只需要在templateUrl中写上script模板的id；注意：这个script模板必须在ng-app中，而且，若replact为true，则也需要一个根标签
        ```javascript
        angular.module('docsTemplateUrlDirective', []).controller('Controller', ['$scope', function($scope) {
          $scope.customer = {
            name: 'Naomi',
            address: '1600 Amphitheatre'
          };
        }])
        .directive('myCustomer', function() {
          return {
            templateUrl: 'my-customer.html'
          };
        });
        ```
    - restrict->设置指令的匹配模式(AEMC)；默认是AE（匹配attribute、element类型指令)；M和C不常用
    ```javascript
    angular.module('docsRestrictDirective', []).controller('Controller', ['$scope', function($scope) {
      $scope.customer = {
        name: 'Naomi',
        address: '1600 Amphitheatre'
      };
    }])
    .directive('myCustomer', function() {
      return {
        restrict: 'E',
        templateUrl: 'my-customer.html'
      };
    });
    // 什么情况下该用元素名，什么情况下该用属性名？ 当创建一个含有自己模板的组件的时候，建议使用元素名，常见情况是，当你想为你的模板创建一个DSL（特定领域语言）的时候。如果仅仅想为已有的元素添加功能，建议使用属性名.
    // 当需要创建一个自己的组件时->创建E型指令
    // 为已有元素添加新功能->创建A型指令
    ```
    - isolate scope
        - 存在原因：若无独立作用域，则在一个作用域下，多个指令无法独立执行；使用独立作用域，可以将指令限制在独立的作用域下执行，互不干扰
        - 创建：在创建directive时指定scope属性
        ```javascript
        angular.module('docsIsolateScopeDirective', []).controller('Controller', ['$scope', function($scope) {
          $scope.naomi = { name: 'Naomi', address: '1600 Amphitheatre' };
          $scope.igor = { name: 'Igor', address: '123 Somewhere' };}]).directive('myCustomer', function() {
          return {
            restrict: 'E',
            scope: {
              customerInfo: '=info' // 将customerInfo绑定到指令所在元素的info特性上，如果外面的特性也叫customerInfo，则可以直接使用缩写形式"="
            },
            templateUrl: 'my-customer-iso.html'
          };
        });
        ```
        - 独立作用域会隔离除你添加到scope: {} 对象中的数据模型之外的一切东西。 因为它可以阻止除你传入的数据模型之外的一切东西改变你内部数据模型的状态。
        - 如果要使你的组件在应用范围内可重用，那么使用scope选项去创建一个独立作用域
        ```html
        <!DOCTYPE html>
        <html lang="en" ng-app="myApp">
        
        <head>
            <meta charset="UTF-8">
            <title>Document</title>
        </head>
        
        <body>
            <div ng-controller="myController">
                {{books}}
                <div book-list book-a="books" book-b="books" book-c="{{title}}"></div>
            </div>
            <script type="text/javascript" src="js/angularjs.js"></script>
            <script>
            angular.module('myApp', []).directive("bookList", function() {
                return {
                    restrict: "EAMC",
                    template: '<div><h1>{{title}}</h1><ul><li ng-repeat="book in books">{{book.name}}</li></ul> </div>', // bookList指令中包含一个booAdd指令
                    replace: true,
                    // scope: false,// scope为独立作用域，当为false，表示直接使用父级作用域，为true，表示创建一个作用域并继承自父作用域；当scope为一个对象时，则表示创建了一个不继承父作用域的继承链的独立作用域(就是可以访问到父作用域，但是无法访问到父作用域之上的作用域)
                    scope: {
                        // &attr表示作用域将父作用域的属性包装成一个函数，从而以函数的形式读写父作用域的属性；一般用在执行父作用域上的某个事件处理函数；若作用域和父作用域的属性名称一要，则可以使用简写形式&,@和=同理
                        a: "&bookA" // 会查找当前指令匹配的元素上的bookA特性，然后取得值books，并将books做为a调用的返回值进行返回；
        
        
                        // =attr会将作用域上的属性和父级的作用域上的属性进行双向绑定，任何一方的修改都会修改另外一方
                        // b: "=bookB" // 会查找当前指令匹配的元素上的bookB特性，会将其值和b进行双向绑定
        
                        // @attr代表只能读取父级作用域上的值，单向的，并只能读取简单值，引用值不行，因为他最终得到的只会是简单值；
                        // c: "@bookC"
                    },
                    controller: function($scope) {
                        $scope.books = $scope.a();
                        console.log($scope.a());
        
                        // $scope.books = $scope.b;
                        // $scope.b.push({
                        //     name: "nodeJs"
                        // });
                        // console.log($scope.b);
        
                        // $scope.title = $scope.c;
                        // console.log($scope.c);
                    },
        
        
                }
            }).controller('myController', ['$scope', function($scope) {
                $scope.books = [{
                    name: "php"
                }, {
                    name: "js"
                }, {
                    name: "java"
                }];
        
                $scope.title = "书籍";
            }]);
            </script>
        </body>
        </html>
        ```
    - compile
        - 主要用在DOM渲染之前( link之前) 改变DOM结构，并不需要$scope参数。它必须返回一个link函数，因此如果指令中compile和link都写了，则link会被覆盖->用的比较少
        ```html
        <!DOCTYPE html>
        <html lang="en" ng-app="myApp">
        
        <head>
            <meta charset="UTF-8">
            <title>Document</title>
        </head>
        
        <body>
            <div ng-controller="myController">
                <div ng-repeat="user in users" my-tag my-tag2></div>
            </div>
            <script type="text/javascript" src="js/angularjs.js"></script>
            <script>
            angular.module('myApp', []).directive('myTag', function() {
                return {
                    restrict: "EAMC",
                    template: '<div>{{user.name}}</div>',
                    replace: true,
                    compile: function(tElement, tAttrs, transclude) { // 主要用来在实际渲染之前修改DOM结构，compile必须返回一个link函数
                        // console.log(tElement); //返回匹配的当前类jQuery对象
                        // console.log(tAttrs); // 返回tElement上的所有attr
                        // console.log(transclude); // 如果指令中transclue为true，则它返回的就是被transclude的原始数据
                        console.log("myTag 编译阶段");
                        // 在实际渲染前变更DOM结构
                        tElement.append(angular.element("<h1>test</h1>"));
                        return { // 若在compile中直接return一个函数，则返回的是postLink函数
                            pre: function(scope, iElement, iAttrs, controller) { // preLink是在compile阶段结束后，link阶段之前触发
                                console.log("myTag preLink");
                            },
                            post: function(scope, iElement, iAttrs, controller) { // postLink是指令link后触发
                                console.log("myTag postLink");
                            }
                        }
                    },
                    link: function(scope, iElement, iAttrs, controller) { // 主要在link中进行绑定事件和操纵DOM；一般定义了compile，就不会定义link了;此处的link其实就是compile中postLink
                        console.log("因为上面执行了，compile，所以我不会再被执行了");
                    }
                }
            }).directive('myTag2', function() {
                return {
                    restrict: "EAMC",
                    compile: function(tElement, tAttrs, transclude) {
                        console.log("myTag2 编译阶段");
                        return {
                            pre: function() {
                                console.log("myTag2 preLink");
                            },
                            post: function() {
                                console.log("myTag2 postLink");
                            }
                        }
                    }
                }
            }).controller("myController", ["$scope", function($scope) {
                $scope.users = [{
                    id: 10,
                    name: "张三"
                }, {
                    id: 20,
                    name: "李四"
                }];
            }]);
            </script>
        </body>     
        </html>
        ```
    - link
        - 主要在这里来操作DOM和添加事件
        - scope->指令所处的作用域(如果有独立作用域，则为独立作用域，否则值为父级的作用域);
        - element->指令所匹配的那个元素
        - attrs->指令匹配元素的所有特性的集合
        - controller->指令需要依赖的controller实例
    	```javascript
        angular.module('docsTimeDirective', []).controller('Controller', ['$scope', function($scope) {
          $scope.format = 'M/d/yy h:mm:ss a';}]).directive('myCurrentTime', ['$interval', 'dateFilter', function($interval, dateFilter) {
        
          function link(scope, element, attrs) {
            var format,
                timeoutId;
        
            function updateTime() {
              element.text(dateFilter(new Date(), format));
            }
        
            scope.$watch(attrs.myCurrentTime, function(value) {
              format = value;
              updateTime();
            });
        
            element.on('$destroy', function() {
              $interval.cancel(timeoutId);
            });
        
            // start the UI update process; save the timeoutId for canceling
            timeoutId = $interval(function() {
              updateTime(); // update DOM
            }, 1000);
          }
        
          return {
            link: link
          };
        }]);
        ```
    - replace->是否替换匹配的元素
        - 若为true则在找到匹配的元素后，会用指令中的template内容替换匹配的内容(包括被匹配的元素)；默认情况下，指令会在找到匹配的元素时，会将匹配元素的内容替换为指令中template的内容；
        - 一般在匹配E时，会选择将其设置为true，因为一般E型指令都是创建新标签，是不符合规范的，所以会选择将其替换
    - transclude->主要用来处理指令嵌套
        - 默认情况下，指令会替换匹配元素内部的内容，这样就无法实现指令的相互嵌套使用(原指令的内容会被新指令全部替换掉)；
        - 当设置transclude为true时，则可以保留原先的指令模板以及对应的作用域；注意，在新指令的模板中要用ng-transclude保留老指令的内容；
        ```javascript
        // js
        angular.module('docsTransclusionDirective', []).controller('Controller', ['$scope', function($scope) {
          $scope.name = 'Tobias';}]).directive('myDialog', function() {
          return {
            restrict: 'E',
            transclude: true,
            scope: {},
            templateUrl: 'my-dialog.html'
          };
        });
        // my-dialog.html
        <h1>新内容</h1><div class="alert" ng-transclude></div> <!-- 指定ng-transclude -->
        ```
        - 仅当你要创建一个包裹任意内容的指令的时候使用transclude: true
        - 创建一个包裹任意内容的dialogBox
        ```javascript
        // js
        angular.module('docsIsoFnBindExample', []).controller('Controller', ['$scope', '$timeout', function($scope, $timeout) {
          $scope.name = 'Tobias';
          $scope.message = '';
          $scope.hideDialog = function(message) {
            $scope.message = message;
            $scope.dialogIsHidden = true;
            $timeout(function() {
              $scope.message = '';
              $scope.dialogIsHidden = false;
            }, 2000);
          };
        }])
        .directive('myDialog', function() {
          return {
            restrict: 'E',
            transclude: true,
            scope: {
              'close': '&onClose' // &prop 用来绑定一个函数到独立作用域，允许独立作用域调用它，同时保留了函数的原来作用域；当你的指令想要开放一个API去绑定特定的行为，在scope选项中使用&prop。
            },
            templateUrl: 'my-dialog-close.html'
          };
        });
        
        // html
        <div ng-controller="Controller">
          {{message}}
          <my-dialog ng-hide="dialogIsHidden" on-close="hideDialog(message)">
            Check out the contents, {{name}}!
          </my-dialog>
        </div>
        
        // my-dialog-close.html
        <div class="alert">
          <a href class="close" ng-click="close({message: 'closing for now'})">&times;</a>
          <div ng-transclude></div> <!-- 保留原先指令内容 -->
        </div>
        ```
        - 创建添加事件的指令
        ```javascript
        angular.module('dragModule', []).directive('myDraggable', ['$document', function($document) {
          return {
            link: function(scope, element, attr) { // 在link中为指令匹配的元素element绑定事件
              var startX = 0, startY = 0, x = 0, y = 0;
        
              element.css({
               position: 'relative',
               border: '1px solid red',
               backgroundColor: 'lightgrey',
               cursor: 'pointer'
              });
        
              element.on('mousedown', function(event) {
                // Prevent default dragging of selected content
                event.preventDefault();
                startX = event.pageX - x;
                startY = event.pageY - y;
                $document.on('mousemove', mousemove);
                $document.on('mouseup', mouseup);
              });
        
              function mousemove(event) {
                y = event.pageY - startY;
                x = event.pageX - startX;
                element.css({
                  top: y + 'px',
                  left:  x + 'px'
                });
              }
        
              function mouseup() {
                $document.off('mousemove', mousemove);
                $document.off('mouseup', mouseup);
              }
            }
          };
        }]);
        ```
        - controller->指令中的controller属性可以用来完成指令间的相互通信
        ```javascript
        // js
        angular.module('docsTabsExample', [])
        .directive('myTabs', function() {
          return {
            restrict: 'E',
            transclude: true,
            scope: {},
            controller: ['$scope', function MyTabsController($scope) {
              var panes = $scope.panes = [];
        
              $scope.select = function(pane) {
                angular.forEach(panes, function(pane) {
                  pane.selected = false;
                });
                pane.selected = true;
              };
        
              this.addPane = function(pane) {// 这个方法需要暴露给其他指令用
                if (panes.length === 0) {
                  $scope.select(pane);
                }
                panes.push(pane);
              };
            }],
            templateUrl: 'my-tabs.html'
          };
        }).directive('myPane', function() {
          return {
            require: '^^myTabs', // 依赖一个myTabs控制器，并在指令的父元素上查找这个控制器
            restrict: 'E',
            transclude: true,
            scope: {
              title: '@' // 相当于@title
            },
            link: function(scope, element, attrs, tabsCtrl) {
              tabsCtrl.addPane(scope);
            },
            templateUrl: 'my-pane.html'
          };
        });
        // index.html
        <my-tabs>
          <my-pane title="Hello">
            <p>Lorem ipsum dolor sit amet</p>
          </my-pane>
          <my-pane title="World">
            <em>Mauris elementum elementum enim at suscipit.</em>
            <p><a href ng-click="i = i + 1">counter: {{i || 0}}</a></p>
          </my-pane>
        </my-tabs>
        // tabs
        <div class="tabbable">
          <ul class="nav nav-tabs">
            <li ng-repeat="pane in panes" ng-class="{active:pane.selected}">
              <a href="" ng-click="select(pane)">{{pane.title}}</a>
            </li>
          </ul>
          <div class="tab-content" ng-transclude></div>
        </div>
        // panel
        <div class="tab-pane" ng-show="selected">
          <h4>{{title}}</h4>
          <div ng-transclude></div>
        </div>
        ```
        - 当你想暴露一个API给其它的指令调用那就用controller,否则用link。
        - controllerAs->给指令中的controller起个别名，并可做controller中的第四个参数传入
        - priority->设置指令执行的优先级顺序(权重)；多个指令时，ng必须知道哪个先执行；默认ng-repeat优先级很高，为1000；->不常用
        - terminal->是否设置当前指令的权重priority 为结束界限；若为true，则节点上小于当前指令权重priority的指令不会被执行，相同权重的会执行
        - require->可以将其他指令传给自己，有以下值
            - directiveName->默认值，会从同一个元素上查找
            - ^directiveName->会在父级上查找
            - ?directiveName->表示指令是可选的，找不到也不会抛出异常

## Animations
- 可参考https://css-tricks.com/animations-the-angular-way/
- 无需引入任何模块，直接利用切换class配合css的过渡，来实现过渡动画->无法做到全兼容；->https://codepen.io/bdsimmons/pen/NqYjaV
- angualr中内置了$animate服务，可以提供简单的动画操作，enter、leave...->核心方法，ngAnimate模块也依赖这个核心服务
- 更强大的动画->引入ngAnimate模块；angualr没有直接包含动画模块(ngAnimate)，需要在引入angular.js后引入angular-animate.js
    - 重点：ngAnimate动画的核心都是基于css，通过变换元素的class类配合过渡和animation进而实现动画，js动画除外
    - 如何使用ngAnimate模块
        - 整个app没有模块，则可以直接指定ng-app="ngAnimate"来从ngAnimate模块启动app，也能有动画效果；->不推荐->http://www.runoob.com/try/try.php?filename=try_ng_animation
        - 当做模块依赖来使用->var app=angular.module("myApp",["ngAnimate"]);
            当引入ngAnimate模块后，就会自动在一些指令执行的特殊时机，为元素添加上对应的class类，可以利用这个配合css实现动画
            ![](Animate.png)
            - css过渡动画->需要设置动画的起点、终点的动画属性值；例如在.ng-enter上设置过渡动画初始值，在.ng-enter-active上设置过渡动画终点值->https://codepen.io/bdsimmons/pen/OPmNxXs
            - css3Animation->css3Animtions，无需在2个class上设置动画，只需要在一个class上设置动画，并给定动画时间即可，所以如上面的只需要在.ng-enter上设置一个animation动画即可，无需在ng-enter-active在设置动画->https://codepen.io/anon/pen/NjJLMZ
            - JS动画->当引入ngAnimate模块后 就自动在app上添加了animation方法，app.animation()；可以通过animation方法，实现动画->https://codepen.io/bdsimmons/pen/YXLZEw
            ![](Animate2.png)

## Module
- 基本用法
    - 创建模块->用angular.module("moduleName",["依赖的模块"]);
    - 获取已有模块->用angular.module("moduleName");注意获取时，没有后面的依赖数组
- 让模块运作起来
    - 声明一个module,然后在ng-app中引用它，即可让app从module中开始运行
- 模块划分
    - 服务模块
    - 指令模块
    - 过滤器模块
    - 一个应用的模块，依赖于上述的三个模块，而且包含应用的初始化及启动代码
    ```javascript
    angular.module('xmpl.service', []) // 服务
      .value('greeter', {
        salutation: 'Hello',
        localize: function(localization) {
          this.salutation = localization.salutation;
        },
        greet: function(name) {
          return this.salutation + ' ' + name + '!';
        }
      })
    
      .value('user', {
        load: function(name) {
          this.name = name;
        }
      });
    
    angular.module('xmpl.directive', []); // directive
    
    angular.module('xmpl.filter', []);// filter
    
    angular.module('xmpl', ['xmpl.service', 'xmpl.directive', 'xmpl.filter']) // xmpl依赖service、directive、filter
    
      .run(function(greeter, user) { // 初始化
        // This is effectively part of the main method initialization code
        greeter.localize({
          salutation: 'Bonjour'
        });
        user.load('World');
      })
    
      .controller('XmplController', function($scope, greeter, user){
        $scope.greeting = greeter.greet(user.name);
      });
    ```
- 模块是配置代码块和运行代码块的集合
    - 配置代码块config->在 provider 注册和配置阶段执行（注：provider 是 ng 服务的一种）。只有 provider 和 constant 可以被注入配置代码块。这是为了防止服务在完全配置好之前被意外地初始化。->config为配置
    - 执行代码块run->在 injector 被创建后执行，被用来启动整个应用。只有服务的实例对象以及 constant 可以被注入到执行代码块。这是为了防止在应用执行期间系统的更进一步的配置。->run为初始化
    ```javascript
    angular.module('myModule', []).
      config(function(injectables) { // provider型注入器
        // 这是配置(config)代码块的范例，你可以有任意多个配置代码块
        // 配置块中你只能注入Provider类（注意：不是由Provider类生成的实例）以及`constant`
      }).
      run(function(injectables) { // instance型注入器
        // 这是运行(run)代码块的范例，你可以有任意个运行代码块
        // 运行块中你只能注入Provider实例（注意：不是Provider类）
      });
    ```
    - 配置代码块的快捷方法
    ```javascript
    angular.module('myModule', []).
    value('a', 123).
    factory('a', function() { return 123; }).
    directive('directiveName', ...).
    filter('filterName', ...);
    
    // 等同于
    
    angular.module('myModule', []).
    config(function($provide, $compileProvider, $filterProvider) {
      $provide.value('a', 123);
      $provide.factory('a', function() { return 123; });
      $compileProvider.directive('directiveName', ...);
      $filterProvider.register('filterName', ...);
    });
    
    //.config等同于设置module函数的第三个参数
    
    angular.module("myModule",[],["$provide","$compileProvider","$filterProvier",function(provide,compileProvier,filterProvider){
      provide.value('a', 123);
      provide.factory('a', function() { return 123; });
      compileProvider.directive('directiveName', ...);
      filterProvider.register('filterName', ...);  
    }]);
    ```
    - 配置语句的执行顺序就是根据它们注册的顺序而定的。唯一的例外是 constant 的定义，它会被调整到所有配置块的最前面执行。
    - 执行代码块
        - 执行代码块是 ng 中最接近 main 函数的一个东西。执行代码块是应用启动时运行的代码。它在所有的服务被配置好以及 注入器(injector)被创建好之后执行。通常，执行代码块包含的代码都很难进行单元测试，正因为如此，它通常应该被丢在一个单独的模块中，这样我们可以在单元测试时忽略它。
- 模块依赖
    - A依赖B，则A的配置阶段要在B的配置阶段完成后进行，执行阶段同理，A的执行要在B的执行结束后。
    - 注意每个模块只能被加载一次，即使有多个别的模块依赖它。

## IE兼容性
- 1.3及以上不再支持IE8->所以如果需要支持IE8，请使用ng1.2.x->1.2的最新版本为1.2.32
- IE7及以下不支持JSON.stringify->使用json2.js
    ```html
    <!--[if lte IE 7]>
      <script src="/path/to/json2.js"></script>
    <![endif]-->
    ```
- 在根元素上添加id="ng-app"并结合ng-app="xxModule"来启动app
    ```html
    <!doctype html>
    <html xmlns:ng="http://angularjs.org" id="ng-app" ng-app="optionalModuleName">
      ...
    </html>
    ```
- 不要使用自定义节点 如<ng:view>，用attribute方式代替如ng-view
- 如果你由于语义或者第三方的Angular组件需要使用tag的方式的话,那么你必须按照如下步骤 make IE happy
    ```html
    <!doctype html>
    <html xmlns:ng="http://angularjs.org" id="ng-app" ng-app="optionalModuleName">
    <head>
    <!--[if lte IE 8]>
      <script>
        document.createElement('ng-include');
        document.createElement('ng-pluralize');
        document.createElement('ng-view');
    
        // Optionally these for CSS
        document.createElement('ng:include');
        document.createElement('ng:pluralize');
        document.createElement('ng:view');
      </script>
    <![endif]-->
    </head>
    <body>
    ...
    </body>
    </html>
    ```
    - 重要的部分:
        - xmlns:ng- 命名空间 - 你需要为每一个将使用的自定义tag注册一个命名空间(译者注:IE作为严格xml模式解析).
        - document.createElement(yourTagName) - 自定义节点创建 - 由于这只是老版本的IE issues，所以你需要按条件加载这些脚本(IE低版本特有的条件注释)。对于每一个需要使用的没有注册命名空间以及非HTML定义的tag你需要利用它来预申明来make IE happy。
- IE在处理关于非标准HTML tag 的问题主要由两类，每种类型又其自己的修复方式.
    - If the tag name starts with my: prefix then it is considered an XML namespace and must have corresponding namespace declaration on <html xmlns:my="ignored">
    - 以`my:`为前缀的tag 考虑到严格的XML命名空间，你必须有相应的命名空间申明,如<html xmlns:my="ignored">。
    - If the tag has no : but it is not a standard HTML tag, then it must be pre-created using document.createElement('my-tag')
    - 没有`:`的非标准HTML tag, 你需要使用`document.createElement('my-tag')`来预申明改节点(译者注:ie-shv)。
    - If you are planning on styling the custom tag with CSS selectors, then it must be pre-created using `document.createElement('my-tag')` regardless of XML namespace.
    - 如果你希望采用CSS选择器的方式，那么你需要使用`document.createElement('my-tag')`预申明，忽略XML命名空间。
- 使用ng-style代替style={{ someCss }};后者在<IE11的版本上无法运行

## Angualr会在执行的某些时间点为标签添加上一些标识用的class类
- ng-scope样式类会在创建了新作用域(Scope)的HTML元素上生成
- ng-binding样式类会在ng-bind 或 绑定了任何数据的元素上生成
- ng-invalid、ng-valid样式类会在进行了验证操作的所有input组件元素上生成
- ng-pristine、ng-dirty
	- angular的input指令给所有新的、还没有与用户交互的input元素附加上ng-pristine类，当用户有任何输入时，则附加上 ng-dirty.

## 国际化I18n和本地化L10n
- 引入特定的语言包
    ```html
    <html ng-app>
    <head>
    ….
      <script src="angular.js"></script>
      <script src="i18n/angular-locale_zh-cn.js"></script>
    ….
    </head>
    </html>
    ```
  
## 启动即bootstrap(这里bootstrap并非指ui库，bootstrap本身就有启动的意思)
- 自动启动
    - 在angular要控制的范围最外层元素上添加ng-app指令
        - 若未指定ng-app的具体值，会从默认模块开始启动；若指定了则从指定模块上启动
        - 若需支持IE7，则还上在ng-app除添加id="ng-app"
        - 若需支持老式ng:风格的指令，则还需要在html上添加xml命名空间
            ```html
            <html xmlns:ng="http://angularjs.org">
            ```
- 手动启动
    - 使用angular.bootstrap
        ```javascript
        <!doctype html>
        <html>
        <body>
          <div ng-controller="MyController">
            Hello {{greetMe}}!
          </div>
          <script src="http://code.angularjs.org/snapshot/angular.js"></script>
        
          <script>
            angular.module('myApp', [])
              .controller('MyController', ['$scope', function ($scope) {
                $scope.greetMe = 'World';
              }]);
        
            angular.element(document).ready(function() {
              angular.bootstrap(document, ['myApp']); // 手动启动
            });
          </script>
        </body>
        </html>
        ```
    - bootstrap要在需要的module创建或加载完成后，才能调用
    - 当手动启动app时，就不应当在用ng-app指令了

## 安全性
- 不要混用前台和后台的模板
- 不要使用用户的输入动态生成模板
- Do not run user input through $scope.$eval
- 考虑使用ngCSP模块

## Providers
- injector可以创建两种对象
	- 专有对象->angular框架提供的，如controller、filter、directives等
	- 服务->服务的API由开发人员自己制定->说明服务可以自己定制
- injector需要知道如何创建服务，它需要一个"图纸"
    - 图纸provider、factory、service、value、constant
    - 最底层的图纸是provider，其余四种图纸都是基于provider的语法糖（在provider上又封装了一层）
- value图纸
    - 用来创建可在运行阶段使用的常量
        ```javascript
        var myApp = angular.module('myApp', []);
        myApp.value('clientId', 'a12345654321x');
        myApp.controller('DemoController', ['clientId', function DemoController(clientId) {
            this.clientId = clientId;
        }
        ]);
        <html ng-app="myApp">
            <body ng-controller="DemoController as demo">
            Client ID: {{demo.clientId}}
        </body>
        </html>
        ```
- factory图纸
    - 可以创建任何类型的服务
    - factory图纸相较value图纸增加了下面功能
        - 可以有依赖
        - 服务初始化
        - 延迟/惰性初始化
    - factory图纸通过一个拥有0～n个参数(参数表示该服务对其他服务的依赖)的函数来创建服务，而函数返回值就是factory图纸创建的服务实例。
        ```javascript
        myApp.factory('clientId', function clientIdFactory() {
          return 'a12345654321x';// 返回的是服务的实例，不过这里clientId是一个常量，所以还是用value靠谱
        });
        ```
    - 更适合用factory的例子，计算token
        ```javascript
        myApp.factory('apiToken', ['clientId', function apiTokenFactory(clientId) {//将工厂方法命名为"Factory"是最佳实践（比如，apiTokenFactory）.虽然这种命名方式不是强制性的，但是它有助于浏览代码仓库或者在调试器里跟踪调用堆栈。
          var encrypt = function(data1, data2) {
            // NSA-proof加密算法：
            return (data1 + ':' + data2).toUpperCase();
          };
        
          var secret = window.localStorage.getItem('myApp.secret');
          var apiToken = encrypt(clientId, secret);
        
          return apiToken;
        }
        ]);
        ``````
        
- service图纸
    - 必须返回引用类型
    - 自定义类型，并携带token
        ```javascript
        function UnicornLauncher(apiToken) {
          this.launchedCount = 0;
          this.launch() {
            // 带上apiToken来发起远程调用
            ...
            this.launchedCount++;
          }}
        myApp.factory('unicornLauncher', ["apiToken", function(apiToken) {// 使用factory图纸来实现
          return new UnicornLauncher(apiToken);
        }]);
        myApp.service('unicornLauncher', ["apiToken", UnicornLauncher]);// 使用services语法更加简介，在内部还是会像factory一样，new 一下
        ```

- provider图纸
    - 注意必须实现$get方法
    - 其它图纸的底层都是通过provider实现的，如factory
        ```javascript 
        function factory(name, factoryFn) {
            return provider(name, { $get: factoryFn }); 
        }
        ```
    - 当你需要为在应用运行前就必须设置好的全局配置项提供API时，你才需要用到provider图纸
    - 假设我们的unicornLauncher服务是如此棒，以至于有好多应用都用到它。默认情况下，发射器将独角兽发射到太空中不需要任何保护屏障。但是在某些星球上，由于大气层非常厚，我们在将独角兽送去做星际旅行前必须将它们包裹在铝箔里，不然它们在穿越大气层时就被烧毁了。在一些应用里，需要设置发射器在每次发射时都使用铝箔屏蔽，如果我们能按需配置这一点那就太棒了。我们可以像下面这样让它变得可配置
        ```javascript
        myApp.provider('unicornLauncher', function UnicornLauncherProvider() {
          var useTinfoilShielding = false;
          this.useTinfoilShielding = function(value) {
            useTinfoilShielding = !!value;
          };
          this.$get = ["apiToken", function unicornLauncherFactory(apiToken) {
            // 这里我们假设UnicornLauncher的构造函数也被改造得支持useTinfoilShielding参数了
            return new UnicornLauncher(apiToken, useTinfoilShielding);
          }];
        });
        myApp.config(["unicornLauncherProvider", function(unicornLauncherProvider) {
          unicornLauncherProvider.useTinfoilShielding(true);
        }]);
        ```

- constant图纸
    - 用来创建可以在配置阶段使用的图纸
    - 在angular开始创建服务之前，angular会配置和实例化所有provider，此时服务还不能用，因为他们还没有被创建(只是provider被实例化了，由provider创建并返回的服务此时还没有被创建)；一旦配置阶段结束，与provider的交互就被禁止了，而创建服务的过程开始；->所以在配置阶段，没有服务可用，这就导致了一些没有依赖用value写的常量也无法被使用->使用constant
    - 假设在配置阶段提供了发射独角兽的星球名称，那么我们的unicornLauncher服务就能通过这个名字来标识一个独角兽。星球名是各个应用特有的，并且在应用运行时也会被各个控制器使用。我们可以像下面的代码那样把星球名定义为一个常量
        ```javascript
        myApp.constant('planetName', 'Greasy Giant');
        myApp.config(['unicornLauncherProvider', 'planetName', function(unicornLauncherProvider, planetName) {// 在配置阶段，使用constant，因为value无法使用
          unicornLauncherProvider.useTinfoilShielding(true);
          unicornLauncherProvider.stampText(planetName);
        }]);
        ``````

- value图纸也可以在控制器、模板、指令中使用
    ``` javascript
    myApp.controller('DemoController', ["clientId", "planetName", function DemoController(clientId, planetName) {
      this.clientId = clientId;
      this.planetName = planetName;
    }]);
    <html ng-app="myApp">
      <body ng-controller="DemoController as demo">
       Client ID: {{demo.clientId}}
       <br>
       Planet Name: {{demo.planetName}}
      </body>
    </html>
    ```
- 总结
	- injector用五种图纸来创建服务和专有对象
	- provider图纸是最底层的方法，其他的图纸都是基于其之上的语法糖
	- provider是最复杂的图纸类型，除非你正在构建需要全局配置的可复用代码，否则不要使用它
	- 除了控制器，其他所有专用对象都是通过factory图纸来定义的(factory定义的，都是单例对象，而controller不是单例的)
	- factory和service是最常用的图纸。它们之间的唯一区别就是service图纸存在一个new过程，所以最好返回一个构造函数，而factory可以创建任意类型
	- constant用在配置时，value用在运行时
	![](Provider.png)
        ```html
        * 有直接使用new操作符预先初始化的开销。
        ** 在配置阶段，服务对象是不能被访问的，但Provider实例是可以被访问的。（参见我们上面列举的unicornLauncherProvider例子）。
        ```

## HTML Compiler
- HTML compiler 让开发者可以教浏览器一些新的语法技能
- Compiler是 Angular 提供的一项服务，用来遍历DOM节点，查找特定的属性。编译过程分为两个阶段：
    - 编译compile：遍历DOM节点，收集所有的指令，返回一个连接函数（link func）
    - 连接link：将上一步收集到的每个指令与其所在的作用域（scope）连接生成一个实时视图。任何作用域中的模型改变都会实时在视图中反映出来，同时任何用户与视图的交互则会映射到作用域的模型中。这样，作用域中的数据模型就成了唯一的数据源。
- 指令directive
    - 在编译过程中，遇到特定的HTML结构（也就是指令）时，指令所声明的行为操作会被触发
    - 指令其实就是在编译器遍历DOM时碰到就需要执行的函数。
- 指令是如何被编译的
    - 知道 Angular 的编译是在DOM节点上发生而非字符串上是很重要的。如果你自己手动调用 $compile 时，如果你传给它一个字符串，显然是要报错的。所以，在你传值给 $compile 之前，用 angular.element 将字符串转化为DOM。
    - 编译流程
        - $compile 遍历DOM节点，匹配指令。
            - 如果编译器发现某个元素匹配一个指令，那么这个指令就被添加到指令列表中（该列表与DOM元素对应）。一个元素可能匹配到多个指令（译注：也就是一个元素里面可能有多个指令）。
        - 当所有指令都匹配到相应的元素时，编译器按照指令的 priority 属性来排列指令的编译顺序。
            - 然后依次执行每个指令的 compile 函数。每个 compile 函数有一次更改该指令所对应的DOM模板的机会。然后，每个 compile 函数返回一个 link 函数。这些函数构成一个“合并的”连接函数，它会调用每个指令返回的 link 函数。
        - 之后，$compile 调用第二步返回的连接函数，将模板和对应的作用域连接。而这又会依次调用连接函数中包含的每个指令对应的 link 函数，进而在各个DOM元素上注册监听器以及在相应的 scope 中设置对应的 $watchs。
        - 用代码表示大概是下面流程
            ```javascript
            var $compile = ...; // 已经存在的编译器

            var scope = ...;// 作用域
            
            var parent = ...; // DOM element where the compiled template can be appended，要被追加内容的DOM元素
            
            var html = '<div ng-bind="exp"></div>';// 指令模板字符串
            
            // Step 1: parse HTML into DOM element，将html字符串解析为DOM，因为如果传给$compile字符串会报错
            
            var template = angular.element(html);
            
            // Step 2: compile the template，遍历整个template的DOM，找到所有指令，并按priority排序，并依次执行每个指令里的compile函数，每个compile函数会返回一个link函数，以供第三部用
            
            var linkFn = $compile(template);
            
            // Step 3: link the compiled template with the scope.，$compile会依次调用第二步返回的link函数，将模板和对应的作用域连接，会依次调用连接函数中包含的每个指令对应的 link 函数，进而在各个DOM元素上注册监听器以及在相应的 scope 中设置对应的 $watches
            
            var element = linkFn(scope);
            
            // Step 4: Append to DOM (optional)，可选的将一切就绪的dom追加到html中
            
            parent.appendChild(element);
            ```
        - 总结：compile(找指令、排序、依次执行指令中compile函数并返回link函数)->执行每个指令中的link(DOM和作用域进行连接，设置事件监听并添加$watch)

## ngModel模块
- 当需要对数据进行深层处理时，可以用ngModel模块->一般是指令，这个ngModel对应的就是指令匹配元素上的ng-model
    ![](Ngmodel.png)

> 主要参考
> http://docs.ngnice.com/guide
> https://code.angularjs.org/1.2.32/docs/guide
> 小猫杯的angular视频教程
> 大漠穷秋的angular实战












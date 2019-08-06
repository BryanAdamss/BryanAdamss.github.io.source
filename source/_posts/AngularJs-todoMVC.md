---
title: AngularJs-todoMVC
tags:
  - Angular
  - todoMVC
categories:
  - 前端
date: 2017-07-13 11:23:52
---

# AngularJs-todoMVC 源码解释

> github上的[todoMVC仓库](https://github.com/tastejs/todomvc)是一个帮助你选择前端MVC框架的项目
> 项目中包含了绝大多数前端MVC框架实现Todo application的范例，让你能比较不同的框架实现同一个应用的差异。进而让你做出最佳选择。
> Todo application的具体效果，可以看这个http://todomvc.com/examples/angularjs/#/
> 对于新手来说，是个很不错的学习范例。
> 本文选取的是其中的angularJs范例，对其做了简单分析。
> 分析源码已经上传至github，https://github.com/BryanAdamss/SourceSave/tree/master/TodoMVC/angularjs
> 源码下载后，请在服务器中打开

## 目录结构
主要根据功能不同，放在了不同文件夹中
- angularjs/
    - js/
        - controllers/
            - todoCtrl.js->最主要的一个控制器
        - directives/
            - todoEscape.js->实现按下esc键，恢复到原先编辑状态的指令
            - todoFocus.js->再编辑input显示，聚焦的指令
        - services/
            - todoStorage.js->实现本地localStorge
        - app.js->入口文件，包含了路由配置
    - node_modules/
        - angular/
        - angular-resource/
        - angular-route/
        - todomvc-app-css/->页面主要样式文件
        - todomvc-common/->一些通用的css样式和js helper
    - index.html

## index.html
相关说明全部写在注释里了

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="utf-8">
    <meta name="renderer" content="webkit">
    <meta name="keywords" content="我是关键字">
    <meta name="description" content="我是网站描述">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta content="telephone=no,email=no" name="format-detection" />
    <meta name="full-screen" content="yes">
    <meta name="x5-fullscreen" content="true">
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=no,minimal-ui" />
    <link rel="stylesheet" href="node_modules/todomvc-common/base.css">
    <link rel="stylesheet" href="node_modules/todomvc-app-css/index.css">
    <title>Angular | TodoMVC</title>
    <style>
    [ng-cloak] {
        /*防止闪屏*/
        display: none;
    }
    </style>
</head>

<body ng-app="todomvc">
    <ng-view></ng-view>
    <script type="text/ng-template" id="todomvc-index.html">
        <!-- 模板 -->
        <section id="todoapp">
            <header id="header">
                <h1>todos</h1>
                <!-- form提交时，触发addTodo()事件 -->
                <form id="todo-form" ng-submit="addTodo()">
                    <!-- 新todo的输入框，值绑定到newTodo上，根据状态saving来禁用 -->
                    <input id="new-todo" placeholder="What needs to be done?" ng-model="newTodo" ng-disabled="saving" autofocus>
                </form>
            </header>
            <!-- #main根据todos的长度来显示隐藏 -->
            <section id="main" ng-show="todos.length" ng-cloak>
                <!-- #toggle-all 布尔值绑定到allChecked上，点击时触发markAll -->
                <input id="toggle-all" type="checkbox" ng-model="allChecked" ng-click="markAll(allChecked)">
                <label for="toggle-all">Mark all as complete</label>
                <ul id="todo-list">
                    <!-- 遍历每个todo，并通过statusFilter进行过滤，通过todo.completed、editedTodo来切换class -->
                    <li ng-repeat="todo in todos | filter:statusFilter track by $index" ng-class="{completed: todo.completed, editing: todo == editedTodo}">
                        <div class="view">
                            <!-- todo前的复选框，值绑定到todo.completed，change时触发toggleCompleted事件，并传入对应todo -->
                            <input class="toggle" type="checkbox" ng-model="todo.completed" ng-change="toggleCompleted(todo)">
                            <!-- 展示用label，双击时触发editTodo，并传入对应todo -->
                            <label ng-dblclick="editTodo(todo)">{{todo.title}}</label>
                            <!-- 删除按钮，点击时，移除对应todo -->
                            <button class="destroy" ng-click="removeTodo(todo)"></button>
                        </div>
                        <!-- 隐藏的再编辑表单，在表单提交时触发saveEdits -->
                        <form ng-submit="saveEdits(todo, 'submit')">
                            <!-- 再编辑input，值绑定到todo.title并不去除前后空格；按下esc时触发reverEdits事件，恢复到之前状态；失去焦点时自动提交；当双击展示用label时，todo和editedTodo相等，会触发todo-focus指令，显示再编辑input-->
                            <input class="edit" ng-trim="false" ng-model="todo.title" todo-escape="revertEdits(todo)" ng-blur="saveEdits(todo, 'blur')" todo-focus="todo == editedTodo">
                        </form>
                    </li>
                </ul>
            </section>
            <footer id="footer" ng-show="todos.length" ng-cloak>
                <!-- #todo-count 展示剩余待做todo数量 -->
                <span id="todo-count"><strong>{{remainingCount}}</strong>
                        <!-- 当count为1显示'item left'，否则显示'items left' -->
                        <ng-pluralize count="remainingCount" when="{ one: 'item left', other: 'items left' }"></ng-pluralize>
                    </span>
                <ul id="filters">
                    <!-- 过滤状态，点击时触发$routeChangeSuccess事件，改变statusFilter，进而改变展示的数据 -->
                    <li>
                        <a ng-class="{selected: status == ''} " href="#/">All</a>
                    </li>
                    <li>
                        <a ng-class="{selected: status == 'active'}" href="#/active">Active</a>
                    </li>
                    <li>
                        <a ng-class="{selected: status == 'completed'}" href="#/completed">Completed</a>
                    </li>
                </ul>
                <!-- 清除所有已完成todo，点击时触发clearCompletedTodos -->
                <button id="clear-completed" ng-click="clearCompletedTodos()" ng-show="completedCount">Clear completed</button>
            </footer>
        </section>
    </script>
    <!-- 资源文件 -->
    <script src="node_modules/todomvc-common/base.js"></script>
    <script src="node_modules/angular/angular.js"></script>
    <script src="node_modules/angular-route/angular-route.js"></script>
    <script src="node_modules/angular-resource/angular-resource.js"></script>
    <!-- 逻辑文件 -->
    <script src="js/app.js"></script>
    <script src="js/controllers/todoCtrl.js"></script>
    <script src="js/services/todoStorage.js"></script>
    <script src="js/directives/todoFocus.js"></script>
    <script src="js/directives/todoEscape.js"></script>
</body>

</html>

```

## app.js
app.js是入口文件，主要是创建了模块，并配置了路由
```javascript
/*global angular */

/**
 * The main TodoMVC app module
 *
 * @type {angular.Module}
 */
angular.module('todomvc', ['ngRoute', 'ngResource'])
    .config(['$routeProvider', function($routeProvider) {
        'use strict';
        var routeConfig = {
            controller: 'TodoCtrl',
            templateUrl: 'todomvc-index.html', // 指定模板
            resolve: {
                store: function(todoStorage) { // 在跳转路由之前载入正确的module
                    // Get the correct module (API or localStorage).
                    return todoStorage.then(function(module) {
                        module.get(); // Fetch the todo records in the background.
                        return module;
                    });
                }
            }
        };
        // 路由跳转
        $routeProvider
            .when('/', routeConfig)
            .when('/:status', routeConfig)
            .otherwise({
                redirectTo: '/'
            });
    }]);

```

## todoStorage.js
这个文件是一个服务，主要实现了数据在localStorge中的存储和读写
其实这一块没怎么看懂，主要是不太理解ngResource模块的作用，不过大概知道是存储和读取数据用的
```javascript
/*global angular */

/**
 * Services that persists and retrieves todos from localStorage or a backend API
 * if available.
 *
 * They both follow the same API, returning promises for all changes to the
 * model.
 */
// 这一块是懵逼的...大概就是将数据存储在localStorage中

angular.module('todomvc')
    .factory('todoStorage', function($http, $injector) {
        'use strict';

        // Detect if an API backend is present. If so, return the API module, else
        // hand off the localStorage adapter
        return $http.get('/api')
            .then(function() {
                return $injector.get('api');
            }, function() {
                return $injector.get('localStorage');
            });
    })

.factory('api', function($resource) {
    'use strict';

    var store = {
        todos: [],

        api: $resource('/api/todos/:id', null, {
            update: { method: 'PUT' }
        }),

        clearCompleted: function() {
            var originalTodos = store.todos.slice(0);

            var incompleteTodos = store.todos.filter(function(todo) {
                return !todo.completed;
            });

            angular.copy(incompleteTodos, store.todos);

            return store.api.delete(function() {}, function error() {
                angular.copy(originalTodos, store.todos);
            });
        },

        delete: function(todo) {
            var originalTodos = store.todos.slice(0);

            store.todos.splice(store.todos.indexOf(todo), 1);
            return store.api.delete({ id: todo.id },
                function() {},
                function error() {
                    angular.copy(originalTodos, store.todos);
                });
        },

        get: function() {
            return store.api.query(function(resp) {
                angular.copy(resp, store.todos);
            });
        },

        insert: function(todo) {
            var originalTodos = store.todos.slice(0);

            return store.api.save(todo,
                    function success(resp) {
                        todo.id = resp.id;
                        store.todos.push(todo);
                    },
                    function error() {
                        angular.copy(originalTodos, store.todos);
                    })
                .$promise;
        },

        put: function(todo) {
            return store.api.update({ id: todo.id }, todo)
                .$promise;
        }
    };

    return store;
})

.factory('localStorage', function($q) {
    'use strict';

    var STORAGE_ID = 'todos-angularjs';

    var store = {
        todos: [],

        _getFromLocalStorage: function() {
            return JSON.parse(localStorage.getItem(STORAGE_ID) || '[]');
        },

        _saveToLocalStorage: function(todos) {
            localStorage.setItem(STORAGE_ID, JSON.stringify(todos));
        },

        clearCompleted: function() {
            var deferred = $q.defer();

            var incompleteTodos = store.todos.filter(function(todo) {
                return !todo.completed;
            });

            angular.copy(incompleteTodos, store.todos);

            store._saveToLocalStorage(store.todos);
            deferred.resolve(store.todos);

            return deferred.promise;
        },

        delete: function(todo) {
            var deferred = $q.defer();

            store.todos.splice(store.todos.indexOf(todo), 1);

            store._saveToLocalStorage(store.todos);
            deferred.resolve(store.todos);

            return deferred.promise;
        },

        get: function() {
            var deferred = $q.defer();

            angular.copy(store._getFromLocalStorage(), store.todos);
            deferred.resolve(store.todos);

            return deferred.promise;
        },

        insert: function(todo) {
            var deferred = $q.defer();

            store.todos.push(todo);

            store._saveToLocalStorage(store.todos);
            deferred.resolve(store.todos);

            return deferred.promise;
        },

        put: function(todo, index) {
            var deferred = $q.defer();

            store.todos[index] = todo;

            store._saveToLocalStorage(store.todos);
            deferred.resolve(store.todos);

            return deferred.promise;
        }
    };

    return store;
});

```

## todoEscape.js
这是一个指令，主要完成按下esc键，恢复再编辑input到原先状态
```javascript
/*global angular */

/**
 * Directive that executes an expression when the element it is applied to gets
 * an `escape` keydown event.
 */
// esc键绑定事件
// 当按下Escape键时，执行attrs.todoEscape的表达式。
angular.module('todomvc')
    .directive('todoEscape', function() {
        'use strict';
        var ESCAPE_KEY = 27;
        return function(scope, elem, attrs) { // 直接返回一个函数，实际上就是link函数；在link函数中绑定事件

            elem.bind('keydown', function(event) {
                if (event.keyCode === ESCAPE_KEY) { // 按下esc，触发attrs.todoEscape对应的事件
                    scope.$apply(attrs.todoEscape);
                }
            });

            scope.$on('$destroy', function() { // 销毁时，解除绑定
                elem.unbind('keydown');
            });
        };
    });

```

## todoFocus.js
这个指令主要完成再编辑input的显示和聚焦
```javascript
/*global angular */

/**
 * Directive that places focus on the element it is applied to when the
 * expression it binds to evaluates to true
 */
angular.module('todomvc')
    .directive('todoFocus', function todoFocus($timeout) {
        'use strict';
        return function(scope, elem, attrs) { // 在二次编辑的input上绑定事件
            scope.$watch(attrs.todoFocus, function(newVal, oldVal) {
                // 当双击时，newVal为true
                if (newVal) {
                    $timeout(function() {
                        elem[0].focus();
                    }, 0, false);
                }
            });
        };
    });

```

## todoCtrl.js
这是重头性，关键性逻辑全写在这
```javascript
/*global angular */

/**
 * The main controller for the app. The controller:
 * - retrieves and persists the model via the todoStorage service
 * - exposes the model to the template and provides event handlers
 */
angular.module('todomvc')
    .controller('TodoCtrl', function TodoCtrl($scope, $routeParams, $filter, store) {
        'use strict';

        var todos = $scope.todos = store.todos; // 从localStorge中取出所有todo

        $scope.newTodo = ''; // 用来保存新创建的todo
        $scope.editedTodo = null; // 用来保存编辑过的todo

        $scope.$watch('todos', function() { // 深度观察todos的值
            $scope.remainingCount = $filter('filter')(todos, { completed: false }).length; // 更新未完成的todo数量
            $scope.completedCount = todos.length - $scope.remainingCount; // 更新完成的todo数量
            $scope.allChecked = !$scope.remainingCount; // 是否全部完成
        }, true);

        // Monitor the current route for changes and adjust the filter accordingly.
        $scope.$on('$routeChangeSuccess', function() { // 观察路由跳转，并更新用来过滤的statusFilter
            var status = $scope.status = $routeParams.status || '';
            $scope.statusFilter = (status === 'active') ? { completed: false } : (status === 'completed') ? { completed: true } : {};
        });

        $scope.addTodo = function() { // 输入框提交时触发
            var newTodo = { // 创建新todo
                title: $scope.newTodo.trim(), //newTodo是绑定在input输入框上
                completed: false
            };

            if (!newTodo.title) { // 空值，则不提交
                return;
            }

            $scope.saving = true; // saving用来标识input的禁用状态，为true则禁用

            store.insert(newTodo) // 插入新todo
                .then(function success() { // 成功则重置newTodo
                    $scope.newTodo = '';
                })
                .finally(function() {
                    $scope.saving = false; // 最后取消input的禁用状态
                });
        };

        $scope.editTodo = function(todo) { // 已添加的todo上双击时触发，会将双击的todo传入
            $scope.editedTodo = todo; // 保存正在编辑的todo
            // Clone the original todo to restore it on demand.
            $scope.originalTodo = angular.extend({}, todo); // 保留原先的todo，以备不时之需
        };

        $scope.saveEdits = function(todo, event) { // 再编辑input提交或者blur时触发
            // Blur events are automatically triggered after the form submit event.
            // This does some unfortunate logic handling to prevent saving twice.
            if (event === 'blur' && $scope.saveEvent === 'submit') { // 提交时，会自动触发一次blur，所以手动阻止
                $scope.saveEvent = null;
                return;
            }

            $scope.saveEvent = event; // 保存事件类型(blur或submit)

            if ($scope.reverted) { // 如果编辑后按esc，取消了编辑，则不保存
                // Todo edits were reverted-- don't save.
                $scope.reverted = null;
                return;
            }

            todo.title = todo.title.trim(); // 保存新编辑title

            if (todo.title === $scope.originalTodo.title) { // title未发生改变，则不保存
                $scope.editedTodo = null;
                return;
            }

            store[todo.title ? 'put' : 'delete'](todo)
                .then(function success() {}, function error() { // 保存出错，则恢复title
                    todo.title = $scope.originalTodo.title;
                })
                .finally(function() { // 最后，重置editedTodo
                    $scope.editedTodo = null;
                });
        };

        $scope.revertEdits = function(todo) { // todoEscape时触发，将再编辑input恢复到编辑前的状态，会传入需要恢复的todo
            todos[todos.indexOf(todo)] = $scope.originalTodo;
            $scope.editedTodo = null;
            $scope.originalTodo = null;
            $scope.reverted = true;
        };

        $scope.removeTodo = function(todo) { // 删除todo
            store.delete(todo);
        };

        $scope.saveTodo = function(todo) { // 保存todo
            store.put(todo);
        };

        $scope.toggleCompleted = function(todo, completed) { // 切换完成状态
            if (angular.isDefined(completed)) { // 如果completed曾经定义过，则直接使用
                todo.completed = completed;
            }

            // 更新localStorge上的todo的complete
            store.put(todo, todos.indexOf(todo))
                .then(function success() {}, function error() { // 保存出错，则恢复
                    todo.completed = !todo.completed;
                });
        };

        $scope.clearCompletedTodos = function() { // 清除所有已经完成的todo
            store.clearCompleted();
        };

        $scope.markAll = function(completed) { // 将所有todo置为已完成
            todos.forEach(function(todo) {
                if (todo.completed !== completed) {
                    $scope.toggleCompleted(todo, completed);
                }
            });
        };
    });
```


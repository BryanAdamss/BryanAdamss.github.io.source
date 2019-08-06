---
title: design-pattern
tags:
  - 设计模式
categories:
  - 前端
date: 2018-04-25 10:18:10
---
> 此文是《JavaScript设计模式与开发实践》的读书笔记
> 本文所有源码可在[这里](https://github.com/BryanAdamss/SourceSave/tree/master/Sjms)找到

# JavaScript设计模式与开发实践

## 基础知识

### 鸭子类型
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 鸭子类型
    // 如果它走起来像鸭子，叫起来也像鸭子，那它就是鸭子
    // 鸭子类型指导我们只关注对象的行为，而不关注对象本身
    var duck = {
        duckSinging: function() {
            console.log("嘎嘎");
        }
    };
    var chicken = {
        duckSinging: function() {
            console.log("嘎嘎");
        }
    };
    // 合唱团
    var choir = [];
    var joinChoir = function(animal) {
        if (animal && typeof animal.duckSinging === "function") {
            choir.push(animal);
            console.log("恭喜加入合唱团");
            console.log("合唱团当前人数:" + choir.length);
        }
    };
    joinChoir(duck);
    joinChoir(chicken);
    </script>
</body>

</html>
```

### 多态
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 多态:同一操作应用在不同的对象上面，可以产生不同的解释和不同的执行结果
    // 即给不同的对象发送同一个消息的时候，这些对象会根据这个消息分别给出不同的反馈
    var makeSound = function(animal) {
        if (animal instanceof Duck) {
            console.log("嘎嘎");
        } else if (animal instanceof Chicken) {
            console.log("咯咯");
        }
    };
    var Duck = function() {};
    var Chicken = function() {};
    makeSound(new Duck());
    makeSound(new Chicken());
    // 这里是不优雅的，因为如果再来一条狗，需要修改makeSound
    </script>
</body>

</html>

```

### 多态2
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 多态背后是将"做什么"和"谁去做，怎么做"分离开来，即将不变的和可变的分离开来
    // 把不变的隔离出来，把可变的部分封装起来,叫是不变的，动物怎么叫是可变的

    // 隔离出不变的
    var makeSound = function(animal) {
        animal.sound();
    };

    // 封装可变的
    var Duck = function() {};

    Duck.prototype.sound = function() {
        console.log("嘎嘎");
    };

    var Chicken = function() {};

    Chicken.prototype.sound = function() {
        console.log("咯咯");
    };

    makeSound(new Duck());
    makeSound(new Chicken());
    // 添加新动物
    var Dog = function() {};
    Dog.prototype.sound = function() {
        console.log("汪汪");
    };
    makeSound(new Dog());
    </script>
</body>

</html>

```

### 多态3
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    var googleMap = {
        show: function() {
            console.log('开始渲染谷歌地图');
        }
    };
    var baiduMap = {
        show: function() {
            console.log('开始渲染百度地图');
        }
    };
    var renderMap = function(type) {
        if (type === 'google') {
            googleMap.show();
        } else if (type === 'baidu') {
            baiduMap.show();
        }
    };
    renderMap('google'); // 输出：开始渲染谷歌地图
    renderMap('baidu'); // 输出：开始渲染百度地图

    // 如果添加其他地图，则需要在renderMap中继续判断，这里显示地图是不变的，显示哪个，怎么显示是可变的，所以可将show隔离(提取)出来

    var renderMap2 = function(map) {
        if (map.show instanceof Function) {
            map.show();
        }
    };
    // 添加其他地图
    var sosoMap = {
        show: function() {
            console.log("开始渲染Soso地图");
        }
    };
    renderMap2(googleMap); // 输出：开始渲染谷歌地图
    renderMap2(baiduMap); // 输出：开始渲染百度地图
    renderMap2(sosoMap); // 输出:开始渲染Soso地图
    </script>
</body>

</html>

```

### 封装数据
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // js一般通过函数来创建作用域来封装数据
    var myObj = (function() {
        // 私有变量
        var _name = "haha";
        return {
            // 公开方法
            getName: function() {
                return _name;
            }
        }
    })();
    console.log(myObj.getName()); // "haha"
    console.log(myObj._name); // undefined
    </script>
</body>

</html>

```

### 封装实现
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 封装实现，例如sort()函数的具体实现你并不知道，它对外界来说是透明的，不可见的
    var arr = [1, 3, 5, 2, 4, 9, 7];
    console.log(arr.sort());
    </script>
</body>

</html>

```

### 原型模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 原型模式是用来创建对象的一种模式，如果想创建一个对象有2种方法
    // 1.先指定类型，然后通过类型创建对象
    // 2.找到一个对象做为原型，通过克隆创建一个一模一样的对象
    // 原型模式是选用的第二种
    // 使用场景:创建一个跟某个对象一模一样的对象时
    // 实现关键:语言本身是否提供了克隆的方法,js中可用Object.create()
    var Plane = function() {
        this.blood = 100;
        this.attackLevel = 1;
        this.defenseLevel = 1;
    };
    var plane = new Plane();
    plane.blood = 500;
    plane.attackLevel = 3;
    plane.defenseLevel = 4;
    // Object.create的兼容
    Object.create = Object.create || function(obj) {
        var F = function() {};
        F.prototype = obj;
        return new F();
    };
    // 克隆
    var clonePlane = Object.create(plane);
    console.log(clonePlane);
    console.log(clonePlane.blood);
    console.log(clonePlane.attackLevel);
    console.log(clonePlane.defenseLevel);
    </script>
</body>

</html>

```

### this
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // js中this是在函数被调用时才确定的，它是指向被调用时的执行环境
    // 1.全局环境下this->window
    console.log(this); // window
    // 2.当函数做为对象方法调用时,this指向此对象
    var a = {
        name: "aaaa",
        getName: function() {
            console.log(this.name);
        }
    };
    a.getName(); // aaaa
    (a.getName)(); // aaaa

    (a.getName = a.getName)(); // 空
    (a.getName || a.getName)(); // 空
    (a.getName, a.getName)(); // 空

    // 注意this是指向最靠近方法的对象
    var b = {
        name: "bbbb"
    };
    b.bar = a;
    b.bar.getName(); // aaaa
    // 3.匿名函数的执行环境具有全局性，所以this通常指向window
    (function() {
        console.log(this); // window
    })();
    // 4.做为构造器调用时,this指向被新创建的对象
    var Person = function(name) {
        this.name = name;
    };
    var person = new Person("haha");
    console.log(person.name);
    // 5.Function.prototype.call 或Function.prototype.apply 可以动态地改变传入函数的this
    a.getName.call(b); // bbbb 相当于b.getName()
    a.getName.apply(b); // bbbb 相当于b.getName()
    </script>
</body>

</html>

```

### call、apply
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // call、apply都可以改变函数中this的指向

    // 第一个参数都指定了函数体内this的指向，如果传入null,则代表在全局环境下调用。apply的第二个参数为带下标的集合(类数组),call参数个数不限制，会按顺序依依对应到形参上
    var func = function(a, b, c) {
        console.log([a, b, c]);
    };
    func.apply(null, [1, 2, 3]); // 输出 [ 1, 2, 3 ]
    func.call(null, 1, 2, 3); // 输出 [ 1, 2, 3 ]

    // call、apply通常用来实现函数借用，用来在类数组(arguments)上调用数组的方法
    // 函数的参数列表arguments 是一个类数组对象，虽然它也有“下标”，但它并非真正的数组，所以也不能像数组一样，进行排序操作或者往集合里添加一个新的元素
    // arguments转成真正的数组的时候，可以借用Array.prototype.slice方法
    // 想截取arguments列表中的头一个元素时，可以借用Array.prototype.shift方法。
    (function() {
        Array.prototype.push.call(arguments, 6);
        console.log(arguments); // 输出[4,5,6]
    })(4, 5);

    // 通过call、apply实现绑定this
    Function.prototype.bind = function() {
        var self = this, // 保存原函数
            context = [].shift.call(arguments), // 需要绑定的this上下文,arguments上为shift方法，通过call来实现调用数组的shift方法
            args = [].slice.call(arguments); // 剩余的参数转成数组
        return function() { // 返回一个新的函数
            return self.apply(context, [].concat.call(args, [].slice.call(arguments)));
            // 执行新的函数的时候，会把之前传入的context 当作新函数体内的this
            // 并且组合两次分别传入的参数，作为新函数的参数
        }
    };
    var obj = {
        name: 'sven'
    };
    var func = function(a, b, c, d) {
        alert(this.name); // 输出：sven
        alert([a, b, c, d])
    }.bind(obj, 1, 2); // 将func中的this绑定到obj上，并传入了默认参数
    func(3, 4); // 输出：[ 1, 2, 3, 4 ]
    </script>
</body>

</html>

```

### 闭包
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div>1</div>
    <div>2</div>
    <div>3</div>
    <div>4</div>
    <div>5</div>
    <script>
    // 函数中声明另外一个函数就会出现闭包
    // 经典案例，每个都弹出5
    var nodes = document.getElementsByTagName('div');
    // for (var i = 0, len = nodes.length; i < len; i++) {
    //     nodes[i].onclick = function() {
    //         alert(i);
    //     };
    // };
    // 这是因为onclick事件是异步触发的，当触发时for循环早结束了，i已经变成5了，所以事件处理函数中引用的i都为5
    // 修正
    for (var i = 0, len = nodes.length; i < len; i++) {
        (function(i) {
            nodes[i].onclick = function() {
                alert(i);
            }
        })(i)
    };
    </script>
</body>

</html>

```

### 利用闭包创建测试类型函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    var Type = {};
    for (var i = 0, type; type = ['String', 'Array', 'Number'][i++];) { // 当语句2条件为false时，就会退出for循环，当i为3时，type[3]为undefined，为false跳出循环
        // 利用闭包创建判断类型的函数
        (function(type) {
            Type['is' + type] = function(obj) {
                return Object.prototype.toString.call(obj) === '[object ' + type + ']';
            }
        })(type)
    };
    Type.isArray([]); // 输出：true
    Type.isString("str"); // 输出：true
    </script>
</body>

</html>

```

### 闭包封装变量
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 乘积
    // 1.普通方法
    var mult = function() {
        var a = 1;
        for (var i = 0, len = arguments.length; i < len; i++) {
            a *= arguments[i];
        }
        return a;
    };
    console.log(mult(8, 9)); // 72
    // 2.进阶添加缓存机制
    var cache = {};
    var mult2 = function() {
        var args = Array.prototype.join.call(arguments, ","); // 将实参数组转换为字符串
        if (cache[args]) {
            // cache中存在原先的计算结果，则直接返回结果
            console.log("我是从缓存2取的");
            return cache[args];
        }
        var a = 1;
        for (var i = 0, len = arguments.length; i < len; i++) {
            a *= arguments[i];
        }
        // 将结果存入缓存中
        cache[args] = a;
        return a;

    };
    console.log(mult2(8, 9));
    // console.log(cache);
    console.log(mult2(8, 9));
    // 3.超进阶使用闭包封装cache，因为这个cache是单独属于mult3的，所以可以封装到mult3中
    var mult3 = (function() {
        var cache3 = {};
        return function() {
            var args = Array.prototype.join.call(arguments, ","); // 将实参数组转换为字符串
            if (args in cache3) {
                // cache中存在原先的计算结果，则直接返回结果
                console.log("我是从缓存3取的");
                return cache3[args];
            }
            var a = 1;
            for (var i = 0, len = arguments.length; i < len; i++) {
                a *= arguments[i];
            }
            // 将结果存入缓存中
            cache3[args] = a;
            return a;
        };
    })();
    console.log(mult3(8, 10));
    console.log(mult3(8, 10));
    // console.log(cache3); // error
    // 4.Max版本提炼函数
    var mult4 = (function() {
        var cache4 = {};
        // 将重复使用到的计算提炼出来
        var calcuate = function() {
            var a = 1;
            for (var i = 0, len = arguments.length; i < len; i++) {
                a *= arguments[i];
            }
            return a;
        };
        return function() {
            var args = Array.prototype.join.call(arguments, ","); // 将实参数组转换为字符串
            if (args in cache4) {
                // cache中存在原先的计算结果，则直接返回结果
                console.log("我是从缓存4取的");
                return cache4[args];
            }
            // 全局环境下调用calcuate
            cache4[args] = calcuate.apply(null, arguments);
            return cache4[args];
        };
    })();
    console.log(mult4(9, 9));
    console.log(mult4(9, 9));
    // console.log(calcuate);// error
    </script>
</body>

</html>

```

### 回调函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    var appendDiv = function(callback) {
        for (var i = 0; i < 100; i++) {
            var div = document.createElement("div");
            div.innerHTML = i;
            document.body.appendChild(div);
            if (typeof callback === "function") {
                callback(div);
            }
        }
    };
    appendDiv();
    // 通过回调函数来确定是否隐藏
    appendDiv(function(node) {
        node.style.display = "none";
    });
    </script>
</body>

</html>

```

### AOP
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // AOP 面向切片编程，将跟核心业务无关的功能抽离出来，再通过动态织入
    Function.prototype.before = function(beforefn) {
        var __self = this; // 保存原函数的引用

        return function() { // 返回包含了原函数和新函数的"代理"函数
            beforefn.apply(this, arguments); // 执行新函数，修正this
            return __self.apply(this, arguments); // 执行原函数
        }
    };
    Function.prototype.after = function(afterfn) {
        var __self = this;
        return function() {
            var ret = __self.apply(this, arguments);
            afterfn.apply(this, arguments);
            return ret;
        }
    };
    var func = function() {
        console.log(2);
    };
    func = func.before(function() {
        console.log(1);
    }).after(function() {
        console.log(3);
    });

    func(); // 1,2,3
    </script>
</body>

</html>

```

### 函数柯里化
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 柯里化一般用于创建已经设置好了一个或多个参数的函数(创建带有默认参数值函数的过程)
    var curry = function(fn) {
        var _args = Array.prototype.slice.call(arguments, 1); //保存默认参数, 一般curry接收的第一个参数为需要柯里化的函数，后面的实参都是要设置为默认值的参数
        console.log(_args);
        return function() {
            var innerArgs = Array.prototype.slice.call(arguments); // 新参数
            var finalArgs = _args.concat(innerArgs); // 最终参数
            return fn.apply(null, finalArgs); // 在全局调用柯里化好的函数
        };
    };

    function add(num1, num2) {
        return num1 + num2;
    }

    var curriedAdd = curry(add, 5); // curriedAdd此时已经有一个默认参数5了， 使用时就只用再传入一个参数
    console.log(curriedAdd(3)); //8
    </script>
</body>

</html>

```

### 反柯理化
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 将特有函数普通化
    Function.prototype.uncurrying = function() {
        var _self = this; // 保存原函数
        return function() {
            var obj = Array.prototype.shift.call(arguments); // 保存调用环境
            return _self.apply(obj, arguments); // 调用原函数
        };
    };
    // 反柯里化的另一种写法
    // Function.prototype.uncurrying = function() {
    //     var _self = this;
    //     return function() {
    //         return Function.prototype.call.apply(_self, arguments);
    //     };
    // };

    var push = Array.prototype.push.uncurrying();

    (function() {
        push(arguments, 4);
        console.log(arguments); // 输出：[1, 2, 3, 4]
    })(1, 2, 3);
    </script>
</body>

</html>

```

### 函数截流
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 有些时候函数会被系统频繁的触发,如window.onresize事件、mousemove、上传进度时会频繁触发相应的处理函数
    // 实现函数节流可通过setTimeout延迟执行来解决
    var throttle = function(fn, interval) {
        var _self = fn, //保存需要被延迟的函数
            firstTime = true, // 是否首次调用
            intervalTime = interval || 500, // 间隔调用时间，默认500毫秒
            timer; // 定时器
        return function() {
            var args = arguments,
                _me = this;
            if (firstTime) { // 如果第一次，则无需延迟，直接调用
                _self.apply(_me, args);
                return firstTime = false;
            }
            if (timer) { // 如果定时器存在，说明前一次执行还没有完成
                return false;
            }
            timer = setTimeout(function() { // 延迟intervalTime后执行
                clearTimeout(timer);
                timer = null;
                _self.apply(_me, args);
            }, intervalTime);
        };
    };

    window.onresize = throttle(function() {
        console.log(1); // 只触发了几次
    }, 1000);
    // window.onresize = function() {
    //     console.log(1); // 几十次
    // };
    </script>
</body>

</html>
```

### 分时函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 当一次性往DOM中添加大量节点，浏览器会卡死，所以需要分时间添加节点
    // var ary = [];
    // for (var i = 1; i <= 99999; i++) {
    //     ary.push(i); // 假设ary 装载了1000 个好友的数据
    // };
    // var renderFriendList = function(data) {
    //     for (var i = 0, l = data.length; i < l; i++) {
    //         var div = document.createElement('div');
    //         div.innerHTML = i;
    //         document.body.appendChild(div);
    //     }
    // };
    // renderFriendList(ary);


    /**
     * 实现分时的函数，在intervalTime时间间隔内执行count次fn函数
     * @author cgh
     * @time   2017-03-06
     * @param  {Array}   	ary    			每次fn执行需要的参数数组
     * @param  {Function} 	fn    			处理函数
     * @param  {Number}   	count 			每个时间间隔内执行的次数
     * @param  {Number}   	intervalTime 	时间间隔
     * @return {Function}       			函数
     */
    var timeChunk = function(ary, fn, count, interval) {
        var obj, t,
            len = ary.length,
            intervalTime = interval || 200; // 默认时间间隔200ms
        var start = function() {
            for (var i = 0; i < Math.min(count || 1, len); i++) {
                obj = ary.shift();
                fn(obj);
            }
        };
        return function() {
            t = setInterval(function() {
                if (ary.length === 0) {
                    return clearInterval(t);
                }
                start();
            }, intervalTime);
        };
    };

    var ary = [];
    for (var i = 1; i <= 800; i++) {
        ary.push(i);
    };
    // 每隔500ms创建8个节点插入到DOM中
    var renderFriendList = timeChunk(ary, function(n) {
        var div = document.createElement('div');
        div.innerHTML = n;
        document.body.appendChild(div);
    }, 8, 500);
    renderFriendList();
    </script>
</body>

</html>

```

### 惰性载入函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div id="div1">点我绑定事件</div>
    <script type="text/javascript">
    var addEvent = function(elem, type, handler) {
        // 首次点击时判断绑定方式，后续点击时无需再判断
        if (window.addEventListener) {
            addEvent = function(elem, type, handler) {
                elem.addEventListener(type, handler, false);
            }
        } else if (window.attachEvent) {
            addEvent = function(elem, type, handler) {
                elem.attachEvent('on' + type, handler);
            }
        }
        addEvent(elem, type, handler);
    };
    var div = document.getElementById('div1');
    addEvent(div, 'click', function() {
        alert(1);
    });
    addEvent(div, 'click', function() {
        alert(2);
    });
    </script>
</body>

</html>

```

## 单例模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 单例模式:保证一个类仅有一个实例，并提供一个访问它的全局访问点
    // 使用场景:当某些对象只需要一个的时候，如线程池、全局缓存、window等
    // 使用举例:如登录页，点击按钮生成登录框，这个登录框只会生成一次，不会每次点击都生成一个
    var Singleton = function(name) {
        this.name = name;
        this.instance = null;
    };
    Singleton.prototype.getName = function() {
        alert(this.name);
    };
    Singleton.getInstance = function(name) {
        if (!this.instance) {
            this.instance = new Singleton(name);
        }
        return this.instance;
    };
    var a = Singleton.getInstance("AAA");
    var b = Singleton.getInstance("BBB");
    console.log(a === b); //true
    // 此种方法通过getInstance来获取实例，和正常的new不一样，用户必须知道这是个单例模式，也就是用户必须知道是通过getInstance方法来获取实例而不是new方式，所就这种方式对用户来说"不透明"
    </script>
</body>

</html>

```

### 透明的单例模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 实例化时不是通过特殊的方法来实现，实例化和平常一样使用new
    var CreateDiv = (function() {
        var instance;
        var CreateDiv = function(html) {
            if (instance) {
                return instance;
            }
            this.html = html;
            this.init();
            return instance = this;
        };
        CreateDiv.prototype.init = function() {
            var div = document.createElement("div");
            div.innerHTML = this.html;
            document.body.appendChild(div);
        };
        return CreateDiv;
    })();
    var a = new CreateDiv("div1");
    var b = new CreateDiv("div2");
    console.log(a === b); // true
    // 不足:此处的构造函数做了2件事，1.创建对象并初始化2.保证只有一个对象。如果某天需要在页面中创建多个div，则需要手动修改此构造函数，所以推荐用代理实现单例模式
    </script>
</body>

</html>

```

### 用代理实现单例模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    var CreateDiv = function(html) {
        this.html = html;
        this.init();
    };
    CreateDiv.prototype.init = function() {
        var div = document.createElement("div");
        div.innerHTML = this.html;
        document.body.appendChild(div);
    };
    // 引入代理,来保证实例的单一性
    var ProxySingletonCreateDiv = (function() {
        var instance;
        return function(html) {
            if (!instance) {
                instance = new CreateDiv(html);
            }
            return instance;
        };
    })();
    var a = new ProxySingletonCreateDiv("div1");
    var b = new ProxySingletonCreateDiv("div2");
    console.log(a === b); //true
    // 这样创建div和保证单一性就分开了，后期也好修改
    // 这些方法都和传统的面向对象编程一样，先有类，再创建实例
    </script>
</body>

</html>

```

### js中的单例模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 单例模式的核心是只有一个实例，并提供全局访问，所以在JS中无需脱裤子放屁，先创建个类，再从类创建实例，可以直接用全局变量(对象)来创建单例
    // 全局对象满足单例的两个核心点，一个实例并提供全局访问，虽然全局对象不是单例模式，但在JS中会把全局对象当做单例模式来用
    // 但全局变量(对象)会造成全局变量污染，所以需要避免，可通过下面方式

    // 使用命名空间
    var namespace1 = {
        a: function() {
            alert(1);
        },
        b: function() {
            alert(2);
        }
    };
    // 使用闭包封装私有变量
    var user = (function() {
        var _name = "aaa";
        var _age = 18;
        return {
            getUserInfo: functioin() {
                return _name + ":" + _age;
            }
        }
    })();
    </script>
</body>

</html>

```

### 惰性单例
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="loginBtn">登录</button>
    <script type="text/javascript">
    // 惰性单例指的是在需要的时候才创建对象实例，实际在19中Singleton.getInstance就是在需要的时候创建实例，并非一开始就创建
    // 利用全局变量(对象)和惰性单例来实现登录弹窗对象的创建
    // var loginLayer = (function() {
    //     var div = document.createElement("div");
    //     div.innerHTML = "我是登录窗口";
    //     div.style.display = 'none';
    //     document.body.appendChild(div);
    //     return div;
    // })();
    // document.getElementById('loginBtn').onclick = function() {
    //     loginLayer.style.display = 'block';
    // };
    // 上面这种在页面加载的时候就会创建一个登录窗口，如果用户根本不点登录的时候也会创建，所以会浪费
    // 改进
    // var createLoginLayer = function() {
    //     var div = document.createElement("div");
    //     div.innerHTML = "我是登录窗口";
    //     div.style.display = 'none';
    //     document.body.appendChild(div);
    //     return div;
    // };

    // document.getElementById('loginBtn').onclick = function() {
    //     var loginLayer = createLoginLayer();
    //     loginLayer.style.display = 'block';
    // };
    // 这样虽然不会一载入就创建，但是没有保证单例，多次点击会创建多个
    // 改进
    var createLoginLayer = (function() {
        var div;
        return function() {
            if (!div) {
                div = document.createElement("div");
                div.innerHTML = "我是登录窗口";
                div.style.display = "none";
                document.body.appendChild(div);
            }
            return div;
        };
    })();
    document.getElementById('loginBtn').onclick = function() {
        var loginLayer = createLoginLayer();
        loginLayer.style.display = 'block';
    };
    // 上面的方式已经能实现基本功能，但是仔细观察会发现，他将创建对象和管理单例的逻辑放在了一起，加入下次想创建一个iframe，就必须将上面代码再抄一遍
    </script>
</body>

</html>

```

### 通用的惰性单例
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="loginBtn">登录</button>
    <button id="loginBtn2">登录2</button>
    <script type="text/javascript">
    // 将23中的管理单例的逻辑给隔离(抽象)出来，实现创建和管理的分离
    // 管理单例
    var getSingle = function(fn) {
        var result;
        return function() {
            return result || (result = fn.apply(this, arguments));
        };
    };
    // 创建login对象
    var createLoginLayer = function() {
        var div = document.createElement('div');
        div.innerHTML = '我是登录浮窗';
        div.style.display = 'none';
        document.body.appendChild(div);
        return div;
    };

    var createSingleLoginLayer = getSingle(createLoginLayer);
    document.getElementById('loginBtn').onclick = function() {
        var loginLayer = createSingleLoginLayer();
        loginLayer.style.display = 'block';
    };
    // 创建iframe对象
    var createIframe = function() {
        var iframe = document.createElement('iframe');
        document.body.appendChild(iframe);
        return iframe;
    }
    var createSingleIframe = getSingle(createIframe);

    document.getElementById('loginBtn2').onclick = function() {
        var loginLayer = createSingleIframe();
        loginLayer.src = 'http://baidu.com';
    };
    </script>
</body>

</html>

```

## 策略模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 策略模式:定义一系列算法，把他们一个一个封装起来，并且使他们可以相互替换。策略模式定义了算法家族，分别封装起来，让他们之间可以互相替换，此模式让算法的变化不会影响到使用算法的客户。它将算法的实现和使用分离
    // 应用场景:当需要根据不同策略(算法)计算时;实践中，不仅可以封装算法，也可以用来封装几乎任何类型的规则，是要在分析过程中需要在不同时间应用不同的业务规则，就可以考虑是要策略模式来处理各种变化。
    // 使用举例:根据工资和绩效等级计算年终奖、缓动动画、表单验证，这三种都是会有多中不同的策略，如年终奖有SAB三等级3个策略，最终都是要得到年终奖。缓动动画有ease-in,ease-out...，最后是要得到一个动画。表单验证需要验证数字、邮箱...等，但最终都要得到是否验证通过这一结果。都是需要根据不同策略，得出某结果
    // 组成部分:1.一组策略类，封装了算法，并负责具体计算2.环境类Contex，负责接收请求，并把请求委托给某个具体的策略类

    // var calculateBonus = function(performanceLevel, salary) {
    //     if (performanceLevel === "S") {
    //         return salary * 4;
    //     }
    //     if (performanceLevel === "A") {
    //         return salary * 3;
    //     }
    //     if (performanceLevel === "B") {
    //         return salary * 2;
    //     }
    // };
    // console.log(calculateBonus("B", 2000)); //4000
    // console.log(calculateBonus("S", 4000)); //16000
    // 上面的缺点
    // 函数庞大，包含了很多if-else，这些要覆盖所有的逻辑分支
    // 缺乏弹性，如果要增加C等级，或修改倍数，必须进入函数内部修改
    // 算法复用性差

    // 使用组合函数重构
    var performanceS = function(salary) {
        return salary * 4;
    };
    var performanceA = function(salary) {
        return salary * 3;
    };
    var performanceB = function(salary) {
        return salary * 2;
    };
    var calculateBonus = function(performanceLevel, salary) {
        if (performanceLevel === 'S') {
            return performanceS(salary);
        }
        if (performanceLevel === 'A') {
            return performanceA(salary);
        }
        if (performanceLevel === 'B') {
            return performanceB(salary);
        }
    };
    calculateBonus('A', 10000); // 输出：30000
    // 虽然解决了算法复用，但是依旧庞大，并缺乏弹性
    </script>
</body>

</html>

```

### 传统面向对象中的策略模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 使用传统面向对象方式实现策略模式，将不同策略封装到不同的类中
    // 一组策略类
    var performanceS = function() {};
    performanceS.prototype.calculate = function(salary) {
        return salary * 4;
    }
    var performanceA = function() {};
    performanceA.prototype.calculate = function(salary) {
        return salary * 3;
    }
    var performanceB = function() {};
    performanceB.prototype.calculate = function(salary) {
        return salary * 2;
    }
    var Bonus = function() {
        this.salary = null; // 工资
        this.strategy = null; // 绩效等级对应的策略对象
    };
    Bonus.prototype.setSalary = function(salary) {
        this.salary = salary;
    };
    Bonus.prototype.setStrategy = function(strategy) {
        this.strategy = strategy;
    };
    // 环境类，把请求委托给具体的策略对象
    Bonus.prototype.getBonus = function() {
        return this.strategy.calculate(this.salary); // 把计算具体奖金的工作委托给某个策略对象
    };

    var bonus = new Bonus(); // 实例化一个年终奖对象
    bonus.setSalary(1000); // 添加工资
    bonus.setStrategy(new performanceS()); // 传入绩效等级
    console.log(bonus.getBonus()); // 获得年终奖
    </script>
</body>

</html>

```

### js中的策略模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 传统的策略模式对象，必须要从类中创建，但是JS中可以直接创建，环境类对象也不用从类中创建，可以直接创建
    // 一组策略
    var strategies = {
        "S": function(salary) {
            return salary * 4;
        },
        "A": function(salary) {
            return salary * 3;
        },
        "B": function(salary) {
            return salary * 2;
        }
    };
    // 负责把请求委托给具体的某个策略类
    var calculateBonus = function(level, salary) {
        return strategies[level](salary);
    };
    console.log(calculateBonus("S", 3000)); //12000
    console.log(calculateBonus("A", 1000)); //2000
    </script>
</body>

</html>

```

### 使用策略模式实现缓动动画
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div style="position:absolute;background:blue" id="div">我是div</div>
    <script type="text/javascript">
    // 将各种缓动动画的算法封装起来
    var tween = {
        // t是动画已消耗的时间、b小球原始位置、c小球目标位置、d动画持续的总时间
        linear: function(t, b, c, d) {
            return c * t / d + b;
        },
        easeIn: function(t, b, c, d) {
            return c * (t /= d) * t + b;
        },
        strongEaseIn: function(t, b, c, d) {
            return c * (t /= d) * t * t * t * t + b;
        },
        strongEaseOut: function(t, b, c, d) {
            return c * ((t = t / d - 1) * t * t * t * t + 1) + b;
        },
        sineaseIn: function(t, b, c, d) {
            return c * (t /= d) * t * t + b;
        },
        sineaseOut: function(t, b, c, d) {
            return c * ((t = t / d - 1) * t * t + 1) + b;
        }
    };
    var Animate = function(dom) {
        this.dom = dom; // 进行运动的dom 节点
        this.startTime = 0; // 动画开始时间
        this.startPos = 0; // 动画开始时，dom 节点的位置，即dom 的初始位置
        this.endPos = 0; // 动画结束时，dom 节点的位置，即dom 的目标位置
        this.propertyName = null; // dom 节点需要被改变的css 属性名
        this.easing = null; // 缓动算法
        this.duration = null; // 动画持续时间
    };
    // start负责启动动画
    /**
     * 启动动画
     * @author cgh
     * @time   2017-03-07
     * @param  {String}   propertyName 要运动的属性
     * @param  {Number}   endPos       结束位置
     * @param  {Time}   duration     持续时间
     * @param  {Obj}   easing       缓动算法
     * @return {[type]}                [description]
     */
    Animate.prototype.start = function(propertyName, endPos, duration, easing) {
        this.startTime = +new Date; // 动画启动时间
        this.startPos = this.dom.getBoundingClientRect()[propertyName]; // dom 节点初始位置
        this.propertyName = propertyName; // dom 节点需要被改变的CSS 属性名
        this.endPos = endPos; // dom 节点目标位置
        this.duration = duration; // 动画持续事件
        this.easing = tween[easing]; // 缓动算法
        var self = this;
        var timeId = setInterval(function() { // 启动定时器，开始执行动画
            if (self.step() === false) { // 如果动画已结束，则清除定时器
                clearInterval(timeId);
            }
        }, 1000 / 60);
    };
    // 定义小球每一帧的动画
    Animate.prototype.step = function() {
        var t = +new Date; // 取得当前时间
        if (t >= this.startTime + this.duration) { // (1)
            this.update(this.endPos); // 更新小球的CSS 属性值
            return false;
        }
        var pos = this.easing(t - this.startTime, this.startPos, this.endPos - this.startPos, this.duration);
        // pos 为小球当前位置
        this.update(pos); // 更新小球的CSS 属性值
    };
    // 负责更新动画
    Animate.prototype.update = function(pos) {
        this.dom.style[this.propertyName] = pos + 'px';
    };
    var div = document.getElementById('div');
    var animate = new Animate(div);
    animate.start('left', 500, 1000, 'strongEaseOut');
    // animate.start('top', 1500, 500, 'strongEaseIn');
    </script>
</body>

</html>

```

### 使用策略模式完成表单验证
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <form id="registerForm" method="post">
        请输入用户名：
        <input type="text" name="userName" /> 请输入密码：
        <input type="text" name="password" /> 请输入手机号码：
        <input type="text" name="phoneNumber" />
        <button>提交</button>
    </form>
    <script type="text/javascript">
    // 策略模式的主要目的是将算法的实现和使用分离
    // 但在广义的情况下，也可以用策略模式将一些业务规则封装起来，只要这些业务规则指向的目标一致，并可被替换使用即可。如表单验证
    // var registerForm = document.getElementById('registerForm');
    // registerForm.onsubmit = function() {
    //     if (registerForm.userName.value === '') {
    //         alert('用户名不能为空');
    //         return false;
    //     }
    //     if (registerForm.password.value.length < 6) {
    //         alert('密码长度不能少于6 位');
    //         return false;
    //     }
    //     if (!/(^1[3|5|8][0-9]{9}$)/.test(registerForm.phoneNumber.value)) {
    //         alert('手机号码格式不正确');
    //         return false;
    //     }
    // };
    // 不足很明显，函数比较庞大、缺乏弹性、复用性差
    // 策略模式改进

    // 封装校验逻辑
    /***********************策略对象**************************/
    var strategies = {
        isNonEmpty: function(value, errorMsg) {
            if (value === '') {
                return errorMsg;
            }
        },
        minLength: function(value, length, errorMsg) {
            if (value.length < length) {
                return errorMsg;
            }
        },
        isMobile: function(value, errorMsg) {
            if (!/(^1[3|5|8][0-9]{9}$)/.test(value)) {
                return errorMsg;
            }
        }
    };
    /***********************Validator 类**************************/
    var Validator = function() {
        this.cache = [];
    };
    Validator.prototype.add = function(dom, rules) {
        var self = this;
        for (var i = 0, rule; rule = rules[i++];) {
            (function(rule) {
                var strategyAry = rule.strategy.split(':');
                var errorMsg = rule.errorMsg;
                self.cache.push(function() {
                    var strategy = strategyAry.shift();
                    strategyAry.unshift(dom.value);
                    strategyAry.push(errorMsg);
                    return strategies[strategy].apply(dom, strategyAry);
                });
            })(rule)
        }
    };
    Validator.prototype.start = function() {
        for (var i = 0, validatorFunc; validatorFunc = this.cache[i++];) {
            var errorMsg = validatorFunc();
            if (errorMsg) {
                return errorMsg;
            }
        }
    };
    /***********************客户调用代码**************************/
    var registerForm = document.getElementById('registerForm');
    var validataFunc = function() {
        var validator = new Validator();
        validator.add(registerForm.userName, [{
            strategy: 'isNonEmpty',
            errorMsg: '用户名不能为空'
        }, {
            strategy: 'minLength:6',
            errorMsg: '用户名长度不能小于10 位'
        }]);
        validator.add(registerForm.password, [{
            strategy: 'minLength:6',
            errorMsg: '密码长度不能小于6 位'
        }]);
        validator.add(registerForm.phoneNumber, [{
            strategy: 'isMobile',
            errorMsg: '手机号码格式不正确'
        }]);
        var errorMsg = validator.start();
        return errorMsg;
    }
    registerForm.onsubmit = function() {
        var errorMsg = validataFunc();
        if (errorMsg) {
            alert(errorMsg);
            return false;
        }
    };
    // 策略模式优点
    // 策略模式利用组合、委托和多态等技术和思想，可以有效地避免多重条件选择语句。
    // 策略模式提供了对开放— 封闭原则的完美支持，将算法封装在独立的strategy中，使得它们易于切换，易于理解，易于扩展。
    // 策略模式中的算法也可以复用在系统的其他地方，从而避免许多重复的复制粘贴工作。
    // 在策略模式中利用组合和委托来让Context 拥有执行算法的能力，这也是继承的一种更轻便的替代方案。

    // 缺点
    // 使用策略模式会在程序中增加许多策略类或者策略对象，但实际上这比把它们负责的逻辑堆砌在Context 中要好。
    // 要使用策略模式，必须了解所有的strategy，必须了解各个strategy 之间的不同点，这样才能选择一个合适的strategy,此时strategy 要向客户暴露它的所有实现，这是违反最少知识原则的
    </script>
</body>

</html>

```

## 代理模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 代理模式:为一个对象提供一个代用品或占位符，以便控制对它的访问
    // 应用场景:当不方便直接访问某个对象时
    // 分类:
    // JS中常用:
    // 虚拟代理模式:会把一些开销很大的对象，延迟到真正需要它的时候再创建(常用)
    // 缓存代理模式:可以为一些开销很大的运算结果提供暂时的缓存，在下次运算时，如果传递进来的参数跟之前一致，则可直接返回缓存的结果(常用)
    // 
    // 不常用:
    // 保护代理模式:代理B可以帮A过滤掉一些请求
    // 防火墙代理模式:控制网络资源的访问，保护主体不让"坏人"接近
    // 远程代理模式:为一个对象在不同的地址空间提供局部代表，在Java 中，远程代理可以是另一个虚拟机中的对象。
    // 保护代理模式：用于对象应该有不同访问权限的情况。
    // 智能引用代理模式：取代了简单的指针，它在访问对象时执行一些附加操作，比如计算一个对象被引用的次数。
    // 写时复制代理：通常用于复制一个庞大对象的情况。写时复制代理延迟了复制的过程，当对象被真正修改时，才对它进行复制操作。写时复制代理是虚拟代理的一种变体，DLL（操作系统中的动态链接库）是其典型运用场景。

    // 使用举例:小明托人送花、虚拟代理实现图片预加载、缓存代理异步ajax请求

    // 小明委托B给A送花
    // var Flower = function() {};
    // // 发起者
    // var xiaoMing = {
    //     sendFolower: function(target) {
    //         var flower = new Flower();
    //         target.receiveFlower(flower);
    //     }
    // };
    // // 接收者
    // var A = {
    //     receiveFlower: function(flower) {
    //         console.log("收到花" + flower);
    //     }
    // };
    // // 被委托的对象
    // var B = {
    //     receiveFlower: function(flower) {
    //         A.receiveFlower(flower);
    //     }
    // };
    // xiaoMing.sendFolower(B);

    // 不足，上面这个例子看不出小明自己送和托B送有什么区别，因为B只是简单的把花递给了A
    // 考虑下面情况
    // 在A心情好的时候小明送花，成功率很高。如果心情不好的时候送花，则成功率为0。小明才认识A，并不知道她心情的变化，而A的好朋友B却很了解，所以小明只管把花交给B，B负责监听A，在A心情好的时候将花传递给A

    var Flower = function() {};

    var xiaoMing = {
        sendFolower: function(target) {
            var flower = new Flower();
            target.receiveFlower(flower);
        }
    };

    var A = {
        receiveFlower: function(flower) {
            console.log("收到花" + flower);
        },
        listenGoodMood: function(fn) {
            setTimeout(function() { // 假设1s后心情变好
                fn();
            }, 3000);
        }

    };

    var B = {
        receiveFlower: function(flower) {
            A.listenGoodMood(function() { // 监听A的心情
                A.receiveFlower(flower);
            });
        }
    };
    xiaoMing.sendFolower(B);
    </script>
</body>

</html>
```

### 虚拟代理模式实现图片预加载
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 在Web 开发中，图片预加载是一种常用的技术，如果直接给某个img标签节点设置src属性，由于图片过大或者网络不佳，图片的位置往往有段时间会是一片空白。常见的做法是先用一张loading 片占位，然后用异步的方式加载图片，等图片加载好了再把它填充到img节点里，这种场景就很适合使用虚拟代理
    // 非代理模式实现
    // var MyImage = (function() {
    //     var imgNode = document.createElement('img');
    //     document.body.appendChild(imgNode);
    //     var img = new Image;
    //     img.onload = function() {
    //         imgNode.src = img.src;
    //     };
    //     return {
    //         setSrc: function(src) {
    //             imgNode.src = 'http://img.zcool.cn/community/013cb15648986a32f87512f6d87dc8.gif';
    //             img.src = src;
    //         }
    //     }
    // })();
    // MyImage.setSrc('http://img1.3lian.com/2015/a1/24/d/17.jpg');
    // 上面违反了单一职责原则，MyImage既负责给img设置src外，还负责预加载图片

    // 虚拟代理模式
    // 普通本体对象，对外提供一个setSrc接口
    var myImage = (function() {
        var imgNode = document.createElement('img');
        document.body.appendChild(imgNode);
        return {
            setSrc: function(src) {
                imgNode.src = src;
            }
        }
    })();
    // 代理对象
    var proxyImage = (function() {
        var img = new Image();
        img.onload = function() {
            // 真图请求完成，将真图放到页面上
            myImage.setSrc(this.src);
        }
        return {
            setSrc: function(src) {
                // 设置loading图
                myImage.setSrc('http://img.zcool.cn/community/013cb15648986a32f87512f6d87dc8.gif');
                // 设置真图地址
                img.src = src;
            }
        }
    })();
    proxyImage.setSrc('http://img1.3lian.com/2015/a1/24/d/17.jpg');
    </script>
</body>

</html>

```

### 虚拟代理合并http请求
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <input type="checkbox" id="1"></input>1
    <input type="checkbox" id="2"></input>2
    <input type="checkbox" id="3"></input>3
    <input type="checkbox" id="4"></input>4
    <input type="checkbox" id="5"></input>5
    <input type="checkbox" id="6"></input>6
    <input type="checkbox" id="7"></input>7
    <input type="checkbox" id="8"></input>8
    <input type="checkbox" id="9"></input>9
    <script type="text/javascript">
    // 将多个HTPP请求打包成一个，一次发往服务器
    var synchronousFile = function(id) {
        console.log("开始同步文件,id为:" + id);
    };
    var checkbox = document.getElementsByTagName("input");
    for (var i = 0, c; c = checkbox[i++];) {
        c.onclick = function() {
            // 这样每点一次就同步一次，对服务器伤害很大，可以将多个打包了，再一并发送
            if (this.checked === true) {
                proxySynchronousFile(this.id);
            }
        };
    }
    // 添加代理函数，负责手机一段时间内的请求
    var proxySynchronousFile = (function() {
        var cache = [],
            timer;
        return function(id) {
            // 存入缓存
            cache.push(id);
            if (timer) {
                return;
            }
            timer = setTimeout(function() {
                // 执行同步
                synchronousFile(cache.join(","));
                // 清除
                clearTimeout(timer);
                timer = null;
                cache.length = 0;
            }, 2000);
        }
    })();
    </script>
</body>

</html>

```

### 虚拟代理在惰性加载中的应用
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 在按特定按键时才加载另外一个js文件
    // var cache = [];
    // var miniConsole = {
    //     log: function() {
    //         var args = arguments;
    //         cache.push(function() {
    //             return miniConsole.log.apply(miniConsole, args);
    //         });
    //     }
    // };
    // var handler = function(ev) {
    //     if (ev.keyCode === 32) {
    //         // 按空格才会创建脚本
    //         var script = document.createElement("script");
    //         script.src = "js/33.js";
    //         script.onload = function() {
    //             for (var i = 0, fn; fn = cache[i++];) {
    //                 fn();
    //             }
    //         };
    //         document.body.appendChild(script);
    //     }
    // };
    // document.body.addEventListener("keydown", handler, false);
    // miniConsole.log(3);
    // 这样的问题是在每次按空格都会重复创建script
    // 改进
    var miniConsole = (function() {
        var cache = [];
        var handler = function(ev) {
            if (ev.keyCode === 32) {
                console.log(ev);
                var script = document.createElement('script');
                script.onload = function() {
                    for (var i = 0, fn; fn = cache[i++];) {
                        fn();
                    }
                };
                script.src = 'js/33.js';
                document.body.appendChild(script);
                document.body.removeEventListener('keydown', handler); // 只加载一次33.js
            }
        };
        document.body.addEventListener('keydown', handler, false);
        return {
            log: function() {
                var args = arguments;
                cache.push(function() {
                    return miniConsole.log.apply(miniConsole, args);
                });
            }
        }
    })();
    miniConsole.log(11); // 开始打印log
    </script>
</body>

</html>

```

### 缓存代理
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 可以为一些开销很大的运算结果提供暂时的缓存，在下次运算时，如果传递进来的参数跟之前一致，则可直接返回缓存的结果
    // 计算乘积
    var mult = function() {
        var a = 1;
        for (var i = 0, len = arguments.length; i < len; i++) {
            a *= arguments[i];
        }
        return a;
    };
    console.log(mult(2, 3));
    console.log(mult(2, 3, 4));
    // 代理函数
    var proxyMult = (function() {
        var cache = {};
        return function() {
            var args = Array.prototype.join.call(arguments, ",");
            if (args in cache) {
                console.log("来自缓存");
                return cache[args];
            }
            return cache[args] = mult.apply(this, arguments);
        };
    })();
    console.log(proxyMult(1, 2, 3, 4));
    console.log(proxyMult(1, 2, 3, 4));
    </script>
</body>

</html>

```

### 用高阶函数动态创建缓存代理
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    /**************** 计算乘积 *****************/
    var mult = function() {
        var a = 1;
        for (var i = 0, l = arguments.length; i < l; i++) {
            a = a * arguments[i];
        }
        return a;
    };
    /**************** 计算加和 *****************/
    var plus = function() {
        var a = 0;
        for (var i = 0, l = arguments.length; i < l; i++) {
            a = a + arguments[i];
        }
        return a;
    };
    /**************** 创建缓存代理的工厂 *****************/
    var createProxyFactory = function(fn) {
        var cache = {};
        return function() {
            var args = Array.prototype.join.call(arguments, ',');
            if (args in cache) {
                console.log("来自缓存");
                return cache[args];
            }
            return cache[args] = fn.apply(this, arguments);
        }
    };
    var proxyMult = createProxyFactory(mult),
        proxyPlus = createProxyFactory(plus);
    console.log((proxyMult(1, 2, 3, 4))); // 输出：24
    console.log((proxyMult(1, 2, 3, 4))); // 输出：24
    console.log((proxyPlus(1, 2, 3, 4))); // 输出：10
    console.log((proxyPlus(1, 2, 3, 4))); // 输出：10
    </script>
</body>

</html>

```

## 迭代器模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 迭代器模式:提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。它将迭代过程从业务逻辑中抽象出来。
    // 分类:
    // 内部迭代器:内部已经定义好了迭代规则，它完全接手整个迭代过程，外部只需要一次初始调用。优点:不用关心迭代器内部实现。缺点:优点也是缺点，因为迭代器中的迭代规则以被提前规定好
    // 外部迭代器:外部迭代器必须显式地请求迭代下一个元素
    // 使用场景是:对于集合内部结果常常变化各异，我们不想暴露其内部结构的话，但又响让客户代码透明底访问其中的元素，这种情况下我们可以使用迭代器模式。
    // 使用举例:很多语言都内置了迭代器如Array.prototype.forEach、jQuery中的$.each

    // 实现自己的each(一个内部迭代器)
    var each = function(ary, callback) {
        for (var i = 0, len = ary.length; i < len; i++) {
            callback.call(ary[i], i, ary[i]); // 在ary[i]下调用callback,传入索引i和元素ary[i]
        }
    };
    // each([1, 2, 3, 4], function(i, n) {
    //     console.log(([i, n]));
    // });

    // 用each完成判断两个数组是否相等
    // var compare = function(ary1, ary2) {
    //     // 长度不同肯定不相等
    //     if (ary1.length !== ary2.length) {
    //         throw new Error("不相等");
    //     }
    //     // 比较内部元素值
    //     each(ary1, function(i, n) {
    //         if (n !== ary2[i]) {
    //             throw new Error("不相等");
    //         }
    //     });
    //     console.log("相等");
    // };
    // compare([1, 2, 3], [1, 2, 4]);
    // 使用外部迭代器改写判断数组相等
    var Iterator = function(obj) {
        var current = 0;
        var next = function() {
            current += 1;
        };
        var isDone = function() {
            return current >= obj.length;
        };
        var getCurrItem = function() {
            return obj[current];
        };
        return {
            next: next,
            isDone: isDone,
            getCurrItem: getCurrItem
        }
    };
    var compare = function(iterator1, iterator2) {
        while (!iterator1.isDone() && !iterator2.isDone()) {
            if (iterator1.getCurrItem() !== iterator2.getCurrItem()) {
                throw new Error("不相等");
            }
            iterator1.next();
            iterator2.next();
        }
        console.log("相等");
    };
    var iterator1 = Iterator([1, 2, 3]);
    var iterator2 = Iterator([1, 2, 3]);
    compare(iterator1, iterator2); // 输出：iterator1 和iterator2 相等
    </script>
</body>

</html>

```

### 迭代类数组对象和字面量对象
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 本质上，只要对象拥有length属性的都可以迭代，arguments和对象字面量也可以迭代，仿照jquery的$.each写一个通用的迭代器
    each = function(obj, callback) {
        var value, i = 0,
            length = obj.length,
            isArray = isArraylike(obj);
        if (isArray) {
            // 类数组对象
            for (; i < length; i++) {
                value = callback.call(obj[i], i, obj[i]);
                // 约定回调函数返回false时，终止迭代
                if (value === false) {
                    break;
                }
            }
        } else {
            // 普通对象
            for (i in obj) {
                value = callback.call(obj[i], i, obj[i]);
                if (value === false) {
                    break;
                }
            }
        }
        return obj;
    };
    </script>
</body>

</html>

```

### 倒序迭代器
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 倒序迭代器
    var reverseEach = function(ary, callback) {
        for (var l = ary.length - 1; l >= 0; l--) {
            callback(l, ary[l]);
        }
    };
    reverseEach([0, 1, 2], function(i, n) {
        console.log(n); // 分别输出：2, 1 ,0
    });
    </script>
</body>

</html>

```

### 迭代器模式实现上传空间选择
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // var getUploadObj = function() {
    //     try {
    //         return new ActiveXObject("TXFTNActiveX.FTNUpload"); // IE 上传控件
    //     } catch (e) {
    //         if (supportFlash()) { // supportFlash 函数未提供
    //             var str = '<object type="application/x-shockwave-flash"></object>';
    //             return $(str).appendTo($('body'));
    //         } else {
    //             var str = '<input name="file" type="file"/>'; // 表单上传
    //             return $(str).appendTo($('body'));
    //         }
    //     }
    // };
    // 上面的难以阅读、扩展、维护
    // 将获取上传对象封装起来
    var getActiveUploadObj = function() {
        try {
            return new ActiveXObject("TXFTNActiveX.FTNUpload"); // IE 上传控件
        } catch (e) {
            return false;
        }
    };
    var getFlashUploadObj = function() {
        if (isSupportFlash()) { // isSupportFlash 函数未提供
            var str = '<object type="application/x-shockwave-flash"></object>';
            return document.body.innerHTML += str;
        }
        return false;
    };
    var getFormUpladObj = function() {
        var str = '<input name="file" type="file" class="ui-file"/>'; // 表单上传
        return document.body.innerHTML += str;
    };
    // 迭代器
    var iteratorUploadObj = function() {
        for (var i = 0, fn; fn = arguments[i++];) {
            var uploadObj = fn();
            if (uploadObj !== false) {
                return uploadObj;
            }
        }
    };
    var uploadObj = iteratorUploadObj(getActiveUploadObj, getFlashUploadObj, getFormUpladObj);
    console.log(uploadObj);
    </script>
</body>

</html>

```

## 观察者模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 观察者模式又叫发布-订阅模式。它定义对象间的一种在对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。JS中一般用事件模型来替代传统的观察者模式
    // 使用场景:应用于异步编程中，这是一种替代传递回调函数的方案。可以取代对象之间硬编码的通知机制，一个对象不用再显式地调用另外一个对象的某个接口
    // 使用举例:买房，到达预售时间，售楼部(发布者)会打电话(通过花名册，一个缓存列表，拨打电话过程就是在执行回调函数的过程)通知买房者(订阅者)可以购买了。

    // var salesOffices = {}; // 定义售楼处，发布者
    // salesOffices.clientList = []; // 缓存列表，花名册，存放订阅者的回调函数
    // salesOffices.listen = function(fn) { // 增加订阅者
    //     this.clientList.push(fn); // 订阅者的回调函数添加到缓存列表中
    // };
    // salesOffices.trigger = function() { // 发布消息
    //     for (var i = 0, fn; fn = this.clientList[i++];) {
    //         fn.apply(this, arguments); // arguments是发布消息时带上的参数
    //     }
    // };
    // salesOffices.listen(function(price, squareMeter) { // 小明订阅消息
    //     console.log("我是小明订阅的");
    //     console.log('价格= ' + price);
    //     console.log('squareMeter= ' + squareMeter);
    // });
    // salesOffices.listen(function(price, squareMeter) { // 小红订阅消息
    //     console.log("我是小红订阅的");
    //     console.log('价格= ' + price);
    //     console.log('squareMeter= ' + squareMeter);
    // });
    // // 发布消息，此时会通知所有订阅者
    // salesOffices.trigger(2000000, 88); // 输出：200 万，88 平方米
    // salesOffices.trigger(3000000, 110); // 输出：300 万，110 平方米

    // 上面出现的问题时，假如小明只想买88平的，售楼部却把110平的也推送给他了
    var salesOffices = {};
    salesOffices.clientList = {};
    salesOffices.listen = function(key, fn) { // key为订阅者想订阅的某个消息类型，例如小明想订阅的88平
        if (!this.clientList[key]) { // 如果还没订阅过此类消息，则给该类消息创建一个缓存列表
            this.clientList[key] = [];
        }
        this.clientList[key].push(fn); // 将订阅的消息添加到缓存列表中
    };
    salesOffices.trigger = function() {
        var key = Array.prototype.shift.call(arguments); // 取出消息类型
        var fns = this.clientList[key]; // 此消息类型对应的回调函数列表
        if (!fns || fns.length === 0) { // 没有订阅该消息，则返回
            return false;
        }
        for (var i = 0, fn; fn = fns[i++];) {
            fn.apply(this, arguments); // 发布消息
        }
    };

    salesOffices.listen('squareMeter88', function(price) { // 小明订阅88 平方米房子的消息
        console.log('价格= ' + price); // 输出： 2000000
    });

    salesOffices.listen('squareMeter88', function(price) { // 小周订阅88 平方米房子的消息
        console.log('价格= ' + price); // 输出： 2000000
    });

    salesOffices.listen('squareMeter110', function(price) { // 小红订阅110 平方米房子的消息
        console.log('价格= ' + price); // 输出： 3000000
    });

    salesOffices.trigger('squareMeter88', 2000000); // 发布88 平方米房子的价格，此时只有小明和小周会收到订阅消息

    salesOffices.trigger('squareMeter110', 3000000); // 发布110 平方米房子的价格，只有小红能收到消息
    </script>
</body>

</html>

```

### 观察者的通用实现
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    var event = {
        clientList: {},
        listen: function(key, fn) {
            if (!this.clientList[key]) {
                this.clientList[key] = [];
            }
            this.clientList[key].push(fn);
        },
        trigger: function() {
            var key = Array.prototype.shift.call(arguments);
            var fns = this.clientList[key];
            if (!fns || fns.length === 0) {
                return false;
            }
            for (var i = 0, fn; fn = fns[i++];) {
                fn.apply(this, arguments);
            }
        },
        remove: function(key, fn) {
            var fns = this.clientList[key];
            if (!fns) {
                return false;
            }
            if (!fn) {
                fns && (fns.length = 0);
            } else {
                for (var l = fns.length - 1; l >= 0; l--) { // 反向遍历订阅的回调函数列表
                    var _fn = fns[l];
                    if (_fn === fn) {
                        fns.splice(l, 1); // 删除订阅者的回调函数
                    }
                }
            }
        }
    };


    var cloneObj = function(sourceObject) {
        var str, newObj = sourceObject.constructor === Array ? [] : {};
        if (typeof sourceObject !== 'object') {
            return;
        } else {
            for (var i in sourceObject) {
                newObj[i] = typeof sourceObject[i] === 'object' ?
                    cloneObj(sourceObject[i]) : sourceObject[i];
            }
        }
        return newObj;
    };



    var salesOffices = cloneObj(event);

    salesOffices.listen('squareMeter88', fn1 = function(price) { // 小明订阅售楼1 88平消息
        console.log('售楼1通知小明价格= ' + price);
    });
    salesOffices.listen('squareMeter88', fn2 = function(price) { // 小周订阅售楼1 88平消息
        console.log('售楼1通知小周价格= ' + price);
    });
    salesOffices.listen('squareMeter100', fn3 = function(price) { // 小红订阅售楼1 100平消息
        console.log('售楼1通知小红价格= ' + price);
    });

    // 售楼1发布88平米的消息
    salesOffices.trigger('squareMeter88', 20000); // 输出：20000
    // 售楼1发布100平米的消息
    salesOffices.trigger('squareMeter100', 30000); // 输出：20000
    console.log("=======================");;

    console.log("删除小明订阅售楼1");
    salesOffices.remove('squareMeter88', fn1); // 删除小明的订阅

    salesOffices.trigger('squareMeter88', 20000); // 输出：20000
    salesOffices.trigger('squareMeter100', 30000); // 输出：20000


    console.log("=======================");

    var salesOffices2 = cloneObj(event);
    salesOffices2.listen('squareMeter88', fn21 = function(price) { // 小明订阅售楼2 88平消息
        console.log('售楼2通知小明价格= ' + price);
    });
    salesOffices2.listen('squareMeter88', fn22 = function(price) { // 小红订阅售楼2 100平消息
        console.log('售楼2通知小红价格= ' + price);
    });
    // 发布88平米消息
    salesOffices2.trigger('squareMeter88', 25000); // 输出：25000
    console.log("=======================");;

    console.log("删除小明订阅售楼2");
    salesOffices2.remove('squareMeter88', fn21); // 删除小明的订阅

    salesOffices2.trigger('squareMeter88', 35000); // 输出：35000
    </script>
</body>

</html>

```

### 使用观察者模式实现网站登录
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <p>假如我们正在开发一个商城网站，网站里有header 头部、nav 导航、消息列表、购物车等模块。这几个模块的渲染有一个共同的前提条件，就是必须先用ajax异步请求获取用户的登录信息。 这是很正常的，比如用户的名字和头像要显示在header 模块里，而这两个字段都来自用户登录后返回的信息。至于ajax 请求什么时候能成功返回用户信息，这点我们没有办法确定</p>
    <script type="text/javascript" src="js/jquery-2.2.4.min.js"></script>
    <script type="text/javascript" src="js/mock-min.js"></script>
    <script type="text/javascript">
    var event = {
        clientList: {},
        listen: function(key, fn) {
            if (!this.clientList[key]) {
                this.clientList[key] = [];
            }
            this.clientList[key].push(fn);
        },
        trigger: function() {
            var key = Array.prototype.shift.call(arguments);
            var fns = this.clientList[key];
            if (!fns || fns.length === 0) {
                return false;
            }
            for (var i = 0, fn; fn = fns[i++];) {
                fn.apply(this, arguments);
            }
        },
        remove: function(key, fn) {
            var fns = this.clientList[key];
            if (!fns) {
                return false;
            }
            if (!fn) {
                fns && (fns.length = 0);
            } else {
                for (var l = fns.length - 1; l >= 0; l--) { // 反向遍历订阅的回调函数列表
                    var _fn = fns[l];
                    if (_fn === fn) {
                        fns.splice(l, 1); // 删除订阅者的回调函数
                    }
                }
            }
        }
    };
    var cloneObj = function(sourceObject) {
        var str, newObj = sourceObject.constructor === Array ? [] : {};
        if (typeof sourceObject !== 'object') {
            return;
        } else {
            for (var i in sourceObject) {
                newObj[i] = typeof sourceObject[i] === 'object' ?
                    cloneObj(sourceObject[i]) : sourceObject[i];
            }
        }
        return newObj;
    };
    // 使用 Mock模拟数据
    Mock.setup({
        timeout: '500-1000'
    });
    Mock.mock('http://test.cn', {
        "userName": "@cname",
        "sex|1": ["男", "女"],
        "avator": Mock.Random.image('100x100', '#894FC4', '#FFF', 'png', '头像'),
        "test|1-3": 3
    });
    // 模拟请求
    $.ajax({
        url: 'http://test.cn',
        dataType: 'json'
    }).done(function(data, status, xhr) {
        // 请求成功，触发登录成功
        login.trigger("loginSucc", data);
    });

    var login = cloneObj(event);

    var header = (function() {
        // 订阅登录的登录成功消息
        login.listen("loginSucc", function(data) {
            header.setAvator(data.avator);
        });
        return {
            setAvator: function(data) {
                console.log("设置Header模块头像");
            }
        }
    })();
    var nav = (function() {
        login.listen("loginSucc", function(data) {
            nav.setAvator(data.avator);
        });
        return {
            setAvator: function(data) {
                console.log("设置Nav模块头像");
            }
        }
    })();
    </script>
</body>

</html>

```

### 全局发布订阅-对象
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="count">点我</button>
    <div id="show"></div>
    <script type="text/javascript">
    var Event = (function() {
        var clientList = {},
            listen, trigger, remove;
        listen = function(key, fn) {
            if (!clientList[key]) {
                clientList[key] = [];
            }
            clientList[key].push(fn);
        };
        trigger = function() {
            var key = Array.prototype.shift.call(arguments);
            var fns = clientList[key];
            if (!fns || fns.length === 0) {
                return false;
            }
            for (var i = 0, fn; fn = fns[i++];) {
                fn.apply(this, arguments);
            }
        };
        remove = function(key, fn) {
            var fns = clientList[key];
            if (!fns) {
                return false;
            }
            if (!fn) {
                fns && (fns.length = 0);
            } else {
                for (var l = fns.length - 1; l >= 0; l--) {
                    var _fn = fns[l];
                    if (_fn === fn) {
                        fns.spice(l, 1);
                    }
                }
            }
        };
        return {
            listen: listen,
            trigger: trigger,
            remove: remove
        };
    })();
    Event.listen('squareMeter88', function(price) { // 小红订阅消息
        console.log('价格= ' + price); // 输出：'价格=2000000'
    });
    Event.trigger('squareMeter88', 2000000); // 售楼处发布消息

    // 利用全局的发布-订阅对象，可以实现模块间的通信
    // 点击按钮，div中显示总的点击次数
    var a = (function() {
        var count = 0;
        var button = document.getElementById("count");
        button.onclick = function() {
            Event.trigger("add", count++);
        };
    })();
    var b = (function() {
        var div = document.getElementById("show");
        Event.listen("add", function(count) {
            div.innerHTML = count;
        });
    })();
    </script>
</body>

</html>

```

### 支持先发布后订阅、命名空间的观察者模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <pre>
	有时，需要先发布，再订阅。如42中的登录，获取登录信息可能很快，但其他模块可能加载的很慢，就会造成已经发布了登录成功的消息，但其他模块还没有订阅的错误。
	所以需要让观察者模式支持先缓存要发布的消息，等到有模块订阅消息的时候再正式发布消息。
	另外，Event对象中只存在一个clientList，当订阅多了，就可能出现命名冲突的问题。所以还需要支持创建命名空间。
</pre>
    <script>
    var Event = (function() {
        var global = this,
            Event, _default = "default";
        Event = function() {
            var _listen, _trigger, _remove, _slice = Array.prototype.slice,
                _shift = Array.prototype.shift,
                _unshift = Array.prototype.unshift,
                namespaceCache = {},
                _create, find, each = function(ary, fn) {
                    var ret;
                    for (var i = 0, l = ary.length; i < l; i++) {
                        var n = ary[i];
                        ret = fn.call(n, i, n);
                    }
                    return ret;
                };
            _listen = function(key, fn, cache) {
                if (!cache[key]) {
                    cache[key] = [];
                }
                cache[key].push(fn);
            };
            _remove = function(key, cache, fn) {
                if (cache[key]) {
                    if (fn) {
                        for (var i = cache[key].length; i >= 0; i--) {
                            if (cache[key][i] === fn) {
                                cache[key].splice(i, 1);
                            }
                        }
                    } else {
                        cache[key] = {};
                    }
                }
            };
            _trigger = function() {
                var cache = _shift.call(arguments),
                    key = _shift.call(arguments),
                    args = arguments,
                    _self = this,
                    ret, stack = cache[key];
                if (!stack || !stack.length) {
                    return;
                }
                return each(stack, function() {
                    return this.apply(_self, args);
                });
            };
            _create = function(namespace) {
                var namespace = namespace || _default;
                var cache = {},
                    offlineStack = [], // 离线事件 
                    ret = {
                        listen: function(key, fn, last) {
                            _listen(key, fn, cache);
                            if (offlineStack === null) {
                                return;
                            }
                            if (last === "last") {
                                offlineStack.length && offlineStack.pop()();
                            } else {
                                each(offlineStack, function() {
                                    this();
                                });
                            }
                            offlineStack = null;
                        },
                        one: function(key, fn, last) {
                            _remove(key, cache);
                            this.listen(key, fn, last);
                        },
                        remove: function(key, fn) {
                            _remove(key, cache, fn);
                        },
                        trigger: function() {
                            var fn, args, _self = this;
                            _unshift.call(arguments, cache);
                            args = arguments;
                            fn = function() {
                                return _trigger.apply(_self, args);
                            };
                            if (offlineStack) {
                                return offlineStack.push(fn);
                            }
                            return fn();
                        }
                    };
                return namespace ?
                    (namespaceCache[namespace] ? namespaceCache[namespace] :
                        namespaceCache[namespace] = ret) : ret;
            };
            return {
                create: _create,
                one: function(key, fn, last) {
                    var event = this.create();
                    event.one(key, fn, last);
                },
                remove: function(key, fn) {
                    var event = this.create();
                    event.remove(key, fn);
                },
                listen: function(key, fn, last) {
                    var event = this.create();
                    event.listen(key, fn, last);
                },
                trigger: function() {
                    var event = this.create();
                    event.trigger.apply(this, arguments);
                }
            };
        }();
        return Event;
    })();
    /************** 先发布后订阅 ********************/
    Event.trigger('click', 1);
    Event.listen('click', function(a) {
        console.log(a); // 输出：1
    });
    /************** 使用命名空间 ********************/
    Event.create('namespace1').listen('click', function(a) {
        console.log(a); // 输出：1
    });
    Event.create('namespace1').trigger('click', 1);
    Event.create('namespace2').listen('click', function(a) {
        console.log(a); // 输出：2
    });
    Event.create('namespace2').trigger('click', 2);
    </script>
</body>

</html>

```

## 命令模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button1">点击按钮1</button>
    <button id="button2">点击按钮2</button>
    <button id="button3">点击按钮3</button>
    <script>
    // 命令模式:命令模式是最简单和优雅的模式之一，命令模式中的命令（command）指的是一个执行某些特定事情的指令。
    // 使用场景:有时候需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是什么。此时希望用一种松耦合的方式来设计程序，使得请求发送者和请求接收者能够消除彼此之间的耦合关系。
    // 使用举例:拿订餐来说，客人需要向厨师发送请求，但是完全不知道这些厨师的名字和联系方式，也不知道厨师炒菜的方式和步骤。 命令模式把客人订餐的请求封装成command 对象，也就是订餐中的订单对象。这个对象可以在程序中被四处传递，就像订单可以从服务员手中传到厨师的手中。这样一来，客人不需要知道厨师的名字，从而解开了请求调用者和请求接收者之间的耦合关系。

    // 菜单程序
    // 绘制按钮和点击按钮的具体行为分离
    // 点击了按钮之后，必须向某些负责具体行为的对象发送请求，这些对象就是请求的接收者。但是目前并不知道接收者是什么对象，也不知道接收者究竟会做什么。此时我们需要借助命令对象的帮助，以便解开按钮和负责具体行为对象之间的耦合。

    // // 传统面向对象语言中的命令模式
    // var button1 = document.getElementById('button1'),
    //     button2 = document.getElementById('button2'),
    //     button3 = document.getElementById('button3');
    // // 负责往按钮上安装命令
    // var setCommand = function(button, command) {
    //     button.onclick = function() {
    //         command.execute();
    //     };
    // };
    // var MenuBar = {
    //     refresh: function() {
    //         console.log('刷新菜单目录');
    //     }
    // };
    // var SubMenu = {
    //     add: function() {
    //         console.log('增加子菜单');
    //     },
    //     del: function() {
    //         console.log('删除子菜单');
    //     }
    // };
    // // 构建命令
    // var RefreshMenuBarCommand = function(receiver) {
    //     this.receiver = receiver;
    // };
    // RefreshMenuBarCommand.prototype.execute = function() {
    //     this.receiver.refresh();
    // };
    // var AddSubMenuCommand = function(receiver) {
    //     this.receiver = receiver;
    // };
    // AddSubMenuCommand.prototype.execute = function() {
    //     this.receiver.add();
    // };
    // var DelSubMenuCommand = function(receiver) {
    //     this.receiver = receiver;
    // };
    // DelSubMenuCommand.prototype.execute = function() {
    //     console.log('删除子菜单');
    // };
    // // 实例化命令
    // var refreshMenuBarCommand = new RefreshMenuBarCommand(MenuBar);
    // var addSubMenuCommand = new AddSubMenuCommand(SubMenu);
    // var delSubMenuCommand = new DelSubMenuCommand(SubMenu);
    // // 安装命令
    // setCommand(button1, refreshMenuBarCommand);
    // setCommand(button2, addSubMenuCommand);
    // setCommand(button3, delSubMenuCommand);

    // // 在js中使用闭包实现命令模式
    // var button1 = document.getElementById('button1');
    // var setCommand = function(button, func) {
    //     button.onclick = function() {
    //         func();
    //     };
    // }
    // var MenuBar = {
    //     refresh: function() {
    //         console.log("刷新菜单界面");
    //     }
    // };
    // var RefreshMenuBarCommand = function(receiver) {
    //     return function() {
    //         receiver.refresh();
    //     };
    // };
    // var refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);
    // setCommand(button1, refreshMenuBarCommand);

    // 为了更明确的表达当前正在用命令模式，或者将来有可能提供撤销操作时，可以将执行函数改为调用execute方法
    var RefreshMenuBarCommand = function(receiver) {
        return {
            execute: function() {
                receiver.refresh();
            }
        }
    };
    var button1 = document.getElementById('button1');
    var setCommand = function(button, command) {
        button.onclick = function() {
            command.execute();
        };
    };
    var MenuBar = {
        refresh: function() {
            console.log("刷新菜单界面");
        }
    };
    var refreshMenuBarCommand = RefreshMenuBarCommand(MenuBar);
    setCommand(button1, refreshMenuBarCommand);
    </script>
</body>

</html>

```

### 带撤销的命令模式
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div id="ball" style="position:absolute;background:#000;width:50px;height:50px"></div>
    输入小球移动后的位置：
    <input id="pos" />
    <button id="moveBtn">开始移动</button>
    <!--增加取消按钮-->
    <button id="cancelBtn">cancel</button>
    <script type="text/javascript">
    var tween = {
        linear: function(t, b, c, d) {
            return c * t / d + b;
        },
        easeIn: function(t, b, c, d) {
            return c * (t /= d) * t + b;
        },
        strongEaseIn: function(t, b, c, d) {
            return c * (t /= d) * t * t * t * t + b;
        },
        strongEaseOut: function(t, b, c, d) {
            return c * ((t = t / d - 1) * t * t * t * t + 1) + b;
        },
        sineaseIn: function(t, b, c, d) {
            return c * (t /= d) * t * t + b;
        },
        sineaseOut: function(t, b, c, d) {
            return c * ((t = t / d - 1) * t * t + 1) + b;
        }
    };
    var Animate = function(dom) {
        this.dom = dom;
        this.startTime = 0;
        this.startPos = 0;
        this.endPos = 0;
        this.propertyName = null;
        this.easing = null;
        this.duration = null;
    };

    Animate.prototype.start = function(propertyName, endPos, duration, easing) {
        this.startTime = +new Date;
        this.startPos = this.dom.getBoundingClientRect()[propertyName];
        this.propertyName = propertyName;
        this.endPos = endPos;
        this.duration = duration;
        this.easing = tween[easing];
        var self = this;
        var timeId = setInterval(function() {
            if (self.step() === false) {
                clearInterval(timeId);
            }
        }, 1000 / 60);
    };
    Animate.prototype.step = function() {
        var t = +new Date;
        if (t >= this.startTime + this.duration) {
            this.update(this.endPos);
            return false;
        }
        var pos = this.easing(t - this.startTime, this.startPos, this.endPos - this.startPos, this.duration);
        this.update(pos);
    };
    Animate.prototype.update = function(pos) {
        this.dom.style[this.propertyName] = pos + 'px';
    };

    // 以上为运动算法
    // 下面通过命令模式实现运动以及撤销功能

    var ball = document.getElementById('ball');
    var pos = document.getElementById('pos');
    var moveBtn = document.getElementById('moveBtn');
    var cancelBtn = document.getElementById('cancelBtn');
    var MoveCommand = function(receiver, pos) {
        this.receiver = receiver;
        this.pos = pos;
        this.oldPos = null;
    };
    MoveCommand.prototype.execute = function() {
        this.receiver.start('left', this.pos, 1000, 'strongEaseOut');
        // 记录小球开始移动前的位置
        this.oldPos = this.receiver.dom.getBoundingClientRect()[this.receiver.propertyName];
    };
    MoveCommand.prototype.undo = function() {
        // 回到小球移动前记录的位置
        this.receiver.start('left', this.oldPos, 1000, 'strongEaseOut');
    };
    var moveCommand;
    moveBtn.onclick = function() {
        var animate = new Animate(ball);
        moveCommand = new MoveCommand(animate, pos.value);
        moveCommand.execute();
    };
    cancelBtn.onclick = function() {
        moveCommand.undo(); // 撤销命令
    };
    </script>
</body>

</html>

```

### 使用命令模式实现重做
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="replay">播放录像</button>
    <script>
    var Ryu = {
        attack: function() {
            console.log('攻击');
        },
        defense: function() {
            console.log('防御');
        },
        jump: function() {
            console.log('跳跃');
        },
        crouch: function() {
            console.log('蹲下');
        }
    };
    var makeCommand = function(receiver, state) { // 创建命令
        return function() {
            receiver[state]();
        }
    };
    var commands = {
        "119": "jump", // W
        "115": "crouch", // S
        "97": "defense", // A
        "100": "attack" // D
    };
    var commandStack = []; // 保存命令的堆栈
    document.onkeypress = function(ev) {
        var keyCode = ev.keyCode,
            command = makeCommand(Ryu, commands[keyCode]);
        if (command) {
            command(); // 执行命令
            commandStack.push(command); // 将刚刚执行过的命令保存进堆栈
        }
    };
    document.getElementById('replay').onclick = function() { // 点击播放录像
        var command;
        while (command = commandStack.shift()) { // 从堆栈里依次取出命令并执行
            command();
        }
    };
    </script>
</body>

</html>

```

### 宏命令
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    var closeDoorCommand = {
        execute: function() {
            console.log('关门');
        }
    };
    var openPcCommand = {
        execute: function() {
            console.log('开电脑');
        }
    };
    var openQQCommand = {
        execute: function() {
            console.log('登录QQ');
        }
    };
    var MacroCommand = function() {
        return {
            commandList: [],
            add: function(command) {
                this.commandList.push(command);
            },
            execute: function() {
                for (var i = 0, command; command = this.commandList[i++];) {
                    command.execute();
                }
            }
        };
    };
    var macroCommand = MacroCommand();
    macroCommand.add(closeDoorCommand);
    macroCommand.add(openPcCommand);
    macroCommand.add(openQQCommand);
    macroCommand.execute();
    </script>
</body>

</html>

```

## 组合模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 组合模式:组合模式就是用小的子对象来构建更大的对象，而这些小的子对象本身也许是由更小的“孙对象”构成的。
    // 组合模式将对象组合成树形结构，以表示“部分-整体”的层次结构。组合模式可以非常方便地描述对象部分-整体层次结构。 除了用来表示树形结构之外，组合模式的另一个好处是通过对象的多态性表现，使得用户对单个对象和组合对象的使用具有一致性。利用对象多态性统一对待组合对象和单个对象。利用对象的多态性表现，可以使客户端忽略组合对象和单个对象的不同。在组合模式中，客户将统一地使用组合结构中的所有对象，而不需要关心它究竟是组合对象还是单个对象。
    // 使用场景:表示对象的部分-整体层次结构。组合模式可以方便地构造一棵树来表示对象的部分-整体结构。特别是我们在开发期间不确定这棵树到底存在多少层次的时候。在树的构造最终完成之后，只需要通过请求树的最顶层对象，便能对整棵树做统一的操作。在组合模式中增加和删除树的节点非常方便，并且符合开放-封闭原则;客户希望统一对待树中的所有对象。组合模式使客户可以忽略组合对象和叶对象的区别，客户在面对这棵树的时候，不用关心当前正在处理的对象是组合对象还是叶对象，也就不用写一堆if、else语句来分别处理它们。组合对象和叶对象会各自做自己正确的事情，这是组合模式最重要的能力。
    // 使用举例:扫描文件夹,文件夹和文件之间的关系，非常适合用组合模式来描述。文件夹里既可以包含文件，又可以包含其他文件夹

    /***************文件夹*******************/
    var Folder = function(name) {
        this.name = name;
        this.files = [];
    };
    Folder.prototype.add = function(file) {
        this.files.push(file);
    };
    Folder.prototype.scan = function() {
        console.log("开始扫描文件夹:" + this.name);
        for (var i = 0, file, files = this.files; file = files[i++];) {
            file.scan();
        }
    };
    /*****************文件*******************/
    var File = function(name) {
        this.name = name;
    };
    File.prototype.add = function() {
        throw new Error("文件下面不能再添加文件");
    };
    File.prototype.scan = function() {
        console.log("开始扫描文件:" + this.name);
    };
    /****创建文件夹及文件****/
    var folder = new Folder('学习资料');
    var folder1 = new Folder('JavaScript');
    var folder2 = new Folder('jQuery');
    var file1 = new File('JavaScript 设计模式与开发实践');
    var file2 = new File('精通jQuery');
    var file3 = new File('重构与模式')
    folder1.add(file1);
    folder2.add(file2);
    folder.add(folder1);
    folder.add(folder2);
    var folder3 = new Folder('Nodejs');
    var file4 = new File('深入浅出Node.js');
    folder3.add(file4);
    var file5 = new File('JavaScript 语言精髓与编程实践');
    folder.add(folder3);
    folder.add(file5);
    /****扫描文件夹****/
    folder.scan();
    </script>
</body>

</html>

```

### 组合模式-引用父对象
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 在49的基础上要添加删除功能，例如删除某个文件的时候，其实是在其上层文件夹中删除该文件。此时文件就必须要保存上层对象的引用
    var Folder = function(name) {
        this.name = name;
        this.parent = null; //增加this.parent 属性
        this.files = [];
    };
    Folder.prototype.add = function(file) {
        file.parent = this; //设置父对象
        this.files.push(file);
    };
    Folder.prototype.scan = function() {
        console.log('开始扫描文件夹: ' + this.name);
        for (var i = 0, file, files = this.files; file = files[i++];) {
            file.scan();
        }
    };
    // 增加删除功能
    Folder.prototype.remove = function() {
        if (!this.parent) { //根节点或者树外的游离节点
            return;
        }
        for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
            var file = files[l];
            if (file === this) {
                files.splice(l, 1);
            }
        }
    };
    var File = function(name) {
        this.name = name;
        this.parent = null;
    };
    File.prototype.add = function() {
        throw new Error('不能添加在文件下面');
    };
    File.prototype.scan = function() {
        console.log('开始扫描文件: ' + this.name);
    };
    File.prototype.remove = function() {
        if (!this.parent) { //根节点或者树外的游离节点
            return;
        }
        for (var files = this.parent.files, l = files.length - 1; l >= 0; l--) {
            var file = files[l];
            if (file === this) {
                files.splice(l, 1);
            }
        }
    };
    var folder = new Folder('学习资料');
    var folder1 = new Folder('JavaScript');
    var file1 = new Folder('深入浅出Node.js');

    folder1.add(new File('JavaScript 设计模式与开发实践'));
    folder.add(folder1);
    folder.add(file1);
    folder.scan();
    console.log("/******移除文件夹******/");
    folder1.remove();
    folder.scan();
    </script>
</body>

</html>

```

## 模板方法模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 模板方法模式:一种基于继承的设计模式
    // 构成:第一部分是抽象父类，第二部分是具体的实现子类;通常在抽象父类中封装了子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法;父类封装了子类的算法框架和方法的执行顺序，子类继承父类之后，父类通知子类执行这些方法，好莱坞原则很好地诠释了这种设计技巧，即高层组件调用底层组件。
    // 使用场景:从大的方面来讲，模板方法模式常被架构师用于搭建项目的框架，架构师定好了框架的骨架，程序员继承框架的结构之后，负责往里面填空。
    // 在Web 开发中也能找到很多模板方法模式的适用场景
    // 比如我们在构建一系列的UI 组件这些组件的构建过程一般如下所示：
    // (1) 初始化一个div 容器；
    // (2) 通过ajax 请求拉取相应的数据；
    // (3) 把数据渲染到div 容器里面，完成组件的构造；
    // (4) 通知用户组件渲染完毕。
    // 我们看到， 任何组件的构建都遵循上面的4步，其中第(1)步和第(4)步是相同的。第(2)步不同的地方只是请求ajax的远程地址，第(3)步不同的地方是渲染数据的方式。于是我们可以把这4个步骤都抽象到父类的模板方法里面，父类中还可以顺便提供第(1)步和第(4)步的具体实现。当子类继承这个父类之后，会重写模板方法里面的第(2)步和第(3)步。
    // 使用举例:泡咖啡和泡茶
    // 泡咖啡
    // (1) 把水煮沸
    // (2) 用沸水冲泡咖啡
    // (3) 把咖啡倒进杯子
    // (4) 加糖和牛奶
    // 泡茶
    // (1) 把水煮沸
    // (2) 用沸水浸泡茶叶
    // (3) 把茶水倒进杯子
    // (4) 加柠檬
    // 二者基本步骤是相同，不同的地方有
    // 原料不同。一个是咖啡，一个是茶，但我们可以把它们都抽象为“饮料”。
    // 泡的方式不同。咖啡是冲泡，而茶叶是浸泡，我们可以把它们都抽象为“泡”。
    // 加入的调料不同。一个是糖和牛奶，一个是柠檬，但我们可以把它们都抽象为“调料”。
    // 抽象之后
    // (1) 把水煮沸
    // (2) 用沸水冲泡饮料
    // (3) 把饮料倒进杯子
    // (4) 加调料

    // 抽象父类-饮料类
    var Beverage = function() {};
    // 烧水
    Beverage.prototype.boilWater = function() {
        console.log("把水煮沸");
    };
    // 泡
    Beverage.prototype.brew = function() {
        throw new Error('子类必须重写brew方法');
    }; // 空方法，应该由子类重写，若子类忘记重写，可在父类上添加一个错误提醒信息
    // 倒入杯子
    Beverage.prototype.pourInCup = function() {
        throw new Error('子类必须重写pourInCup方法');
    }; // 空方法，应该由子类重写
    // 加调料
    Beverage.prototype.addCondiments = function() {
        throw new Error('子类必须重写addCondiments方法');
    }; // 空方法，应该由子类重写
    Beverage.prototype.init = function() {
        this.boilWater();
        this.brew();
        this.pourInCup();
        this.addCondiments();
    };
    // 抽象子类-coffee
    var Coffee = function() {};
    // 继承饮料类
    Coffee.prototype = new Beverage();
    // 重写泡、倒入杯子、加调料方法
    Coffee.prototype.brew = function() {
        console.log('用沸水冲泡咖啡');
    };
    Coffee.prototype.pourInCup = function() {
        console.log('把咖啡倒进杯子');
    };
    Coffee.prototype.addCondiments = function() {
        console.log('加糖和牛奶');
    };
    // coffee实例
    var Coffee = new Coffee();
    Coffee.init();
    // 抽象子类-tea
    var Tea = function() {};
    Tea.prototype = new Beverage();
    Tea.prototype.brew = function() {
        console.log('用沸水浸泡茶叶');
    };
    Tea.prototype.pourInCup = function() {
        console.log('把茶倒进杯子');
    };
    Tea.prototype.addCondiments = function() {
        console.log('加柠檬');
    };
    var tea = new Tea();
    tea.init();
    // 上面Beverage.prototype.init 被称为模板方法。该方法中封装了子类的算法框架，它作为一个算法的模板，指导子类以何种顺序去执行哪些方法
    </script>
</body>

</html>

```

### 钩子方法
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 在50中实现了下面的流程，如果有客人不加调料，该如何实现
    // (1) 把水煮沸
    // (2) 用沸水冲泡饮料
    // (3) 把饮料倒进杯子
    // (4) 加调料
    // 钩子方法（hook）可以用来解决这个问题，放置钩子是隔离变化的一种常见手段。我们在父类中容易变化的地方放置钩子，钩子可以有一个默认的实现，究竟要不要“挂钩”，这由子类自行决定。钩子方法的返回结果决定了模板方法后面部分的执行步骤，也就是程序接下来的走向，这样一来，程序就拥有了变化的可能。
    var Beverage = function() {};
    Beverage.prototype.boilWater = function() {
        console.log("把水煮沸");
    };
    Beverage.prototype.brew = function() {
        throw new Error('子类必须重写brew 方法');
    };
    Beverage.prototype.pourInCup = function() {
        throw new Error('子类必须重写pourInCup 方法');
    };
    Beverage.prototype.addCondiments = function() {
        throw new Error('子类必须重写addCondiments 方法');
    };
    Beverage.prototype.customerWantsCondiments = function() {
        return true; // 默认需要调料
    };
    Beverage.prototype.init = function() {
        this.boilWater();
        this.brew();
        this.pourInCup();
        if (this.customerWantsCondiments()) { // 如果挂钩返回true，则需要调料
            this.addCondiments();
        }
    };
    var CoffeeWithHook = function() {};
    CoffeeWithHook.prototype = new Beverage();
    CoffeeWithHook.prototype.brew = function() {
        console.log('用沸水冲泡咖啡');
    };
    CoffeeWithHook.prototype.pourInCup = function() {
        console.log('把咖啡倒进杯子');
    };
    CoffeeWithHook.prototype.addCondiments = function() {
        console.log('加糖和牛奶');
    };
    CoffeeWithHook.prototype.customerWantsCondiments = function() {
        return window.confirm('请问需要调料吗？');
    };
    var coffeeWithHook = new CoffeeWithHook();
    coffeeWithHook.init();

    // 好莱坞原则
    // 好莱坞无疑是演员的天堂，但好莱坞也有很多找不到工作的新人演员，许多新人演员在好莱坞把简历递给演艺公司之后就只有回家等待电话。有时候该演员等得不耐烦了，给演艺公司打电话询问情况，演艺公司往往这样回答：“不要来找我，我会给你打电话。”
    // 在设计中，这样的规则就称为好莱坞原则。在这一原则的指导下，我们允许底层组件将自己挂钩到高层组件中， 而高层组件会决定什么时候、 以何种方式去使用这些底层组件， 高层组件对待底层组件的方式， 跟演艺公司对待新人演员一样，都是“ 别调用我们，我们会调用你”。
    </script>
</body>

</html>
```

### 模板方法模式在js中的实现
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    var Beverage = function(param) {
        var boilWater = function() {
            console.log("把水煮沸");
        };
        var brew = param.brew || function() {
            throw new Error("必须传递brew方法");
        };
        var pourInCup = param.pourInCup || function() {
            throw new Error("必须传递pourInCup方法");
        };
        var addCondiments = param.addCondiments || function() {
            throw new Error("必须传递addCondiments方法");
        };
        var F = function() {};
        F.prototype.init = function() {
            boilWater();
            brew();
            pourInCup();
            addCondiments();
        };
        return F;
    };
    var Coffee = Beverage({
        brew: function() {
            console.log('用沸水冲泡咖啡');
        },
        pourInCup: function() {
            console.log('把咖啡倒进杯子');
        },
        addCondiments: function() {
            console.log('加糖和牛奶');
        }
    });
    var Tea = Beverage({
        brew: function() {
            console.log('用沸水浸泡茶叶');
        },
        pourInCup: function() {
            console.log('把茶倒进杯子');
        },
        addCondiments: function() {
            console.log('加柠檬');
        }
    });
    var coffee = new Coffee();
    coffee.init();
    var tea = new Tea();
    tea.init();
    // 在这段代码中，我们把brew、pourInCup、addCondiments 这些方法依次传入Beverage 函数，    Beverage 函数被调用之后返回构造器F。 F 类中包含了“ 模板方法” F.prototype.init。 跟继承得到的效果一样， 该“ 模板方法” 里依然封装了饮料子类的算法框架。
    </script>
</body>

</html>

```

## 享元模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 享元模式:享元(flyweight)模式是一种用于性能优化的模式，“ fly” 在这里是苍蝇的意思， 意为蝇量级。 享元模式的核心是运用共享技术来有效支持大量细粒度的对象。

    // 享元模式的目标是尽量减少共享对象的数量，享元模式要求将对象的属性划分为内部状态与外部状态（状态在这里通常指属性）,关于如何划分内部状态和外部状态，下面的几条经验提供了一些指引。
    // 内部状态存储于对象内部。
    // 内部状态可以被一些对象共享。
    // 内部状态独立于具体的场景，通常不会改变。
    // 外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享。
    // 我们便可以把所有内部状态相同的对象都指定为同一个共享的对象。而外部状态可以从对象身上剥离出来，并储存在外部。
    // 剥离了外部状态的对象成为共享对象，外部状态在必要时被传入共享对象来组装成一个完整的对象。虽然组装外部状态成为一个完整对象的过程需要花费一定的时间，但却可以大大减少系统中的对象数量，相比之下，这点时间或许是微不足道的。因此，享元模式是一种用时间换空间的优化模式。

    // 使用场景:
    // 一个程序中使用了大量的相似对象。
    // 由于使用了大量对象，造成很大的内存开销。
    // 对象的大多数状态都可以变为外部状态。
    // 剥离出对象的外部状态之后，可以用相对较少的共享对象取代大量对象。
    // 使用举例:假设有个内衣工厂，目前的产品有50种男式内衣和50种女士内衣，为了推销产品，工厂决定生产一些塑料模特来穿上他们的内衣拍成广告照片。 


    // 正常情况下需要50 个男模特和50个女模特，然后让他们每人分别穿上一件内衣来拍照
    // var Model = function(sex, underwear) {
    //     this.sex = sex;
    //     this.underwear = underwear;
    // };
    // Model.prototype.takePhoto = function() {
    //     console.log("sex=" + this.sex + ";underwear=" + this.underwear);
    // };
    // for (var i = 1; i <= 50; i++) {
    //     var maleModel = new Model("male", "underwear" + i);
    //     maleModel.takePhoto();
    // }
    // for (var j = 1; j <= 50; j++) {
    //     var femaleModel = new Model("female", "underwear" + j);
    //     femaleModel.takePhoto();
    // }
    // 这样会产生100个对象，如果有10000种内衣，则此程序会爆炸

    // 优化，其实只要一个男模特、一个女模特，让他们分别试穿50件内衣然后拍照即可
    var Model = function(sex) {
        this.sex = sex;
    };
    Model.prototype.takePhoto = function() {
        console.log("sex=" + this.sex + ";underwear=" + this.underwear);
    };
    var maleModel = new Model('male'),
        femaleModel = new Model('female');
    for (var i = 1; i <= 50; i++) {
        maleModel.underwear = 'underwear' + i;
        maleModel.takePhoto();
    };
    for (var j = 1; j <= 50; j++) {
        femaleModel.underwear = 'underwear' + j;
        femaleModel.takePhoto();
    };
    // 上例中性别是内部状态，内衣是外部状态
    // 可以被对象共享的属性通常被划分为内部状态，如同不管什么样式的衣服，都可以按照性别不同，穿在同一个男模特或者女模特身上，模特的性别就可以作为内部状态储存在共享对象的内部。而外部状态取决于具体的场景，并根据场景而变化，就像例子中每件衣服都是不同的，它们不能被一些对象共享，因此只能被划分为外部状态。
    </script>
</body>

</html>
```

### 享元模式重构上传
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 普通上传
    // var id = 0;
    // window.startUpload = function(uploadType, files) {
    //     for (var i = 0, file; file = files[i++];) {
    //         var uploadObj = new Upload(uploadType, file.fileName, file.fileSize);
    //         uploadObj.init(id++);
    //     }
    // };
    // var Upload = function(uploadType, fileName, fileSize) {
    //     this.uploadType = uploadType;
    //     this.fileName = fileName;
    //     this.fileSize = fileSize;
    //     this.dom = null;
    // };
    // Upload.prototype.init = function(id) {
    //     var that = this;
    //     this.id = id;
    //     this.dom = document.createElement("div");
    //     this.dom.innerHTML = '<span>文件名称:' + this.fileName + ', 文件大小: ' + this.fileSize + '</span>' +
    //         '<button class="delFile">删除</button>';
    //     this.dom.querySelector(".delFile").onclick = function() {
    //         that.delFile();
    //     };
    //     document.body.appendChild(this.dom);
    // };
    // Upload.prototype.delFile = function() {
    //     if (this.fileSize < 3000) {
    //         return this.dom.parentNode.removeChild(this.dom);
    //     }
    //     if (window.confirm("确定要删除该文件吗？" + this.fileName)) {
    //         return this.dom.parentNode.removeChild(this.dom);
    //     }
    // };
    // startUpload('plugin', [{
    //     fileName: '1.txt',
    //     fileSize: 1000
    // }, {
    //     fileName: '2.html',
    //     fileSize: 3000
    // }, {
    //     fileName: '3.txt',
    //     fileSize: 5000
    // }]);
    // startUpload('flash', [{
    //     fileName: '4.txt',
    //     fileSize: 1000
    // }, {
    //     fileName: '5.html',
    //     fileSize: 3000
    // }, {
    //     fileName: '6.txt',
    //     fileSize: 5000
    // }]);
    // 上面的问题在于，有多少个要上传的文件，就要需要创建多少个upload对象

    // 使用享元模式重构
    // uploadType是内部状态，fileName和fileSize都是外部状态
    var Upload = function(uploadType) {
        this.uploadType = uploadType;
    };
    Upload.prototype.delFile = function(id) {
        uploadManager.setExternalState(id, this);
        if (this.fileSize < 3000) {
            return this.dom.parentNode.removeChild(this.dom);
        }
        if (window.confirm('确定要删除该文件吗? ' + this.fileName)) {
            return this.dom.parentNode.removeChild(this.dom);
        }
    };
    var UploadFactory = (function() {
        var createdFlyWeightObjs = {};
        return {
            create: function(uploadType) {
                if (createdFlyWeightObjs[uploadType]) {
                    return createdFlyWeightObjs[uploadType];
                }
                return createdFlyWeightObjs[uploadType] = new Upload(uploadType);
            }
        };
    })();
    var uploadManager = (function() {
        var uploadDatabase = {};
        return {
            add: function(id, uploadType, fileName, fileSize) {
                var flyWeightObj = UploadFactory.create(uploadType);
                var dom = document.createElement("div");
                dom.innerHTML =
                    '<span>文件名称:' + fileName + ', 文件大小: ' + fileSize + '</span>' +
                    '<button class="delFile">删除</button>';
                dom.querySelector('.delFile').onclick = function() {
                    flyWeightObj.delFile(id);
                }
                document.body.appendChild(dom);
                uploadDatabase[id] = {
                    fileName: fileName,
                    fileSize: fileSize,
                    dom: dom
                };
                return flyWeightObj;
            },
            setExternalState: function(id, flyWeightObj) {
                var uploadData = uploadDatabase[id];
                for (var i in uploadData) {
                    flyWeightObj[i] = uploadData[i];
                }
            }
        };
    })();
    var id = 0;
    window.startUpload = function(uploadType, files) {
        for (var i = 0, file; file = files[i++];) {
            var uploadObj = uploadManager.add(++id, uploadType, file.fileName, file.fileSize);
        }
    };
    startUpload('plugin', [{
        fileName: '1.txt',
        fileSize: 1000
    }, {
        fileName: '2.html',
        fileSize: 3000
    }, {
        fileName: '3.txt',
        fileSize: 5000
    }]);
    startUpload('flash', [{
        fileName: '4.txt',
        fileSize: 1000
    }, {
        fileName: '5.html',
        fileSize: 3000
    }, {
        fileName: '6.txt',
        fileSize: 5000
    }]);
    </script>
</body>

</html>

```

### 对象池
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 对象池维
    // 护一个装载空闲对象的池子，如果需要对象的时候，不是直接new，而是转从对象池里获取。如
    // 果对象池里没有空闲对象，则创建一个新的对象，当获取出的对象完成它的职责之后， 再进入
    // 池子等待被下次获取。
    // 对象池技术的应用非常广泛，HTTP 连接池和数据库连接池都是其代表应用。在Web 前端开
    // 发中，对象池使用最多的场景大概就是跟DOM 有关的操作。很多空间和时间都消耗在了DOM
    // 节点上，如何避免频繁地创建和删除DOM 节点就成了一个有意义的话题。
    // 对象池跟享元模式的思想有点相似,虽然innerHTML 的值A、B、C、D 等也可以看成节点的外部状态，但在这里我们并没有主动分离内部状态和外部状态的过程。
    var toolTipFactory = (function() {
        var toolTipPool = [];
        return {
            create: function() {
                if (toolTipPool.length === 0) {
                    // 对象池为空则创建一个div
                    var div = document.createElement("div");
                    document.body.appendChild(div);
                    return div;
                } else {
                    // 不为空则取出一个
                    return toolTipPool.shift();
                }
            },
            recover: function(tooltipDom) {
                // 对象池回收dom
                return toolTipPool.push(tooltipDom);
            }
        };
    })();
    // 创建2个小气泡
    var ary = [];
    for (var i = 0, str; str = ["A", "B"][i++];) {
        var toolTip = toolTipFactory.create();
        toolTip.innerHTML = str;
        ary.push(toolTip);
    }
    console.log(ary);
    // 地图需要重绘，所以先回收之前创建的2个气泡
    for (var i = 0, toolTip; toolTip = ary[i++];) {
        toolTipFactory.recover(toolTip);
    };
    // 创建6个气泡
    for (var i = 0, str; str = ['A', 'B', 'C', 'D', 'E', 'F'][i++];) {
        var toolTip = toolTipFactory.create();
        toolTip.innerHTML = str;
    };
    </script>
</body>

</html>
```

### 通用对象池
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 通用的创建对象池的工厂
    var objectPoolFactory = function(creactObjFn) {
        var objectPool = [];
        return {
            create: function() {
                var obj = objectPool.length === 0 ? creactObjFn.apply(this, arguments) : objectPool.shift();
                return obj;
            },
            recover: function(obj) {
                objectPool.push(obj);
            }
        };
    };
    var iframeFactory = objectPoolFactory(function() {
        var iframe = document.createElement("iframe");
        document.body.appendChild(iframe);
        iframe.onload = function() {
            // 防止iframe重复加载的bug
            iframe.onload = null;
            // 加载完成后回收节点，供下次使用
            iframeFactory.recover(iframe);
        };
        return iframe;
    });
    var iframe1 = iframeFactory.create();
    iframe1.src = 'http://www.baidu.com';
    var iframe2 = iframeFactory.create();
    iframe2.src = 'http://www.qq.com';
    setTimeout(function() {
        var iframe3 = iframeFactory.create();
        iframe3.src = 'http://www.163.com';
    }, 3000);
    </script>
</body>

</html>

```

## 职责链模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 职责链模式:使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系， 将这些对象连成一条链， 并沿着这条链传递该请求， 直到有一个对象处理它为止。
    // 使用场景:类似队列的操作时可以用
    // 使用举例:公司针对支付过定金的用户有一定的优惠政策。在正式购买后，已经支付过500元定金的用户会收到100元的商城优惠券，200元定金的用户可以收到50 元的优惠券，而之前没有支付定金的用户只能进入普通购买模式，也就是没有优惠券，且在库存有限的情况下不一定保证能买到。我们的订单页面是PHP 吐出的模板，在页面加载之初，PHP 会传递给页面几个字段。
    // orderType：表示订单类型（定金用户或者普通购买用户），code 的值为1 的时候是500 元定金用户，为2 的时候是200 元定金用户，为3 的时候是普通购买用户。
    // pay：表示用户是否已经支付定金，值为true 或者false, 虽然用户已经下过500 元定金的订单，但如果他一直没有支付定金，现在只能降级进入普通购买模式。
    // stock：表示当前用于普通购买的手机库存数量，已经支付过500 元或者200 元定金的用户不受此限制。

    // 用普通流程写
    // var order = function(orderType, pay, stock) {
    //     if (orderType === 1) { // 500 元定金购买模式
    //         if (pay === true) { // 已支付定金
    //             console.log('500 元定金预购, 得到100 优惠券');
    //         } else { // 未支付定金，降级到普通购买模式
    //             if (stock > 0) { // 用于普通购买的手机还有库存
    //                 console.log('普通购买, 无优惠券');
    //             } else {
    //                 console.log('手机库存不足');
    //             }
    //         }
    //     } else if (orderType === 2) { // 200 元定金购买模式
    //         if (pay === true) {
    //             console.log('200 元定金预购, 得到50 优惠券');
    //         } else {
    //             if (stock > 0) {
    //                 console.log('普通购买, 无优惠券');
    //             } else {
    //                 console.log('手机库存不足');
    //             }
    //         }
    //     } else if (orderType === 3) {
    //         if (stock > 0) {
    //             console.log('普通购买, 无优惠券');
    //         } else {
    //             console.log('手机库存不足');
    //         }
    //     }
    // };
    // order(1, true, 500); // 输出： 500 元定金预购, 得到100 优惠券

    // 用职责链来重写
    // 先把500 元订单、200 元订单以及普通购买分成3个函数
    // 接下来把orderType、pay、stock 这3 个字段当作参数传递给500 元订单函数，如果该函数不符合处理条件，则把这个请求传递给后面的200 元订单函数，如果200 元订单函数依然不能处理该请求，则继续传递请求给普通购买函数
    // 500 元订单
    var order500 = function(orderType, pay, stock) {
        if (orderType === 1 && pay === true) {
            console.log('500 元定金预购, 得到100 优惠券');
        } else {
            order200(orderType, pay, stock); // 将请求传递给200 元订单
            // order200 和order500 耦合在一起
        }
    };
    // 200 元订单
    var order200 = function(orderType, pay, stock) {
        if (orderType === 2 && pay === true) {
            console.log('200 元定金预购, 得到50 优惠券');
        } else {
            orderNormal(orderType, pay, stock); // 将请求传递给普通订单
        }
    };
    // 普通购买订单
    var orderNormal = function(orderType, pay, stock) {
        if (stock > 0) {
            console.log('普通购买, 无优惠券');
        } else {
            console.log('手机库存不足');
        }
    };
    // 测试结果：
    order500(1, true, 500); // 输出：500 元定金预购, 得到100 优惠券
    order500(1, false, 500); // 输出：普通购买, 无优惠券
    order500(2, true, 500); // 输出：200 元定金预购, 得到500 优惠券
    order500(3, false, 500); // 输出：普通购买, 无优惠券
    order500(3, false, 0); // 输出：手机库存不足
    // 相比上面一个去掉了很多if判断，但是仍然存在每个处理对象相互耦合的问题，在order500中有order200
    // 如果有天我们要增加300 元预订或者去掉200 元预订，意味着就必须改动这些业务函数内部。就像一根环环相扣打了死结的链条，如果要增加、拆除或者移动一个节点，就必须得先砸烂这根链条
    </script>
</body>

</html>
```

### 灵活可拆分的职责链节点
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 58中的第二个例子很不灵活，链中的每个节点无法灵活拆分和重组
    var order500 = function(orderType, pay, stock) {
        if (orderType === 1 && pay === true) {
            console.log('500 元定金预购，得到100 优惠券');
        } else {
            return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
        }
    };
    var order200 = function(orderType, pay, stock) {
        if (orderType === 2 && pay === true) {
            console.log('200 元定金预购，得到50 优惠券');
        } else {
            return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
        }
    };
    var orderNormal = function(orderType, pay, stock) {
        if (stock > 0) {
            console.log('普通购买，无优惠券');
        } else {
            console.log('手机库存不足');
        }
    };
    // Chain.prototype.setNextSuccessor 指定在链中的下一个节点
    // Chain.prototype.passRequest 传递请求给某个节点
    var Chain = function(fn) {
        this.fn = fn;
        this.successor = null; //表示在链中的下一个节点
    };
    Chain.prototype.setNextSuccessor = function(successor) {
        return this.successor = successor;
    };
    Chain.prototype.passRequest = function() {
        var ret = this.fn.apply(this, arguments);
        if (ret === "nextSuccessor") {
            return this.successor && this.successor.passRequest.apply(this.successor, arguments);
        }
        return ret;
    };
    // 将订单函数包装成职责链的节点
    var chainOrder500 = new Chain(order500);
    var chainOrder200 = new Chain(order200);
    var chainOrderNormal = new Chain(orderNormal);
    // 然后指定节点在职责链中的顺序
    chainOrder500.setNextSuccessor(chainOrder200);
    chainOrder200.setNextSuccessor(chainOrderNormal);
    // 传递请求
    chainOrder500.passRequest(1, true, 500); // 输出：500 元定金预购，得到100 优惠券
    chainOrder500.passRequest(2, true, 500); // 输出：200 元定金预购，得到50 优惠券
    chainOrder500.passRequest(3, true, 500); // 输出：普通购买，无优惠券
    chainOrder500.passRequest(1, false, 0); // 输出：手机库存不足
    </script>
</body>

</html>

```

### 异步的职责链
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 在59中我们让每个节点函数同步返回一个特定的值"nextSuccessor"，来表示是否把请求传递给下一个节点。而在现实开发中，我们经常会遇到一些异步的问题，比如我们要在节点函数中发起一个ajax异步请求，异步请求返回的结果才能决定是否继续在职责链中passRequest。
    var Chain = function(fn) {
        this.fn = fn;
        this.successor = null; //表示在链中的下一个节点
    };
    Chain.prototype.setNextSuccessor = function(successor) {
        return this.successor = successor;
    };
    Chain.prototype.passRequest = function() {
        var ret = this.fn.apply(this, arguments);
        if (ret === "nextSuccessor") {
            return this.successor && this.successor.passRequest.apply(this.successor, arguments);
        }
        return ret;
    };
    // 添加next，表示手动传递请求给下一节点
    Chain.prototype.next = function() {
        return this.successor && this.successor.passRequest.apply(this.successor, arguments);
    };
    var fn1 = new Chain(function() {
        console.log(1);
        return 'nextSuccessor';
    });
    var fn2 = new Chain(function() {
        console.log(2);
        var self = this;
        setTimeout(function() {
            self.next();
        }, 1000);
    });
    var fn3 = new Chain(function() {
        console.log(3);
    });
    fn1.setNextSuccessor(fn2).setNextSuccessor(fn3);
    fn1.passRequest();
    </script>
</body>

</html>
```

### 用AOP实现职责链
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    Function.prototype.after = function(fn) {
        var self = this;
        return function() {
            var ret = self.apply(this, arguments);
            if (ret === 'nextSuccessor') {
                return fn.apply(this, arguments);
            }
            return ret;
        }
    };
    var order500 = function(orderType, pay, stock) {
        if (orderType === 1 && pay === true) {
            console.log('500 元定金预购，得到100 优惠券');
        } else {
            return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
        }
    };
    var order200 = function(orderType, pay, stock) {
        if (orderType === 2 && pay === true) {
            console.log('200 元定金预购，得到50 优惠券');
        } else {
            return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
        }
    };
    var orderNormal = function(orderType, pay, stock) {
        if (stock > 0) {
            console.log('普通购买，无优惠券');
        } else {
            console.log('手机库存不足');
        }
    };
    var order = order500.after(order200).after(orderNormal);
    order(1, true, 500); // 输出：500 元定金预购，得到100 优惠券
    order(2, true, 500); // 输出：200 元定金预购，得到50 优惠券
    order(1, false, 500); // 输出：普通购买，无优惠券
    </script>
</body>

</html>

```

### 用职责链获取文件上传对象
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    Function.prototype.after = function(fn) {
        var self = this;
        return function() {
            var ret = self.apply(this, arguments);
            if (ret === 'nextSuccessor') {
                return fn.apply(this, arguments);
            }
            return ret;
        }
    };
    var getActiveUploadObj = function() {
        try {
            return new ActiveXObject("TXFTNActiveX.FTNUpload"); // IE 上传控件
        } catch (e) {
            return 'nextSuccessor';
        }
    };
    var getFlashUploadObj = function() {
        if (supportFlash()) {
            var str = '<object type="application/x-shockwave-flash"></object>';
            return $(str).appendTo($('body'));
        }
        return 'nextSuccessor';
    };
    var getFormUpladObj = function() {
        return $('<form><input name="file" type="file"/></form>').appendTo($('body'));
    };
    var getUploadObj = getActiveUploadObj.after(getFlashUploadObj).after(getFormUpladObj);
    console.log(getUploadObj());
    </script>
</body>

</html>

```

## 中介者模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 中介者模式:程序由大大小小的单一对象组成，所有这些对象都按照某种关系和规则来通信。当对象多了以后，相互通信会变的非常封闭式。中介者模式的作用就是解除对象与对象之间的紧耦合关系。增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可。中介者使各对象之间耦合松散，而且可以独立地改变它们之间的交互。中介者模式使网状的多对多关系变成了相对简单的一对多关系
    // 使用场景:当对象之间相互引用非常多时
    // 使用举例:泡泡堂游戏、购买商品
    function Player(name, teamColor) {
        this.partners = []; // 队友列表
        this.enemies = []; // 敌人列表
        this.state = 'live'; // 玩家状态
        this.name = name; // 角色名字
        this.teamColor = teamColor; // 队伍颜色
    };
    Player.prototype.win = function() { // 玩家团队胜利
        console.log('winner: ' + this.name);
    };
    Player.prototype.lose = function() { // 玩家团队失败
        console.log('loser: ' + this.name);
    };
    Player.prototype.die = function() { // 玩家死亡
        var all_dead = true;
        this.state = 'dead'; // 设置玩家状态为死亡
        for (var i = 0, partner; partner = this.partners[i++];) { // 遍历队友列表
            if (partner.state !== 'dead') { // 如果还有一个队友没有死亡，则游戏还未失败
                all_dead = false;
                break;
            }
        }
        if (all_dead === true) { // 如果队友全部死亡
            this.lose(); // 通知自己游戏失败
            for (var i = 0, partner; partner = this.partners[i++];) { // 通知所有队友玩家游戏失败
                partner.lose();
            }
            for (var i = 0, enemy; enemy = this.enemies[i++];) { // 通知所有敌人游戏胜利
                enemy.win();
            }
        }
    };
    var playerFactory = function(name, teamColor) {
        var newPlayer = new Player(name, teamColor); // 创建新玩家
        for (var i = 0, player; player = players[i++];) { // 通知所有的玩家，有新角色加入
            if (player.teamColor === newPlayer.teamColor) { // 如果是同一队的玩家
                player.partners.push(newPlayer); // 相互添加到队友列表
                newPlayer.partners.push(player);
            } else {
                player.enemies.push(newPlayer); // 相互添加到敌人列表
                newPlayer.enemies.push(player);
            }
        }
        players.push(newPlayer);
        return newPlayer;
    };
    var players = [];
    //红队：
    var player1 = playerFactory('皮蛋', 'red'),
        player2 = playerFactory('小乖', 'red'),
        player3 = playerFactory('宝宝', 'red'),
        player4 = playerFactory('小强', 'red');
    //蓝队：
    var player5 = playerFactory('黑妞', 'blue'),
        player6 = playerFactory('葱头', 'blue'),
        player7 = playerFactory('胖墩', 'blue'),
        player8 = playerFactory('海盗', 'blue');
    player1.die();
    player2.die();
    player4.die();
    player3.die();
    // 问题,每个玩家和其他玩家都是紧紧耦合在一起的。当每个对象的状态发生改变，比如角色移动、吃到道具或者死亡时，都必须要显式地遍历通知其他对象。当玩家变的很多，则会爆炸
    </script>
</body>

</html>

```

### 中介者模式改造泡泡堂游戏
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    function Player(name, teamColor) {
        this.name = name; // 角色名字
        this.teamColor = teamColor; // 队伍颜色
        this.state = 'alive'; // 玩家生存状态
    };
    Player.prototype.win = function() {
        console.log(this.name + ' won ');
    };
    Player.prototype.lose = function() {
        console.log(this.name + ' lost');
    };
    /*******************玩家死亡*****************/
    Player.prototype.die = function() {
        this.state = 'dead';
        playerDirector.reciveMessage('playerDead', this); // 给中介者发送消息，玩家死亡
    };
    /*******************移除玩家*****************/
    Player.prototype.remove = function() {
        playerDirector.reciveMessage('removePlayer', this); // 给中介者发送消息，移除一个玩家
    };
    /*******************玩家换队*****************/
    Player.prototype.changeTeam = function(color) {
        playerDirector.reciveMessage('changeTeam', this, color); // 给中介者发送消息，玩家换队
    };
    var playerFactory = function(name, teamColor) {
        var newPlayer = new Player(name, teamColor); // 创造一个新的玩家对象
        playerDirector.reciveMessage('addPlayer', newPlayer); // 给中介者发送消息，新增玩家
        return newPlayer;
    };
    var playerDirector = (function() {
        var players = {}, // 保存所有玩家
            operations = {}; // 中介者可以执行的操作
        /****************新增一个玩家***************************/
        operations.addPlayer = function(player) {
            var teamColor = player.teamColor; // 玩家的队伍颜色
            players[teamColor] = players[teamColor] || []; // 如果该颜色的玩家还没有成立队伍，则新成立一个队伍
            players[teamColor].push(player); // 添加玩家进队伍
        };
        /****************移除一个玩家***************************/
        operations.removePlayer = function(player) {
            var teamColor = player.teamColor, // 玩家的队伍颜色
                teamPlayers = players[teamColor] || []; // 该队伍所有成员
            for (var i = teamPlayers.length - 1; i >= 0; i--) { // 遍历删除
                if (teamPlayers[i] === player) {
                    teamPlayers.splice(i, 1);
                }
            }
        };
        /****************玩家换队***************************/
        operations.changeTeam = function(player, newTeamColor) { // 玩家换队
            operations.removePlayer(player); // 从原队伍中删除
            player.teamColor = newTeamColor; // 改变队伍颜色
            operations.addPlayer(player); // 增加到新队伍中
        };
        operations.playerDead = function(player) { // 玩家死亡
            var teamColor = player.teamColor,
                teamPlayers = players[teamColor]; // 玩家所在队伍
            var all_dead = true;
            for (var i = 0, player; player = teamPlayers[i++];) {
                if (player.state !== 'dead') {
                    all_dead = false;
                    break;
                }
            }
            if (all_dead === true) { // 全部死亡
                for (var i = 0, player; player = teamPlayers[i++];) {
                    player.lose(); // 本队所有玩家lose
                }
                for (var color in players) {
                    if (color !== teamColor) {
                        var teamPlayers = players[color]; // 其他队伍的玩家
                        for (var i = 0, player; player = teamPlayers[i++];) {
                            player.win(); // 其他队伍所有玩家win
                        }
                    }
                }
            }
        };
        var reciveMessage = function() {
            var message = Array.prototype.shift.call(arguments); // arguments 的第一个参数为消息名称
            operations[message].apply(this, arguments);
        };
        return {
            reciveMessage: reciveMessage
        }
    })();

    // 红队：
    var player1 = playerFactory('皮蛋', 'red'),
        player2 = playerFactory('小乖', 'red'),
        player3 = playerFactory('宝宝', 'red'),
        player4 = playerFactory('小强', 'red');
    // 蓝队：
    var player5 = playerFactory('黑妞', 'blue'),
        player6 = playerFactory('葱头', 'blue'),
        player7 = playerFactory('胖墩', 'blue'),
        player8 = playerFactory('海盗', 'blue');
    player1.changeTeam('blue');
    player2.die();
    player3.die();
    player4.die();
    </script>
</body>

</html>

```

### 普通方法购买商品
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>选择颜色:
    <select id="colorSelect">
        <option value="">请选择</option>
        <option value="red">红色</option>
        <option value="blue">蓝色</option>
    </select>
    选择内存:
    <select id="memorySelect">
        <option value="">请选择</option>
        <option value="32G">32G</option>
        <option value="16G">16G</option>
    </select>
    输入购买数量:
    <input type="text" id="numberInput" />
    <br/> 您选择了颜色:
    <div id="colorInfo"></div>
    <br/> 您选择了内存:
    <div id="memoryInfo"></div>
    <br/> 您输入了数量:
    <div id="numberInfo"></div>
    <br/>
    <button id="nextBtn" disabled="true">请选择手机颜色和购买数量</button>
    <script>
    // 假设我们正在编写一个手机购买的页面，在购买流程中，可以选择手机的颜色以及输入购买数量，同时页面中有两个展示区域，分别向用户展示刚刚选择好的颜色和数量。还有一个按钮动态显示下一步的操作，我们需要查询该颜色手机对应的库存，如果库存数量少于这次的购买数量，按钮将被禁用并且显示库存不足，反之按钮可以点击并且显示放入购物车。
    var colorSelect = document.getElementById('colorSelect'),
        numberInput = document.getElementById('numberInput'),
        colorInfo = document.getElementById('colorInfo'),
        numberInfo = document.getElementById('numberInfo'),
        nextBtn = document.getElementById('nextBtn');
    var goods = { // 手机库存
        "red|32G": 3, // 红色32G，库存数量为3
        "red|16G": 0,
        "blue|32G": 1,
        "blue|16G": 6
    };
    colorSelect.onchange = function() {
        var color = this.value,
            memory = memorySelect.value,
            stock = goods[color + '|' + memory];
        number = numberInput.value, // 数量
            colorInfo.innerHTML = color;
        if (!color) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择手机颜色';
            return;
        }
        if (!memory) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择内存大小';
            return;
        }
        if (!(/^[0-9]*[1-9][0-9]*$/).test(number)) { // 输入购买数量是否为正整数
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请输入正确的购买数量';
            return;
        }
        if (number > stock) { // 当前选择数量没有超过库存量
            nextBtn.disabled = true;
            nextBtn.innerHTML = '库存不足';
            return;
        }
        nextBtn.disabled = false;
        nextBtn.innerHTML = '放入购物车';
    };
    numberInput.oninput = function() {
        var color = colorSelect.value, // 颜色
            number = this.value, // 数量
            memory = memorySelect.value,
            stock = goods[color + '|' + memory];
        number = numberInput.value, // 数量
            numberInfo.innerHTML = number;
        if (!color) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择手机颜色';
            return;
        }
        if (!memory) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择内存大小';
            return;
        }
        if (!(/^[0-9]*[1-9][0-9]*$/).test(number)) { // 输入购买数量是否为正整数
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请输入正确的购买数量';
            return;
        }
        if (number > stock) { // 当前选择数量没有超过库存量
            nextBtn.disabled = true;
            nextBtn.innerHTML = '库存不足';
            return;
        }
        nextBtn.disabled = false;
        nextBtn.innerHTML = '放入购物车';
    };
    memorySelect.onchange = function() {
        var color = colorSelect.value, // 颜色
            number = numberInput.value, // 数量
            memory = this.value,
            stock = goods[color + '|' + memory]; // 该颜色手机对应的当前库存
        memoryInfo.innerHTML = memory;
        if (!color) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择手机颜色';
            return;
        }
        if (!memory) {
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请选择内存大小';
            return;
        }
        if (!(/^[0-9]*[1-9][0-9]*$/).test(number)) { // 输入购买数量是否为正整数
            nextBtn.disabled = true;
            nextBtn.innerHTML = '请输入正确的购买数量';
            return;
        }
        if (number > stock) { // 当前选择数量没有超过库存量
            nextBtn.disabled = true;
            nextBtn.innerHTML = '库存不足';
            return;
        }
        nextBtn.disabled = false;
        nextBtn.innerHTML = '放入购物车';
    };
    // 上面的方式，每个对象都耦合在一起，当修改一个地方，需要在每个事件里同步修改，非常不利于维护。
    </script>
</body>

</html>

```

### 中介者模式购买商品
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>选择颜色:
    <select id="colorSelect">
        <option value="">请选择</option>
        <option value="red">红色</option>
        <option value="blue">蓝色</option>
    </select>
    选择内存:
    <select id="memorySelect">
        <option value="">请选择</option>
        <option value="16G">16G</option>
        <option value="32G">32G</option>
    </select>
    选择CPU:
    <select id="cpuSelect">
        <option value="">请选择</option>
        <option value="800">800</option>
        <option value="801">801</option>
    </select>
    输入购买数量:
    <input type="text" id="numberInput" />
    <br/> 您选择了颜色:
    <div id="colorInfo"></div>
    <br/> 您选择了内存:
    <div id="memoryInfo"></div>
    <br/>您选择了cpu:
    <div id="cpuInfo"></div>
    <br/> 您输入了数量:
    <div id="numberInfo"></div>
    <br/>
    <button id="nextBtn" disabled="true">请选择手机颜色和购买数量</button>
    <script>
    // 引入中介者模式,所有的节点对象只跟中介者通信
    var goods = { // 手机库存
        "red|32G|800": 3, // 颜色red，内存32G，cpu800，对应库存数量为3
        "red|16G|801": 0,
        "blue|32G|800": 1,
        "blue|16G|801": 6
    };
    var mediator = (function() {
        var colorSelect = document.getElementById('colorSelect'),
            memorySelect = document.getElementById('memorySelect'),
            numberInput = document.getElementById('numberInput'),
            colorInfo = document.getElementById('colorInfo'),
            cpuInfo = document.getElementById('cpuInfo'),
            memoryInfo = document.getElementById('memoryInfo'),
            numberInfo = document.getElementById('numberInfo'),
            nextBtn = document.getElementById('nextBtn'),
            cpuSelect = document.getElementById('cpuSelect');
        return {
            changed: function(obj) {
                var color = colorSelect.value,
                    memory = memorySelect.value,
                    number = numberInput.value,
                    cpu = cpuSelect.value,
                    stock = goods[color + '|' + memory + '|' + cpu];
                if (obj === colorSelect) {
                    colorInfo.innerHTML = color;
                } else if (obj === memorySelect) {
                    memoryInfo.innerHTML = memory;
                } else if (obj === cpuSelect) {
                    cpuInfo.innerHTML = cpu;
                } else if (obj === numberInput) {
                    numberInfo.innerHTML = number;
                }
                if (!color) {
                    nextBtn.disabled = true;
                    nextBtn.innerHTML = "请选择手机颜色";
                    return;
                }
                if (!memory) {
                    nextBtn.disabled = true;
                    nextBtn.innerHTML = "请选择手机内存大小";
                    return;
                }
                if (!cpu) {
                    nextBtn.disabled = true;
                    nextBtn.innerHTML = "请选择CPU";
                    return;
                }
                if (!(/^[0-9]*[1-9][0-9]*$/).test(number)) { // 输入购买数量是否为正整数
                    nextBtn.disabled = true;
                    nextBtn.innerHTML = '请输入正确的购买数量';
                    return;
                }
                if (!stock || number > stock) { // 当前选择数量没有超过库存量
                    nextBtn.disabled = true;
                    nextBtn.innerHTML = '库存不足';
                    return;
                }
                nextBtn.disabled = false;
                nextBtn.innerHTML = "放入购物车";
            }
        };
    })();
    colorSelect.onchange = function() {
        mediator.changed(this);
    };
    memorySelect.onchange = function() {
        mediator.changed(this);
    };
    cpuSelect.onchange = function() {
        mediator.changed(this);
    };
    numberInput.oninput = function() {
        mediator.changed(this);
    };
    // 当再添加一个cpu型号时，可直接在mediator中添加即可
    </script>
</body>

</html>
```

## 装饰者模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 装饰者模式:装饰者模式可以动态地给某个对象添加一些额外的职责，而不会影响从这个类中派生的其他对象。装饰者模式能够在不改变对象自身的基础上，在程序运行期间给对象    动态地添加职责。
    // 使用场景:当需要给对象动态的添加行为时
    // 使用举例:统计代码、插件式验证等

    // 传统装饰者模式
    // var Plane = function() {}
    // Plane.prototype.fire = function() {
    //     console.log('发射普通子弹');
    // };
    // var MissileDecorator = function(plane) {
    //     this.plane = plane;
    // }
    // MissileDecorator.prototype.fire = function() {
    //     this.plane.fire();
    //     console.log('发射导弹');
    // };
    // var AtomDecorator = function(plane) {
    //     this.plane = plane;
    // };
    // AtomDecorator.prototype.fire = function() {
    //     this.plane.fire();
    //     console.log('发射原子弹');
    // };
    // var plane = new Plane();
    // plane = new MissileDecorator(plane);
    // plane = new AtomDecorator(plane);
    // plane.fire();
    // // 分别输出： 发射普通子弹、发射导弹、发射原子弹

    // js中的装饰者模式
    var plane = {
        fire: function() {
            console.log("发射普通");
        }
    };
    var missileDecorator = function() {
        console.log('发射导弹');
    }
    var atomDecorator = function() {
        console.log('发射原子弹');
    }
    var fire1 = plane.fire;
    plane.fire = function() {
        fire1();
        missileDecorator();
    }
    var fire2 = plane.fire;
    plane.fire = function() {
        fire2();
        atomDecorator();
    }
    plane.fire();
    // 分别输出： 发射普通子弹、发射导弹、发射原子弹
    </script>
</body>

</html>

```

### 装饰函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // 装饰函数->为函数添加功能

    // 最low的添加功能，直接改写原函数
    // var a = function() {
    //     alert(1);
    // };

    // var a = function() {
    //     alert(1);
    //     alert(2);
    // };

    // 保存原函数的引用，这种方法必须得维护_a这个中间量，如果装饰链过长或要装饰的函数过多，中间量就会变的很多;还会出现this劫持问题
    // var a = function() {
    //     alert(1);
    // };
    // var _a = a;
    // a = function() {
    //     _a();
    //     alert(2);
    // };
    // a();

    // 上面方法会出现下面的this劫持问题
    // var _getElementById = document.getElementById;
    // document.getElementById = function(id) {
    //     alert(1);
    //     return _getElementById(id); // (1)
    // }
    // var button = document.getElementById('button'); // error
    // 报错原因是getElementById内部是要用到this,调用_getElementById时this是指向window而不是document，所以会报错

    // 为解决上面问题，可以将document动态传入
    var _getElementById = document.getElementById;
    document.getElementById = function() {
        alert(1);
        return _getElementById.apply(document, arguments);
    }
    var button = document.getElementById('button');
    </script>
</body>

</html>

```

### 用AOP来装饰函数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    Function.prototype.before = function(beforefn) {
        var __self = this; // 保存原函数的引用
        return function() { // 返回包含了原函数和新函数的"代理"函数
            beforefn.apply(this, arguments); // 执行新函数，且保证this 不被劫持，新函数接受的参数
            // 也会被原封不动地传入原函数，新函数在原函数之前执行
            return __self.apply(this, arguments); // 执行原函数并返回原函数的执行结果，
            // 并且保证this 不被劫持
        }
    };
    Function.prototype.after = function(afterfn) {
        var __self = this;
        return function() {
            var ret = __self.apply(this, arguments);
            afterfn.apply(this, arguments);
            return ret;
        }
    };
    document.getElementById = document.getElementById.before(function() {
        alert("取button之前");
    });
    var button = document.getElementById('button');
    console.log(button);


    window.onload = function() {
        alert(1);
    }
    window.onload = (window.onload || function() {}).after(function() {
        alert(2);
    }).after(function() {
        alert(3);
    }).after(function() {
        alert(4);
    });

    // 上面的AOP 实现是在Function.prototype 上添加before 和after 方法，但许多人不喜欢这种污染原型的方式，那么我们可以做一些变通，把原函数和新函数都作为参数传入before 或者after 方法
    var before = function(fn, beforefn) {
        return function() {
            beforefn.apply(this, arguments);
            return fn.apply(this, arguments);
        }
    }
    var a = before(
        function() {
            alert(3)
        },
        function() {
            alert(2)
        }
    );
    a = before(a, function() {
        alert(1);
    });
    a();
    </script>
</body>

</html>

```

### 用AOP实现数据上报
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button tag="login" id="button">点击打开登录浮层</button>
    <script>
    // 一般在业务完成后要要求添加上数据统计的功能

    // 普通上报
    // var showLogin = function() {
    //     console.log('打开登录浮层');
    //     log(this.getAttribute('tag'));
    // }
    // var log = function(tag) {
    //     console.log('上报标签为: ' + tag);
    //     // (new Image).src = 'http:// xxx.com/report?tag=' + tag; // 真正的上报代码略
    // }
    // document.getElementById('button').onclick = showLogin;
    // showLogin既打开浮层又上报

    // 使用AOP分离
    Function.prototype.after = function(afterfn) {
        var __self = this;
        return function() {
            var ret = __self.apply(this, arguments);
            afterfn.apply(this, arguments);
            return ret;
        }
    };
    var showLogin = function() {
        console.log('打开登录浮层');
    };
    var log = function() {
        console.log('上报标签为: ' + this.getAttribute('tag'));
    };
    showLogin = showLogin.after(log); // 打开登录浮层之后上报数据
    document.getElementById('button').onclick = showLogin;
    </script>
</body>

</html>

```

### 用AOP动态改变函数的参数
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 1,2公用了arguments，所以可以在1中将arguments改变
    Function.prototype.before = function(beforefn) {
        var __self = this;
        return function() {
            beforefn.apply(this, arguments); // (1)
            return __self.apply(this, arguments); // (2)
        }
    };
    var func = function(param) {
        console.log(param); // 输出： {a: "a", b: "b"}
    }
    func = func.before(function(param) {
        param.b = 'b';
    });
    func({
        a: 'a'
    });
    </script>
</body>

</html>

```

### 利用AOP动态改变参数实现发送ajax前添加token
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 需要在每次发送ajax前添加一个验证用的token

    // 通常做法
    // var getToken = function() {
    //     return 'Token';
    // }
    // var ajax = function(type, url, param) {
    //     param = param || {};
    //     param.Token = getToken(); // 发送ajax 请求的代码略...
    // };

    // 如果要将这个ajax发布到网上，可能有些人就不需要token
    // 改进
    Function.prototype.before = function(beforefn) {
        var __self = this; // 保存原函数的引用
        return function() { // 返回包含了原函数和新函数的"代理"函数
            beforefn.apply(this, arguments); // 执行新函数，且保证this 不被劫持，新函数接受的参数
            // 也会被原封不动地传入原函数，新函数在原函数之前执行
            return __self.apply(this, arguments); // 执行原函数并返回原函数的执行结果，
            // 并且保证this 不被劫持
        }
    };
    var ajax = function(type, url, param) {
        console.log(param);
    };
    var getToken = function() {
        return "Token";
    };
    // 在ajax发送之前添加token
    ajax = ajax.before(function(type, url, param) {
        param.Token = getToken();
    });
    ajax("get", 'http:// xxx.com/userinfo', {
        name: 'sven'
    });
    </script>
</body>

</html>
```

### 插件式的表单验证
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>用户名：
    <input id="username" type="text" /> 密码：
    <input id="password" type="password" />
    <input id="submitBtn" type="button" value="提交">
    <script>
    // 正常写法
    // var username = document.getElementById('username'),
    //     password = document.getElementById('password'),
    //     submitBtn = document.getElementById('submitBtn');
    // var formSubmit = function() {
    //     if (username.value === '') {
    //         return alert('用户名不能为空');
    //     }
    //     if (password.value === '') {
    //         return alert('密码不能为空');
    //     }
    //     var param = {
    //         username: username.value,
    //         password: password.value
    //     }
    //     ajax('http:// xxx.com/login', param); // ajax 具体实现略
    // }
    // submitBtn.onclick = function() {
    //     formSubmit();
    // };

    // 存在的问题formSubmit承担了过多的职责，一要验证，二要负责提交

    // 将验证逻辑分离到validata中
    // var validata = function() {
    //     if (username.value === '') {
    //         alert('用户名不能为空');
    //         return false;
    //     }
    //     if (password.value === '') {
    //         alert('密码不能为空');
    //         return false;
    //     }
    // };
    // var formSubmit = function() {
    //     if (validata() === false) { // 校验未通过
    //         return;
    //     }
    //     var param = {
    //         username: username.value,
    //         password: password.value
    //     }
    //     ajax('http:// xxx.com/login', param);
    // };
    // submitBtn.onclick = function() {
    //     formSubmit();
    // };
    // 此时，虽将验证逻辑抽离，但formSubmit中还需要validata的返回值
    // 再次优化

    Function.prototype.before = function(beforefn) {
        var __self = this;
        return function() {
            if (beforefn.apply(this, arguments) === false) {
                // beforefn 返回false 的情况直接return，不再执行后面的原函数
                return;
            }
            return __self.apply(this, arguments);
        }
    };
    var validata = function() {
        if (username.value === '') {
            alert('用户名不能为空');
            return false;
        }
        if (password.value === '') {
            alert('密码不能为空');
            return false;
        }
    };
    var formSubmit = function() {
        var param = {
            username: username.value,
            password: password.value
        }
        ajax('http:// xxx.com/login', param);
    };
    // 利用AOP动态织入验证逻辑
    formSubmit = formSubmit.before(validata);
    submitBtn.onclick = function() {
        formSubmit();
    }
    </script>
</body>

</html>

```

### AOP的注意事项
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    // before,after返回的实际上是一个新函数，所以原先保存在函数上的属性会丢失;这种装饰方式也叠加了函数的作用域，如果装饰的链条过长，性能上也会受到一些影响。
    Function.prototype.after = function(afterfn) {
        var __self = this;
        return function() {
            var ret = __self.apply(this, arguments);
            afterfn.apply(this, arguments);
            return ret;
        }
    };
    var func = function() {
        alert(1);
    }
    func.a = 'a';
    func = func.after(function() {
        alert(2);
    });
    alert(func.a); // 输出：undefined

    // 代理模式和装饰者模式的不同
    // 代理模式和装饰者模式最重要的区别在于它们的意图和设计目的。代理模式的目的是，当直接访问本体不方便或者不符合需要时，为这个本体提供一个替代者。本体定义了关键功能，而代理提供或拒绝对它的访问，或者在访问本体之前做一些额外的事情。装饰者模式的作用就是为对象动态加入行为。换句话说，代理模式强调一种关系（Proxy 与它的实体之间的关系），这种关系可以静态的表达，也就是说，这种关系在一开始就可以被确定。而装饰者模式用于一开始不能确定对象的全部功能时。代理模式通常只有一层代理本体的引用，而装饰者模式经常会形成一条长长的装饰链。
    </script>
</body>

</html>

```

## 状态模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 状态模式:允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。状态模式的关键是区分事物内部的状态，事物内部状态的改变往往会带来事物的行为改变。往往将状态封装成一个类，跟此状态有关的所有行为都被封装在这个类的内部
    // 使用场景:程序有多种状态，并状态改变会影响其他行为时
    // 使用举例:一个开关的两个状态(开、关)，将影响灯泡的状态
    var Light = function() {
        this.state = 'off'; // 给电灯设置初始状态off
        this.button = null; // 电灯开关按钮
    };
    Light.prototype.init = function() {
        var button = document.createElement('button'),
            self = this;
        button.innerHTML = '开关';
        this.button = document.body.appendChild(button);
        this.button.onclick = function() {
            self.buttonWasPressed();
        }
    };
    Light.prototype.buttonWasPressed = function() {
        if (this.state === 'off') {
            console.log('开灯');
            this.state = 'on';
        } else if (this.state === 'on') {
            console.log('关灯');
            this.state = 'off';
        }
    };
    var light = new Light();
    light.init();

    //这个世界上的电灯并非只有一种。许多酒店里有另外一种电灯，这种电灯也只有一个开关，但它的表现是：第一次按下打开弱光，第二次按下打开强光，第三次才是关闭电灯
    Light.prototype.buttonWasPressed = function() {
        if (this.state === 'off') {
            console.log('弱光');
            this.state = 'weakLight';
        } else if (this.state === 'weakLight') {
            console.log('强光');
            this.state = 'strongLight';
        } else if (this.state === 'strongLight') {
            console.log('关灯');
            this.state = 'off';
        }
    };
    // 上面的缺点
    // buttonWasPressed方法违反了开放-封闭原则，每次新增或修改light的状态都需要改动buttonWasPressed方法
    // 所有跟状态有关的行为都封装在了buttonWasPressed方法里，以后若需要修改可能会造成此方法的无限膨胀
    // 状态的切换非常不明显，仅仅表现在对state变量的赋值，实际过程中可能会遗忘，也无法一目了然的知道总共有多少状态
    // 状态之间的切换关系依靠if...else来切换，维护麻烦
    </script>
</body>

</html>

```

### 状态模式改进电灯程序
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 通常我们谈到封装，一般都会优先封装对象的行为，而不是对象的状态。但在状态模式中刚好相反，状态模式的关键是把事物的每种状态都封装成单独的类，跟此种状态有关的行为都被封装在这个类的内部，所以button 被按下的的时候，只需要在上下文中，把这个请求委托给当前的状态对象即可，该状态对象会负责渲染它自身的行为

    // OffLightState
    var OffLightState = function(light) {
        this.light = light;
    };
    OffLightState.prototype.buttonWasPressed = function() {

        console.log("弱光"); // offLightState对应的行为
        this.light.setState(this.light.weakLightState); // 切换到weakLightState
    };
    // WeakLightState：
    var WeakLightState = function(light) {
        this.light = light;
    };
    WeakLightState.prototype.buttonWasPressed = function() {
        console.log('强光'); // weakLightState 对应的行为
        this.light.setState(this.light.strongLightState); // 切换状态到strongLightState
    };
    // StrongLightState：
    var StrongLightState = function(light) {
        this.light = light;
    };
    StrongLightState.prototype.buttonWasPressed = function() {
        console.log('超强光'); // strongLightState 对应的行为
        this.light.setState(this.light.superStrongLightState); // 切换状态到offLightState
    };
    var SuperStrongLightState = function(light) {
        this.light = light;
    };
    SuperStrongLightState.prototype.buttonWasPressed = function() {
        console.log('关灯');
        this.light.setState(this.light.offLightState);
    };
    var Light = function() {
        console.log(this);
        this.offLightState = new OffLightState(this);
        this.weakLightState = new WeakLightState(this);
        this.strongLightState = new StrongLightState(this);
        this.superStrongLightState = new SuperStrongLightState(this); // 新增superStrongLightState 对象
        this.button = null;
    };
    Light.prototype.init = function() {
        var button = document.createElement("button"),
            self = this;
        this.button = document.body.appendChild(button);
        this.button.innerHTML = "开关";
        this.currState = this.offLightState; // 设置当前状态
        this.button.onclick = function() {
            self.currState.buttonWasPressed();
        };
    };
    Light.prototype.setState = function(newState) {
        this.currState = newState;
    };
    var light = new Light();
    light.init();

    // 添加新状态时
    </script>
</body>

</html>

```

### 状态模式的通用结构
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 首先定义了Light 类，Light类在这里也被称为上下文（Context）。随后在Light 的构造函数中，我们要创建每一个状态类的实例对象，Context将持有这些状态对象的引用，以便把请求委托给状态对象。用户的请求，即点击button的动作也是实现在Context中的
    var Light = function() {
        this.offLightState = new OffLightState(this); // 持有状态对象的引用
        this.weakLightState = new WeakLightState(this);
        this.strongLightState = new StrongLightState(this);
        this.superStrongLightState = new SuperStrongLightState(this);
        this.button = null;
    };
    Light.prototype.init = function() {
        var button = document.createElement('button'),
            self = this;
        this.button = document.body.appendChild(button);
        this.button.innerHTML = '开关';
        this.currState = this.offLightState; // 设置默认初始状态
        this.button.onclick = function() { // 定义用户的请求动作
            self.currState.buttonWasPressed();
        }
    };
    // 接下来可能是个苦力活，我们要编写各种状态类，light对象被传入状态类的构造函数，状态对象也需要持有light对象的引用，以便调用light 中的方法或者直接操作light 对象
    var OffLightState = function(light) {
        this.light = light;
    };
    OffLightState.prototype.buttonWasPressed = function() {
        console.log('弱光');
        this.light.setState(this.light.weakLightState);
    };
    </script>
</body>

</html>

```

### 状态模式实现文件上传
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script type="text/javascript">
    //文件在扫描状态中，是不能进行任何操作的，既不能暂停也不能删除文件，只能等待扫描完成。扫描完成之后，根据文件的md5 值判断，若确认该文件已经存在于服务器，直接跳到上传完成状态。如果该文件的大小超过允许上传的最大值，或者该文件已经损坏，则跳往上传失败状态。剩下的情况下才进入上传中状态。
    // 上传过程中可以点击暂停按钮来暂停上传，暂停后点击同一个按钮会继续上传。
    // 扫描和上传过程中，点击删除按钮无效，只有在暂停、上传完成、上传失败之后，才能    删除文件。

    // 正常实现
    window.external.upload = function(state) {
        console.log(state); // 可能为sign、uploading、done、error
    };
    var plugin = (function() {
        var plugin = document.createElement('embed');
        plugin.style.display = 'none';
        plugin.type = 'application/txftn-webkit';
        plugin.sign = function() {
            console.log('开始文件扫描');
        }
        plugin.pause = function() {
            console.log('暂停文件上传');
        };
        plugin.uploading = function() {
            console.log('开始文件上传');
        };
        plugin.del = function() {
            console.log('删除文件上传');
        }
        plugin.done = function() {
            console.log('文件上传完成');
        }
        document.body.appendChild(plugin);
        return plugin;
    })();

    var Upload = function(fileName) {
        this.plugin = plugin;
        this.fileName = fileName;
        this.button1 = null;
        this.button2 = null;
        this.state = 'sign'; // 设置初始状态为waiting
    };
    Upload.prototype.init = function() {
        var that = this;
        this.dom = document.createElement('div');
        this.dom.innerHTML =
            '<span>文件名称:' + this.fileName + '</span>\
<button data-action="button1">扫描中</button>\
<button data-action="button2">删除</button>';
        document.body.appendChild(this.dom);
        this.button1 = this.dom.querySelector('[data-action="button1"]'); // 第一个按钮
        this.button2 = this.dom.querySelector('[data-action="button2"]'); // 第二个按钮
        this.bindEvent();
    };
    Upload.prototype.bindEvent = function() {
        var self = this;
        this.button1.onclick = function() {
            if (self.state === 'sign') { // 扫描状态下，任何操作无效
                console.log('扫描中，点击无效...');
            } else if (self.state === 'uploading') { // 上传中，点击切换到暂停
                self.changeState('pause');
            } else if (self.state === 'pause') { // 暂停中，点击切换到上传中
                self.changeState('uploading');
            } else if (self.state === 'done') {
                console.log('文件已完成上传, 点击无效');
            } else if (self.state === 'error') {
                console.log('文件上传失败, 点击无效');
            }
        };
        this.button2.onclick = function() {
            if (self.state === 'done' || self.state === 'error' || self.state === 'pause') {
                // 上传完成、上传失败和暂停状态下可以删除
                self.changeState('del');
            } else if (self.state === 'sign') {
                console.log('文件正在扫描中，不能删除');
            } else if (self.state === 'uploading') {
                console.log('文件正在上传中，不能删除');
            }
        };
    };
    Upload.prototype.changeState = function(state) {
        switch (state) {
            case 'sign':
                this.plugin.sign();
                this.button1.innerHTML = '扫描中，任何操作无效';
                break;
            case 'uploading':
                this.plugin.uploading();
                this.button1.innerHTML = '正在上传，点击暂停';
                break;
            case 'pause':
                this.plugin.pause();
                this.button1.innerHTML = '已暂停，点击继续上传';
                break;
            case 'done':
                this.plugin.done();
                this.button1.innerHTML = '上传完成';
                break;
            case 'error':
                this.button1.innerHTML = '上传失败';
                break;
            case 'del':
                this.plugin.del();
                this.dom.parentNode.removeChild(this.dom);
                console.log('删除完成');
                break;
        };
        this.state = state;
    };
    var uploadObj = new Upload('JavaScript 设计模式与开发实践');
    uploadObj.init();
    window.external.upload = function(state) { // 插件调用JavaScript 的方法
        uploadObj.changeState(state);
    };
    window.external.upload('sign'); // 文件开始扫描
    setTimeout(function() {
        window.external.upload('uploading'); // 1 秒后开始上传
    }, 1000);
    setTimeout(function() {
        window.external.upload('done'); // 5 秒后上传完成
    }, 5000);
    </script>
</body>

</html>

```

### 状态模式重构文件上传
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    window.external.upload = function(state) {
        console.log(state); // 可能为sign、uploading、done、error
    };
    var plugin = (function() {
        var plugin = document.createElement('embed');
        plugin.style.display = 'none';
        plugin.type = 'application/txftn-webkit';
        plugin.sign = function() {
            console.log('开始文件扫描');
        }
        plugin.pause = function() {
            console.log('暂停文件上传');
        };
        plugin.uploading = function() {
            console.log('开始文件上传');
        };
        plugin.del = function() {
            console.log('删除文件上传');
        }
        plugin.done = function() {
            console.log('文件上传完成');
        }
        document.body.appendChild(plugin);
        return plugin;
    })();
    var Upload = function(fileName) {
        this.plugin = plugin;
        this.fileName = fileName;
        this.button1 = null;
        this.button2 = null;
        this.signState = new SignState(this); // 设置初始状态为waiting
        this.uploadingState = new UploadingState(this);
        this.pauseState = new PauseState(this);
        this.doneState = new DoneState(this);
        this.errorState = new ErrorState(this);
        this.currState = this.signState; // 设置当前状态
    };
    Upload.prototype.init = function() {
        var that = this;
        this.dom = document.createElement('div');
        this.dom.innerHTML =
            '<span>文件名称:' + this.fileName + '</span>\
<button data-action="button1">扫描中</button>\
<button data-action="button2">删除</button>';
        document.body.appendChild(this.dom);
        this.button1 = this.dom.querySelector('[data-action="button1"]');
        this.button2 = this.dom.querySelector('[data-action="button2"]');
        this.bindEvent();
    };
    Upload.prototype.bindEvent = function() {
        var self = this;
        this.button1.onclick = function() {
            self.currState.clickHandler1();
        }
        this.button2.onclick = function() {
            self.currState.clickHandler2();
        }
    };
    Upload.prototype.sign = function() {
        this.plugin.sign();
        this.currState = this.signState;
    };
    Upload.prototype.uploading = function() {
        this.button1.innerHTML = '正在上传，点击暂停';
        this.plugin.uploading();
        this.currState = this.uploadingState;
    };
    Upload.prototype.pause = function() {
        this.button1.innerHTML = '已暂停，点击继续上传';
        this.plugin.pause();
        this.currState = this.pauseState;
    };
    Upload.prototype.done = function() {
        this.button1.innerHTML = '上传完成';
        this.plugin.done();
        this.currState = this.doneState;
    };
    Upload.prototype.error = function() {
        this.button1.innerHTML = '上传失败';
        this.currState = this.errorState;
    };
    Upload.prototype.del = function() {
        this.plugin.del();
        this.dom.parentNode.removeChild(this.dom);
    };
    var StateFactory = (function() {
        var State = function() {};
        State.prototype.clickHandler1 = function() {
            throw new Error('子类必须重写父类的clickHandler1 方法');
        }
        State.prototype.clickHandler2 = function() {
            throw new Error('子类必须重写父类的clickHandler2 方法');
        }
        return function(param) {
            var F = function(uploadObj) {
                this.uploadObj = uploadObj;
            };
            F.prototype = new State();
            for (var i in param) {
                F.prototype[i] = param[i];
            }
            return F;
        }
    })();
    var SignState = StateFactory({
        clickHandler1: function() {
            console.log('扫描中，点击无效...');
        },
        clickHandler2: function() {
            console.log('文件正在上传中，不能删除');
        }
    });
    var UploadingState = StateFactory({
        clickHandler1: function() {
            this.uploadObj.pause();
        },
        clickHandler2: function() {
            console.log('文件正在上传中，不能删除');
        }
    });
    var PauseState = StateFactory({
        clickHandler1: function() {
            this.uploadObj.uploading();
        },
        clickHandler2: function() {
            this.uploadObj.del();
        }
    });
    var DoneState = StateFactory({
        clickHandler1: function() {
            console.log('文件已完成上传, 点击无效');
        },
        clickHandler2: function() {
            this.uploadObj.del();
        }
    });
    var ErrorState = StateFactory({
        clickHandler1: function() {
            console.log('文件上传失败, 点击无效');
        },
        clickHandler2: function() {
            this.uploadObj.del();
        }
    });
    var uploadObj = new Upload('JavaScript 设计模式与开发实践');
    uploadObj.init();
    window.external.upload = function(state) {
        uploadObj[state]();
    };
    window.external.upload('sign');
    setTimeout(function() {
        window.external.upload('uploading'); // 1 秒后开始上传
    }, 1000);
    setTimeout(function() {
        window.external.upload('done'); // 5 秒后上传完成
    }, 5000);
    </script>
</body>

</html>

```

### 状态模式注意事项
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 状态模式的优缺点
    // 状态模式定义了状态与行为之间的关系，并将它们封装在一个类里。通过增加新的状态类，很容易增加新的状态和转换。
    // 避免Context 无限膨胀，状态切换的逻辑被分布在状态类中，也去掉了Context 中原本过多的条件分支。
    // 用对象代替字符串来记录当前状态，使得状态的切换更加一目了然。
    // Context 中的请求动作和状态类中封装的行为可以非常容易地独立变化而互不影响。

    // 缺点
    // 状态模式的缺点是会在系统中定义许多状态类，编写20 个状态类是一项枯燥乏味的工作，而且系统中会因此而增加不少对象。另外，由于逻辑分散在状态类中，虽然避开了不受欢迎的条件分支语句，但也造成了逻辑分散的问题，我们无法在一个地方就看出整个状态机的逻辑。

    // 状态模式的性能优化点
    // 有两种选择来管理state 对象的创建和销毁。第一种是仅当state 对象被需要时才创建并随后销毁，另一种是一开始就创建好所有的状态对象，并且始终不销毁它们。如果state对象比较庞大，可以用第一种方式来节省内存，这样可以避免创建一些不会用到的对象并及时地回收它们。但如果状态的改变很频繁，最好一开始就把这些state 对象都创建出来，也没有必要销毁它们，因为可能很快将再次用到它们。
    // 在本章的例子中，我们为每个Context 对象都创建了一组state 对象，实际上这些state对象之间是可以共享的，各Context 对象可以共享一个state 对象，这也是享元模式的应用场景之一。

    // 状态模式和策略模式的关系
    // 相同点
    // 策略模式和状态模式的相同点是，它们都有一个上下文、一些策略或者状态类，上下文把请求委托给这些类来执行。
    // 不同点
    // 策略模式中的各个策略类之间是平等又平行的，它们之间没有任何联系，所以客户必须熟知这些策略类的作用，以便客户可以随时主动切换算法；而在状态模式中，状态和状态对应的行为是早已被封装好的，状态之间的切换也早被规定完成，“改变行为”这件事情发生在状态模式内部。对客户来说，并不需要了解这些细节。这正是状态模式的作用所在。
    </script>
</body>

</html>

```

### js版本的状态机
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 状态模式是状态机的实现之一，但在JavaScript 这种“无类”语言中，没有规定让状态对象一定要从类中创建而来。另外一点，JavaScript 可以非常方便地使用委托技术，并不需要事先让一个对象持有另一个对象。下面的状态机选择了通过Function.prototype.call 方法直接把请求委托给某个字面量对象来执行。

    // 改写灯泡
    var Light = function() {
        this.currState = FSM.off; // 设置当前状态
        this.button = null;
    };
    Light.prototype.init = function() {
        var button = document.createElement("button"),
            self = this;
        button.innerHTML = "已关灯";
        this.button = document.body.appendChild(button);
        this.button.onclick = function() {
            self.currState.buttonWasPressed.call(self); // 把请求委托给FSM状态机
        };
    };
    var FSM = {
        off: {
            buttonWasPressed: function() {
                console.log('关灯');
                this.button.innerHTML = '下一次按我是开灯';
                this.currState = FSM.on;
            }
        },
        on: {
            buttonWasPressed: function() {
                console.log('开灯');
                this.button.innerHTML = '下一次按我是关灯';
                this.currState = FSM.off;
            }
        }
    };
    var light = new Light();
    light.init();
    </script>
</body>

</html>

```

### 使用delegate函数实现状态机
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    var delegate = function(client, delegation) {
        return {
            buttonWasPressed: function() { // 将客户的操作委托给delegation 对象
                return delegation.buttonWasPressed.apply(client, arguments);
            }
        }
    };
    var FSM = {
        off: {
            buttonWasPressed: function() {
                console.log('关灯');
                this.button.innerHTML = '下一次按我是开灯';
                this.currState = this.onState;
            }
        },
        on: {
            buttonWasPressed: function() {
                console.log('开灯');
                this.button.innerHTML = '下一次按我是关灯';
                this.currState = this.offState;
            }
        }
    };
    var Light = function() {
        this.offState = delegate(this, FSM.off);
        this.onState = delegate(this, FSM.on);
        this.currState = this.offState; // 设置初始状态为关闭状态
        this.button = null;
    };
    Light.prototype.init = function() {
        var button = document.createElement('button'),
            self = this;
        button.innerHTML = '已关灯';
        this.button = document.body.appendChild(button);
        this.button.onclick = function() {
            self.currState.buttonWasPressed();
        }
    };
    var light = new Light();
    light.init();
    </script>
</body>

</html>

```

### 表驱动的有限状态机
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 参考:https://github.com/jakesgordon/javascript-state-machine
    var fsm = StateMachine.create({
        initial: 'off',
        events: [{
            name: 'buttonWasPressed',
            from: 'off',
            to: 'on'
        }, {
            name: 'buttonWasPressed',
            from: 'on',
            to: 'off'
        }],
        callbacks: {
            onbuttonWasPressed: function(event, from, to) {
                console.log(arguments);
            }
        },
        error: function(eventName, from, to, args, errorCode, errorMessage) {
            console.log(arguments); // 从一种状态试图切换到一种不可能到达的状态的时候
        }
    });
    button.onclick = function() {
        fsm.buttonWasPressed();
    }
    </script>
</body>

</html>

```

## 适配器模式

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 适配器模式:适配器模式的作用是解决两个软件实体间的接口不兼容的问题,它不考虑这些接口是怎样实现的，也不考虑它们将来可能会如何演化。适配器模式不需要改变已有的接口，就能够使它们协同作用。使用适配器模式之后，原本由于接口不兼容而不能工作的两个软件实体可以一起工作。
    // 使用场景:当两个接口不一样时，可以采用亡羊补牢的方式实现一个适配器，来提供一个统一的接口
    // 使用举例:百度谷歌地图提供的展示地图的API不同，一个show,一个display
    var googleMap = {
        show: function() {
            console.log('开始渲染谷歌地图');
        }
    };
    var baiduMap = {
        display: function() {
            console.log('开始渲染百度地图');
        }
    };
    var baiduMapAdapter = {
        show: function() {
            return baiduMap.display();
        }
    };
    renderMap(googleMap); // 输出：开始渲染谷歌地图
    renderMap(baiduMapAdapter); // 输出：开始渲染百度地图
    </script>
</body>

</html>

```

## 单一职责原则

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 就一个类而言，应该仅有一个引起它变化的原因。在JS中类用的很少，单一职责更多的是体现在对象和方法上
    // SRP原则体现为：一个对象（方法）只做一件事情。->职责分离
    // SRP原则在代理模式、迭代器模式、单例模式、装饰者模式中都有体现

    // 何时用职责分离:
    // 如果随着需求的变化，有2个职责总是同时变化，则不必分离他们
    // 如果两个职责以后确定不会发生变化，则不用分离

    // SRP原则优缺点
    // 优点:降低了单个类或者对象的复杂度，按照职责把对象分解成更小的粒度，这有助于代码的复用，也有利于进行单元测试。当一个职责需要变更的时候，不会影响到其他的职责。
    // 缺点:会增加编写代码的复杂度。当我们按照职责把对象分解成更小的粒度之后，实际上也增大了这些对象之间相互联系的难度。
    </script>
</body>

</html>

```

## 最少知识原则

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 最少知识原则（LKP）说的是一个软件实体应当尽可能少地与其他实体发生相互作用。这里的软件实体是一个广义的概念，不仅包括对象，还包括系统、类、模块、函数、变量等。
    // 最少知识原则要求我们在设计程序时，应当尽量减少对象之间的交互。如果两个对象之间不必彼此直接通信，那么这两个对象就不要发生直接的相互联系。常见的做法是引入一个第三者对象，来承担这些对象之间的通信作用。如果一些对象需要向另一些对象发起请求，可以通过第三者对象来转发这些请求。
    // 中介者模式、外观模式(定义了一个高层接口去封装一组“子系统”，可以直接定义调用高层接口完成一组事情，也可调用子系统中的某一个接口完成某一个特定事情)中都有体现。
    </script>
</body>

</html>

```

## 开放封闭原则

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 在面向对象的程序设计中，开放-封闭原则（OCP）是最重要的一条原则
    // 软件实体（类、模块、函数）等应该是可以扩展的，但是不可修改。(进行扩展，但不能修改源代码->保留源码基础上增加代码)
    // 开放-封闭原则的思想：当需要改变一个程序的功能或者给这个程序增加新功能的时候，可以使用增加代码的方式，但是不允许改动程序的源代码。
    // 挑选出最容易发生变化的地方，然后构造抽象来封闭这些变化。在不可避免发生修改的时候，尽量修改那些相对容易修改的地方。
    // 发布-订阅、模板方法、策略、代理、职责链等模式都有体现

    // 过多的条件分支语句是造成程序违反开放-封闭原则的一个常见原因。每当需要增加一个新的if时，就必须修改源代码。
    // 当看到很多if时，应考虑能否使用对象的多态性来重构(找出变化部分，将其封装起来，这样就可以将变化的和稳定的隔离开来)

    // 反例，很多if，违反开放-封闭原则
    // var makeSound = function(animal) {
    //     if (animal instanceof Duck) {
    //         console.log('嘎嘎嘎');
    //     } else if (animal instanceof Chicken) {
    //         console.log('咯咯咯');
    //     }
    // };
    // var Duck = function() {};
    // var Chicken = function() {};
    // makeSound(new Duck()); // 输出：嘎嘎嘎
    // makeSound(new Chicken()); // 输出：咯咯咯

    // 利用多态的思想，我们把程序中不变的部分隔离出来（动物都会叫），然后把可变的部分封装起来（不同类型的动物发出不同的叫声），这样一来程序就具有了可扩展性。当我们想让一只狗发出叫声时，只需增加一段代码即可，而不用去改动原有的makeSound 函数
    var makeSound = function(animal) {
        animal.sound();
    };
    var Duck = function() {};
    Duck.prototype.sound = function() {
        console.log('嘎嘎嘎');
    };
    var Chicken = function() {};
    Chicken.prototype.sound = function() {
        console.log('咯咯咯');
    };
    makeSound(new Duck()); // 嘎嘎嘎
    makeSound(new Chicken()); // 咯咯咯
    /********* 增加动物狗，不用改动原有的makeSound 函数 ****************/
    var Dog = function() {};
    Dog.prototype.sound = function() {
        console.log('汪汪汪');
    };
    makeSound(new Dog()); // 汪汪汪

    // 实现开放-封闭原则的其他方法:放置钩子见52、使用回调(特殊钩子)
    </script>
</body>

</html>

```

## 接口和面向接口编程

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 接口:
    // 第一种含义:我们经常说一个库或者模块对外提供了某某API 接口。通过主动暴露的接口来通信，可以隐藏软件系统内部的工作细节
    // 第二种含义:是一些语言提供的关键字，比如Java 的interface。interface 关键字可以产生一个完全抽象的类。这个完全抽象的类用来表示一种契约，专门负责建立类与类之间的联系。
    // 第三种含义:“面向接口编程”中的接口，接口的含义在这里体现得更为抽象。->接口是对象能响应的请求的集合。
    </script>
</body>

</html>

```

## 代码重构技巧

### 基本定义
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <script>
    // 1.提炼函数
    var getUserInfo = function() {
        ajax('http:// xxx.com/userInfo', function(data) {
            // 有打印数据的操作，可提炼出来
            console.log('userId: ' + data.userId);
            console.log('userName: ' + data.userName);
            console.log('nickName: ' + data.nickName);
        });
    };
    // 改进
    var getUserInfo = function() {
        ajax('http:// xxx.com/userInfo', function(data) {
            printDetails(data);
        });
    };
    var printDetails = function(data) {
        console.log('userId: ' + data.userId);
        console.log('userName: ' + data.userName);
        console.log('nickName: ' + data.nickName);
    };

    // 2.合并重复的条件片段
    var paging = function(currPage) {
        // 每个分支里面都有jump
        if (currPage <= 0) {
            currPage = 0;
            jump(currPage); // 跳转
        } else if (currPage >= totalPage) {
            currPage = totalPage;
            jump(currPage); // 跳转
        } else {
            jump(currPage); // 跳转
        }
    };
    // 改进
    var paging = function(currPage) {
        if (currPage <= 0) {
            currPage = 0;
        } else if (currPage >= totalPage) {
            currPage = totalPage;
        }
        jump(currPage); // 把jump 函数独立出来
    };

    function jump(page) {
        xxxxx
    }

    // 3.把条件分支语句提炼成函数
    var getPrice = function(price) {
        var date = new Date();
        if (date.getMonth() >= 6 && date.getMonth() <= 9) { // 可将判断夏天提取出来，单独使用
            return price * 0.8;
        }
        return price;
    };
    // 改进
    var isSummer = function() {
        var date = new Date();
        return date.getMonth() >= 6 && date.getMonth() <= 9;
    };
    var getPrice = function(price) {
        if (isSummer()) { // 夏天
            return price * 0.8;
        }
        return price;
    };

    // 4.合理使用循环
    var createXHR = function() {
        var xhr;
        try {
            xhr = new ActiveXObject('MSXML2.XMLHttp.6.0');
        } catch (e) {
            try {
                xhr = new ActiveXObject('MSXML2.XMLHttp.3.0');
            } catch (e) {
                xhr = new ActiveXObject('MSXML2.XMLHttp');
            }
        }
        return xhr;
    };
    var xhr = createXHR();
    // 可用循环一次生成
    var createXHR = function() {
        var versions = ['MSXML2.XMLHttp.6.0ddd', 'MSXML2.XMLHttp.3.0', 'MSXML2.XMLHttp'];
        for (var i = 0, version; version = versions[i++];) {
            try {
                return new ActiveXObject(version);
            } catch (e) {}
        }
    };
    var xhr = createXHR();

    // 5.提前让函数退出代替嵌套条件分支
    var del = function(obj) {
        var ret;
        if (!obj.isReadOnly) { // 不为只读的才能被删除
            if (obj.isFolder) { // 如果是文件夹
                ret = deleteFolder(obj);
            } else if (obj.isFile) { // 如果是文件
                ret = deleteFile(obj);
            }
        }
        return ret;
    };
    // 改进
    var del = function(obj) {
        if (obj.isReadOnly) { // 反转if 表达式
            return;
        }
        if (obj.isFolder) {
            return deleteFolder(obj);
        }
        if (obj.isFile) {
            return deleteFile(obj);
        }
    };

    // 6.传递对象参数代替过长的参数列表
    var setUserInfo = function(id, name, address, sex, mobile, qq) {
        console.log('id= ' + id);
        console.log('name= ' + name);
        console.log('address= ' + address);
        console.log('sex= ' + sex);
        console.log('mobile= ' + mobile);
        console.log('qq= ' + qq);
    };
    setUserInfo(1314, 'sven', 'shenzhen', 'male', '137********', 377876679);
    // 改进
    var setUserInfo = function(obj) {
        console.log('id= ' + obj.id);
        console.log('name= ' + obj.name);
        console.log('address= ' + obj.address);
        console.log('sex= ' + obj.sex);
        console.log('mobile= ' + obj.mobile);
        console.log('qq= ' + obj.qq);
    };
    setUserInfo({
        id: 1314,
        name: 'sven',
        address: 'shenzhen',
        sex: 'male',
        mobile: '137********',
        qq: 377876679
    });

    // 7.尽量减少参数数量
    var draw = function(width, height, square) {};
    // 改进
    var draw = function(width, height) {
        var square = width * height;
    };

    // 8.少用三目运算符，仅在简单判断时使用，复杂情况下用if
    if (!aup || !bup) {
        return a === doc ? -1 :
            b === doc ? 1 :
            aup ? -1 :
            bup ? 1 :
            sortInput ?
            (indexOf.call(sortInput, a) - indexOf.call(sortInput, b)) :
            0;
    }

    // 9.分解大型类
    // 10.用return退出多重循环
    var func = function() {
        for (var i = 0; i < 10; i++) {
            for (var j = 0; j < 10; j++) {
                if (i * j > 30) {
                    return;
                }
            }
        }
        console.log(i); // 这句代码没有机会被执行
    };
    // 改进
    var print = function(i) {
        console.log(i);
    };
    var func = function() {
        for (var i = 0; i < 10; i++) {
            for (var j = 0; j < 10; j++) {
                if (i * j > 30) {
                    return print(i);
                }
            }
        }
    };
    func();
    </script>
</body>

</html>

```
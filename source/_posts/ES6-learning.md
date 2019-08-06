---
title: ES6-learning
tags:
  - ES6
categories:
  - 前端
date: 2017-09-26 16:56:17
---

# ES6学习笔记
> 最近在看阮老师的ES6教程，所以特此记录一些重点。
> 例子使用的大都是阮老师的例子，感谢阮老师的无私奉献。

## 简介
- es6泛指下一代js标准，主要涵盖了es2015,es2016,es2017；一般说的es6主要指的es2015

## let
- let实际为js提供了`块级作用域`，用法类似`var`，但它所声明的变量，仅在`let`所在的代码块中有效。
```javascript
if(true){
    let a=3;
}
console.log(a);// 报错
```
- for循环中的计数变量非常适合使用`let`
    - 使用`var`声明的
    ```javascript
    var a = [];
    for (var i = 0; i < 10; i++) {
      a[i] = function () {
        console.log(i);
      };
    }
    a[6](); // 10，因为i是全局变量，全局有效，调用时得到的是最后一次的值
    ```
    - 使用`let`声明的
    ```javascript
    var a = [];
    for (let i = 0; i < 10; i++) {
      a[i] = function () {
        console.log(i);
      };
    }
    a[6](); // 6，使用let后，每轮循环的i，仅在本轮循环中有效，这样其实每轮循环都有一个新的i值。后台负责记录上一次的i值；
    ```
    - for循环设置循环变量的部分是一个父作用域，循环体是一个子作用域，使用let声明相同的变量不会相互干扰
    ```javascript
    for (let i = 0; i < 3; i++) {
      let i = 'abc';
      console.log(i);
    }
    // abc
    // abc
    // abc
    ```
- 使用`let`，不存在变量声明提升
```javascript
// var 的情况
console.log(foo); // 输出undefined，因为存在var foo;被提升了
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```
- 暂时性死区
    - 个人理解：ES6在执行时，可能存在一个预先检查的过程，只要检查到某个代码块中使用`let`声明某变量后，那在`let`声明之前任何使用此变量的操作(包括带有安全防范机制的typeof)都将报错。
    ```javascript
    var tmp = 123;
    
    if (true) {
      tmp = 'abc'; // ReferenceError，检查到这个代码块使用了let声明了temp，所以在let声明之前使用tmp报错，即使外部有全局变量。
      let tmp;
    }
    ```
    - 使用`typeof`也会报错
    ```javascript
    typeof x;// 报错，显示x还未定义，如果没有let，因为typeof存在安全防范机制，所以返回的是'undefined'
    let x;
    ```
- 不允许重复声明
    - `let`不允许在相同作用域内，重复声明同一个变量
    ```javascript
    // 报错
    function func() {
      let a = 10;
      var a = 1;
    }
    
    // 报错
    function func() {
      let a = 10;
      let a = 1;
    }
    ```
    - 不能在函数内部重新声明参数
    ```javascript
    function func(arg) {
      let arg; // 报错
    }
    
    function func(arg) {
      {
        let arg; // 不报错
      }
    }
    ```
- ES6块级作用域
    - `let`实际上为 JavaScript 新增了块级作用域
    - 块级作用域的任意嵌套
    ```javascript
    {{{{{let insane = 'Hello World'}}}}};
    ```
    - 外层作用域无法读取内层作用域的变量，内层作用域可以定义外层作用域的同名变量
    ```
    {{{{
      {let insane = 'Hello World'}
      console.log(insane); // 报错
    }}}};
    
    {{{{
      let insane = 'Hello World';
      {let insane = 'Hello World'}
    }}}};
    ```
    - 最好不要在块级作用域中使用`function`声明函数(可以使用函数表达式来创建函数)

## const
- `const`基本和`let`类似，只在声明所在的块级作用域内有效、存在暂时性死区、同一作用域不可重复声明；重要的是一旦声明，常量的值就不能改变。
```javascript
const PI = 3.1415;
PI // 3.1415

PI = 3;
```
- `const`声明一个变量时，必须立即给其赋值，不能先声明，后期再赋值
```javascript
const foo;// SyntaxError: Missing initializer in const declaration
foo=3;
```

## 解构赋值
- ES6允许按照一定的模式，从数组和对象中提取值，对变量进行赋值，这过程称之为解构过程Destructuring

### 数组的解构赋值
```javascript
let [a, b, c] = [1, 2, 3];
```
- 这种写法属于“模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值。
```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```
- 解构不成功时，变量的值就等于undefined
```javascript
let [bar, foo] = [1];// bar为1，foo为undefined
```
- 不完全解构
    - 等号左边的模式，只匹配一部分的等号右边的数组；解构依然会成功
    ```javascript
    let [x, y] = [1, 2, 3];
    x // 1
    y // 2
    
    let [a, [b], d] = [1, [2, 3], 4];
    a // 1
    b // 2
    d // 4
    ```
- 如果等号的右边不是数组（或者严格地说，不是可遍历的结构，参见《Iterator》一章），那么将会报错
```javascript
// 报错
let [foo] = 1;// 转为对象以后不具备 Iterator 接口
let [foo] = false;// 转为对象以后不具备 Iterator 接口
let [foo] = NaN;// 转为对象以后不具备 Iterator 接口
let [foo] = undefined;// 转为对象以后不具备 Iterator 接口
let [foo] = null;// 转为对象以后不具备 Iterator 接口
let [foo] = {};// 本身就不具备 Iterator 接口
```
- 解构时可以有默认值
```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```
    - 默认值只有在当一个成员严格相等于(`===`)`undefined`时(本身值为`undefined`，或者没有值)，才会生效，这就是上面，最后的y为`'b'`的原因
    ```javascript
    let [a = 1] = [];// 无值，默认值生效
    a // 1

    let [x = 1] = [undefined];// 值为undefined，默认值生效
    x // 1
    
    let [x = 1] = [null];// 因为null!==undefined，默认值并未生效
    x // null
    ```

### 对象的解构赋值
```javascript
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
```
- 数组是有顺序的，所以解构是按照顺序来匹配，而对象没有固定的顺序，所以结构时变量必须和属性同名，才能取到正确的值
```javascript
let { bar, foo } = { foo: "aaa", bar: "bbb" };// 匹配和顺序无关
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };// 找不到同名的属性，所以变量取值不成功
baz // undefined
```
- 如果想把取到的值赋给另外一个变量则必须用下面的写法
```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };// foo取到的值'aaa'被赋给了baz变量
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```
- 对象解构赋值的本质
    - {模式名:变量名,模式名:变量名...}={属性名:值,属性名:值...}
    - 通过左侧模式名去找对应的属性名，取到对应属性名的值后，将值赋值给模式名后面的变量名；
    - 当左侧模式后面的变量名没有时，会取模式的名字；
    ```javascript
    let { foo, bar } = { foo: "aaa", bar: "bbb" };// 和下面的是等价的
    let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
    ```
    - 对应关系:模式名->属性名；变量名->值  
    ```javascript
    let { foo: baz } = { foo: "aaa", bar: "bbb" };
    baz // "aaa"
    foo // error: foo is not defined，foo不是变量而是一个模式
    ```
- 对象解构和数组解构一样可以嵌套解构 
    - 这时的p是模式并非变量
    ```javascript
    let obj = {
      p: [
        'Hello',
        { y: 'World' }
      ]
    };
    
    let { p: [x, { y }] } = obj;
    x // "Hello"
    y // "World"
    ```
    - 取得p的值
    ```javascript
    let obj = {
      p: [
        'Hello',
        { y: 'World' }
      ]
    };
    
    let { p, p: [x, { y }] } = obj;// 相当于let { p:p, p: [x, { y }] } = obj
    x // "Hello"
    y // "World"
    p // ["Hello", {y: "World"}]
    ```
    - 多层嵌套
    ```javascript
    const node = {
      loc: {
        start: {
          line: 1,
          column: 5
        }
      }
    };
    
    let { loc, loc: { start }, loc: { start: { line }} } = node;// 相当于let { loc:loc, loc: { start:start }, loc: { start: { line:line }} } = node
    line // 1
    loc  // Object {start: Object}
    start // Object {line: 1, column: 5}
    ```
- 默认值
    ```javascript
    var {x = 3} = {};
    x // 3
    
    var {x, y = 5} = {x: 1};
    x // 1
    y // 5
    
    var {x: y = 3} = {};
    y // 3
    
    var {x: y = 3} = {x: 5};// 模式是x，变量是y，并且y有个默认值3
    y // 5
    
    var { message: msg = 'Something went wrong' } = {};
    msg // "Something went wrong"
    ```
    - 默认值生效的条件是，对象的属性值严格等于`undefined`
    ```javascript
    var {x = 3} = {x: undefined};
    x // 3
    
    var {x = 3} = {x: null};// null!==undefined，所以默认值不生效
    x // null
    ```
- 解构失败，则变量的值为`undefined`
```javascript
let {foo} = {bar: 'baz'};
foo // undefined
```
- 如果解构模式是嵌套的对象，而且子对象所在的父属性不存在，那么将会报错
```javascript
// 报错
let {foo: {bar}} = {baz: 'baz'};// 因为foo根本就没找到此时foo为undefined，在undefined下找bar肯定报错
```
- 针对已经声明的变量使用解构赋值时，需要在最外围加上括号
```javascript
// 错误的写法
let x;
{x} = {x: 1};// SyntaxError: syntax error 语法块不能被赋值

// 正确的写法
let x;
({x} = {x: 1});
```
- 可以将原生对象的属性方法解构到变量上
```javascript
let { log, sin, cos } = Math;
```
- 运用解构获取数组的首个和末尾元素
```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

### 非对象的解构赋值
- 解构赋值的规则是，只要等号右边的值不是对象或数组，就先将其转为对象。由于undefined和null无法转为对象，所以对它们进行解构赋值，都会报错。
```javascript
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```
- 字符串的解构赋值
    - 字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象(包装对象)。
    ```javascript
    const [a, b, c, d, e] = 'hello';
    a // "h"
    b // "e"
    c // "l"
    d // "l"
    e // "o"
    let {length : len} = 'hello';
    len // 5
    ```
- 数值和布尔值的解构赋值
    - 解构赋值时，如果等号右边是数值和布尔值，则会先转为对象。
```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```
- 函数参数的解构赋值
    - 例子
    ```javascript
    function add([x, y]){
      return x + y;
    }
    add([1, 2]); // 3
    
    [[1, 2], [3, 4]].map(([a, b]) => a + b);
    // [ 3, 7 ]
    ```
    - 参数解构使用默认值
    ```javascript
    function move({x = 0, y = 0} = {}) {
      return [x, y];
    }
    
    move({x: 3, y: 8}); // [3, 8]；{x = 0, y = 0}={x: 3, y: 8}
    move({x: 3}); // [3, 0]；{x = 0, y = 0}={x: 3}
    move({}); // [0, 0]；{x = 0, y = 0}={}
    move(); // [0, 0]；{x = 0, y = 0}={}
    
    // 另一种写法
    function move({x, y} = { x: 0, y: 0 }) {
      return [x, y];
    }
    
    move({x: 3, y: 8}); // [3, 8]；{x, y}={x: 3, y: 8}
    move({x: 3}); // [3, undefined]；{x, y}={x: 3}
    move({}); // [undefined, undefined]；{x, y}={}
    move(); // [0, 0]；{x, y}={ x: 0, y: 0 }
    ```
- `undefined`会触发函数参数的默认值
```javascript
[1, undefined, 3].map((x = 'yes') => x);// 索引1为undefined，所以默认值生效
// [ 1, 'yes', 3 ]
```

### 圆括号问题
- 对于编译器来说，一个式子到底是模式，还是表达式，没有办法从一开始就知道，必须解析到（或解析不到）等号才能知道.ES6 的规则是，只要有可能导致解构的歧义，就不得使用圆括号。**建议只要有可能，就不要在模式中放置圆括号。**
- 不能用圆括号的情况
    - 变量声明语句
    ```javascript
    // 全部报错
    let [(a)] = [1];
    
    let {x: (c)} = {};
    let ({x: c}) = {};
    let {(x: c)} = {};
    let {(x): c} = {};
    
    let { o: ({ p: p }) } = { o: { p: 2 } };
    ```
    - 函数参数
        - 函数参数也属于变量声明，因此不能带有圆括号
        ```javascript
        // 报错
        function f([(z)]) { return z; }
        // 报错
        function f([z,(x)]) { return x; }
        ```
    - 赋值语句的模式中
    ```javascript
    // 全部报错
    ({ p: a }) = { p: 42 };
    ([a]) = [5];
    
    // 报错
    [({ p: a }), { x: c }] = [{}, {}];
    ```
- 可以使用圆括号的情况
    - 赋值语句的非模式部分，可以使用圆括号。 
    ```javscript
    [(b)] = [3]; // 正确
    ({ p: (d) } = {}); // 正确
    [(parseInt.prop)] = [3]; // 正确
    ```

### 解构赋值的用途
- 交换变量的值
```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```
- 从函数返回多个值
```javascript
// 返回一个数组，然后通过解构赋值赋值给变量
function example() {
  return [1, 2, 3];
}

let [a, b, c] = example();

// 返回一个对象，然后通过解构赋值赋值给变量
function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```
- 函数参数的定义
```javascript
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```
- **提取JSON数据**
```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```
- 函数参数的默认值
```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```
- 遍历Map结构
```javascript
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world

// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```
- 输入模块的指定方法
```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```

## 字符串的扩展

### 使用大括号表示超过ffff的字符
```javascript
"𠮷"的unicode编码为\u20BB7，超过ffff，在es5中必须拆分成两个来写
"\uD842\uDFB7"
// "𠮷"

es6中可以使用{}包裹来完成超过ffff字符的显示
"\u{20BB7}"
// "𠮷"
```

### codePointAt
- codePointAt方法会正确返回32位的UTF-16字符的十进制表示。对于那些两个字节储存的常规字符，它的返回结果与charCodeAt方法相同
```javascript
let s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
```
- codePointAt方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。
```javascript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

### String.fromCodePoin
- String.fromCharCode的升级版本能将超过ffff的编码转换成字符
```javascript
String.fromCharCode(0x20BB7)
// "ஷ"
String.fromCodePoint(0x20BB7)
// "𠮷"
```

### at
- charAt的升级版，返回对应位置字符，可识别超过ffff的字符
```javascript
'abc'.charAt(0) // "a"
'𠮷'.charAt(0) // "\uD842",𠮷是32bit的，它返回的是高16位的编码

'abc'.at(0) // "a"
'𠮷'.at(0) // "𠮷"
```

### 字符串的遍历器接口
- es6为字符串提供遍历接口，可以使用for...of遍历，并能遍历超过ffff编码的字符
```javascript
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"

let text = String.fromCodePoint(0x20BB7);
for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```

### includes(), startsWith(), endsWith()
- includes返回布尔值，表示是否找到了参数字符串
- startsWith返回布尔值，表示参数字符串是否在原字符串的头部。
- endsWith返回布尔值，表示参数字符串是否在原字符串的尾部。
```javascript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```

### repeat
- 重复某个字符串，并返回新的字符串
```javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```

### padStart(),padEnd()
- 补齐字符串长度
```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'
'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'

// 如果原字符串的长度，等于或大于指定的最小长度，则返回原字符串。
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'

// 如果用来补全的字符串与原字符串，两者的长度之和超过了指定的最小长度，则会截去超出位数的补全字符串。
'abc'.padStart(10, '0123456789')// '0123456abc'

// 如果省略第二个参数，默认使用空格补全长度。
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```
- padStart的常见用途是为数值补全指定位数
```javascript
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```

### 模板字符串
- 模板字符串（template string）是增强版的字符串，用反引号（`）标识。它可以当作普通字符串使用，也可以用来定义多行字符串，或者在字符串中嵌入变量
```javascript
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```
- 模板字符串中嵌入变量，需要将变量名写在${}之中。
```javascript
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 传统写法为
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}

// 大括号内部可以放入任意的JavaScript表达式，可以进行运算，以及引用对象属性
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"

// 模板字符串之中还能调用函数
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```
- 模板字符串甚至还能嵌套
```javascript
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;

const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```

### 标签模板
- 函数名后紧跟一个模板字符串
- 标签模板其实不是模板，而是函数调用的一种特殊形式。“标签”指的就是函数，紧跟在后面的模板字符串就是它的参数。
```javascript
alert`123`
// 等同于
alert(123)
```
- 如果模板字符里面有变量，就不是简单的调用了，而是会将模板字符串先处理成多个参数，再调用函数
    - 会将除模板字符串中变量以外的字符串分隔成一个个字符串保存到数组中并传入函数中，并将模板字符串中变量按顺序依次传入函数中 
    ```javascript
    let a = 5;
    let b = 10;
    
    tag`Hello ${ a + b } world ${ a * b }`;
    // 等同于
    tag(['Hello ', ' world ', ''], 15, 50);
    
    function tag(stringArr, value1, value2){
        // stringArr为['Hello ', ' world ', ''],value1为15,value2为50
        // ...
    }
    
    // 等同于
    function tag(stringArr, ...values){
        // stringArr为['Hello ', ' world ', ''],values[15,20]
        // ...
    }
    ```
- 用途
    - 过滤用户恶意输入、i18n国际化
    ```javascript
    // 一般${sender}为用户的输入
    let message =
      SaferHTML`<p>${sender} has sent you a message.</p>`;
    
    function SaferHTML(templateData) {
      let s = templateData[0];// 原有字符串数组
      for (let i = 1; i < arguments.length; i++) {// i从1开始
        let arg = String(arguments[i]);// 实际取到的为用户输入
    
        // 转义用户输入中的特殊符号
        s += arg.replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;");
    
        // 拼接上原有字符串的后面部分
        s += templateData[i];
      }
      return s;
    }
    
    let sender = '<script>alert("abc")</script>'; // 恶意代码
    let message = SaferHTML`<p>${sender} has sent you a message.</p>`;
    
    console.log(message);
    // <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
    ```

### String.raw
- 返回用`\`转移的字符串串
```javascript
String.raw`Hi\n${2+3}!`;
// "Hi\\n5!"

String.raw`Hi\u000A!`;
// 'Hi\\u000A!'
```

## 正则的扩展
- 老正则还不会用...先放着，后期再补吧(希望我能想起来吧...)

## 数值的扩展
- 扩展了一大批方法...(先过一遍，留个印象，用的时候再查吧)

## 函数的扩展

### 函数参数的默认值
- 利用`=`直接写在形参后面
```javascript
function log(x, y = 'World') {// y如果===undefined，则y的默认值生效
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```
- 参数变量是默认声明的，所以不能用let或const再次声明。
```javascript
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
```
- 参数默认值不是传值的，而是每次都重新计算默认值表达式的值
```javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```
- 与解构赋值默认值结合使用
```javascript
function foo({x, y = 5}) {// 解构赋值默认值
  console.log(x, y);
}

foo({}) // undefined 5；{x, y = 5}={}
foo({x: 1}) // 1 5；{x, y = 5}={x:1}
foo({x: 1, y: 2}) // 1 2；{x, y = 5}:{x:1,y:2}
foo() // TypeError: Cannot read property 'x' of undefined；{x, y = 5}=undefined所以报错

function foo({x, y = 5} = {}) {// 解构赋值默认值+形参默认值，就不会出现报错
  console.log(x, y);
}

foo() // undefined 5
```
    - 不同写法
    ```javascript
    // 写法1
    function m1(x,y) {// 形参(无默认值)
      return [x, y];
    }
    m1(1,3);//[1,3]
    m1();//[undefined,undefined]
    
    // 写法2
    function m1(x,y=5) {// 形参(有默认值)
      return [x, y];
    }
    m1(1,3);//[1,3]
    m1(3);//[3,5]
    m1();//[undefined,5]
    
    // 写法3
    function m1({x , y}) {// 解构赋值(无默认值)
      return [x, y];
    }
    m1(1,3);//[undefined,undefined];{x,y}=1,3;左右模式不一致，所以解构赋值不成功，得到undefined
    m1({x:1,y:3});//[1,3]；{x,y}={x:1,y:3}
    m1({x:1});//[1,undefined]；{x,y}={x:1}
    m1({});[undefined,undefined]；{x , y}={}；未找到对应属性名，所以得到undefined
    m1();// err；{x,y}=undefined->所以报错
    
    // 写法4
    function m1({x = 0, y = 5}) {// 解构赋值(有默认值)
      return [x, y];
    }
    m1(1,3);//[0,5];{x = 0, y = 5}=1,3；左右模式不匹配，得到undefined，因而解构默认值生效
    m1({x:1,y:3});//[1,3]；{x = 0, y = 5}={x:1,y:3}
    m1({x:1});//[1,undefined]；{x,y}={x:1}
    m1({});//[0,5]；{x = 0, y = 5}={}；未找到对应属性名，所以得到undefined，因而解构默认值生效
    m1();// err；{x = 0, y = 5}=undefined->所以报错
    
    // 写法5
    function m1({x , y}={}) {// 解构赋值(无默认值)+形参(有默认值，空对象)
      return [x, y];
    }
    m1(1,3);//[undefined,undefined];{x,y}=1,3;左右模式不一致，所以解构赋值不成功，得到undefined
    m1({x:1,y:3});//[1,3]；{x , y}={x:1,y:3}
    m1({x:1});//[1,undefined]；{x , y}={x:1}
    m1({});//[undefined,undefined]；{x , y}={}；未找到对应属性名，所以得到undefined
    m1();//[undefined,undefined]；{x , y}={}
    
    
    // 写法6
    function m1({x , y}={x:3,y:4}) {// 解构赋值(无默认值)+形参(有默认值，非空对象)
      return [x, y];
    }
    m1(1,3);//[undefined,undefined];{x,y}=1,3;左右模式不一致，所以解构赋值不成功，得到undefined
    m1({x:1,y:3});//[1,3]；{x , y}={x:1,y:3}
    m1({x:1});//[1,undefined]；{x , y}={x:1}
    m1({});//[undefined,undefined]；{x , y}={}；未找到对应属性名，所以得到undefined
    m1();//[3,4]；{x , y}={x:3,y:4}；未传参数，所以形参的默认值{x:3,y:4}生效，参与解构
    
    // 写法7
    function m1({x = 0, y = 5}={}) {// 解构赋值(有默认值)+形参(有默认值，空对象)
      return [x, y];
    }
    m1(1,3);//[0,5];传入参数跟想要的类型不一致，所以行参默认值生效{x = 0, y = 5}={};未找到对应属性名，所以解构默认值生效
    m1({x:1,y:3});//[1,3]；{x = 0, y = 5}={x:1,y:3}// 默认值{}不生效
    m1({x:1});//[1,5]；{x = 0, y = 5}={x:1} 
    m1({});//[0,5]；{x = 0, y = 5}={}；未找到对应属性名，所以得到undefined，解构的默认值生效
    m1();//[0,5]；{x = 0, y = 5}={}；未传参数，所以形参的默认值{}生效，参与解构，未解构到对应属性名，所以解构的默认值生效
    
    // 写法8
    function m1({x = 0, y = 5}={x:4,y:8}) {// 解构赋值(有默认值)+形参(有默认值，非空对象)
      return [x, y];
    }
    m1(1,3);//[0,5];传入参数跟想要的类型不一致，所以行参默认值生效{x = 0, y = 5}={};未找到对应属性名，所以解构默认值生效
    m1({x:1,y:3});//[1,3]；{x = 0, y = 5}={x:1,y:3}
    m1({x:1});//[1,5]；{x = 0, y = 5}={x:1} 
    m1({});//[0,5]；{x = 0, y = 5}={}；未找到对应属性名，所以得到undefined，解构的默认值生效
    m1();//[4,8]；{x = 0, y = 5}={x:4,y:8}；未传参数，所以形参的默认值{x:4,y:8}生效，参与解构
    ```
- 参数默认值的位置
    - 要设置默认值的参数，应该放在参数列表的尾部，因为这样方便看出调用时省略了哪些参数
    ```javascript
    // 例一
    function f(x = 1, y) {
      return [x, y];
    }
    
    f() // [1, undefined]
    f(2) // [2, undefined])
    f(, 1) // 报错
    f(undefined, 1) // [1, 1]
    
    // 例二
    function f(x, y = 5, z) {
      return [x, y, z];
    }
    
    f() // [undefined, 5, undefined]
    f(1) // [1, 5, undefined]
    f(1, ,2) // 报错
    f(1, undefined, 2) // [1, 5, 2]
    ```
- 函数的length
    - 返回没有指定默认值的参数个数，指定了默认值后，length属性将失真
    ```javascript
    (function (a) {}).length // 1
    (function (a = 5) {}).length // 0
    (function (a, b, c = 5) {}).length // 2
    ```
    - 如果设置了默认值的参数不是尾参数，那么length属性也不再计入后面的参数了。
    ```javascript
    (function (a = 0, b, c) {}).length // 0
    (function (a, b = 1, c) {}).length // 1
    ```
- 作用域
    - 一旦设置了参数的默认值，函数进行声明初始化时，参数会形成一个单独的作用域（context）。等到初始化结束，这个作用域就会消失。这种语法行为，在不设置参数默认值时，是不会出现的；
    - 设置默认值时，会在形参声明处生成一个单独的作用域。它只可以受全局作用影响，不受函数体内的作用域影响
    ```javascript
    var x = 1;
    function f(x, y = x) {// 等价于let x;let y=x;因为形参处已经声明了x，所以y=x后，xy都为同一个值->实际传进来的值，而不是全局的x
      console.log(x,y);
    }
    f(2) // 2
    
    function f(y = x) {// let y=x;外部没有定义x所以报错(形参不受函数体内影响)
      let x = 2;
      console.log(y);
    }
    f() // ReferenceError: x is not defined
    
    var x = 1;
    function foo(x = x) {// let x=x;由于tdz，x在声明结束前无法使用x
      // ...
    }
    foo() // ReferenceError: x is not defined
    
    let foo = 'outer';
    function bar(func = () => foo) {
      let foo = 'inner';
      console.log(func());
    }
    bar(); // outer
    
    var x = 1;
    function foo(x, y = function() { x = 2; }) {// 因为形参出已经有了x,所以y的匿名函数中的x指向的也就是形参处的x，它就不会再受外界影响了
      var x = 3;// 在函数体内重新声明了一个x和外界的x不是同一个
      y();//调用后，改变的是形参的x
      console.log(x);// 根据就近原则，找到的是函数体中的x，所以打印不是行参的x，如果去掉var x=3;则打印的是形参的x
    }
    foo() // 3
    x // 1
    ```
- 参数默认值的应用
    - 利用参数默认值，可以指定某一个参数不得省略，如果省略就抛出一个错误
    ```javascript
    function throwIfMissing() {
      throw new Error('Missing parameter');
    }
    
    function foo(mustBeProvided = throwIfMissing()) {
      return mustBeProvided;
    }
    
    foo()
    // Error: Missing parameter
    ```

### rest参数
- 用于获取函数的剩余参数，将其保存到一个数组中，这样就不需要使用arguments对象了
```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```
- 和`arguments`的区别
    - `arguments`是一个类数组，并不是真正的数组
    - rest得到的是一个真正的数组，可以直接调用数组的方法
    ```javascript
    // arguments变量的写法
    function sortNumbers() {
      return Array.prototype.slice.call(arguments).sort();
    }
    
    // rest参数的写法
    const sortNumbers = (...numbers) => numbers.sort();
    ```
- rest参数必须放在形参列表的尾部
```javascript
// 报错
function f(a, ...b, c) {
  // ...
}
```
- 函数的length属性，不包括 rest 参数。
```javascript
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

### 严格模式
- 只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错。
```javascript
// 报错
function doSomething(a, b = a) {
  'use strict';
  // code
}

// 报错
const doSomething = function ({a, b}) {
  'use strict';
  // code
};

// 报错
const doSomething = (...a) => {
  'use strict';
  // code
};

const obj = {
  // 报错
  doSomething({a, b}) {
    'use strict';
    // code
  }
};
```
- 解决方法
    - 设置全局的严格模式，不推荐
    ```javascript
    'use strict';
    
    function doSomething(a, b = a) {
      // code
    }
    ```
    - 函数包在一个无参数的立即执行函数里面
    ```javascript
    const doSomething = (function () {
      'use strict';
      return function(value = 42) {
        return value;
      };
    }());
    ```

### name属性
- 返回函数名，es5就有，但es6做了一些修改
```javascript
var f = function () {};
// ES5
f.name // ""
// ES6
f.name // "f"

const bar = function baz() {};
// ES5
bar.name // "baz"
// ES6
bar.name // "baz"

(new Function).name // "anonymous"

function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

### 箭头函数
- 基础语法
```javascript
(param1, param2, paramN) => { 多条语句；return 表达式; }
(param1, param2, paramN) => 表达式
// 等价于：(param1, param2, paramN) => { return 表达式; }
/* 当删除大括号时，它将是一个隐式的返回值，这意味着我们不需要指定我们返回*/

// 如果只有一个参数，圆括号是可选的:
(singleParam) => { statements;return 表达式; }
singleParam => { statements; return 表达式;}

// 如果箭头函数 无参数 , 必须使用 ()圆括号:
() => { statements; return 表达式;} 
```
- 高级语法
```javascript
//返回一个对象时，函数体外要加圆括号，否则会被当成语法块，进而语法错误
params => ({foo: bar})

// 支持 剩余参数和默认参数:
(param1, param2, ...rest) => { statements; return 表达式; }
(param1 = defaultValue1, param2, …, paramN = defaultValueN) => { statements; return 表达式; }
// 也支持参数列表中的解构赋值
let f = ([a, b] = [1, 2], {x: c} = {x: a + b}) => a + b + c; // a=1; b=2; x=c; c=a+b=3;
f();  // 6
```
- 注意事项
    - 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象
    - 准确的说应该是，箭头函数本身没有`this`，它的`this`是继承最近父作用域的(更准确的说是直接使用的最近父作用域的`this`)，即最近父作用域被调用时的`this`是什么，它的`this`就是什么；这过程类似变量的溯源查找过程。
    ```javascript
    // 正常情况
    function foo(){
        setTimeout(function(){
            console.log('id',this.id);
        },100);
    
        // 会输出21，setTimeout内部是这样的
        function setTimeout(fn,delay){
            fn();// fn不是做为方法调用，也不是new，更没有使用call、apply、bind做显示绑定，而是属于直接调用，所以内部this指向了window，进而最终输出21
        }
    }
    var id = 21;
    foo.call({ id: 42 });// 21
    
    // 箭头函数
    function foo() {
        setTimeout(() => {
            console.log('id:', this.id);
        }, 100);
    
        // 箭头函数没有this,它的this是用的最近父作用域foo被调用时的this，
        // foo调用时，this被绑定到了{ id: 42 }，所以箭头函数用的this也是{ id: 42 }，进而最终输出了42
    }
    
    var id = 21;
    foo.call({ id: 42 });// 42
    ```
    - 如何快速判断箭头函数的`this`，直接找它定义时的直接父函数或者直接父对象
    ```
    function foo() {  
        setTimeout(() => {// 这个箭头函数定义时，直接父函数为foo，所以它的this是foo被调用时的this
            console.log('id:', this.id);
        }, 100);
    }
    
    var id = 21;
    foo.call({ id: 42 });// 42
    ```
    - 箭头函数没有`arguments`、`super`、`new.target`，如果要使用`arguments`，可以用rest参数代替
    ```javascript
    function foo() {
      setTimeout(() => {
        console.log('args:', arguments);
      }, 100);
    }
    
    foo(2, 4, 6, 8)
    // args: [2, 4, 6, 8]
    ```
    - 箭头函数没有自己的`this`，所以更不能使用`call`、`apply`、`bind`，这些方法无法改变this的指向
    ```
    (function() {
      return [
        (() => this.x).bind({ x: 'inner' })()
      ];
    }).call({ x: 'outer' });
    // ['outer']
    ```
    - 箭头函数不能使用`new`，否则会报错

### 尾调用优化
- 尾调用
    - 指某个函数的最后一步是调用另一个函数
    ```javascript
    function f(x){
      return g(x);
    }
    
    // 情况一
    function f(x){
      let y = g(x);
      return y;
    }
    
    // 情况二
    function f(x){
      return g(x) + 1;
    }
    
    // 情况三
    function f(x){
      g(x);
    }
    ```
    - 尾调用不一定出现在函数尾部，只要是最后一步操作即可
    ```javascript
    function f(x) {
      if (x > 0) {
        return m(x)
      }
      return n(x);
    }
    ```
- 尾调用优化
    - 如果在函数A的内部调用函数B，那么在A的调用帧上方，还会形成一个B的调用帧。等到B运行结束，将结果返回到A，B的调用帧才会消失。如果函数B内部还调用函数C，那就还有一个C的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。
    - 尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。
    ```javascript
    function f() {
      let m = 1;
      let n = 2;
      return g(m + n);
    }
    f();
    
    // 等同于
    function f() {
      return g(3);
    }
    f();
    
    // 等同于
    g(3);
    ```
    - 上面代码中，如果函数g不是尾调用，函数f就需要保存内部变量m和n的值、g的调用位置等信息。但由于调用g之后，函数f就结束了，所以执行到最后一步，完全可以删除f(x)的调用帧，只保留g(3)的调用帧。
    - 只有不再用到外层函数的内部变量，内层函数的调用帧才会取代外层函数的调用帧，否则就无法进行“尾调用优化”。
- 尾递归
    - 函数调用自身，称为递归。如果尾调用自身，就称为尾递归
    - 递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误
    ```javascript
    // 普通递归
    function factorial(n) {
      if (n === 1) return 1;
      return n * factorial(n - 1);
    }
    
    factorial(5) // 120
    // 尾递归
    function factorial(n, total) {
      if (n === 1) return total;
      return factorial(n - 1, n * total);
    }
    
    factorial(5, 1) // 120
    ```
- 递归函数的改写
    - 尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是把所有用到的内部变量改写成函数的参数。
    - 比如上面的例子，阶乘函数 factorial 需要用到一个中间变量total，那就把这个中间变量改写成函数的参数。
    ```javascript
    function factorial(n, total = 1) {
      if (n === 1) return total;
      return factorial(n - 1, n * total);
    }
    
    factorial(5) // 120
    ```
- ES6 的尾调用优化只在严格模式下开启，正常模式是无效的
    - 非严格模式下，可以通过将递归改写成循环，进而模拟尾递归调用的优化效果(减少调用帧)

## 数组的扩展

### 扩展运算符
- 可以理解为rest参数的逆运算；rest参数是将逗号分隔的参数列表转换成数组，而扩展运算符号则是将数组转换成逗号分隔的列表
```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]


let arr=[];
const numbers = [4, 38];
function push(array, ...items) {// arr,[4,38]
  array.push(...items);// items为数组，无法做为push的参数，所以使用了扩展运算符，转换成了参数列表
}
push(arr,...numbers);// 相当于push(arr,4,38)
console.log(arr);//[4,38]
```
- 扩展运算符与正常的函数参数可以结合使用，非常灵活
```javascript
function f(v, w, x, y, z) { }
const args = [0, 1];
f(-1, ...args, 2, ...[3]);
```
- 用途
    - 替代数组的`apply`方法
    ```javascript
    // ES5 的写法
    function f(x, y, z) {
      // ...
    }
    var args = [0, 1, 2];
    f.apply(null, args);
    
    // ES6的写法
    function f(x, y, z) {
      // ...
    }
    let args = [0, 1, 2];
    f(...args);

    // 取最大值
    // ES5 的写法
    Math.max.apply(null, [14, 3, 77])
    
    // ES6 的写法
    Math.max(...[14, 3, 77])
    
    // 等同于
    Math.max(14, 3, 77);
    ```
    - 合并数组
    ```javascript
    // ES5
    [1, 2].concat(more)
    // ES6
    [1, 2, ...more]
    
    var arr1 = ['a', 'b'];
    var arr2 = ['c'];
    var arr3 = ['d', 'e'];
    
    // ES5的合并数组
    arr1.concat(arr2, arr3);
    // [ 'a', 'b', 'c', 'd', 'e' ]
    
    // ES6的合并数组
    [...arr1, ...arr2, ...arr3]
    // [ 'a', 'b', 'c', 'd', 'e' ]
    ```
    - 扩展运算符可以与解构赋值结合起来，用于生成数组
    ```javascript
    // ES5
    a = list[0], rest = list.slice(1)
    // ES6
    [a, ...rest] = list
    
    const [first, ...rest] = [1, 2, 3, 4, 5];
    first // 1
    rest  // [2, 3, 4, 5]
    
    const [first, ...rest] = [];
    first // undefined
    rest  // []
    
    const [first, ...rest] = ["foo"];
    first  // "foo"
    rest   // []
    ```
        - 如果将扩展运算符用于数组赋值，只能放在参数的最后一位，否则会报错。
        ```javascript
        const [...butLast, last] = [1, 2, 3, 4, 5];
        // 报错
        
        const [first, ...middle, last] = [1, 2, 3, 4, 5];
        // 报错
        ```
    - 函数返回值
    ```javascript
    let dateFields = readDateFields(database);
    let d = new Date(...dateFields);// Data不接收数组，所以转换成参数列表
    ```
    - 字符串转数组
    ```javascript
    [...'hello']
    // [ "h", "e", "l", "l", "o" ]
    ```
    - 实现了`Iterator`接口的对象，都能转换成数组
    ```javascript
    let nodeList = document.querySelectorAll('div');// nodelist有Iterator接口，所以可以转换成数组
    let array = [...nodeList];
    
    let map = new Map([// Map有Iterator接口
      [1, 'one'],
      [2, 'two'],
      [3, 'three'],
    ]);
    let arr = [...map.keys()]; // [1, 2, 3]
    
    const go = function*(){// Generator 函数运行后，返回一个遍历器对象，因此也可以使用扩展运算符。
      yield 1;
      yield 2;
      yield 3;
    };
    [...go()] // [1, 2, 3]
    ```

### Array.from
- 将类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）转换成数组
- 所谓类似数组的对象，本质特征只有一点，即必须有length属性。因此，任何有length属性的对象，都可以通过Array.from方法转为数组，而此时扩展运算符就无法转换
```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};
// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']
// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']

// NodeList对象
let ps = document.querySelectorAll('p');
Array.from(ps).forEach(function (p) {
  console.log(p);
});

// arguments对象
function foo() {
  var args = Array.from(arguments);
  // ...
}

Array.from('hello')// es6中字符串有Iterator接口
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']

Array.from({ length: 2 }, () => 'jack')// 对象有length，所以是类数组对象，可以用from转换
// ['jack', 'jack']
```

### Array.of()
- Array.of方法用于将一组值，转换为数组。弥补数组构造函数Array()的不足，因为参数个数的不同，会导致Array()的行为有差异。
- Array.of总是返回参数值组成的数组。如果没有参数，就返回一个空数组。
```javascript
Array() // []
Array(3) // [, , ,]，只有一个参数时，是指定的数组长度
Array(3, 11, 8) // [3, 11, 8]

Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
```

### 数组实例的 copyWithin()
- 将指定位置的成员复制到其他位置（会覆盖原有成员），然后返回当前数组，所以会修改当前数组
```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
target（必需）：从该位置开始替换数据。
start（可选）：从该位置开始读取数据，默认为0。如果为负值，表示倒数。
end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数。

[1, 2, 3, 4, 5].copyWithin(0, 3)// 读取从索引3开始到结束的数据4,5并替换从0开始的数据
// [4, 5, 3, 4, 5]

// 将3号位复制到0号位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相当于3号位，-1相当于4号位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 将3号位复制到0号位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 将2号位到数组结束，复制到0号位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// 对于没有部署 TypedArray 的 copyWithin 方法的平台
// 需要采用下面的写法
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

### 数组实例的 find() 和 findIndex()
- 数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined
```javascript
[1, 4, -5, 10].find((n) => n < 0)
// -5

[1, 5, 10, 15].find(function(value, index, arr) {//当前值，索引，原数组
  return value > 9;
}) // 10
```
- 返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1
```javascript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

### 数组实例的fill()
- fill方法使用给定值，填充一个数组。
```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]

['a', 'b', 'c'].fill(7, 1, 2)// 将7填充到[1,2)区间
// ['a', 7, 'c']
```

### 数组实例的 entries()，keys() 和 values()
- entries()，keys()和values()——用于遍历数组。它们都返回一个遍历器对象，可以用for...of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。
```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

### 数组实例的 includes() 
- Array.prototype.includes方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似
```javascript
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

### 数组的空位
- 数组的空位指，数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位。
```javascript
Array(3) // [, , ,]
```
- 空位不是undefined，一个位置的值等于undefined，依然是有值的。空位是没有任何值，in运算符可以说明这一点
```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```
- ES5 对空位的处理，已经很不一致了，大多数情况下会忽略空位；ES6 则是明确将空位转为undefined。
```javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]

[...['a',,'b']]
// [ "a", undefined, "b" ]

[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]

new Array(3).fill('a') // ["a","a","a"]

let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1

// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

## 对象的扩展

### 属性的简洁表示法
- ES6允许直接写入变量和函数，作为对象的属性和方法
```javascript
const foo = 'bar';
const baz = {foo};
baz // {foo: "bar"}
// 等同于
const baz = {foo: foo};

// 属性简写
function f(x, y) {
  return {x, y};
}
// 等同于
function f(x, y) {
  return {x: x, y: y};
}
f(1, 2) // Object {x: 1, y: 2}

// 方法简写
const o = {
  method() {
    return "Hello!";
  }
};
// 等同于
const o = {
  method: function() {
    return "Hello!";
  }
};

// 实际例子
let birth = '2000/01/01';
const Person = {
  name: '张三',
  //等同于birth: birth
  birth,
  // 等同于hello: function ()...
  hello() { console.log('我的名字是', this.name); }
};

// 用在函数返回值
function getPoint() {
  const x = 1;
  const y = 10;
  return {x, y};
}
getPoint()// {x:1, y:10}

// 属性的赋值器（setter）和取值器（getter），事实上也是采用这种写法
const cart = {
  _wheels: 4,
  get wheels () {
    return this._wheels;
  },
  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('数值太小了！');
    }
    this._wheels = value;
  }
}

// 简洁写法的属性名总是字符串，这会导致一些看上去比较奇怪的结果
const obj = {
  class () {}
};
// 等同于
var obj = {
  'class': function() {}
};

// 如果某个方法的值是一个 Generator 函数，前面需要加上星号
const obj = {
  * m() {
    yield 'hello world';
  }
};
```

### 属性名表达式
- ES6 允许字面量定义对象时，用表达式作为对象的属性名，即把表达式放在方括号内。
```javascript
let lastWord = 'last word';
const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};
a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"

// 表达式还可以用于定义方法名
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};
obj.hello() // hi

// 属性名表达式与简洁表示法，不能同时使用，会报错
// 报错
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] };
// 正确
const foo = 'bar';
const baz = { [foo]: 'abc'};

// 属性名表达式如果是一个对象，默认情况下会自动将对象转为字符串[object Object]
const keyA = {a: 1};
const keyB = {b: 2};
const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};
myObject // Object {[object Object]: "valueB"}
```

### 方法的 name 属性
- 函数的name属性，返回函数名。对象方法也是函数，因此也有name属性
```javascript
const person = {
  sayName() {
    console.log('hello!');
  },
};
person.sayName.name   // "sayName"

(new Function()).name // "anonymous"
var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"
```

### Object.is()
- ES5 比较两个值是否相等，只有两个运算符：相等运算符（==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。
- `Object.is`它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。和`===`不同点在于一是+0不等于-0，二是NaN等于自身
```javascript
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

### Object.assign()
- 类似`jQuery`的`$.extend()`方法，可用来合并对象
```javascript
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3 };
Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}

// 重名属性，后面对象会覆盖前面的
const target = { a: 1, b: 1 };

const source1 = { b: 2, c: 2 };
const source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```
- Object.assign拷贝的属性是有限制的，只拷贝源对象的自身属性（不拷贝继承属性），也不拷贝不可枚举的属性（enumerable: false）
```javascript
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }
```
- Object.assign方法实行的是浅拷贝，而不是深拷贝。也就是说，如果源对象某个属性的值是对象，那么目标对象拷贝得到的是这个对象的引用
```javascript
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);
obj1.a.b = 2;
obj2.a.b // 2

// 同名属性，直接替换引用
const target = { a: { b: 'c', d: 'e' } }
const source = { a: { b: 'hello' } }
Object.assign(target, source);
// { a: { b: 'hello' } }
```
- Object.assign可以用来处理数组，但是会把数组视为对象
```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```
- 用途
    - 为对象添加属性
    ```javascript
    class Point {
      constructor(x, y) {
        Object.assign(this, {x, y});// 将x属性和y属性添加到Point类的对象实例
      }
    }
    ```
    - 为对象添加方法
    ```javascript
    Object.assign(SomeClass.prototype, {
      someMethod(arg1, arg2) {
        ···
      },
      anotherMethod() {
        ···
      }
    });
    
    // 等同于下面的写法
    SomeClass.prototype.someMethod = function (arg1, arg2) {
      ···
    };
    SomeClass.prototype.anotherMethod = function () {
      ···
    };
    ```
    - 克隆对象
    ```javascript
    // 克隆原始对象自身的值，不能克隆它继承的值
    function clone(origin) {
      return Object.assign({}, origin);
    }
    
    // 克隆同时保持继承链
    function clone(origin) {
      let originProto = Object.getPrototypeOf(origin);
      return Object.assign(Object.create(originProto), origin);
    }
    ```
    - 合并多个对象
    ```javascript
    // 将多个对象合并到某个对象
    const merge = (target, ...sources) => Object.assign(target, ...sources);
    
    // 合并后返回一个新对象
    const merge = (...sources) => Object.assign({}, ...sources);
    ```
    - 为属性指定默认值
    ```javascript
    const DEFAULTS = {
      logLevel: 0,
      outputFormat: 'html'
    };
    
    function processContent(options) {
      options = Object.assign({}, DEFAULTS, options);
      console.log(options);
      // ...
    }
    ```

### 属性的可枚举性和遍历
- Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象
```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```
- 四个操作会忽略enumerable为false的属性
    - for...in循环：只遍历对象自身的和**继承**的可枚举的属性。
    - Object.keys()：返回对象自身的所有可枚举的属性的键名。
    - JSON.stringify()：只串行化对象自身的可枚举的属性。
    - Object.assign()： 忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性。
    - 只有for...in会返回继承的属性，其他三个方法都会忽略继承的属性，只处理对象自身的属性
    - 总的来说，操作中引入继承的属性会让问题复杂化，大多数时候，我们只关心对象自身的属性。所以，尽量不要用for...in循环，而用Object.keys()代替。
- 属性遍历
    - for...in
        - for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）
    - Object.keys(obj)
        - Object.keys返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）
    - Object.getOwnPropertyNames(obj)
        - Object.getOwnPropertyNames返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）
    - Object.getOwnPropertySymbols(obj)
        - Object.getOwnPropertySymbols返回一个数组，包含对象自身的所有 Symbol 属性。
    - Reflect.ownKeys(obj)
        - Reflect.ownKeys返回一个数组，包含对象自身的所有属性，不管属性名是 Symbol 或字符串，也不管是否可枚举。
    - 遍历规则
        - 首先遍历所有属性名为数值的属性，按照数字排序。
        - 其次遍历所有属性名为字符串的属性，按照生成时间排序。
        - 最后遍历所有属性名为 Symbol 值的属性，按照生成时间排序。
        ```javascript
        Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
        // ['2', '10', 'b', 'a', Symbol()]
        ```

### Object.getOwnPropertyDescriptors
- 返回指定对象所有自身属性（非继承属性）的描述对象。
```javascript
const obj = {
  foo: 123,
  get bar() { return 'abc' }
};

Object.getOwnPropertyDescriptors(obj)
// { foo:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
```
- 主要是为了解决Object.assign()无法正确拷贝get属性和set属性的问题。
```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};
const target1 = {};
Object.assign(target1, source);
Object.getOwnPropertyDescriptor(target1, 'foo')
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }

// 正确拷贝set方法
const source = {
  set foo(value) {
    console.log(value);
  }
};
const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }

// 上面代码中，两个对象合并的逻辑可以写成一个函数。
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```
- 配合Object.create方法，将对象属性克隆到一个新对象。这属于浅拷贝
```javascript
const clone = Object.create(Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj));
// 或者
const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

### __proto__属性，Object.setPrototypeOf()，Object.getPrototypeOf()
- __proto__属性（前后各两个下划线），用来读取或设置当前对象的prototype对象，浏览器内部方法
```javascript
// es6的写法
const obj = {
  method: function() { ... }
};
obj.__proto__ = someOtherObj;

// es5的写法
var obj = Object.create(someOtherObj);
obj.method = function() { ... };
```
- Object.setPrototypeOf()
    - Object.setPrototypeOf方法的作用与__proto__相同，用来设置一个对象的prototype对象，返回参数对象本身。它是 ES6 正式推荐的设置原型对象的方法
    ```javascript
    // 格式
    Object.setPrototypeOf(object, prototype)
    // 用法
    const o = Object.setPrototypeOf({}, null);
    
    // 等同于
    function (obj, proto) {
      obj.__proto__ = proto;
      return obj;
    }
    ```
- Object.getPrototypeOf()
    - 该方法与Object.setPrototypeOf方法配套，用于读取一个对象的原型对象
    ```javascript
    function Rectangle() {
      // ...
    }
    
    const rec = new Rectangle();
    
    Object.getPrototypeOf(rec) === Rectangle.prototype
    // true
    
    Object.setPrototypeOf(rec, Object.prototype);
    Object.getPrototypeOf(rec) === Rectangle.prototype
    // false
    ```

### Object.keys()，Object.values()，Object.entries()
- Object.keys方法，返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键名
- Object.values方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值
- Object.entries方法返回一个数组，成员是参数对象自身的（不含继承的）所有可遍历（enumerable）属性的键值对数组
```javascript
var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]

let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```
- Object.entries的基本用途是遍历对象的属性
```javascript
let obj = { one: 1, two: 2 };
for (let [k, v] of Object.entries(obj)) {
  console.log(
    `${JSON.stringify(k)}: ${JSON.stringify(v)}`
  );
}
// "one": 1
// "two": 2
```

### 对象的扩展运算符
- 解构赋值
```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }

// 解构赋值不会拷贝继承自原型对象的属性。
let o1 = { a: 1 };
let o2 = { b: 2 };
o2.__proto__ = o1;
let { ...o3 } = o2;
o3 // { b: 2 }
o3.a // undefined
```
- 可用于取出参数对象的所有可遍历属性，拷贝到当前对象之中，这等同于使用Object.assign方法。它只拷贝了对象实例的属性
```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }

let aClone = { ...a };
// 等同于
let aClone = Object.assign({}, a);

// 想完整克隆一个对象，还拷贝对象原型的属性
// 写法一
const clone1 = {
  __proto__: Object.getPrototypeOf(obj),
  ...obj
};

// 写法二
const clone2 = Object.assign(
  Object.create(Object.getPrototypeOf(obj)),
  obj
);
```
- 用于合并两个对象
```javascript
let ab = { ...a, ...b };
// 等同于
let ab = Object.assign({}, a, b);
```

### Null 传导运算符
- 如果读取对象内部的某个属性，往往需要判断一下该对象是否存在
- 
```javascript
// 以前写法
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';

// 现在写法
// 如果 a 是 null 或 undefined, 返回 undefined
// 否则返回 a.b.c().d
a?.b.c().d
// 如果 a 是 null 或 undefined，下面的语句不产生任何效果
// 否则执行 a.b = 42
a?.b = 42
// 如果 a 是 null 或 undefined，下面的语句不产生任何效果
delete a?.b
```

## Symbol

### 是啥？
- Symbol 是一种特殊的、不可变的数据类型，**可以作为对象属性的标识符使用，用它创建的属性名是绝对唯一的，不会产生冲突**；Symbol 数据类型是一个原始数据类型；
- 创建
```javascript
let s = Symbol();
typeof s // "symbol"

let s1 = Symbol('foo');
s1 // Symbol(foo)
```
    - 通过`Symbol()`创建的symbol与其他任何值都不相等
    ```javascript
    // 没有参数的情况
    let s1 = Symbol();
    let s2 = Symbol();
    s1 === s2 // false
    
    // 有参数的情况
    let s1 = Symbol('foo');
    let s2 = Symbol('foo');
    s1 === s2 // false
    ```
    - Symbol 值不能与其他类型的值进行运算
    ```javascript
    let sym = Symbol('My symbol');
    "your symbol is " + sym
    // TypeError: can't convert symbol to string
    `your symbol is ${sym}`
    // TypeError: can't convert symbol to string
    ```
    - Symbol 值可以显式转为字符串，也可以转为布尔值，但是不能转为数值。
    ```javascript
    // Symbol 值可以显式转为字符串
    let sym = Symbol('My symbol');
    String(sym) // 'Symbol(My symbol)'
    sym.toString() // 'Symbol(My symbol)'
    
    let sym = Symbol();
    Boolean(sym) // true
    !sym  // false
    
    if (sym) {
      // ...
    }
    
    Number(sym) // TypeError
    sym + 2 // TypeError
    ```
- 做为属性名使用
```javascript
let mySymbol = Symbol();

// 第一种写法
let a = {};
a[mySymbol] = 'Hello!';

// 第二种写法
let a = {// 对象的增强属性名写法
  [mySymbol]: 'Hello!'
};

// 第三种写法
let a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello!' });

// 以上写法都得到同样结果
a[mySymbol] // "Hello!"
```
    - 做为属性名使用时，不能使用`.`语法，使用点语法会被当成字符串处理
    ```javascript
    const mySymbol = Symbol();
    const a = {};
    
    a.mySymbol = 'Hello!';
    a[mySymbol] // undefined
    a['mySymbol'] // "Hello!"
    ```

### 属性名的遍历
- Symbol 作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。
```javascript
const obj = {};
let a = Symbol('a');
let b = Symbol('b');

obj[a] = 'Hello';
obj[b] = 'World';

const objectSymbols = Object.getOwnPropertySymbols(obj);

objectSymbols
// [Symbol(a), Symbol(b)]
```

### Symbol的共享(重复使用)
- 有时，我们想重复使用某个Symbol，我们知道通过`Symbol()`方法，生成的symbol是绝对唯一的，即使描述符一样
```javascript
let s1 = Symbol('foo');
let s2 = Symbol('foo');
s1===s2 // false
```
- `Symbol.for`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值
```javascript
let s1 = Symbol.for('foo');
let s2 = Symbol.for('foo');
s1 === s2 // true

Symbol.for("bar") === Symbol.for("bar")
// true

Symbol("bar") === Symbol("bar")
// false
```

### 用Symbol实现单例模式
- 传统实现
```javascript
// mod.js
function A() {
  this.foo = 'hello';
}
if (!global._foo) {
  global._foo = new A();
}
module.exports = global._foo;

// 加载
const a = require('./mod.js');
console.log(a.foo);
```
    - 全局变量global._foo是可写的，任何文件都可以修改
- 使用`Symbol`
```javascript
// mod.js
const FOO_KEY = Symbol.for('foo');

function A() {
  this.foo = 'hello';
}

if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}

module.exports = global[FOO_KEY];
```
    - 上面代码中，可以保证global[FOO_KEY]不会被无意间覆盖，但还是可以被改写
    ```javascript
    const a = require('./mod.js');
    global[Symbol.for('foo')] = 123;
    ```

## Set和Map数据结构

### Set数据结构
- 它类似于数组，但是**成员的值都是唯一的**，**没有重复的值**。
- 使用`new Set()`创建
```javascript
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4，重复值不会被保留
```
    - Set 函数可以接受一个数组（或者具有 iterable 接口的其他数据结构）作为参数，用来初始化
    ```javascript
    // 例一
    const set = new Set([1, 2, 3, 4, 4]);
    [...set]
    // [1, 2, 3, 4]，自动删除了重复的值，可利用这方法实现数组去重
    
    // 例二
    const items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
    items.size // 5
    
    // 例三
    function divs () {
      return [...document.querySelectorAll('div')];
    }
    
    const set = new Set(divs());
    set.size // 56
    
    // 类似于
    divs().forEach(div => set.add(div));
    set.size // 56
    ```
    - `Set`判断值是否重复，类似`===`，不过`NaN`等于自身；任意两个对象是不相等的
    ```javascript
    let set = new Set();
    let a = NaN;
    let b = NaN;
    set.add(a);
    set.add(b);
    set // Set {NaN}
    
    let set = new Set();
    set.add({});
    set.size // 1
    set.add({});
    set.size // 2
    ```
- 属性和方法
    - 属性
        - Set.prototype.constructor：构造函数，默认就是Set函数。
        - Set.prototype.size：返回Set实例的成员总数。
    - 操作方法
        - add(value)：添加某个值，返回Set结构本身。
        - delete(value)：删除某个值，返回一个布尔值，表示删除是否成功。
        - has(value)：返回一个布尔值，表示该值是否为Set的成员。
        - clear()：清除所有成员，没有返回值。
    - 遍历方法
        - keys()：返回键名的遍历器
        - values()：返回键值的遍历器
        - entries()：返回键值对的遍历器
        - forEach()：使用回调函数遍历每个成员
    ```javascript
    s.add(1).add(2).add(2);// 注意2被加入了两次
    s.size // 2
    s.has(1) // true
    s.has(2) // true
    s.has(3) // false
    s.delete(2);
    s.has(2) // false
    
    let set = new Set(['red', 'green', 'blue']);
    for (let item of set.keys()) {
      console.log(item);
    }
    // red
    // green
    // blue
    for (let item of set.values()) {
      console.log(item);
    }
    // red
    // green
    // blue
    for (let item of set.entries()) {
      console.log(item);
    }
    // ["red", "red"]
    // ["green", "green"]
    // ["blue", "blue"]
    
    // 可直接使用for..of遍历
    let set = new Set(['red', 'green', 'blue']);
    for (let x of set) {
      console.log(x);
    }
    // red
    // green
    // blue
    
    let set = new Set([1, 2, 3]);
    set.forEach((value, key) => console.log(value * 2) )// 使用forEach做附加操作
    // 2
    // 4
    // 6
    
    // set配合filter完成交集、并集、差集
    let a = new Set([1, 2, 3]);
    let b = new Set([4, 3, 2]);
    // 并集
    let union = new Set([...a, ...b]);
    // Set {1, 2, 3, 4}
    // 交集
    let intersect = new Set([...a].filter(x => b.has(x)));
    // set {2, 3}
    // 差集
    let difference = new Set([...a].filter(x => !b.has(x)));
    // Set {1}
    ```
- Array.from方法可以将 Set 结构转为数组。
```javascript
const items = new Set([1, 2, 3, 4, 5]);
const array = Array.from(items);

// 利用from方法完成数组的去重
function dedupe(array) {
  return Array.from(new Set(array));
}
dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

### WeakSet
- WeakSet 结构与 Set 类似，也是不重复的值的集合。但是，它与 Set 有两个区别
    - WeakSet 的成员只能是对象，而不能是其他类型的值
    - 垃圾回收机制不考虑 WeakSet 对该对象的引用；如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中
        - WeakSet 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失
        - WeakSet 的成员是不适合引用的，因为它会随时消失
        - 由于 WeakSet 内部有多少个成员，取决于垃圾回收机制有没有运行，运行前后很可能成员个数是不一样的，而垃圾回收机制何时运行是不可预测的，**因此 ES6 规定 WeakSet 不可遍历**
        ```javascript
        const ws = new WeakSet();
        ws.add(1)
        // TypeError: Invalid value used in weak set
        ws.add(Symbol())
        // TypeError: invalid value used in weak set
        
        // WeakSet 可以接受一个数组或类似数组的对象作为参数(实际上，任何具有 Iterable 接口的对象，都可以作为 WeakSet 的参数)
        const a = [[1, 2], [3, 4]];
        const ws = new WeakSet(a);// 会将a数组的成员添加到ws中，而不是a数组自身；所以数组成员必须是对象
        // WeakSet {[1, 2], [3, 4]}
        
        const b = [3, 4];
        const ws = new WeakSet(b); // b的成员不是对象，所以会出错
        // Uncaught TypeError: Invalid value used in weak set(…)
        ```
- 方法
    - add、delete、has，同set相比没有clear方法
    ```javascript
    const ws = new WeakSet();
    const obj = {};
    const foo = {};
    
    ws.add(window);
    ws.add(obj);
    ws.has(window); // true
    ws.has(foo);    // false
    ws.delete(window);
    ws.has(window);    // false
    
    // WeakSet没有size属性，没有办法遍历它的成员。
    ws.size // undefined
    ws.forEach // undefined
    
    ws.forEach(function(item){ console.log('WeakSet has ' + item)})
    // TypeError: undefined is not a function
    ```

### Map数据结构
- 类似Object结构，都是key-value结构，不同的地方是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object 结构提供了“字符串—值”的对应，Map结构提供了“值—值”的对应，是一种更完善的 Hash 结构实现。如果你需要“键值对”的数据结构，Map 比 Object 更合适。
- 提供了一种更加灵活方便的一一映射的结构
```javascript
const m = new Map();
const o = {p: 'Hello World'};
m.set(o, 'content')
m.get(o) // "content"
m.has(o) // true
m.delete(o) // true
m.has(o) // false

// 接收数组做为参数
const map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);
map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"

// 上面的等同于
const items = [
  ['name', '张三'],
  ['title', 'Author']
];

const map = new Map();

items.forEach(// 取得每个item的key,value，然后再添加到map中
  ([key, value]) => map.set(key, value)
);
```
- 属性和方法
    - 属性
        - size属性返回 Map 结构的成员总数。
    - 操作方法
        - set方法设置键名key对应的键值为value，然后返回整个 Map 结构。如果key已经有值，则键值会被更新，否则就新生成该键。
        - get方法读取key对应的键值，如果找不到key，返回undefined。
        - has方法返回一个布尔值，表示某个键是否在当前 Map 对象之中。
        - delete方法删除某个键，返回true。如果删除失败，返回false。
        - clear方法清除所有成员，没有返回值。
    - 遍历方法
        - keys()：返回键名的遍历器。
        - values()：返回键值的遍历器。
        - entries()：返回所有成员的遍历器。
        - forEach()：遍历 Map 的所有成员
    ```javascript
    // size
    const map = new Map();
    map.set('foo', true);
    map.set('bar', false);
    map.size // 2
    
    // set
    const m = new Map();
    m.set('edition', 6)        // 键是字符串
    m.set(262, 'standard')     // 键是数值
    m.set(undefined, 'nah')    // 键是 undefined
    
    // set方法返回的是当前的Map对象，因此可以采用链式写法。
    let map = new Map()
      .set(1, 'a')
      .set(2, 'b')
      .set(3, 'c');
    
    // get
    const m = new Map();
    const hello = function() {console.log('hello');};
    m.set(hello, 'Hello ES6!') // 键是函数
    m.get(hello)  // Hello ES6!
    
    // has
    const m = new Map();
    m.set('edition', 6);
    m.set(262, 'standard');
    m.set(undefined, 'nah');
    m.has('edition')     // true
    m.has('years')       // false
    m.has(262)           // true
    m.has(undefined)     // true
    
    // delete
    const m = new Map();
    m.set(undefined, 'nah');
    m.has(undefined)     // true
    m.delete(undefined)
    m.has(undefined)       // false
    
    // clear
    let map = new Map();
    map.set('foo', true);
    map.set('bar', false);
    map.size // 2
    map.clear()
    map.size // 0
    
    // 遍历方法
    const map = new Map([
      ['F', 'no'],
      ['T',  'yes'],
    ]);
    
    for (let key of map.keys()) {
      console.log(key);
    }
    // "F"
    // "T"
    
    for (let value of map.values()) {
      console.log(value);
    }
    // "no"
    // "yes"
    
    for (let item of map.entries()) {
      console.log(item[0], item[1]);
    }
    // "F" "no"
    // "T" "yes"
    
    // 或者
    for (let [key, value] of map.entries()) {
      console.log(key, value);
    }
    // "F" "no"
    // "T" "yes"
    
    // 等同于使用map.entries()
    for (let [key, value] of map) {
      console.log(key, value);
    }
    // "F" "no"
    // "T" "yes"
    
    // forEach，第二个参数可以绑定this
    const reporter = {
      report: function(key, value) {
        console.log("Key: %s, Value: %s", key, value);
      }
    };
    
    map.forEach(function(value, key, map) {
      this.report(key, value);
    }, reporter);
    ```
- map转数组可以使用`...`扩展运算符
```javascript
const map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```
- 结合数组的map方法、filter方法，可以实现 Map 的遍历和过滤（Map 本身没有map和filter方法）
```javascript
const map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

const map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 产生 Map 结构 {1 => 'a', 2 => 'b'}

const map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 产生 Map 结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```
- 与其他数据结构的互相转换
    - Map 转为数组
    ```javascript
    const myMap = new Map()
      .set(true, 7)
      .set({foo: 3}, ['abc']);
    [...myMap]
    // [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
    ```
    - 数组 转为 Map
    ```javascript
    new Map([
      [true, 7],
      [{foo: 3}, ['abc']]
    ])
    // Map {
    //   true => 7,
    //   Object {foo: 3} => ['abc']
    // }
    ```
    - Map 转为对象
    ```javascript
    function strMapToObj(strMap) {
      let obj = Object.create(null);
      for (let [k,v] of strMap) {
        obj[k] = v;
      }
      return obj;
    }
    
    const myMap = new Map()
      .set('yes', true)
      .set('no', false);
    strMapToObj(myMap)
    // { yes: true, no: false }
    ```
    - 对象转为 Map
    ```javascript
    function objToStrMap(obj) {
      let strMap = new Map();
      for (let k of Object.keys(obj)) {
        strMap.set(k, obj[k]);
      }
      return strMap;
    }
    
    objToStrMap({yes: true, no: false})
    // Map {"yes" => true, "no" => false}
    ```
    - Map 转为 JSON
    ```javascript
    // Map 的键名都是字符串，这时可以选择转为对象 JSON。
    function strMapToJson(strMap) {
      return JSON.stringify(strMapToObj(strMap));
    }
    
    let myMap = new Map().set('yes', true).set('no', false);
    strMapToJson(myMap)
    // '{"yes":true,"no":false}'
    
    // Map 的键名有非字符串，这时可以选择转为数组 JSON
    function mapToArrayJson(map) {
      return JSON.stringify([...map]);
    }
    let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
    mapToArrayJson(myMap)
    // '[[true,7],[{"foo":3},["abc"]]]'
    ```
    - JSON 转为 Map
    ```javascript
    function jsonToStrMap(jsonStr) {
      return objToStrMap(JSON.parse(jsonStr));
    }
    
    jsonToStrMap('{"yes": true, "no": false}')
    // Map {'yes' => true, 'no' => false}
    ```

### WeakMap
- 类似WeakSet
```javascript
// WeakMap 可以使用 set 方法添加成员
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2

// WeakMap 也可以接受一个数组，
// 作为构造函数的参数
const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
wm2.get(k2) // "bar"
```
- WeakMap与Map的区别有两点。
    - WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名
    - WeakMap的键名所指向的对象，不计入垃圾回收机制
```javascript
const map = new WeakMap();
map.set(1, 2)
// TypeError: 1 is not an object!
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
map.set(null, 2)
// TypeError: Invalid value used as weak map key
```
- WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏
    - WeakMap 应用的典型场合就是 DOM 节点作为键名
    ```javascript
    let myElement = document.getElementById('logo');
    let myWeakmap = new WeakMap();
    
    myWeakmap.set(myElement, {timesClicked: 0});
    
    myElement.addEventListener('click', function() {
      let logoData = myWeakmap.get(myElement);
      logoData.timesClicked++;
    }, false);
    
    // 上面代码中，myElement是一个 DOM 节点，每当发生click事件，就更新一下状态。我们将这个状态作为键值放在 WeakMap 里，对应的键名就是myElement。一旦这个 DOM 节点删除，该状态就会自动消失，不存在内存泄漏风险。
    ```
    - 部署私有属性
    ```javascript
    const _counter = new WeakMap();
    const _action = new WeakMap();
    
    class Countdown {
      constructor(counter, action) {
        _counter.set(this, counter);
        _action.set(this, action);
      }
      dec() {
        let counter = _counter.get(this);
        if (counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
          _action.get(this)();
        }
      }
    }
    
    const c = new Countdown(2, () => console.log('DONE'));
    
    c.dec()
    c.dec()
    // DONE
    ```

## Proxy
- Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”
    - Proxy相当于在目标对象上添加了一层拦截器，在某些操作生效之前，可以在拦截器中做一些操作，例如在设置某个值前，检查值是否符合要求
    ```javascript
    var proxy = new Proxy({}, {// 代理了{}对象的所有get操作，在实际操作生效前，拦截了并返回新值
      get: function(target, property) {
        return 35;
      }
    });
    
    proxy.time // 35
    proxy.name // 35
    proxy.title // 35
    
    // 拦截器可以代理很多类型的操作，在这些操作真正生效前，进行拦截
    var handler = {
      get: function(target, name) {
        if (name === 'prototype') {
          return Object.prototype;
        }
        return 'Hello, ' + name;
      },
    
      apply: function(target, thisBinding, args) {
        return args[0];
      },
    
      construct: function(target, args) {
        return {value: args[1]};
      }
    };
    
    var fproxy = new Proxy(function(x, y) {
      return x + y;
    }, handler);
    
    fproxy(1, 2) // 1
    new fproxy(1,2) // {value: 2}
    fproxy.prototype === Object.prototype // true
    fproxy.foo // "Hello, foo"
    ```
- Proxy中的this问题
    - 在 Proxy 代理的情况下，目标对象内部的this关键字会指向 Proxy 代理。
    ```javascript
    const target = {
      m: function () {
        console.log(this === proxy);
      }
    };
    const handler = {};
    
    const proxy = new Proxy(target, handler);
    
    target.m() // false
    proxy.m()  // true，代理后，this指向了proxy而不是原先的target
    ```

## Reflect
- 应该是js在设计之初有些不合理的地方，例如将一些明显是语言内部的方法放到了`Object`上，现在要改正，需要一个容器来装载这些方法，所以将这些改良方法都放在了`Reflect`对象上
- 特点
    - 现阶段，某些方法同时在Object和Reflect对象上部署，未来的新方法将只部署在Reflect对象上。也就是说，从Reflect对象上可以拿到语言内部的方法。
    - 修改某些Object方法的返回结果，让其变得更合理。
    - 让Object操作都变成函数行为。
    - Reflect对象的方法与Proxy对象的方法一一对应，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。

## Promise对象
- 可以和`jQuery`中的延迟对象以及promise对比理解
- 基本用法
```javascript
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    return resolve(value);
  } else {
    return reject(error);
  }
});

promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```
- `Promise`新建后就会立即执行
```javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  return resolve();
});

promise.then(function() {
  console.log('resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// resolved
```
- 返回另一个异步操作
```javascript
var p1 = new Promise(function (resolve, reject) {
  // ...
});

var p2 = new Promise(function (resolve, reject) {
  // ...
  return resolve(p1);// p1和p2都是 Promise 的实例，但是p2的resolve方法将p1作为参数，即一个异步操作的结果是返回另一个异步操作，这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会立刻执行
});

```

### p.then
- promise.then(resolvedFn,rejectFn);
    - 基于前一个promise的状态分别调用不同的回调函数
    ```javascript
    getJSON("/post/1.json").then(function(post) {
      return getJSON(post.commentURL);
    }).then(function funcA(comments) {
      console.log("resolved: ", comments);
    }, function funcB(err){
      console.log("rejected: ", err);
    });
    ```

### p.catch
- Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数。
```javascript
p.then((val) => console.log('fulfilled:', val))
  .catch((err) => console.log('rejected', err));

// 等同于
p.then((val) => console.log('fulfilled:', val))
  .then(null, (err) => console.log("rejected:", err));
```

### p.all
- Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。
    - 当多个promise都resolve了，则调用then的resolvFn，否则都调用rejectFn
```javascript
// promises是包含6个 Promise 实例的数组，只有这6个实例的状态都变成fulfilled，或者其中有一个变为rejected，才会调用Promise.all方法后面的回调函数。
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

### p.race
- Promise.race方法同样是将多个Promise实例，包装成一个新的Promise实例，哪个promise状态先改变，则后面的回调将根据其状态做不同的调用
```javascript
var p = Promise.race([p1, p2, p3]);
// 只要p1、p2、p3之中有一个实例率先改变状态，p的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给p的回调函数。

const p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
]);
p.then(response => console.log(response));
p.catch(error => console.log(error));
// 如果5秒之内fetch方法无法返回结果，变量p的状态就会变为rejected，从而触发catch方法指定的回调函数
```

### Promise.resolve
- 用来将一个对象转换成ES6的Promise对象
- 参数是一个Promise实例，则直接返回这个Promise实例
- 参数是一个thenable对象，Promise.resolve方法会将这个对象转为Promise对象，然后就立即执行thenable对象的then方法
```javascript
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```
- 参数不是具有then方法的对象，或根本就不是对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved
```javascript
var p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```
- 不带有任何参数，直接返回一个resolved状态的Promise对象。
```javascript
var p = Promise.resolve();

p.then(function () {
  // ...
});
```

### Promise.reject()
- Promise.reject(reason)方法也会返回一个新的 Promise 实例，该实例的状态为rejected。
```javascript
var p = Promise.reject('出错了');
// 等同于
var p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```

## Iterator 和 for...of 循环

### Iterator
- 遍历器（Iterator）它是一种接口，为各种不同的数据结构提供统一的访问机制；
- 默认的 Iterator 接口部署在数据结构的Symbol.iterator属性，或者说，一个数据结构只要具有Symbol.iterator属性，就可以认为是“可遍历的”（iterable）
```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();
iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }// done表示本次遍历结束
```
- 原生具备 Iterator 接口的数据结构如下，对于原生部署 Iterator 接口的数据结构，不用自己写遍历器生成函数，原生的Obj不具备遍历器接口
    - Array
    - Map
    - Set
    - String
    - TypedArray
    - 函数的 arguments 对象
    - NodeList 对象
- 某些情况下，会默认调用Iterator 接口
    - 解构赋值
    - 扩展运算符
    - yield*
    - 任何接受数组作为参数的场合，其实都调用了遍历器接口

### for...of
- 所有部署了Iterator接口的数据结构，都可以使用for...of来进行遍历
- 所有原生具备Iterator接口的都可以直接使用for...of遍历
```javascript
// 数组
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}

const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for(let v of obj) {
  console.log(v); // red green blue
}

// Set、Map
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```
- 有些数据结构是在现有数据结构的基础上，计算生成的。比如，ES6的数组、Set、Map 都部署了keys()、values()、entries方法，调用后都返回遍历器对象
```javascript
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
```
- 类似数组的对象
```javascript
// 字符串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList对象
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```
- 对象不可直接使用for...of遍历
```javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (let e of es6) {
  console.log(e);
}
// TypeError: es6[Symbol.iterator] is not a function
```
    - 使用Object.keys方法将对象的键名生成一个数组，然后遍历这个数组。
    ```javascript
    for (var key of Object.keys(someObject)) {
      console.log(key + ': ' + someObject[key]);
    }
    ```

### 遍历语法的对比
- for
    - 繁琐
    ```javascript
    for (var index = 0; index < myArray.length; index++) {
      console.log(myArray[index]);
    }
    ```
- forEach方法
    - 无法中途跳出forEach循环，break命令或return命令都不能奏效。
    ```javascript
    myArray.forEach(function (value) {
      console.log(value);
    });
    ```
- for...in
    - for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
    - 遍历顺序在某些情况下还不固定
    - 主要是为对象部署的
- for...of
    - 只要部署了遍历接口的数据结构都可使用
    - 不同于forEach方法，它可以与break、continue和return配合使用。
    ```javascript
    for (var n of fibonacci) {
      if (n > 1000)
        break;
      console.log(n);
    }
    ```

## Generator 函数的语法

### 基本概念
- Generator函数总是返回一个遍历器对象，通过调用遍历器对象的next()方法，让函数一步一步的执行
- 语法形式
```javascript
function* helloWorldGenerator() {// function后跟一个*
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();// 被调用时，返回一个遍历器对象，可在遍历器对象上调用next()方法
hw.next() // 每次调用next()返回的对象的value值都是yield后表达式对应的值
// { value: 'hello', done: false }

hw.next() // 每次调用next()时，会寻找下一个yield表达式
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```
- yield 表达式
    - yield是暂停标志，调用next()时，遇到yield会暂停，并将后面表达式的值返回，下次调用next()时会按顺序寻找下一个yield；如果是最后一个yield，则继续执行，直到遇到return，将return后面的值做为返回的对象的value值，如果没有return，则返回对象的value值为undefined
    - yield后面的表达式是惰性求值
    ```javascript
    function* gen() {
      yield  123 + 456; // yield后面的表达式123 + 456，不会立即求值，只会在next方法将指针移到这一句时，才会求值
    }
    ```
    - yield表达式只能用在 Generator 函数里面，用在其他地方都会报错。
    ```javascript
    (function (){
      yield 1;
    })()
    // SyntaxError: Unexpected number
    ```
- 与 Iterator 接口的关系
    - 由于Generator 函数返回的是一个遍历器对象，所以可以把 Generator 赋值给对象的Symbol.iterator属性，从而使得该对象具有 Iterator 接口
    ```javascript
    var myIterable = {};
    myIterable[Symbol.iterator] = function* () {
      yield 1;
      yield 2;
      yield 3;
    };
    
    [...myIterable] // [1, 2, 3]
    ```
- next方法的参数
    - yield表达式本身没有返回值，或者说总是返回undefined
    - next方法的参数代表上一个yield表达式返回的值
    ```javascript
    function* f() {
      for(var i = 0; true; i++) {
        var reset = yield i;
        if(reset) { i = -1; }
      }
    }
    
    var g = f();
    
    g.next() // { value: 0, done: false }，第一次调用next，找到yield，此时i为0，暂停后面的执行并返回0做为返回对象的value值
    g.next() // { value: 1, done: false }，在上一次yield后继续执行寻找下一个yield，由于第一次yield默认返回undefined，所以rest为undefined,所以i=-1并未执行，进入第二轮循环，找到yield，此时i为1，暂停后面的执行并返回1做为返回对象的value值
    g.next(true) // { value: 0, done: false }，传入参数true，true被当成上一个yield表达式的返回值，所以rest为true，进而i=-1被执行，进入第三次循环(此时i已经加1了为0)，找到yield，此时i为0，暂停后面的执行并返回0做为返回对象的value值
    ```
- for...of
    - for...of循环可以自动遍历 Generator 函数时生成的Iterator对象，且此时不再需要调用next方法
    ```javascript
    function *foo() {
      yield 1;
      yield 2;
      yield 3;
      yield 4;
      yield 5;
      return 6;
    }
    
    for (let v of foo()) {
      console.log(v);
    }
    // 1 2 3 4 5，不会返回return后的值，所以for...of只会按顺序返回yield后的值
    ```
- Generator.prototype.throw()、Generator.prototype.return()
    - throw用来抛出错误
    - return可以返回给定的值，并且终结遍历 Generator 函数
- 在Generator函数内调用另外一个Generator函数
    - 使用yield*表达式
    ```javascript
    function* inner() {
      yield 'hello!';
    }
    
    function* outer1() {
      yield 'open';
      yield inner();
      yield 'close';
    }
    
    var gen = outer1()
    gen.next().value // "open"
    gen.next().value // 返回一个遍历器对象
    gen.next().value // "close"
    
    function* outer2() {
      yield 'open'
      yield* inner()
      yield 'close'
    }
    
    var gen = outer2()
    gen.next().value // "open"
    gen.next().value // "hello!"
    gen.next().value // "close"
    ```
- 作为对象属性的Generator函数可以采用简写形式
```javascript
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};

// 简写形式
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```

### 应用
- 异步操作的同步化表达
```javascript
function* main() {
  var result = yield request("http://some.url");// 1.此处被暂停，进行request异步请求
  var resp = JSON.parse(result);// 对2处异步请求的返回值做相应处理
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){// 异步请求成功
    it.next(response);// 2.手动调用next，并将异步请求的返回值做为上一次yield表达式的值
  });
}

var it = main();// 返回遍历器对象
it.next();// 执行next，寻找第一个yield
```
- 控制流管理
```javascript
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // Do something with value4
      });
    });
  });
});

// promise改写
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(function (value4) {
    // Do something with value4
  }, function (error) {
    // Handle any error from step1 through step4
  })
  .done();

// generator改写
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}

scheduler(longRunningTask(initialValue));

function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
```
- 部署 Iterator 接口
    - 原生obj上因为没有Iterator 接口无法使用for...of，可以利用Generator 函数，可以在任意对象上部署 Iterator 接口。
    ```javascript
    function* iterEntries(obj) {
      let keys = Object.keys(obj);
      for (let i=0; i < keys.length; i++) {
        let key = keys[i];
        yield [key, obj[key]];
      }
    }
    
    let myObj = { foo: 3, bar: 7 };
    
    for (let [key, value] of iterEntries(myObj)) {
      console.log(key, value);
    }
    
    // foo 3
    // bar 7
    ```
- 作为数据结构
    - Generator 可以看作是数据结构，更确切地说，可以看作是一个数组结构，因为 Generator 函数可以返回一系列的值，这意味着它可以对任意表达式，提供类似数组的接口。
    ```javascript
    function *doStuff() {
      yield fs.readFile.bind(null, 'hello.txt');
      yield fs.readFile.bind(null, 'world.txt');
      yield fs.readFile.bind(null, 'and-such.txt');
    }
    
    for (task of doStuff()) {
      // task是一个函数，可以像回调函数那样使用它
    }
    ```

## Generator 函数的异步应用

### 传统方法完成异步
- 回调
    - 容易出现callback hell
    ```javascript
    fs.readFile(fileA, 'utf-8', function (err, data) {
      fs.readFile(fileB, 'utf-8', function (err, data) {
        // ...
      });
    });
    ```
- Promise
    - 允许将回调函数的嵌套，改成链式调用
    ```javascript
    var readFile = require('fs-readfile-promise');
    
    readFile(fileA)
    .then(function (data) {
      console.log(data.toString());
    })
    .then(function () {
      return readFile(fileB);
    })
    .then(function (data) {
      console.log(data.toString());
    })
    .catch(function (err) {
      console.log(err);
    });
    ```
    - Promise 的最大问题是代码冗余，原来的任务被 Promise 包装了一下，不管什么操作，一眼看去都是一堆then，原来的语义变得很不清楚

### Generator 函数完成异步任务
- 利用Generator中的`yield`暂停函数的执行，等到恢复后，继续从此处执行；
- 遇到yield命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作，如果去除yield命令，简直一模一样
- 整个 Generator 函数就是一个封装的异步任务，或者说是异步任务的容器。**异步操作需要暂停的地方，都用yield语句注明**。
- **通过next方法接收参数，向Generator函数体内输入数据。**
```javascript
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);// 返回的结果保存在result中
  console.log(result.bio);
}

var g = gen();
var result = g.next();// 得到异步读取的结果

result.value.then(function(data){// Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next方法。
  return data.json();// 转换为json
}).then(function(data){
  g.next(data);// 传入Generator函数
});
```

## async 函数

### async函数是Generator函数的语法糖
```javascript
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

// 使用async改写
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```
- 改进
    - 内置执行器
        - Generator必须手动一步一步的调用next()方法或者是用第三方的执行器自动执行，而async函数内部了执行器，能自动执行
    - 更好的语义
    - 更广的适用性
        - 如果使用第三放的执行器，例如`co`则`yield`后只能是Thunk 函数或 Promise 对象，而async函数内的await命令后面，可以是Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）
    - async返回值是Promise，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作
    - async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖

### 基本用法
- async函数返回一个 Promise 对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会先返回，等到异步操作完成，再接着执行函数体内后面的语句
```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);// 等待getStockSymbol异步完成，再执行下面的
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {// getStockPriceByName是一个async函数，所以返回的是Promise，可以使用then
  console.log(result);
});
```
- async 函数有多种使用形式
```javascript
// 函数声明
async function foo() {}

// 函数表达式
const foo = async function () {};

// 对象的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭头函数
const foo = async () => {};
```

### 语法
- async函数返回一个 Promise 对象。async函数内部return语句返回的值，会成为then方法回调函数的参数。
```javascript
async function f() {// 整体返回一个Promise对象
  return 'hello world';// 内部的return返回的值将当做then方法回调的参数
}

f().then(v => console.log(v))// 'heollo world'传递给了v
// "hello world"
```
- 正常情况下，await命令后面是一个 Promise 对象。如果不是，会被转成一个立即resolve的 Promise 对象
```javascript
async function f() {
  return await 123;// 123会被转换成Promise对象，并立即resolved
}

f().then(v => console.log(v))
// 123
```
- async函数返回的 Promise 对象，必须等到**内部所有await命令后面的 Promise 对象执行完，才会发生状态改变**，除非遇到return语句或者抛出错误。也就是说，只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数
```javascript
async function getTitle(url) {// 所有await命令都执行完成，才会执行then的回调
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

### 注意点
- async函数内部抛出错误，会导致返回的 Promise 对象变为reject状态。抛出的错误对象会被catch方法回调函数接收到
```javascript
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```
- await命令后面的 Promise 对象如果变为reject状态，则reject的参数会被catch方法的回调函数接收到。
```javascript
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了
```
- 只要一个await语句后面的 Promise 变为reject，那么整个async函数都会中断执行
```javascript
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```
- 前一个异步操作失败，也不要中断后面的异步操作，可以将第一个await放在try...catch结构里面，这样不管这个异步操作是否成功，第二个await都会执行。或者await后面的 Promise 对象再跟一个catch方法，处理前面可能出现的错误。
```javascript
async function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}
f()
.then(v => console.log(v))
// hello world

// await后面的 Promise 对象再跟一个catch方法，处理前面可能出现的错误
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}
f()
.then(v => console.log(v))
// 出错了
// hello world
```
- 多个await命令后面的异步操作，如果不存在继发关(依赖)系，最好让它们同时触发。
```javascript
// getBar、getFoo这两个异步请求并无依赖关系，但这种写法getBar必须在getFoo完成后再请求
let foo = await getFoo();
let bar = await getBar();

// 使用Promise.all
let [foo, bar] = await Promise.all([getFoo(), getBar()]);// 二者可以同时请求
```
- await命令只能用在async函数之中，如果用在普通函数，就会报错
```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 报错
  docs.forEach(function (doc) {// forEach参数为普通函数，所以无法使用await
    await db.post(doc);
  });
}
```

### 实例:按顺序完成异步操作
- Promise 的写法如下
```javascript
// 使用fetch方法，同时远程读取一组 URL。每个fetch操作都返回一个 Promise 对象，放入textPromises数组。然后，reduce方法依次处理每个 Promise 对象，然后使用then，将所有 Promise 对象连起来，因此就可以依次输出结果
function logInOrder(urls) {
  // 远程读取所有URL
  const textPromises = urls.map(url => {
    return fetch(url).then(response => response.text());
  });

  // 按次序输出
  textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}
```
- async写法
```javascript
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
// 上面的代码，下一个请求必须在上一个请求完成后再请求，而请求之间无依赖关系，所以可以同时请求，优化如下
async function logInOrder(urls) {
  // 并发读取远程URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序输出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
// 上面代码中，虽然map方法的参数是async函数，但它是并发执行的，因为只有async函数内部是继发执行，外部不受影响。后面的for..of循环内部使用了await，因此实现了按顺序输出。
```

### 异步遍历器
- 类似同步遍历器，不过它调用next()方法后返回的是一个Promise对象，可以使用then方法
```javascript
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```
- 异步遍历器接口返回的是一个异步遍历器，可以使用`for await...of`来遍历
```javascript
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```

### 异步 Generator 函数
- Generator 函数返回一个同步遍历器对象一样，异步 Generator 函数的作用，是返回一个异步遍历器对象。
- 语法上，异步 Generator 函数就是async函数与 Generator 函数的结合
```javascript
// gen是一个异步 Generator 函数，执行后返回一个异步 Iterator 对象。对该对象调用next方法，返回一个 Promise 对象
async function* gen() {
  yield 'hello';
}
const genObj = gen();
genObj.next().then(x => console.log(x));
// { value: 'hello', done: false }

// 同步 Generator 函数
function* map(iterable, func) {
  const iter = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iter.next();
    if (done) break;
    yield func(value);
  }
}

// 异步 Generator 函数
async function* map(iterable, func) {
  const iter = iterable[Symbol.asyncIterator]();
  while (true) {
    const {value, done} = await iter.next();
    if (done) break;
    yield func(value);
  }
}
```
- 异步 Generator 函数出现以后，JavaScript 就有了四种函数形式：`普通函数`、`async 函数`、`Generator 函数`和`异步 Generator 函数`。请注意区分每种函数的不同之处。基本上，如果是一系列按照顺序执行的异步操作（比如读取文件，然后写入新内容，再存入硬盘）存在明显的先后顺序才能得到正确结果时，可以使用 async 函数；如果是一系列产生相同数据结构的异步操作（比如一行一行读取文件），可以使用异步 Generator 函数

## Class的基本语法

### Class
- ES6的`class`(类)可以看做是ES5中类写法的一个语法糖
```javascript
//ES5
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);

//ES6
class Point {
  constructor(x, y) {// 使用new时，会自动调用这个方法
    this.x = x;
    this.y = y;
  }

  toString() {// toString实际是定义在原型对象上的，所以所有实例对象都可以使用此方法
    return '(' + this.x + ', ' + this.y + ')';
  }
}

var p=new Point(1,2);
```
- 类的所有方法都定义在类的prototype属性上面，所以所有实例对象都可以使用此方法
```javascript
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```
- 类的内部所有定义的方法，都是不可枚举的（non-enumerable）
```javascript
// ES6
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]

//ES5写法中，方法是可枚举的
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```
- 类内，默认采用严格模式
- 与函数一样，类也可以使用表达式的形式定义。
```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};

let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined

// 如果类的内部没用到的话，可以省略Me，也就是可以写成下面的形式
const MyClass = class { /* ... */ };
```

### constructor方法
- 类中的constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法；一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加；constructor方法默认返回的实例对象
```javascript
// 没有显示添加constructor方法，会默认添加
class Point {
}

// 等同于
class Point {
  constructor() {}
}
```
- 类必须使用`new`调用，否则会出错
```javascript
class B{}
B();// error
```

### 不存在变量提升
- 类不存在变量提升（hoist），这一点与 ES5 完全不同。
```javascript
new Foo(); // ReferenceError
class Foo {}
```

### 私有属性、方法
- ES6暂不提供私有属性、方法，不过可以模拟
```javascript
// 私有方法
class Widget {
  foo (baz) {
    bar.call(this, baz);
  }

  // ...
}

function bar(baz) {
  return this.snaf = baz;
}

// 私有属性，新提案使用#标识
class Point {
  #x;

  constructor(x = 0) {
    #x = +x; // 写成 this.#x 亦可
  }
}
```

### this
- 类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错
```javascript
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;// 将logger的printName方法单独提取出来
printName(); // TypeError: Cannot read property 'print' of undefined，因为此时this已经不指向logger实例了，所以找不到print方法，解决方法如下

// 直接绑定this
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}

// 使用箭头函数
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }

  // ...
}
```

### Class 的取值函数（getter）和存值函数（setter）
- 存值函数和取值函数是设置在属性的 Descriptor 对象上的
```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html"
);

"get" in descriptor  // true
"set" in descriptor  // true
```

### Class 的 Generator 方法
- 如果某个方法之前加上星号（*），就表示该方法是一个 Generator 函数
```javascript
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {// 返回一个同步遍历器对象，可以使用for...of遍历
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```

### 类的静态方法
- 如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。
```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function，无法在实例上调用
```
- 静态方法包含this关键字，这个this指的是类，而不是实例
```javascript
class Foo {
  static bar () {
    this.baz();
  }
  static baz () {
    console.log('hello');
  }
  baz () {
    console.log('world');
  }
}

Foo.bar() // hello
```
- 父类的静态方法，可以被子类继承
```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod() // 'hello'
```

### 类的静态属性
- 静态属性指的是 Class 本身的属性，即Class.propName，而不是定义在实例对象（this）上的属性。
```javascript
// 暂时只能用下面这种写法
class Foo {
}

Foo.prop = 1;
Foo.prop // 1

// 以下两种写法都无效
class Foo {
  // 写法一
  prop: 2

  // 写法二
  static prop: 2
}

Foo.prop // undefined
```
- 新提案对实例属性和静态属性都规定了新的写法
```javascript
// 类的实例属性可以用等式，写入类的定义之中。以前，我们定义实例属性，只能写在类的constructor方法里面。
class MyClass {
  myProp = 42;

  constructor() {
    console.log(this.myProp); // 42
  }
}

// 类的静态属性，类的静态属性只要在上面的实例属性写法前面，加上static关键字就可以了。
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

### new.target属性
- 一般用在构造函数之中，返回new命令调用的那个构造函数。如果构造函数不是通过new命令调用的，new.target会返回undefined，因此这个属性可以用来确定构造函数是怎么调用的。
```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用new生成实例');
  }
}

// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 生成实例');
  }
}

var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 构造函数不是通过new调用的，报错
```
- 用在ES6中的`class`时，`new.target`返回的是new后跟着的类名(其实本质一样)
```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 输出 true
```
- 子类继承父类时，`new.target`会返回子类，而不是父类。
```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // Square===Rectangle，所以输出 false，
```
- 利用这个特点，可以写出不能独立使用、必须继承后才能使用的类
```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本类不能实例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape();  // 报错
var y = new Rectangle(3, 4);  // 正确
```

## class的继承

### extends
- Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多。
```javascript
// 定义了一个ColorPoint类，该类通过extends关键字，继承了Point类的所有属性和方法。
class Point {
}

class ColorPoint extends Point {
}
```

### super
- **ES5 的继承，实质是先创造子类的实例对象this，然后再将父类的方法添加到this上面（Parent.apply(this)）。ES6 的继承机制完全不同，实质是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this**。
```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)，获取父类的实例(this对象)
    this.color = color;
  }
}
```
- **子类必须在constructor方法中调用super方法，否则新建实例时会报错。这是因为子类没有自己的this对象，而是继承父类的this对象，然后对其进行加工。如果不调用super方法，子类就得不到this对象**。
```javascript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {// 指定了constructor，但没有调用super方法
  }
}

let cp = new ColorPoint(); // ReferenceError
```
- 当未显示指定`constructor`方法时，会默认添加`constructor`方法，并在内部自动调用`super`方法
```javascript
class ColorPoint extends Point {
}

// 等同于
class ColorPoint extends Point {
  constructor(...args) {
    super(...args);
  }
}
```
- 只有调用super之后，才可以使用this关键字，否则会报错。这是因为**子类实例的构建，是基于对父类实例加工，只有super方法才能返回父类实例**
```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError，子类没有自己的this，必须先获得父类的this
    super(x, y);
    this.color = color; // 正确
  }
}
```
### super的使用
- `super`关键字可以当函数使用，也可以当作对象使用
- super作为函数调用时，代表父类的构造函数
```javascript
class A {}

class B extends A {
  constructor() {
    super();// 调用父类的constructor，得到父类的this(实例对象)，进而可以进行加工得到自己的实例对象
  }
}
```
    - **super虽然代表了父类A的构造函数，但是返回的是子类B的实例**，即super内部的this指的是B，因此super()在这里相当于A.prototype.constructor.call(this)。**这里其实就可以理解为，先调用父类的构造函数方法创建一个父类的实例对象，然后拷贝一份给子类，这样子类就拥有了自己的实例对象，然后通过子类自己的构造函数对拷贝过来的实例对象进行修改加工。**
    ```javascript
    class A {
      constructor() {
        console.log(new.target.name);
      }
    }
    class B extends A {
      constructor() {
        super();
      }
    }
    new A() // A
    new B() // B
    ```
    - 作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错。
    ```javascript
    class A {}
    
    class B extends A {
      m() {
        super(); // 报错
      }
    }
    ```
- super作为对象时，在普通方法中，指向**父类的原型对象**；在静态方法中，指向**父类**。
```javascript
// 普通方法中
// 子类B当中的super.p()，就是将super当作一个对象使用。这时，super在普通方法之中，指向A.prototype，所以super.p()就相当于A.prototype.p()
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();

// 由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;// super代表的是A.prototype，实例方法无法引用到
  }
}

let b = new B();
b.m // undefined

// 如果属性定义在父类的原型对象上，super就可以取到。
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```
    - ES6 规定，**通过super调用父类的方法时，super会绑定子类的this**
    ```javascript
    class A {
      constructor() {
        this.x = 1;
      }
      print() {
        console.log(this.x);
      }
    }
    
    class B extends A {
      constructor() {
        super();
        this.x = 2;
      }
      m() {
        super.print();// 调用父类原型对象上的print方法，但是this绑定的是子类的，实际上执行的是super.print.call(this)
      }
    }
    
    let b = new B();
    b.m() // 2
    ```
    - 由于绑定子类的this，所以如果通过super对某个属性赋值，这时super就是this，赋值的属性会变成子类实例的属性
    ```javascript
    class A {
      constructor() {
        this.x = 1;
      }
    }
    
    class B extends A {
      constructor() {
        super();
        this.x = 2;
        super.x = 3;
        console.log(super.x); // undefined
        console.log(this.x); // 3
      }
    }
    
    let b = new B();
    ```
- 如果super作为对象，用在静态方法之中，这时super将指向父类，而不是父类的原型对象
```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```
- 使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。
```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

### 原生构造函数的继承
- ES5中是无法继承原生构造函数，或者继承出来的行为不一致
```javascript
// 子类无法获得原生构造函数的内部属性，通过Array.apply()或者分配给原型对象都不行。原生构造函数会忽略apply方法传入的this，也就是说，原生构造函数的this无法绑定，导致拿不到内部属性。
// ES5 是先新建子类的实例对象this，再将父类的属性添加到子类上，由于父类的内部属性无法获取，导致无法继承原生的构造函数。比如，Array构造函数有一个内部属性[[DefineOwnProperty]]，用来定义新属性时，更新length属性，这个内部属性无法在子类获取，导致子类的length属性行为不正常
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});

var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```
- ES6 允许继承原生构造函数定义子类，因为 ES6 是先新建父类的实例对象this，然后再用子类的构造函数修饰this，使得父类的所有行为都可以继承。
```javascript
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined
```

## Decorator

### 修饰器
- 用来修改类、方法的行为
- 修饰类
```javascript
@testable
class MyTestableClass {
  // ...
}

function testable(target) {// target就是要修饰的类
  target.isTestable = true;
}

MyTestableClass.isTestable // true

// 等同于
class MyTestableClass {
  // ...
}

MyTestableClass = testable(MyTestableClass) || MyTestableClass;

// 一个参数不够用，可以在修饰器外面再封装一层函数
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true) // 返回一个函数，用这个函数来修饰MyTestableClass
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```
- 修饰方法
```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}

function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 类似于
Object.defineProperty(Person.prototype, 'name', descriptor);
```
- 修饰器只能用于类和类的方法，不能用于函数，因为存在函数提升
```javascript
var counter = 0;

var add = function () {
  counter++;
};

@add
function foo() {
}

// 等同于
@add
function foo() {
}

var counter;
var add;

counter = 0;

add = function () {
  counter++;
};
```

## Module语法

### 简介
- ES6 模块不是对象，而是通过export命令显式指定输出的代码，再通过import命令输入。
- import、export命令实现的是静态加载，是在编译阶段加载，而不是运行时再加载，所以效率要比CommonJS 模块的加载方式高
```javascript
// ES6模块
import { stat, exists, readFile } from 'fs';
```

### 严格模式
- ES6 的模块自动采用严格模式
- ES6 模块之中，顶层的this指向undefined，即不应该在顶层代码使用this。

### export命令
- export命令用于规定模块的对外接口
```javascript
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;

// 等同于下面的
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```
- export命令除了输出变量，还可以输出函数或类（class）。
```javascript
export function multiply(x, y) {
  return x * y;
};

export class Person{
  ...
}
```
- 使用`as`关键字给对外接口重新取名
```javascript
function v1() { ... }
function v2() { ... }

export {// v2对外有两个接口名
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```
- export命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。
```javascript
// 报错
export 1;

// 报错
var m = 1;
export m;

// 正确写法
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};

// function、class也必须遵守
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
```
- export语句输出的接口，与其对应的值是动态绑定关系，即通过该接口，可以取到模块内部实时的值
```javascript
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```
- export、import命令可以出现在模块的任何位置，只要处于模块顶层就可以。如果处于块级作用域内，就会报错
```javascript
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```

### import命令
- 使用export命令定义了模块的对外接口以后，其他 JS 文件就可以通过import命令加载这个模块
```javascript
// main.js
import {firstName, lastName, year} from './profile';// 使用解构赋值获得./profile导出值

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```
- 同export一样，可对导入的变量使用`as`重新取名
```javascript
import { lastName as surname } from './profile';
```
- import命令具有提升效果，会提升到整个模块的头部，首先执行
```javascript
foo();

import { foo } from 'my_module';
```
- import命令是静态执行，所以不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构
```javascript
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let module = 'my_module';
import { foo } from module;

// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

### 模块的整体加载
- 用星号（*）指定一个对象，所有export输出值都加载在这个对象上面。
```javascript
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}

// main.js，单独加载
import { area, circumference } from './circle';
console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));

// main.js，整体加载
import * as circle from './circle';
console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```
- 用于挂载输出值的对象不可以改写
```javascript
import * as circle from './circle';

// 下面两行都是不允许的
circle.foo = 'hello';
circle.area = function () {};
```
- **注意，直接使用`import * as xxx`时，是不会导入默认值，会忽略默认值**，如果需要同时导入默认值和所有的非默认值需要这么写
```javascript
import myDefault,* as myObj from './a'
```

### export default 命令
- 使用export命令时，可以添加default来指定默认输出值，import加载时可用任意的变量名来接收输出值
- 显然，一个模块只能有一个默认输出，因此export default命令只能使用一次
```javascript
// export-default.js
export default function () {
  console.log('foo');
}

// import-default.js
import customName from './export-default';// 这时import命令后面，不使用大括号。
customName(); // 'foo'

// export default可用在非匿名函数前，加载的时候，视同匿名函数加载。
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者写成

function foo() {
  console.log('foo');
}

export default foo;
```
- 默认输出值，在导入时，不用加`{}`，非默认导出值需要添加`{}`
```javascript
// 第一组
export default function crc32() { // 输出
  // ...
}

import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};

import {crc32} from 'crc32'; // 输入
```
- 本质上，export default就是输出一个叫做default的变量或方法，然后系统允许你为它取任意名字
```javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as xxx } from 'modules';
// 等同于
// import xxx from 'modules';
```
- 如果想在一条import语句中，同时输入默认方法和其他接口，可以写成下面这样
```javascript
import _, { each, each as forEach } from 'lodash';
```

### export 与 import 的复合写法
- 用在一个模块之中，先输入后输出同一个模块
- **注意，`export *`命令会忽略模块的default方法**
```javascript
export { foo, bar } from 'my_module';
// 等同于
import { foo, bar } from 'my_module';
export { foo, bar };

// 模块的接口改名和整体输出，也可以采用这种写法
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module';

// 默认接口的写法
export { default } from 'foo';

// 具名接口改为默认接口的写法
export { es6 as default } from './someModule';
// 等同于
import { es6 } from './someModule';
export default es6;

// 默认接口也可以改名为具名接口
export { default as es6 } from './someModule';
```

### 模块的继承
```javascript
// circle.js
export function area(radius) {
  return Math.PI * radius * radius;
}
export function circumference(radius) {
  return 2 * Math.PI * radius;
}

// circleplus.js
export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
// 上面代码中的export *，表示再输出circle模块的所有属性和方法。注意，export *命令会忽略circle模块的default方法(因为后面有自己的默认方法)。然后，上面代码又输出了自定义的e变量和默认方法。

```

### 跨模块常量
- 可以将不同常量放在不同的js文件中，然后集中导入到一个js文件中，并集体输出；使用时只用导入集中的js文件即可
```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];

// constants/index.js，将不同常量导入到一个文件，并到处
export {db} from './db';
export {users} from './users';

// script.js，使用时直接导入集中的文件
import {db, users} from './index';
```

### import()
- import命令是编译时加载，无法做到运行时加载(动态加载)，新提案通过`import()`函数实现运行时加载
- import()返回一个 Promise 对象
```javascript
const main = document.querySelector('main');

import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```
- 场景
```javascript
// 按需加载
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});

// 条件加载
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}

// 动态的模块路径
import(f())
.then(...);
```

## Module 的加载实现

### 浏览器加载ES6模块
- 传统方法，加载普通模块
```html
<!-- 页面内嵌的脚本 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部脚本 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>

<!-- defer是“渲染完再执行” -->
<script src="path/to/myModule.js" defer></script>
<!-- async是“下载完就执行” -->
<script src="path/to/myModule.js" async></script>
```
- 浏览器上加载ES6模块
```html
<!-- 需要指定type为module，异步加载，不会造成堵塞浏览器，即等到整个页面渲染完，再执行模块脚本，等同于打开了<script>标签的defer属性。 -->
<script type="module" src="foo.js"></script>

<!-- 等同于 -->
<script type="module" src="foo.js" defer></script>
```
- 对于外部的模块脚本（上例是foo.js），有几点需要注意。
    - 代码是在模块作用域之中运行，而不是在全局作用域运行。模块内部的顶层变量，外部不可见。
    - 模块脚本自动采用严格模式，不管有没有声明use strict。
    - 模块之中，可以使用import命令加载其他模块（.js后缀不可省略，需要提供绝对 URL 或相对 URL），也可以使用export命令输出对外接口。
    - 模块之中，顶层的this关键字返回undefined，而不是指向window。也就是说，在模块顶层使用this关键字，是无意义的。
    - 同一个模块如果加载多次，将只执行一次。
    ```javascript
    import utils from 'https://example.com/js/utils.js';
    
    const x = 1;
    
    console.log(x === window.x); //false
    console.log(this === undefined); // true
    
    delete x; // 句法错误，严格模式禁止删除变量
    ```

### ES6 模块与 CommonJS 模块的差异
- CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用。ES6模块输出的变量是实时更新的
- CommonJS 模块是运行时加载，ES6 模块是编译时输出接口。

### ES6模块的循环加载
- 注意，ES6输出的是一个引用
- 当一个模块已经加载后，不会重复执行
```javascript
// a.js如下
import {bar} from './b.js';
console.log('a.js');
console.log(bar);
export let foo = 'foo';

// b.js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar = 'bar';

// 执行a.js
$ babel-node a.js
b.js
undefined
a.js
bar

// 由于a.js的第一行是加载b.js，所以先执行的是b.js。而b.js的第一行又是加载a.js，这时由于a.js已经开始执行了，所以不会重复执行，而是继续往下执行b.js，所以第一行输出的是b.js。

// 接着，b.js要打印变量foo，这时a.js还没执行完，取不到foo的值，导致打印出来是undefined。b.js执行完，开始执行a.js，这时就一切正常了。
```

## ArrayBuffer

### 简介
- 先做个简单了解，后期用到再补
- ArrayBuffer对象、TypedArray视图和DataView视图是 JavaScript 操作二进制数据的一个接口
- 这个接口的原始设计目的，与 WebGL 项目有关，与显卡交换数据使用传统文本格式，需要转换效率低下，直接使用二进制交流，效率高


## 编程风格

### 块级作用域
- let 取代 var
```javascript
'use strict';

if (true) {
  let x = 'hello';
}

for (let i = 0; i < 10; i++) {
  console.log(i);
}
```
- 在全局环境，不应该设置变量，只应设置常量。建议优先使用const
- 所有的函数都应该设置为常量
```javascript
// bad
var a = 1, b = 2, c = 3;

// good
const a = 1;
const b = 2;
const c = 3;

// best
const [a, b, c] = [1, 2, 3];

const add=(x,y)=>x+y;
```

### 字符串
- 静态字符串一律使用单引号或反引号，不使用双引号。动态字符串使用反引号。
```javascript
// bad
const a = "foobar";
const b = 'foo' + a + 'bar';

// acceptable
const c = `foobar`;

// good
const a = 'foobar';
const b = `foo${a}bar`;
const c = 'foobar';
```

### 解构赋值
- 使用数组成员对变量赋值时，优先使用解构赋值。
```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```
- 函数的参数如果是对象的成员，优先使用解构赋值。
```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {
}
```
- 如果函数返回多个值，优先使用对象的解构赋值，而不是数组的解构赋值。这样便于以后添加返回值，以及更改返回值的顺序
```javascript
// bad
function processInput(input) {
  return [left, right, top, bottom];
}

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
```

### 对象
- 多行定义的对象，最后一个成员以逗号结尾。
```javascript
// bad
const a = { k1: v1, k2: v2, };
const b = {
  k1: v1,
  k2: v2
};

// good
const a = { k1: v1, k2: v2 };
const b = {
  k1: v1,
  k2: v2,
};
```
- 对象尽量静态化，一旦定义，就不得随意添加新的属性。如果添加属性不可避免，要使用Object.assign方法
```javascript
// bad
const a = {};
a.x = 3;

// if reshape unavoidable
const a = {};
Object.assign(a, { x: 3 });

// good
const a = { x: null };
a.x = 3;
```
- 如果对象的属性名是动态的，可以在创造对象的时候，使用属性表达式定义
```javascript
// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```
- 对象的属性和方法，尽量采用简洁表达法，这样易于描述和书写
```javascript
var ref = 'some value';

// bad
const atom = {
  ref: ref,

  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  ref,

  value: 1,

  addValue(value) {
    return atom.value + value;
  },
};
```

### 数组
- 使用扩展运算符（...）拷贝数组
```javascript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```
- Array.from方法，将类似数组的对象转为数组。
```javascript
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```

### 函数
- 立即执行函数可以写成箭头函数的形式
```javascript
(() => {
  console.log('Welcome to the Internet.');
})();
```
- 那些需要使用函数表达式的场合，尽量用箭头函数代替
```javascript
// bad
[1, 2, 3].map(function (x) {
  return x * x;
});

// good
[1, 2, 3].map((x) => {
  return x * x;
});

// best
[1, 2, 3].map(x => x * x);
```
- 箭头函数取代Function.prototype.bind，不应再用self/_this/that绑定 this
```javascript
// bad
const self = this;
const boundMethod = function(...params) {
  return method.apply(self, params);
}

// acceptable
const boundMethod = method.bind(this);

// best
const boundMethod = (...params) => method.apply(this, params);
```
- 所有配置项都应该集中在一个对象，放在最后一个参数，布尔值不可以直接作为参数
```javascript
// bad
function divide(a, b, option = false ) {
}

// good
function divide(a, b, { option = false } = {}) {
}
```
- 不要在函数体内使用arguments变量，使用rest运算符（...）代替
```javascript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```
- 使用默认值语法设置函数参数的默认值
```javascript
// bad
function handleThings(opts) {
  opts = opts || {};
}

// good
function handleThings(opts = {}) {
  // ...
}
```

### Map
- 只有模拟现实世界的实体对象时，才使用Object。如果只是需要key: value的数据结构，使用Map结构。因为Map有内建的遍历机制
```javascript
let map = new Map(arr);

for (let key of map.keys()) {
  console.log(key);
}

for (let value of map.values()) {
  console.log(value);
}

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
```

### Class
- 总是用Class，取代需要prototype的操作；
```javascript
// bad
function Queue(contents = []) {
  this._queue = [...contents];
}
Queue.prototype.pop = function() {
  const value = this._queue[0];
  this._queue.splice(0, 1);
  return value;
}

// good
class Queue {
  constructor(contents = []) {
    this._queue = [...contents];
  }
  pop() {
    const value = this._queue[0];
    this._queue.splice(0, 1);
    return value;
  }
}
```
- 使用extends实现继承
```javascript
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}
inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function() {
  return this._queue[0];
}

// good
class PeekableQueue extends Queue {
  peek() {
    return this._queue[0];
  }
}
```

### 模块
- Module语法是JavaScript模块的标准写法，坚持使用这种写法。使用import取代require，使用export取代module.exports。
```javascript
// bad
const moduleA = require('moduleA');
const func1 = moduleA.func1;
const func2 = moduleA.func2;

// good
import { func1, func2 } from 'moduleA';

// commonJS的写法
var React = require('react');

var Breadcrumbs = React.createClass({
  render() {
    return <nav />;
  }
});

module.exports = Breadcrumbs;

// ES6的写法
import React from 'react';

class Breadcrumbs extends React.Component {
  render() {
    return <nav />;
  }
};

export default Breadcrumbs;
```
- 如果模块只有一个输出值，就使用export default，如果模块有多个输出值，就不使用export default，export default与普通的export不要同时使用
```javascript
// bad
const add=(x,y)=>x+y;
const PI=3.14;

export add;
export default PI;

// good
const add=(x,y)=>x+y;
const PI=3.14;

export add;
export PI;
```
- 不要在模块输入中使用通配符。因为这样可以确保你的模块之中，有一个默认输出（export default）。因为`import *`会忽略模块的默认值
```javascript
// bad
import * as myObject from './importModule';

// good
import myObject from './importModule';
```

> 东西很多，先做个大概了解，后期用到再做更加细致的了解。以上。
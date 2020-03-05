---
title: webpack-test-include-exclude
tags:
  - webpack
  - 工程化
categories:
  - 前端
date: 2020-03-05 12:19:25
---

# webpack test、include、exclude

## 介绍
- webpack `module rules`中的`test、include、exclude`都是针对处理`当前rule`的`loader`做范围限制的，换句话说是做**范围匹配**的
- `loader`会针对依赖图中的所有`module`运行匹配逻辑，如果匹配了，则用当前`loader`进行处理
- 三者可单独指定，也可同时指定
```js
// webpack.config.js

module.exports={

    module:{
    
        rules:[
            {
                test:/\.css$/, // test、include、exclude三者可单独指定，也可同时指定
                // inculde:path.join(__dirname,'./src')
                // inculde:path.join(__dirname,'./static')
                loader:'css-loader'
            }
        ]
    }
}
```

## 相关源码
- 与`test`、`include`、`exclude`相关的匹配规则生成源码，在`RuleSet.js`中
- `module.rules`一般我们会传递一个数组，所以其会逐条`序列化rule(normalizeRule)`
- 在`序列化rule`时，会检查是否既设置了`test、include、exclude`又设置了`resource`字段，`webpack`会报错，因为不允许同时设置这两个字段；[https://webpack.docschina.org/configuration/module/#rule-include](https://webpack.docschina.org/configuration/module/#rule-include)
- 如果未同时设置，则会利用`test、include、exclude`生成一个`condition对象`，然后使用`normalizeCondition`进行序列化，`normalizeCondition`函数总是返回一个`条件函数`
- 在`normalizeCondition`中，会判断`condition对象`的类型
    - 如果是`string`,则返回一个`条件函数`，此`条件函数`内部利用`indexOf`判断`module路径`是否以`condition`开头来匹配
    - 如果是`function`，则将传入的`function`做为`条件函数`返回
    - 如果为正则表达式，则对其的`test`方法先绑定上下文，然后将`test` 方法做为`条件函数`返回
    - 如果是`object`，则遍历`condition对象`的`keys`，调用`normalizeCondition`生成`条件函数`
        - 如果发现`condition对象`的`key`是`or、include、test`，则直接生成一个`条件函数`并`push`到预先定义好的`matchers`数组中
        - 如果`key`是`and`，则遍历`key`对应的`value`数组，生成一个由`条件函数`组成的数组`items`，并将`items`传入`andMatcher`函数，返回一个经过`andMatcher`包装的`条件函数`并将其`push`到`matchers`中；
            - 经过`andMathcer`包装后的`条件函数`，在执行时，要求入参必须让`items`中的所有`条件函数`都返回`true`才通过(且的关系)；相当于所有`条件函数`的返回值使用`&&`连接了
        - 如果`key`是`not、exclude`，则生成一个条件函数，传入`notMatcher`中，进而返回一个被`notMathcer`包装后的`条件函数`，再将包装后的`条件函数``push`到`matchers`中
            - 经过`notMathcer`包装后的`条件函数`，在执行时，要求入参必须让`条件函数`返回`false` 才通过(非的关系)；
    - 经过上面逻辑后，`mathers`数组将包含所有的`条件函数`，最后使用`andMatcher`函数，将`matchers`中保存的`条件函数`使用`&&`连接起来
```js
// RuleSet.js

"use strict";
module.exports = class RuleSet {
  // ...
  
  // 序列化rules
  static normalizeRules(rules, refs, ident) {
    
    // 数组，逐条序列化
    if (Array.isArray(rules)) {
      return rules.map((rule, idx) => {
        return RuleSet.normalizeRule(rule, refs, `${ident}-${idx}`);
      });
    } else if (rules) {
      return [RuleSet.normalizeRule(rules, refs, ident)];
    } else {
      return [];
    }
  }
  
  // 序列化rule
  static normalizeRule(rule, refs, ident) {
     // ...
    
    const newRule = {};
    let useSource;
    let resourceSource;
    let condition;
    
    // 传递了test或include或exclude
    if (rule.test || rule.include || rule.exclude) {
      // test、include、exclude和Rule.resource是互斥的，所以这里需要检查，如果发现都设置了，则报错提示
      checkResourceSource("test + include + exclude");
      
      condition = {
        test: rule.test,
        include: rule.include,
        exclude: rule.exclude
      };
      
      try {
        // 序列化condition
        newRule.resource = RuleSet.normalizeCondition(condition);
      } catch (error) {
        throw new Error(RuleSet.buildErrorMessage(condition, error));
      }
    }
    // ...
    return newRule;
  }
  // 序列化condition，总是返回一个条件函数
  static normalizeCondition(condition) {
    if (!condition) throw new Error("Expected condition but got falsy value");
    // 如果配置数据类型为 string，那么直接使用 indexOf 作为路径匹配规则
    if (typeof condition === "string") {
      return str => str.indexOf(condition) === 0;
    }
    // 如果为 function 函数，那么使用这个开发者自己定义的 function 作为路径匹配规则
    if (typeof condition === "function") {
      return condition;
    }
    // 如果为正则表达式，则绑定上下文后，使用正则来匹配
    if (condition instanceof RegExp) {
      return condition.test.bind(condition);
    }
    // 如果condition是数组，则生成orMatcher，orMathcer是一个高阶函数，会返回一个接收str的函数；其要求str满足items中的一项即可(或的关系)
    if (Array.isArray(condition)) {
      const items = condition.map(c => RuleSet.normalizeCondition(c));
      return orMatcher(items);
    }
    if (typeof condition !== "object")
      throw Error(
        "Unexcepted " +
          typeof condition +
          " when condition was expected (" +
          condition +
          ")"
      );
    // condition是一个对象，则遍历对象的keys
    const matchers = [];
    Object.keys(condition).forEach(key => {
      const value = condition[key];
      switch (key) {
        // 如果key是or、include、test，则根据value值生成一个条件函数
        case "or":
        case "include":
        case "test":
          if (value) matchers.push(RuleSet.normalizeCondition(value));
          break;
        // 如果key是and，则遍历value，生成多个条件函数，并用andMatcher返回一个函数，此函数要求传入的str必须满足items中的所有条件(且的关系)
        case "and":
          if (value) {
            const items = value.map(c => RuleSet.normalizeCondition(c));
            matchers.push(andMatcher(items));
          }
          break;
        // not或exclude，序列化value后，生成一个notMatcher
        case "not":
        case "exclude":
          if (value) {
            const matcher = RuleSet.normalizeCondition(value);
            matchers.push(notMatcher(matcher));
          }
          break;
        default:
          throw new Error("Unexcepted property " + key + " in condition");
      }
    });
    if (matchers.length === 0)
      throw new Error("Excepted condition but got " + condition);
    if (matchers.length === 1) return matchers[0];
    // 将上面生成好的条件函数数组，用且的关系连接起来；符合这条rule的module，就会用对应的loader做转换
    return andMatcher(matchers);
  }
};
function notMatcher(matcher) {
  return function(str) {
    return !matcher(str);
  };
}
function orMatcher(items) {
  return function(str) {
    for (let i = 0; i < items.length; i++) {
      if (items[i](str)) return true;
    }
    return false;
  };
}
function andMatcher(items) {
  return function(str) {
    for (let i = 0; i < items.length; i++) {
      if (!items[i](str)) return false;
    }
    return true;
  };
}
```

## 解释

### 单独指定一个字段
- 单独指定`test`
    - 只要`module`符合`test`指定的正则，就可以使用对应`loader`转换
- 单独指定`include`
    - 只会针对`include`中指定的模块，会使用对应`loader`转换，不管是否真的能转换
- 单独指定`exclude`
    - 会对`exclude`以外的所有模块，使用对应`loader`转换，不管是否真的能转换

### 指定两个字段
- `test + include`
    - 通过源码我们知道，在`normalizeCondition`时，遇到`test、include`都是拿对应字段的`value`直接生成一个`条件函数`，然后`push`进`matchers`中
    - 但要注意在`normalizeCondition`的最后，他们使用`andMatcher`连接了，所以`module`必须同时满足`test`和`include`，才能被对应`loader`转换
    - 所有可以说，同时指定`test`和`include`，二者是`test && include`的关系
- `test + exclude`
    - 因为`exclude`返回的`条件函数`是通过`notMatcher`包装的，所以要求`module的路径`必须不在`exclude`指定范围内，才能被对应`loader`转换
    - 所以同时指定`test + exclude`他们的关系是`test && (!exclude)`
- `include + exclude`
    - 和`test + exclude`类似
    - 同时指定时的关系是`include && (!exclude)`

### 同时指定三个字段
- 同时指定三个字段，跟同时指定两个字段有一定的相同；`test`和`include`是`&`的关系，`exclude`是`!`
- 所以同时指定时三者关系是`test && include && (!exclude)`


## 结论
- 起初，我以为`test`、`include`之间是`或`的关系取并集的操作，发现错了，原来他们取的是交集
- `test`、`include`、`exclude`确定范围时的关系是`test && include && (!exclude)`
- 只要指定了`exclude`，则其匹配的`module`必将不会被对应`loader`转换

## 最佳实践
- 根据官网上的文档有以下最佳实践
- 使用`test + include`，避免使用`exclude`
```js
 // 这里是匹配条件，每个选项都接收一个正则表达式或字符串
// test 和 include 具有相同的作用，都是必须匹配选项
// exclude 是必不匹配选项（优先于 test 和 include）
// 最佳实践：
// - 只在 test 和 文件名匹配 中使用正则表达式
// - 在 include 和 exclude 中使用绝对路径数组
// - 尽量避免 exclude，更倾向于使用 include
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],

```

## 相关项目
- [webpack-chain源码分析](https://github.com/BryanAdamss/webpack-chain-source)
- [使用webpack-chain编写webpack的实践](https://github.com/BryanAdamss/webpack4-template-use-webpack-chain)
- [《深入浅出Webpack》学习demos](https://github.com/BryanAdamss/webpack-dive-into-demos)

## 参考
- https://github.com/CommanderXL/Biu-blog/issues/30
- https://webpack.docschina.org/configuration/
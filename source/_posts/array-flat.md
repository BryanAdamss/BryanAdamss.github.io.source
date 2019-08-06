---
title: array-flat
tags:
  - 扁平化
  - flat
categories:
  - 前端
date: 2019-03-11 23:14:43
---

# 数组扁平化

## 前置知识

### 浅扁平化

- 浅扁平化就是只扁平化一层数组

### 深扁平化

- 实现无线层级的多维数组扁平化

## 方法

### 原始数组

```javascript
var needFlatArr = [1, 2, [3, 4], [[5, 6, 7]], [8, [9, 10], [[11]]]]
```

### 浅扁平化实现

1. `concat + apply`

```javascript
// 1.concat + apply - 浅扁平化
// concat在拼接数组时，若入参中有数组时，则遍历此数组每项(不会递归遍历，仅会遍历一层)，并将其依次拼接到尾部
// [].concat(1,[2,3],[4,5],[[6,7]]) 结果[1,2,3,4,5,[6,7]]
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat
function flatConcatApply(arr) {
  // 相当于[].concat.(arr[0],arr[1],arr[2],...)
  return Array.prototype.concat.apply([], arr)
}
```

2. `concat + reduce`

```javascript
// concat + reduce - 浅扁平化
function flatConcatReduce(arr) {
  return arr.reduce((a, b) => a.concat(b), [])
}
```

### 深扁平化实现

1. 递归实现 - 常规实现 - 深扁平化

```javascript
function flatRecursion(arr) {
  var result = []
  for (var i = 0, len = arr.length; i < len; i++) {
    if (Object.prototype.toString.call(arr[i]) === '[object Array]') {
      result = result.concat(flatRecursion(arr[i]))
    } else {
      result = result.concat(arr[i])
    }
  }
  return result
}
```

2. 递归实现 es6 - 常规实现 - 深扁平化

```javascript
function flatRecursionEs6(arr, result = []) {
  for (let item of arr) {
    if (Array.isArray(item)) {
      flatRecursionEs6(item, result)
    } else {
      result.push(item)
    }
  }
  return result
}
```

3. 递归实现 es6 - 简化 - 深扁平化 - 推荐

```javascript
const flatten = arr =>
  arr.reduce(
    (result, shouldFlatten) =>
      result.concat(
        Array.isArray(shouldFlatten) ? flatten(shouldFlatten) : shouldFlatten
      ),
    []
  )
```

4. 循环实现 - 循环调用浅扁平化函数实现 - lodash 思路 - 深扁平化

```javascript
function flatIterationShallowFlat(arr, deep = 1) {
  let result = arr
  while (deep--) {
    result = flatConcatApply(result)
  }
  return result
}
```

5. toString - 仅支持元素全部为 number 的场景 - 深扁平化

```javascript
function flatToString(arr) {
  return arr
    .toString()
    .split(',')
    .map(item => +item) // 转换为数字类型
}
```

6. 字符串过滤 - 将输入数组转换为字符串并删除所有括号（[]）并将输出解析为数组 - 深扁平化

```javascript
const flatStringFilter = arr =>
  JSON.parse(`[${JSON.stringify(arr).replace(/\[|]/g, '')}]`)
```

7. Generator - 定义生成器函数，并依次调用

```javascript
function* flatG(arr) {
  for (let item of arr) {
    if (Array.isArray(item)) {
      yield* flat(item)
    } else {
      yield item
    }
  }
}

function flatGenerator(arr) {
  let result = []
  for (let val of flatG(arr)) {
    result.push(val)
  }
  return result
}
```

8. 原生 flat - 兼容有问题 chrome 高版本不支持 - 深扁平化

```javascript
;[1, 2, [3]]
  .flat(1) // [1, 2, 3]
  [(1, 2, [3, [4]])].flat(2) // [1, 2, 3, 4]
  [(1, 2, [3, [4, [5]]])].flat(Infinity) // [1, 2, 3, 4, 5] 无线层级
```

9. 原生 flatMap - 等于 map + flat

```javascript
// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
;[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

## 参考

- [https://www.jianshu.com/p/b1fb3434e1f5](https://www.jianshu.com/p/b1fb3434e1f5)
- [http://www.cnblogs.com/front-end-ralph/p/4871332.html](http://www.cnblogs.com/front-end-ralph/p/4871332.html)
- [https://juejin.im/post/5c6b63f6f265da2da53ec057](https://juejin.im/post/5c6b63f6f265da2da53ec057)

---
title: flip
tags:
  - 动画
  - flip
categories:
  - 前端
date: 2019-02-21 22:41:40
---

# flip 动画技术

## 前言

- 尺寸、位置动画会触发重排，导致动画不流畅、动画启动慢
- flip 技术主要的目的是将常见的尺寸(`width、height`)、位置(`top、left...`)动画映射为性能开销小的`transform`动画

## 基本概念

- First: 元素的初始状态
- Last：元素的最终状态
- Invert：反转
- Play：开启动画

## 基本思路

1. First：获取元素的初始状态
2. Last：设置元素的状态为运动结束的最终状态
3. Invert：通过设置相反的`transform`值将元素从最终状态反转为初始状态
4. Play：设置元素`transition`运动属性(缓动、时长等)，再清空`transform`动画来启动动画
5. 可选的 Clean：动画结束，清除操作

## 例子

```javascript
// 获取初始位置
var first = el.getBoundingClientRect()
// 为元素指定一个样式，让元素在最终位置上
el.classList.add('end')

// 获取最终位置
var last = el.getBoundingClientRect()

// 如果有必要也可以对其他样式进行计算，但最好坚持只进行 transform 和 opacity 相关的计算

var invert = first.top - last.top
// 反转
el.style.transform = 'translateY(' + invert + 'px)'
// 等到下一帧，也就是其他所有的样式都已经被应用
requestAnimationFrame(function() {
  // 添加动画相关的设置
  el.classList.add('on-moving')
  // 开始动画
  el.style.transform = ''
})
// 结束时清理
el.addEventListener('transitionend', clean, false)
```

## 实际 demo

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Page Title</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      .box {
        width: 100px;
        height: 100px;
        border: 1px solid red;
        position: absolute;
      }
      .start {
        left: 100px;
        top: 50px;
      }
      .end {
        left: 800px;
        top: 400px;
      }
      .on-moving {
        transition: transform 0.3s linear;
      }
    </style>
  </head>
  <body>
    <div class="box start">box</div>
    <script>
      window.onload = function() {
        var $el = document.querySelector('.box')

        $el.addEventListener('click', boot, false)

        function boot(e) {
          var that = this

          // Last
          this.classList.add('end')

          // Invert
          this.style.transform = 'translate3d(-700px,-350px,0)' // -700为 100-800； -350为 50-400

          // Play
          requestAnimationFrame(function() {
            that.classList.add('on-moving')
            that.style.transform = ''
          })

          // Clean
          $el.addEventListener('transitionend', clean, false)

          function clean(e) {
            var target = e.target
            target.classList.remove('on-moving')
            target.removeEventListener('transitionend', clean, false)
          }
        }
      }
    </script>
  </body>
</html>
```

## 其他 demo

[http://vuejs.github.io/vue-animated-list/example.html](http://vuejs.github.io/vue-animated-list/example.html)
[https://codepen.io/davidkpiano/pen/305a618d4dd75cbe8423183c70d6a43e](https://codepen.io/davidkpiano/pen/305a618d4dd75cbe8423183c70d6a43e)

## 参考链接

- > [https://www.w3cplus.com/animation/high-performance-animations.html](https://www.w3cplus.com/animation/high-performance-animations.html) 
- > [https://www.w3cplus.com/javascript/animating-layouts-with-the-flip-technique.html](https://www.w3cplus.com/javascript/animating-layouts-with-the-flip-technique.html) 
- > [http://luchun.github.io/animating-layouts-with-the-flip-technique/](http://luchun.github.io/animating-layouts-with-the-flip-technique/)
- > [https://codepen.io/davidkpiano/pen/305a618d4dd75cbe8423183c70d6a43e](https://codepen.io/davidkpiano/pen/305a618d4dd75cbe8423183c70d6a43e)

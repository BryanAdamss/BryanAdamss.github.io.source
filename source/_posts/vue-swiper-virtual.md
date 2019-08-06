---
title: vue-swiper-virtual
tags:
  - vue
  - swiper
  - virtual
categories:
  - 前端
date: 2019-03-11 23:16:54
---

# 在 vue 中使用 swiper 的 virtual 特性

## swiper 介绍

- 前端开发中，经常会碰到轮播、翻页等需求。[swiper](https://www.swiper.com.cn/)就是用来解决此类需求的。
- 功能全、覆盖场景多，可适配移动端等特点，让它成为实现此类需求的最好选择。
- 它有专门的`vue`版本，[vue-awesome-swiper](https://github.com/surmon-china/vue-awesome-swiper)

## 背景

- 项目是基于`vue全家桶`开发的，有轮播需求，轮播的 slides 数量较多，slide 的 DOM 结构比较复杂。

## 问题

- 项目中使用的是`vue-awesome-swiper`，它基于`swiper`做了相关封装
- 使用时并没有使用`virtual slides`特性，初始化时会渲染所有`slide`，导致初始渲染速度较慢。在`DOM`结构复杂，slide 数量为 100 时，初始渲染大概需要 2~3s。
- 数据改变触发更新时，`vue-awesome-swiper`会频繁触发`update`操作。容易页面卡死。
- 类似问题[https://github.com/surmon-china/vue-awesome-swiper/issues/424](https://github.com/surmon-china/vue-awesome-swiper/issues/424)
- 见源码

```javascript
// vue-awesome-swiper/src/slide.vue
<template>
<div :class="slideClass">
<slot></slot>
</div>
</template>
<script>
export default {
    name: 'swiper-slide',
    data() {
        return {
             slideClass: 'swiper-slide'
        }
    },
    ready() {
        this.update()
    },
    mounted() {
        this.update()
        if (this.$parent && this.$parent.options && this.$parent.options.slideClass) {
            this.slideClass = this.$parent.options.slideClass
        }
    },
    updated() {
        this.update()
    },
    attached() {
        this.update()
    },
    methods: {
        update() {
            if (this.$parent && this.$parent.swiper) {
                this.$parent.update()
            }
        }
    }
}
</script>
```

-

## 使用 swiper 的 virtual 特性

- `virtual`官方介绍 -`虚拟Slide会在Dom结构中保持尽量少的Slide，只渲染当前可见的slide和前后的slide，而隐藏其他不可见的Slide，每次切换时就渲染新的Slide。当你的Slide很多的时候，尤其是Slide中有复杂的Dom树结构时，性能会有很大的提升。`
- 由于使用`vue-awesome-swiper`出现了一些性能问题，所以我们决定直接使用`swiper`版本并开启`virtual`特性（**注意使用 4.0 版本**）
- 中文文档中，并没有详细介绍如何在`vue`中使用`virtual`特性。
- 英文文档 [http://idangero.us/swiper/api/#virtual](http://idangero.us/swiper/api/#virtual)中详细介绍了如何在`vue、react`中使用`virtual`特性

```javascript
<template>
<!-- ... -->
<div class="swiper-container">
    <div class="swiper-wrapper">
        <!-- 必须在每个slide上设置left -->
        <!-- 勿直接将v-for的index做为key，会出现key重复的问题 -->
        <div class="swiper-slide"
            v-for="(slide, index) in virtualData.slides"
            :key="slide.id"
            :style="{left: `${virtualData.offset}px`}"
        >
        {{curIndex}}
        <br/>
        {{slide}}
        </div>
    </div>
</div>
<!-- ... -->
</template>
<script>

import Swiper from 'swiper/dist/js/swiper.esm.bundle' // 导入swiper
import 'swiper/dist/css/swiper.min.css' // 记得导入样式

export default {
    data() {
        return {
                curIndex:0, // 当前翻页索引
                // 声明virtualData，供swiper使用
                virtualData: {
                slides: [],
            },
        }
    },
    mounted() {
        Api.getSomeSlides().then(res=>{
            const self = this;
            const swiper = new Swiper('.swiper-container', {
            // ...
                virtual: {
                    slides: res.slides, // 需要添加的虚拟slide的内容
                    renderExternal(data) {
                        console.log(data)
                         // 返回一个经过swiper计算后的当前需要渲染的slides相关信息
                        // offset - slides偏移值
                        // from - 首个需要渲染的slide索引
                        // to - 最后一个需要渲染的slide索引
                        // slides - 需要渲染的slides
                        self.virtualData = data
                 },
                },
                on:{
                    slideChange(){
                        // 更新index
                        self.curIndex=this.activeIndex
                    }
                }
            });
        })
    },
};
</script>
```

- 使用后每次仅渲染当前页及前后页
- 每次渲染时，会重新触发`vue`的生命周期
- 其他使用基本和不使用`virtual`特性保持一致
- 若有`swiper`嵌套需求，可在子组件中直接实例化一个`swiper`并设置`nested`选项为`true`

## 总结

- `vue-awesome-swiper`有坑，会频繁调用`update`，导致一定性能问题
- 可直接使用`swiper`替换`vue-awesome-swiper`

## 相关链接

- [vue-awesome-swiper](https://github.com/surmon-china/vue-awesome-swiper)
- [https://github.com/surmon-china/vue-awesome-swiper/issues/424](https://github.com/surmon-china/vue-awesome-swiper/issues/424)
- [http://idangero.us/swiper/api/#virtual](http://idangero.us/swiper/api/#virtual)

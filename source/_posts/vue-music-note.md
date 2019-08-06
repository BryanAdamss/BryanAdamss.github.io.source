---
title: vue-music-note
tags:
  - vue
categories:
  - 前端
date: 2018-01-30 17:08:32
---

> 幕客网音乐app学习笔记


# 幕客网音乐app学习笔记

## vue-cli
- 初始化
    - `vue init webpack vue-music`，如果没有，会自动创建`vue-music`文件夹
- vue-cli的`runtime+compiler` vs `runtime`选择
    - 前者会包含编译器，适合在需要客户端编译模板的时候
    - 后者因为不包含编译器，最终编译出来的大小会比前者小，适用于基于`.vue`文件开发使用了`vue-loader`的情况。一般选择这个
- vue-cli的`eslint`模式
    - 使用`standard`
- 基本命令
    - 可在`package.json`的`scripts`中查询到
    - `npm start`、`npm run dev`启动开发服务器，默认在`localhost:8080`端口；若想在其他电脑查看，可配置`config/index.js`下的`host`字段为本机IP

## 修改vue-cli目录
- 默认目录结构
```
/vue-music
|---build/              // 存放webpack构建相关文件，一般不建议修改
|---config/             // 构建的一些配置参数，如果需要对构建过程做一些修改，可以配置此文件
|---node_modules/
|---src/                // 存放开发的主要文件，一般需要修改这里的目录结构
    assets/             // 存放一些需要webpack处理的静态文件
    components/         // 存放vue的.vue组件
    router/             // 存放路由文件
    App.vue             // 根组件，一般做整体布局用
    main.js             // 入口文件，负责渲染App.vue文件，并进行一些全局性的操作；全局性样式可在这里通过import 'test.scss'导入
|---static/             // 存放不需要webpack处理可以直接使用的静态文件
|---.babelrc            // babel配置
|---.editorconfig
|---.eslintrc.js        // eslint配置文件
|---.gitignore
|---.postcssrc.js       // postcss配置
|---index.html          // spa的主体html文件，需要修改一些head中代码，js代码会通过webpack自动注入
|---package.json        // 保存项目所有的依赖
|---README.md
```
- 修改后的目录，开发时主要修改的是`/src/`目录结构
```
/vue-music
|---build/
|---config/
|---node_modules/
|---src/
    api/                // 存放请求后台接口的js文件以及接口的通用配置参数文件
    base/               // 存放基础(通用)组件，以文件夹做区分
        base1/
            base1.vue
        base2/
            base2.vue
    common/             // 存放一些通用的js、img、字体等静态资源
    components/         // 存放业务组件，组件用不同的文件夹区分，组件自身用到的资源组织到一个文件夹中
        comp1/
            com1Bg.jpg
            comp1.vue
        comp2/
            comp2.vue
    
    router/
    store/              // 存放vuex相关文件
    sass/               // 存放sass样式文件
    App.vue
    main.js
|---static/
|---.babelrc
|---.editorconfig
|---.gitignore
|---.postcssrc.js
|---index.html
|---package.json
|---README.md
```
- `vue-cli`本身不自带样式预处理器，需要手动安装`npm install --save-dev node-sass sass-loader`
- `vue-cli`会配置路径别名`@`指向`/src/`文件夹，所以可以通过`@/components/comp1/comp1`来引用组件；也可以手动配置路径别名，减少书写量。

## vue-router
- 需要在`/src/router/index.js`中导入`vue-router`，并导出一个router的实例，最后在入口js(main.js)中导入并注册
```
// router/index.js
import Vue from 'vue';

import Router from 'vue-router';// 导入vue-router

..
import Recommend from 'components/recommend/recommend'
import Singer from 'components/singer/singer'
...

Vue.use(Router); // 必须使用use方法来注册第三方插件

export default new Router({// 导出一个vue-router实例
  routes: [{
    path: '/',
    redirect: '/recommend',// 没有匹配到的路径全部重定向到/recomend
  }, {
    path: '/recommend',// path一定是个路径，开头必须有/
    name: 'Recommend',
    component: Recommend,
    children: [{// 子路由
      path: ':id',// 传递的参数
      name: 'Disc',
      component: Disc
    }]
})


// src/main.js
import Vue from 'vue'
...
import router from './router'; // 导入
...


new Vue({
  el: '#app',
  router,// 注册
  store,
  render: h => h(App)
})
```
- `router-link`
```
   <router-link tag="div" class="tab-item" to="/recommend" active-class="is-cur"> // 渲染成div，跳转到/recommend路径，并在激活时添加is-cur样式类
      <span class="tab-link">推荐</span>
    </router-link>
```

## 使用babel
- `vue-cli`的babel转义不太清楚，为什么用了很多preset，难道不是用最新的`babel-preset-env`就可以了吗？疑惑中...
- 自己总结的babel使用经验
    - 使用`babel-preset-env`，并配置targets浏览器
    - 结合`useBuiltIns:usage`选项和`babel-polyfill`来完成按需polyfill
    - 参考 https://github.com/babel/babel/tree/master/packages/babel-preset-env

## fastclick
- 取消移动端点击300ms延迟
- 入口文件中导入，并绑定到body上
```
// src/main.js

...
import fastclick from 'fastclick'
...

fastclick.attach(document.body)
```
- 注意:fastclick会拦截click事件，如果一个组件内部需要监听click事件时，可在对应DOM节点上添加`needsclick`样式类告诉fastclick不拦截此DOM的click事件

## import/export
- 通过`import`可以导入各种模块
```
import Test from './components/test/test'; // 相对路径查找
import {getRecommend,getDiscList} from './api/recommend'; // 导入特定方法
import myDefault,* as myObj from './a'; // 导入所有方法(默认方法+其他方法)

import Vue from 'vue'; // 直接以名字开头，会在node_modules下查找
import createLogger from 'vuex/dist/logger'; // 可以在node_modules下按路径查找

// 配置了别名的情况下，可以直接使用名字开头，如果没配置别名则会在node_modules下查找
// webpack.base.conf.js
module.exports = {
 ...
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'components': resolve('src/components'),
    }
  },
}

import Test from 'components/test/test';
```

## jsonp
- 原理:script标签没有同源策略限制。通过动态创建script标签，然后src指向api地址，并附带一个回调函数名，后端用前台传来的回调名将需要的数据包裹并传回前台，前台会自动执行提前写好的回调。jsonp是get请求；
- 下面是jsonp的简单实现，具体可查看:https://github.com/BryanAdamss/SourceSave/blob/master/Plugins/js/vendor/11_jsonp.js
```javascript
(function(root, moduleName, factory) {
    if (typeof define === 'function' && define.amd) {
        define([], function() {
            return (root[moduleName] = factory(root));
        });
    } else {
        root[moduleName] = factory(root);
    }
}(typeof window !== "undefined" ? window : this, "c_jsonp", function(win) {
    if (typeof Object.assign != 'function') {
        Object.defineProperty(Object, "assign", {
            value: function assign(target, varArgs) {
                'use strict';
                if (target == null) {
                    throw new TypeError('Cannot convert undefined or null to object');
                }
                var to = Object(target);

                for (var index = 1; index < arguments.length; index++) {
                    var nextSource = arguments[index];

                    if (nextSource != null) {
                        for (var nextKey in nextSource) {
                            if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
                                to[nextKey] = nextSource[nextKey];
                            }
                        }
                    }
                }
                return to;
            },
            writable: true,
            configurable: true
        });
    }

    function formatParams(obj) { // 格式化参数
        var arr = [];
        for (var key in obj) {
            arr.push(encodeURIComponent(key) + '=' + encodeURIComponent(obj[key]));
        }
        return arr.join('&');
    }

    var configs = {
        url: '',
        data: {},
        callbackKey: 'callback', // 和后台约定的确定回调名的key值
        callbackName: ('jsonpCallback' + Math.random()).replace('.', ''), // 默认的随机回调名
        timeout: 3000, // 超时时间
        success: function(resp) { // 会在请求成功时，调用callbackName对应的函数中执行success(为什么不直接执行success是因为，需要在callbackName对应的函数中做一些其他操作，如删除script、清除定时器)
            console.log(resp);
        },
        error: function() { // 请求出错时，会直接调用
            throw new Error('请求出错!');
        },
    };


    function jsonp(options) {
        var settings = Object.assign({}, configs, options);
        if (!settings.url) {
            throw new Error('url必须传入');
            return;
        }

        var params = '';
        // 格式化参数
        if (settings.data) {
            settings.data[settings.callbackKey] = settings.callbackName;
            params = formatParams(settings.data);
        }

        var timer = null;
        // 超时处理
        if (settings.timeout) {
            timer = setTimeout(function() {
                win[settings.callbackName] = null;
                oHead.removeChild(oScript);
                settings.error && settings.error.call(null);
            }, settings.timeout);
        }

        // 创建一个全局回调函数，等待jsonp调用
        window[settings.callbackName] = function(resp) {
            oHead.removeChild(oScript);
            if (timer) {
                clearTimeout(timer);
            }
            window[settings.callbackName] = null;
            settings.success && settings.success.call(null, resp);
        };


        // 创建script并追加到页面上
        var oHead = document.querySelector('head');
        var oScript = document.createElement('script');

        var hasQuestionMark = settings.url.indexOf('?') < 0 ? false : true;

        var src = '';
        if (hasQuestionMark) {
            src = settings.url + params;
        } else {
            src = settings.url + '?' + params;
        }

        oScript.src = src;

        oHead.appendChild(oScript);

    }

    return jsonp;
}));
```
- `vue`中可以使用`webmodules/jsonp`模块(github下载)来实现jsonp请求
    - 默认是使用回调函数的形式
    - 改造成promise形式
    ```javascript
    import originJsonp from 'jsonp'; // 导入原来回调形式的jsonp
    
    export default function jsonp(url, data, option) {
    
      url += (url.indexOf('?') < 0 ? '?' : '&') + param(data);
    
      return new Promise((resolve, reject) => {// 返回一个promise实例，供后面使用then方法
        originJsonp(url, option, (err, data) => {
          if (!err) {
            resolve(data);
          } else {
            reject(err);
          }
        });
      });
    
    }
    
    function param(data) {// 格式化参数
      let url = '';
      for (var k in data) {
        let value = data[k] !== undefined ? data[k] : '';
        url += `&${k}=${encodeURIComponent(value)}`;
      }
      return url ? url.substring(1) : '';
    }
    
    ```

## 组件何时拉取数据？
- 组件拉取后台数据一般放在`created`钩子中，并将获取数据封装成一个`methods`
```javascript
import {getRecommend,getDiscList} from 'api/recommend'; // 导入实际拉取数据的方法
import {ERR_OK} from 'api/config';// 为保证语义化，可以将一些常见状态定义为常量

export default{
    created(){
        this._getRecommend();
        this._getDiscList();
    },
    methods:{
      _getRecommend(){
        getRecommend().then((res)=>{
          if(res.code===ERR_OK){
            this.recommends=res.data.slider;
          }
        });
      },
      _getDiscList(){
        getDiscList().then((res)=>{
          if(res.code===ERR_OK){
            this.discList=res.data.list;
          }
        });
      },
    }
};
```

## 封装轮播组件
- 底层使用`better-scroll`实现
- 基本原理:外部一个固定尺寸的容器(尺寸better-scroll会自动设置)，内部有一个超出尺寸的内容;动态设置内容的transform；
- 下面的slide是容器，sliderGroup是内容
```html
<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot>
      </slot>
    </div>
    <div class="dots">
      <span v-for="(item,index) in dots" class="dot" :class="{active:index===currentPageIndex}"></span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import {addClass} from 'common/js/dom';

  export default {
    data(){
      return {
        dots:[],
        currentPageIndex:0,// 指示当前dots
      };
    },
    props:{
      loop:{
        type:Boolean,
        default:true,
      },
      autoPlay:{
        type:Boolean,
        default:true,
      },
      interval:{
        type:Number,
        default:4000
      }
    },
    mounted(){
      // DOM准备好时，初始化；可以用this.$nextTick(),更推荐用setTimeout(()=>{},20);
      setTimeout(()=>{
        this._setSliderWidth();
        this._initDots();
        this._initSlider();
        if(this.autoPlay){
          this._play();
        }
      },20);

      window.addEventListener('resize',()=>{
        if(!this.slider){
          return ;
        }
        this._setSliderWidth(true);
        this.slider.refresh();
      });

    },
    destoryed(){
      // 组件切换后，应该停止timer
      clearTimeout(this.timer);
    },
    methods:{
      _setSliderWidth(isResize){
        this.children=this.$refs.sliderGroup.children;

        let width=0;
        let sliderWidth=this.$refs.slider.clientWidth;

        for(let i=0;i<this.children.length;i++){
          let child =this.children[i];
          // 我们不应该要求传入的slot是特定样式的，应该我们主动添加样式，这样可以降低和外部的耦合
          addClass(child,'slider-item');

          child.style.width=sliderWidth+'px';
          width+=sliderWidth;
        }
        
        // 因为better-scroll在循环时，会在前后自动复制一个，所以总宽度需要加2
        if(this.loop && !isResize){
          width+=2*sliderWidth;
        }
    
        // 设置内容的宽度
        this.$refs.sliderGroup.style.width=width + 'px';
      },
      _initDots(){
        // dots为一个只有长度的空数组，方便渲染dots
        // this.children在_setSliderWidth中已经声明
        this.dots=new Array(this.children.length);
      },
      _initSlider(){
        this.slider=new BScroll(this.$refs.slider,{
          scrollX: true,// 横向滚动
          scrollY: false,
          momentum: false,// 动量动画
          snap: {
            loop: true,// 循环
            threshold: 0.1,
            speed:400
          },
        });
        
        // 监听better-scroll派发的scrollEnd事件，更新currentPageIndex
        this.slider.on('scrollEnd',()=>{
          let pageIndex=this.slider.getCurrentPage().pageX;

          // 循环播放会在头尾复制一个，所以index需要-1
          if(this.loop){
            pageIndex-=1;
          }

          this.currentPageIndex=pageIndex;

          if(this.autoPlay){
            clearTimeout(this.timer);
            this._play();
          }
        });
      },
      _play(){
        // 要滚动到的索引值
        let pageIndex=this.currentPageIndex+1;
        
        // 循环，因为存在头尾复制，所以需要再+1
        if(this.loop){
          pageIndex+=1;
        }

        this.timer=setTimeout(()=>{
          // 横向滚动到pageIndex页
          this.slider.goToPage(pageIndex,0,400);
        },this.interval);
      },
    }
  }
</script>
```

## keep-alive
- 用来缓存不活动的组件
```html
<keep-alive>
    <router-view></router-view>
</keep-alive>
```
- 和`transition`组件使用时，要放在`transition`内部
```html
<transition>
    <keep-alive>
        <router-view></router-view>
    </keep-alive>
</transition>
```

## 后端接口代理
- 有些接口会在请求时配置请求头的`Host`和`Referer`来限制随意访问，纯前端无法直接绕过。
- 因为`vue-cli`使用的`express`框架，可以借助`express`框架的`Router`来实现后端接口反向代理工作
- 正反向代理
    - 正向代理
        - 代理人代理的是客户端(VPN)，代理人充当的是客户端，负责接受服务器传来的数据
    - 反向代理
        - 代理人代理的是服务端(负载均衡,其实就是服务器内容分发)，代理人充当的是服务端，提供数据给前台
        - 为何反向代理可以实现跨域请求
            - 同源策略是浏览器的安全策略，不是HTTP协议的一部分。服务器端调用HTTP接口只是使用HTTP协议，不会执行JS脚本，不需要同源策略，也就没有跨越问题。
    - 通俗易懂解释可参考
        - https://www.aliyun.com/jiaocheng/21099.html
        - http://blog.csdn.net/zhanghanboke/article/details/77488894
```javascript
const app = express()

const apiRoutes = express.Router();

apiRoutes.get('/getDiscList', function (req, res) {
  var url = 'https://c.y.qq.com/splcloud/fcgi-bin/fcg_get_diss_by_tag.fcg'
  axios.get(url, {
    headers: {
      // 配置refer、host
      referer: 'https://c.y.qq.com/',
      host: 'c.y.qq.com'
    },
    params: req.query
  }).then((response) => {
    res.json(response.data)
  }).catch((e) => {
    console.log(e)
  })
});

app.use('/api', apiRoutes); // 所有/api下的请求都会由后台发送给远程服务器，成功后返回数据给后台，后台再传给前台(代理人充当了远程服务器的角色，负责提供数据给前台)
```

## 封装scroll组件
- 底层使用`better-scroll`
- 注意:better-scroll的可滚动距离是自动计算r的，所以初始化一定要在DOM渲染好后 
```html
<template>
  <div ref="wrapper">
    <slot></slot>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'

  export default{
    props: {
      // 如何派发scroll事件(0默认值，不派发；1屏幕滑动超过一定时间后派发；2实时派发；3实时派发，而且在缓动时也派发)
      probeType: {
        type: Number,
        default:1
      },
      // 是否主动派发点击事件
      click: {
        type: Boolean,
        default: true
      },
      // 在低版本的better-scroll中需要手动监听数据的变化并重新初始化组件，但高版本的已经不需要手动监听data了
      data: {
        type: Array,
        default: null
      },
      // 是否监听滚动
      listenScroll:{
        type:Boolean,
        default:false
      },
      // 是否派发滚动到底部事件
      pullup:{
        type:Boolean,
        default:false
      },
      // 是否派发beforeScroll事件
      beforeScroll:{
        type:Boolean,
        default:false
      }
    },
    mounted() {
      // DOM准备好时初始化组件
      setTimeout(()=> {
        this._initScroll();
      }, 20)
    },
    methods: {
      _initScroll(){
        if (!this.$refs.wrapper) {
          return;
        }
        // 创建一个scroll
        this.scroll = new BScroll(this.$refs.wrapper, {
          probeType: this.probeType,
          click: this.click
        });

        // 派发滚动事件
        if (this.listenScroll) {
          let me = this;
          this.scroll.on('scroll', (pos) => {
            me.$emit('scroll', pos);
          });
        }

        // 派发滚动到底部事件
        if(this.pullup){
          this.scroll.on('scrollEnd',()=>{
            if(this.scroll.y<=(this.scroll.maxScrollY+50)){
              this.$emit('scrollToEnd');
            }
          });
        }

        // 派发beforeScroll事件
        if(this.beforeScroll){
          this.scroll.on('beforeScrollStart',()=>{
            this.$emit('beforeScroll');
          });
        }
      },
      // 代理并暴露一些常用接口
      enable(){
        this.scroll && this.enable();
      },
      disable(){
        this.scroll && this.disable();
      },
      refresh(){
        console.log('refresh');
        this.scroll && this.scroll.refresh();
      },
      scrollTo(){
        this.scroll&&this.scroll.scrollTo.apply(this.scroll,arguments);
      },
      scrollToElement(){
        this.scroll&&this.scroll.scrollToElement.apply(this.scroll,arguments);
      }
    }
  }
</script>
```
- 注意:fastclick会拦截click事件，如果一个组件内部需要监听click事件时，可在对应DOM节点上添加`needsclick`样式类告诉fastclick不拦截此DOM的click事件

## lazy-load
- 可以使用`vue-lazyload`实现
```html
// main.js
import VueLazyLoad from 'vue-lazyload';

Vue.use(VueLazyLoad,{
  loading:require('common/image/default.png')
});

// 使用时 
<div class="icon">
    <img v-lazy="item.imgurl" alt="" width="60" height="60">
</div>
```

## loading组件
```html
<template>
  <div class="loading">
    <img width="24" height="24" src="./loading.gif">
    <p class="desc">{{title}}</p>
  </div>
</template>
<script type="text/ecmascript-6">
  export default {
    props: {
      title: {
        type: String,
        default: '正在载入...'
      }
    }
  }
</script>
<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .loading
    width: 100%
    text-align: center
    .desc
      line-height: 20px
      font-size: $font-size-small
      color: $color-text-l
</style>
```

## vuex
```javascript
const store = new Vuex.Store({
  state: {
    // 需要全局共享的数据
    count: 1
  },
  mutations: {
    // 注册一个increment mutations
    increment (state,payload) {
      // 变更状态
      state.count += payload.amount
    }
  },
  actions: {
    incrementAsync (context,payload) {
      setTimeout(() => {
        context.commit('increment')
      }, 1000)
    }
  }
})
```
- `states`
    - 需要全局共享的数据
- `getters`
    - 类似计算属性，可以用来访问基于state派生出的一些state，主要用来访问states
- `mutations`
    - 很类似事件，只能通过提交mutations来改变state(方便开发工具跟踪)，主要用来设置states
    - 提交mutaions是改变state的唯一方式
    - 只能是同步任务，异步任务需要`actions`来完成
    - 每一个mutation，store会传入state和可选的payload
- `commit`
    - 用来提交`mutaions`
    - 可以传递额外的参数`payload`给`mutaions`
    ```javascript
    store.commit('increment', {
      amount: 10
    })
    ```
- `actions`
    - 主要用来完成异步任务;需要请求后台的任务全部放在`actions`中
    - `action`不会直接变更状态，而是通过提交mutation来改变state，
    - 每个action，store会传入一个context和可选的payload，context包含store实例的所有方法和属性。因此你可以调用 `context.commit` 提交一个 `mutation`，或者通过 `context.state` 和 `context.getters` 来获取 `state` 和 `getters`
- `dispatch`
    - 用来分发(提交)action
    - 可以传递额外的参数(`payload`)给`actions`
    ```javascript
    store.dispatch('incrementAsync', {
      amount: 10
    })
    ```
- 辅助函数
```javascript
// mapState、mapGetters、mapMutations、mapActions，都是返回一个对象，可以使用扩展预算符解析出来
new Vue({
    el:'#app',
    store,
    computed: {
        localComputed () {},
        // 使用对象展开运算符将此对象混入到外部对象中
        ...mapState([
            count,// 将this.count 映射为 store.state.count
        ]),
        ...mapGetters([
            'doneTodosCount',// 将this.doneTodosCount 映射为 store.getters.doneTodosCount
            'anotherGetter',// 将this.anotherGetter 映射为 store.getters.anotherGetter
        ])
    },
    methods: {
        ...mapMutations([
            'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
        
            // `mapMutations` 也支持载荷：
            'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
        ]),
        ...mapMutations({
            add: 'SET_INCREMENT' // 将 `this.add()` 映射为 `this.$store.commit('SET_INCREMENT')`
        }),
        ...mapActions([
            'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
        
            // `mapActions` 也支持载荷：
            'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
        ]),
        ...mapActions({
            add: 'increment' // 将 `this.add()` 映射为 `this.$store.dispatch('increment')`
        })
    }
});
```
- 实际项目目录
```javascript
|---store/
    actions.js          // 保存所有异步任务
    getters.js          // 保存所有getters
    index.js            // 导入state、getter...来实例化vuex
    mutations-type.js   // 保存所有mutations类型
    mutations.js        // 实际的mutation函数
    state.js            // 所有需要共享的数据

// state.js
// 首先确定哪些数据需要vuex来管理，并给定默认值
import {playMode} from 'common/js/config'
import {loadSearch,loadPlay,loadFavorite} from 'common/js/cache'

// state只放一些基础数据，派生的数据可放在getter中，如currentSong可通过palyList和currentIndex计算而来
const state = {
  singer: {},
  playing: false,
  fullScreen: false,
  playlist: [],
  sequenceList: [],
  mode: playMode.sequence,
  currentIndex: -1,
  disc:{},
  topList:{},
  searchHistory:loadSearch(),
  playHistory:loadPlay(),
  favoriteList:loadFavorite(),
};

export default state;// 导出state对象

// mutation-types.js
// 确定针对state的修改
export const SET_SINGER = 'SET_SINGER'

export const SET_PLAYING_STATE = 'SET_PLAYING_STATE'

export const SET_FULL_SCREEN = 'SET_FULL_SCREEN'

export const SET_PLAYLIST = 'SET_PLAYLIST'

export const SET_SEQUENCE_LIST = 'SET_SEQUENCE_LIST'

export const SET_PLAY_MODE = 'SET_PLAY_MODE'

export const SET_CURRENT_INDEX = 'SET_CURRENT_INDEX'

export const SET_DISC = 'SET_DISC'

export const SET_TOP_LIST = 'SET_TOP_LIST'

export const SET_SEARCH_HISTORY = 'SET_SEARCH_HISTORY'

export const SET_PLAY_HISTORY = 'SET_PLAY_HISTORY'

export const SET_FAVORITE_LIST = 'SET_FAVORITE_LIST'

// mutations.js
// 根据mutaion-types编写具体的mutation来设置state
import * as types from './mutation-types'

const mutations = {
  [types.SET_SINGER](state, singer) {
    state.singer = singer
  },
  [types.SET_PLAYING_STATE](state, flag) {
    state.playing = flag
  },
  [types.SET_FULL_SCREEN](state, flag) {
    state.fullScreen = flag
  },
  [types.SET_PLAYLIST](state, list) {
    state.playlist = list
  },
  [types.SET_SEQUENCE_LIST](state, list) {
    state.sequenceList = list
  },
  [types.SET_PLAY_MODE](state, mode) {
    state.mode = mode
  },
  [types.SET_CURRENT_INDEX](state, index) {
    state.currentIndex = index
  },
  [types.SET_DISC](state, disc) {
    state.disc = disc
  },
  [types.SET_TOP_LIST](state, topList) {
    state.topList = topList
  },
  [types.SET_SEARCH_HISTORY](state, history) {
    state.searchHistory = history
  },
  [types.SET_PLAY_HISTORY](state, history) {
    state.playHistory = history
  },
  [types.SET_FAVORITE_LIST](state, list) {
    state.favoriteList = list
  },
}

export default mutations; // 导出mutations对象

// getters.js
// 编写getter来读取state
export const singer = state => state.singer

export const playing = state=>state.playing

export const fullScreen = state=>state.fullScreen

export const playlist = state=>state.playlist

export const sequenceList = state=>state.sequenceList

export const mode = state=>state.mode

export const currentIndex = state=>state.currentIndex

// 通过playlist、currentIndex计算而来
export const currentSong = (state)=> {
  return state.playlist[state.currentIndex] || {};
}

export const disc = state=>state.disc

export const topList = state=>state.topList

export const searchHistory = state=>state.searchHistory

export const playHistory = state=>state.playHistory

export const favoriteList = state=>state.favoriteList

// actions.js
// 异步操作
import * as types from './mutation-types'
import { playMode } from 'common/js/config'
import { shuffle } from 'common/js/util'
import { saveSearch, deleteSearch, clearSearch, savePlay, saveFavorite, deleteFavorite } from 'common/js/cache'

function findIndex(list, song) {
  return list.findIndex((item) => {
    return item.id === song.id;
  })
}

export const selectPlay = function({ commit, state }, { list, index }) {
  commit(types.SET_SEQUENCE_LIST, list);

  if (state.mode === playMode.random) {
    let randomList = shuffle(list);
    commit(types.SET_PLAYLIST, randomList);
    index = findIndex(randomList, list[index]);
  } else {
    commit(types.SET_PLAYLIST, list);
  }

  commit(types.SET_CURRENT_INDEX, index);
  commit(types.SET_FULL_SCREEN, true);
  commit(types.SET_PLAYING_STATE, true);
};

export const randomPlay = function({ commit }, { list }) {
  commit(types.SET_PLAY_MODE, playMode.random);
  commit(types.SET_SEQUENCE_LIST, list);

  let randomList = shuffle(list);
  commit(types.SET_PLAYLIST, randomList);

  commit(types.SET_CURRENT_INDEX, 0);
  commit(types.SET_FULL_SCREEN, true);
  commit(types.SET_PLAYING_STATE, true);
};

export const insertSong = function({ commit, state }, song) {
  let playlist = state.playlist.slice();
  let sequenceList = state.sequenceList.slice();
  let currentIndex = state.currentIndex;
  // 记录当前歌曲
  let currentSong = playlist[currentIndex];

  // 修改playlist
  // 查询当前列表中是否有待插入的歌曲并返回其索引
  let fpIndex = findIndex(playlist, song);
  // 因为是插入歌曲，所以索引要+1
  currentIndex++;
  // 插入这首歌当当前索引位置
  playlist.splice(currentIndex, 0, song);
  // 如果已经包含这首歌曲
  if (fpIndex > -1) {
    // 如果当前插入的索引大于列表中的序号
    if (currentIndex > fpIndex) {
      playlist.splice(fpIndex, 1);
      currentIndex--;
    } else {
      playlist.splice(fpIndex + 1, 1);
    }
  }

  // 修改sequenceList
  let currentSIndex = findIndex(sequenceList, currentSong) + 1;
  let fsIndex = findIndex(sequenceList, song);
  sequenceList.splice(currentSIndex, 0, song);
  if (fsIndex > -1) {
    if (currentSIndex > fsIndex) {
      sequenceList.splice(fsIndex, 1);
    } else {
      sequenceList.splice(fsIndex + 1, 1);
    }
  }

  commit(types.SET_PLAYLIST, playlist);
  commit(types.SET_SEQUENCE_LIST, sequenceList);
  commit(types.SET_CURRENT_INDEX, currentIndex);
  commit(types.SET_FULL_SCREEN, true);
  commit(types.SET_PLAYING_STATE, true);
};

export const saveSearchHistory = function({ commit }, query) {
  commit(types.SET_SEARCH_HISTORY, saveSearch(query));
};

export const deleteSearchHistory = function({ commit }, query) {
  commit(types.SET_SEARCH_HISTORY, deleteSearch(query));
};

export const clearSearchHistory = function({ commit }) {
  commit(types.SET_SEARCH_HISTORY, clearSearch());
};

export const deleteSong = function({ commit, state }, song) {
  let playlist = state.playlist.slice()
  let sequenceList = state.sequenceList.slice()
  let currentIndex = state.currentIndex

  let pIndex = findIndex(playlist, song)
  playlist.splice(pIndex, 1)

  let sIndex = findIndex(sequenceList, song)
  sequenceList.splice(sIndex, 1)

  if (currentIndex > pIndex || currentIndex === playlist.length) {
    currentIndex--
  }

  commit(types.SET_PLAYLIST, playlist)
  commit(types.SET_SEQUENCE_LIST, sequenceList)
  commit(types.SET_CURRENT_INDEX, currentIndex)

  if (!playlist.length) {
    commit(types.SET_PLAYING_STATE, false)
  } else {
    commit(types.SET_PLAYING_STATE, true)
  }
}

export const deleteSongList = function({ commit }) {
  commit(types.SET_CURRENT_INDEX, -1)
  commit(types.SET_PLAYLIST, [])
  commit(types.SET_SEQUENCE_LIST, [])
  commit(types.SET_PLAYING_STATE, false)
}

export const savePlayHistory = function({ commit }, song) {
  commit(types.SET_PLAY_HISTORY, savePlay(song))
}

export const saveFavoriteList = function({ commit }, song) {
  commit(types.SET_FAVORITE_LIST, saveFavorite(song))
}

export const deleteFavoriteList = function({ commit }, song) {
  commit(types.SET_FAVORITE_LIST, deleteFavorite(song))
}


// index.js
// 入口文件，实例化vuex

import Vue from 'vue'
import Vuex from 'vuex'
import * as actions from './actions'
import * as getters from './getters'
import state from './state'
import mutations from './mutations'
import createLogger from 'vuex/dist/logger';// 调试工具，可以打印出log

Vue.use(Vuex);// vuex是个插件，需要use

const debug = process.env.NODE_ENV !== 'production';

// 导出一个store实例，供main.js使用
export default new Vuex.Store({
  actions,
  getters,
  state,
  mutations,
  strict: debug,// strict模式会针对不通过commit提交mutation报错
  plugins: debug ? [createLogger()] : [],// 使用打印log插件
})

// main.js
// 导入vuex的stroe实例，并注入到vue根组件中

import store from './store'
...

new Vue({
  el: '#app',
  router,
  store,// 导入vuex的stroe实例
  render: h => h(App)
});
```

## js prefix
```javascript
// common/js/dom.js
// 能力检测，判断是哪种前缀
let elementStyle = document.createElement('div').style;
// 利用IIFE得到支持的前缀
let vendor = (()=> {
  // 利用transform做能力检测，来判断支持哪种前缀
  let transformNames = {
    webkit: 'webkitTransform',
    Moz: 'MozTransform',
    O: 'OTransform',
    ms: 'msTransform',
    standard: 'transform'
  };

  for (let key in transformNames) {
    // 支持某种前缀则直接返回
    if (elementStyle[transformNames[key]] !== undefined) {
      return key;
    }
  }

  // 如果没有匹配，则返回false
  return false;
})();

// 添加前缀
export function prefixStyle(style) {
  if (vendor === false) {
    return false;
  }
  // 支持标准，则直接返回
  if (vendor === 'standard') {
    return style;
  }
  // 否则返回prefix后的字符串
  return vendor + style.charAt(0).toUpperCase() + style.substr(1);
}

// 实际调用
// musicList.js
...
import {prefixStyle} from 'common/js/dom';
...

const transform=prefixStyle('transform');
const backdrop=prefixStyle('backdrop-filter');

...

watch:{
      scrollY(newY){
        ...
        this.$refs.layer.style[transform]=`translate3d(0,${translateY}px,0)`;
        ...
        this.$refs.filter.style[backdrop]=`blur(${blur}px)`;
        ...
      }
}
```

## 用js创建animation动画
- 使用`create-keyframe-animation`
```javascript
<!-- 监听一些钩子 -->
<transition name="normal"
      @enter="enter"
      @after-enter="afterEnter"
      @leave="leave"
      @after-leave="afterLeave"
>
...      
</transition>

// 样式
&.normal-enter-active, &.normal-leave-active
    transition: all 0.4s
    .top, .bottom
        transition: all 0.4s cubic-bezier(0.86, 0.18, 0.82, 1.32)
&.normal-enter, &.normal-leave-to
    opacity: 0
    .top
      transform: translate3d(0, -100px, 0)
    .bottom
      transform: translate3d(0, 100px, 0)

// js
...
import animations from 'create-keyframe-animation'
...

methods:{
    enter(el,done){
        const {x,y,scale}=this._getPosAndScale();
        let animation={
            0:{
              transform:`translate3d(${x}px,${y}px,0) scale(${scale})`,
            },
            60:{
              transform:`translate3d(0,0,0) scale(1.1)`,
            },
            100:{
              transform:`translate3d(0,0,0) scale(1)`,
            }
        };
        // 注册一个move动画
        animations.registerAnimation({
            name:'move',
            animation,
            presets:{
              duration:400,
              easing:'linear'
            }
        });
        // 在cdWrapper上调用move动画，结束时一定要调用done
        animations.runAnimation(this.$refs.cdWrapper,'move',done);
    },
    afterEnter(){
        // 运动结束注销move
        animations.unregisterAnimation('move');
        this.$refs.cdWrapper.style.animation = '';
    },
    leave(el,done){
        // leave动画用普通的过渡动画实现即可
        this.$refs.cdWrapper.style.transition='all .3s';
        const {x,y,scale}=this._getPosAndScale();
        this.$refs.cdWrapper.style[transform]=`translate3d(${x}px,${y}px,0)`;
        // 结束时，调用done
        this.$refs.cdWrapper.addEventListener('transitionend',done);
    },
    afterLeave(){
        this.$refs.cdWrapper.style.transition = ''
        this.$refs.cdWrapper.style[transform] = ''
    }
}
```

## 利用svg实现圆形进度
- 利用`dasharray`配合`stroke-dashoffset`来实现进度
```javascript
<template>
  <div class="progress-circle">
    <svg :width="radius" :height="radius" viewBox="0 0 100 100" version="1.1" xmlns="http://www.w3.org/2000/svg">
      <circle class="progress-background" r="50" cx="50" cy="50" fill="transparent"/>
      <circle class="progress-bar" r="50" cx="50" cy="50" fill="transparent" :stroke-dasharray="dashArray"
              :stroke-dashoffset="dashOffset"/>
    </svg>
    <slot></slot>
  </div>
</template>

<script type="text/ecmascript-6">
 export default{
   props:{
     radius:{
       type:Number,
       default:100
     },
     percent:{
       type:Number,
       default:0
     }
   },
   data(){
     return {
       dashArray:Math.PI * 100
     }
   },
   computed:{
     dashOffset(){
       return (1-this.percent)*this.dashArray
     }
   }
 }
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import "~common/stylus/variable"

  .progress-circle
    position: relative
    circle
      stroke-width: 8px
      transform-origin: center
      &.progress-background
        transform: scale(0.9)
        stroke: $color-theme-d
      &.progress-bar
        transform: scale(0.9) rotate(-90deg)
        stroke: $color-theme
</style>
```

## 利用mixin复用组件选项
- 当组件的选项类似是，可以将其抽取成一个公用mixin，然后在组件`mixins`字段中引入即可
```javascript
// mixins.js
import {mapGetters, mapMutations, mapActions} from 'vuex'
...

export const playlistMixin = {
  // 下面定义的选项将在不同组件中复用
  computed: {
    ...mapGetters([
      'playlist'
    ])
  },
  mounted() {
    this.handlePlaylist(this.playlist)
  },
  activated() {
    this.handlePlaylist(this.playlist)
  },
  watch: {
    playlist(newVal) {
      this.handlePlaylist(newVal)
    }
  },
  methods: {
    handlePlaylist() {
      throw new Error('component must implement handlePlaylist method')
    }
  }
};


// playList.vue
...
 import {playerMixin} from 'common/js/mixin'
...

export default {
    mixins:[playerMixin],
    ...
}
```

## 编译打包
- 直接使用`npm run build`

## 打包优化
- 路由懒加载
    - vue官方推荐使用webpack的ensure(这是webpack1.x版本的，2.x推荐使用import)
    ```javascript
    import Vue from 'vue'
    import Router from 'vue-router'
    // import Recommend from 'components/recommend/recommend'
    // import Singer from 'components/singer/singer'
    // import Rank from 'components/rank/rank'
    // import Search from 'components/search/search'
    // import SingerDetail from 'components/singer-detail/singer-detail'
    // import Disc from 'components/disc/disc'
    // import TopList from 'components/top-list/top-list'
    // import UserCenter from 'components/user-center/user-center'
    
    Vue.use(Router)
    
    // 使用import动态加载组件，并在成功后，resolve组件传递给路由
    const Recommend = (resolve) => {
      import('components/recommend/recommend').then((module) => {
        resolve(module)
      })
    }
    
    const Singer = (resolve) => {
      import('components/singer/singer').then((module) => {
        resolve(module)
      })
    }
    
    const Rank = (resolve) => {
      import('components/rank/rank').then((module) => {
        resolve(module)
      })
    }
    
    const Search = (resolve) => {
      import('components/search/search').then((module) => {
        resolve(module)
      })
    }
    
    const SingerDetail = (resolve) => {
      import('components/singer-detail/singer-detail').then((module) => {
        resolve(module)
      })
    }
    
    const Disc = (resolve) => {
      import('components/disc/disc').then((module) => {
        resolve(module)
      })
    }
    
    const TopList = (resolve) => {
      import('components/top-list/top-list').then((module) => {
        resolve(module)
      })
    }
    
    const UserCenter = (resolve) => {
      import('components/user-center/user-center').then((module) => {
        resolve(module)
      })
    }
    
    export default new Router({
      routes: [{
        path: '/',
        redirect: '/recommend'
      }, {
        path: '/recommend',
        name: 'Recommend',
        component: Recommend,
        children: [{
          path: ':id',
          name: 'Disc',
          component: Disc
        }]
      },
        {
          path: '/singer',
          name: 'Singer',
          component: Singer,
          children: [{
            path: ':id',
            name: 'SingerDetail',
            component: SingerDetail
          }]
        }, {
          path: '/rank',
          name: 'Rank',
          component: Rank,
          children:[{
            path:':id',
            name: 'TopList',
            component:TopList
          }]
        }, {
          path: '/search',
          name: 'Search',
          component: Search,
          children: [{
            path: ':id',
            name: 'SingerDetail',
            component: SingerDetail
          }]
        },{
          path:'/user',
          name:'User',
          component:UserCenter
        }
      ]
    })
    ```

## 移动端调试工具vConsole
- 在代码**入口文件main.js**中导入并实例化
- vconsole是不需要调用的，所以使用特定注释规避掉eslint检查
- 上线时，记得清除掉vConsole
```javascript
// main.js
...
import VConsole from 'vconsole'
...

// vconsole是不需要调用的，所以使用特定注释规避掉eslint检查
/* eslint-disable no-unused-vars */
var vConsole=new VConsole();
console.log('test');

...
/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```

## 抓包工具
- win上使用`fiddler`工具
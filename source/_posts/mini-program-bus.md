---
title: mini-program-bus
tags:
  - 小程序
categories:
  - 前端
date: 2018-03-15 13:49:47
---

> 最近在做一个关于公交查询的小程序
> 第一次接触小程序，有很多不熟悉的地方，特在此做个记录

# 微信小程序-公交查询笔记

## 注册小程序
- 可以参照官网的教程https://mp.weixin.qq.com/debug/wxadoc/introduction/index.html?t=1521093163
- 需要注意的是注册邮箱必须是未注册过公众号、个人微信号的邮箱才行

## 目录结构
- ![](catalog.png)
- `app.js`用来注册app程序的，有对应的生命周期钩子，可以在其中做一些初始化工作，app会在整个程序运行的生命周期内可访问，所以一些简单的数据传递、共享，可以通过在app.js中挂载全局变量的形式完成。
- `app.json`用来配置app程序
    - ![](app_json.png)
    - `pages`字段用来指定总共有哪些页面，最上面的是程序启动页；
        - 微信开发工具保存时，会自动刷新并返回到启动页，所以在开发非启动页页面时，可以通过将非启动页调整到`pages`字段最上面，来提高开发效率。
    - `window`主要用来配置app的一些样式(navigationBar样式)、全局功能(下拉刷新)
    - `networkTimeout`用来配置网络的超时时间
- `app.wxss`用来配置全局样式
    - 可以在里面设计基础样式，如全局字体、背景色
    - `wxss`基本和`css`一样，css能用的，wxss基本都能用
- `project.config.json`项目配置
- `pages`文件夹主要用来存放页面，页面相关资源全部保存在一个页面文件中，如图中的`index`
    - 每个页面文件夹(如`index`页面)中又包含页面对应的`js、json(非必须，若设置了会覆盖app.json)、wxml、wxss(会覆盖app.wxss)`
- `utils`用来保存一些工具函数

## API Promisify
- 小程序提供的接口，用的都是回调函数的形式，这在开发时不太友好。不过，小程序提供的api接口的回调名一致，而且支持原生的promise，可以用promise来定义一个Promisify函数来。
```javascript
    // promisify.js
    module.exports = (api) => {
      return (options, ...params) => {
        return new Promise((resolve, reject) => {
          api(Object.assign({}, options, { success: resolve, fail: reject }), ...params);
        });
      }
    }

    // index.js
    const Promisify = require('../../utils/promise.js');
        
    let promisiedGetLocation = Promisify(wx.getLocation);
    promisiedGetLocation({
      type: 'gcj02'
    }).then((res)=>{
        console.log(res);
    });
```

## 不同页面通信
- 可以参考https://github.com/dannnney/weapp-event
```javascript
    // event.js
    var events = {};

    function on(name, self, callback) {
      var tuple = [self, callback];
      var callbacks = events[name];
      if (Array.isArray(callbacks)) {
        callbacks.push(tuple);
      }
      else {
        events[name] = [tuple];
      }
    }
    
    function remove(name, self) {
      var callbacks = events[name];
      if (Array.isArray(callbacks)) {
        events[name] = callbacks.filter((tuple) => {
          return tuple[0] != self;
        })
      }
    }
    
    function emit(name, data) {
      var callbacks = events[name];
      if (Array.isArray(callbacks)) {
        callbacks.map((tuple) => {
          var self = tuple[0];
          var callback = tuple[1];
          callback.call(self, data);
        })
      }
    }
    
    exports.on = on;
    exports.remove = remove;
    exports.emit = emit;
```

## 清除input内容
- 小程序没有提供一个简便的方法清除input的内容，百度了下可以用以下方法实现
    - 将input的value属性绑定到一个变量上，单击时置空变量
        - 场景:实现搜索建议，搜索框带有清除按钮
        - 将input的value绑定到变量a上，并监听input的oninput事件，将e.currentTarget.detail.value赋值给a,点击清除按钮时，置空a。
            - 虽然完成了搜索建议和清除功能，但在手机上，快速删除时，光标会出现闪烁问题

## 坐标转换
- `wx.getLocation()`可以返回wgs84坐标和gcj02坐标
    - wgs84坐标为gps坐标；国内地图公司一般不使用
        - 硬件设备、谷歌地球、chrome devTool中的Sensor
    - gcj02为在wgs84基础上加密的坐标，又称火星坐标系；国内大部分地图公司用的是gcj02
        - 腾讯地图、高德地图国内、谷歌地图国内
    - bd09为百度地图使用的坐标系统，在gcj02的基础又加密了一层
        - 百度地图
```javascript
    // Created by Wandergis on 2015/7/8.
    // 提供了百度坐标（BD09）、国测局坐标（火星坐标，GCJ02）、和WGS84坐标系之间的转换
    
    //定义一些常量
    var x_PI = 3.14159265358979324 * 3000.0 / 180.0;
    var PI = 3.1415926535897932384626;
    var a = 6378245.0;
    var ee = 0.00669342162296594323;
    
    
    // 百度坐标系 (BD-09) 与 火星坐标系 (GCJ-02)的转换
    // 即 百度 转 谷歌、高德
    // @param bd_lon
    // @param bd_lat
    // @returns {*[]}
    //
    function bd09togcj02(bd_lon, bd_lat) {
      var x_pi = 3.14159265358979324 * 3000.0 / 180.0;
      var x = bd_lon - 0.0065;
      var y = bd_lat - 0.006;
      var z = Math.sqrt(x * x + y * y) - 0.00002 * Math.sin(y * x_pi);
      var theta = Math.atan2(y, x) - 0.000003 * Math.cos(x * x_pi);
      var gg_lng = z * Math.cos(theta);
      var gg_lat = z * Math.sin(theta);
      return [gg_lng, gg_lat]
    }
    
    // 火星坐标系 (GCJ-02) 与百度坐标系 (BD-09) 的转换
    // 即谷歌、高德 转 百度
    // @param lng
    // @param lat
    // @returns {*[]}

    function gcj02tobd09(lng, lat) {
      var z = Math.sqrt(lng * lng + lat * lat) + 0.00002 * Math.sin(lat * x_PI);
      var theta = Math.atan2(lat, lng) + 0.000003 * Math.cos(lng * x_PI);
      var bd_lng = z * Math.cos(theta) + 0.0065;
      var bd_lat = z * Math.sin(theta) + 0.006;
      return [bd_lng, bd_lat]
    }
    
    // WGS84转GCj02
    // @param lng
    // @param lat
    // @returns {*[]}

    function wgs84togcj02(lng, lat) {
      if (out_of_china(lng, lat)) {
        return [lng, lat]
      }
      else {
        var dlat = transformlat(lng - 105.0, lat - 35.0);
        var dlng = transformlng(lng - 105.0, lat - 35.0);
        var radlat = lat / 180.0 * PI;
        var magic = Math.sin(radlat);
        magic = 1 - ee * magic * magic;
        var sqrtmagic = Math.sqrt(magic);
        dlat = (dlat * 180.0) / ((a * (1 - ee)) / (magic * sqrtmagic) * PI);
        dlng = (dlng * 180.0) / (a / sqrtmagic * Math.cos(radlat) * PI);
        var mglat = lat + dlat;
        var mglng = lng + dlng;
        return [mglng, mglat]
      }
    }
    
    // GCJ02 转换为 WGS84
    // @param lng
    // @param lat
    // @returns {*[]}

    function gcj02towgs84(lng, lat) {
      if (out_of_china(lng, lat)) {
        return [lng, lat]
      }
      else {
        var dlat = transformlat(lng - 105.0, lat - 35.0);
        var dlng = transformlng(lng - 105.0, lat - 35.0);
        var radlat = lat / 180.0 * PI;
        var magic = Math.sin(radlat);
        magic = 1 - ee * magic * magic;
        var sqrtmagic = Math.sqrt(magic);
        dlat = (dlat * 180.0) / ((a * (1 - ee)) / (magic * sqrtmagic) * PI);
        dlng = (dlng * 180.0) / (a / sqrtmagic * Math.cos(radlat) * PI);
        mglat = lat + dlat;
        mglng = lng + dlng;
        return [lng * 2 - mglng, lat * 2 - mglat]
      }
    }
    
    function transformlat(lng, lat) {
      var ret = -100.0 + 2.0 * lng + 3.0 * lat + 0.2 * lat * lat + 0.1 * lng * lat + 0.2 * Math.sqrt(Math.abs(lng));
      ret += (20.0 * Math.sin(6.0 * lng * PI) + 20.0 * Math.sin(2.0 * lng * PI)) * 2.0 / 3.0;
      ret += (20.0 * Math.sin(lat * PI) + 40.0 * Math.sin(lat / 3.0 * PI)) * 2.0 / 3.0;
      ret += (160.0 * Math.sin(lat / 12.0 * PI) + 320 * Math.sin(lat * PI / 30.0)) * 2.0 / 3.0;
      return ret
    }
    
    function transformlng(lng, lat) {
      var ret = 300.0 + lng + 2.0 * lat + 0.1 * lng * lng + 0.1 * lng * lat + 0.1 * Math.sqrt(Math.abs(lng));
      ret += (20.0 * Math.sin(6.0 * lng * PI) + 20.0 * Math.sin(2.0 * lng * PI)) * 2.0 / 3.0;
      ret += (20.0 * Math.sin(lng * PI) + 40.0 * Math.sin(lng / 3.0 * PI)) * 2.0 / 3.0;
      ret += (150.0 * Math.sin(lng / 12.0 * PI) + 300.0 * Math.sin(lng / 30.0 * PI)) * 2.0 / 3.0;
      return ret
    }
    
    // 判断是否在国内，不在国内则不做偏移
    // @param lng
    // @param lat
    // @returns {boolean}
     
    function out_of_china(lng, lat) {
      return (lng < 72.004 || lng > 137.8347) || ((lat < 0.8293 || lat > 55.8271) || false);
    }
    
    module.exports={
      bd09togcj02: bd09togcj02,
      gcj02tobd09: gcj02tobd09,
      wgs84togcj02: wgs84togcj02,
      gcj02towgs84: gcj02towgs84,
    };
```
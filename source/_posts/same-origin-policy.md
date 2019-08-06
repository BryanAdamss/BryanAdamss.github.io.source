---
title: same-origin-policy
tags:
  - CORS
  - JSONP
  - 跨域
categories:
  - 前端
date: 2018-02-05 10:07:24
---

> 前端时间，项目中遇到一些跨域问题，查阅了一些文档后，特在此做个记录。

# 浏览器的同源策略及规避方法

## 同源策略(same-origin-policy)
- 同源策略主要是用来隔离一些潜在的恶意文件。保证用户信息的安全，防止恶意的网站窃取数据。https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy
- 例如AB两个不同源的网页，如果没有同源策略，AB之间就可以相互访问请求到对方的隐私信息(如cookie)，这样显然是不安全的。

## 什么是同源?
- 同源要求:
    - 协议相同
    - 域名相同
    - 端口相同
- 举例

| URL        | 说明           | 是否允许通信 |
| ------------- |:-------------:| -----:|
| http://www.a.com/a.js <br> http://www.a.com/b.js | 同一域名下 | 允许 |
| http://www.a.com/lab/a.js <br> http://www.a.com/script/b.js | 同一域名下不同文件夹 | 允许 |
| http://www.a.com:8000/a.js <br> http://www.a.com/b.js | 同一域名，不同端口 | 不允许 |
| http://www.a.com/a.js <br> https://www.a.com/b.js | 同一域名，不同协议 | 不允许 |
| http://www.a.com/a.js <br> http://70.32.92.74/b.js | 域名和域名对应ip | 不允许 |
| http://www.a.com/a.js <br> http://script.a.com/b.js | 主域相同，子域不同 | 不允许 |
| http://www.a.com/a.js <br> http://a.com/b.js | 同一域名，不同二级域名（同上） | 不允许（cookie这种情况下也不允许访问） |
| http://www.cnblogs.com/a.js <br> http://www.a.com/b.js | 不同域名 | 不允许 |

## 如何规避同源策略(跨域)
- 虽然同源策略能保证一定的安全性，但是有时候我们需要请求不同源页面的资源，就需要跨过同源策略。

### cookie
- 当AB两个页面，一级域名相同，二级域名不同时，可以通过同时设置`document.domain`为一级域名，就可以实现`cookie`的共享
- A页面是`http://a.test.com`，B页面是`http://b.test.com`，二者因为二级域名不同，所以受同源策略的限制，无法共享cookie。可以同时设置`document.domain='test.com'`来解决
```javascript
// A页面
document.domain='test.com'
document.cookie = "testName=cgh"; // A页面设置cookie

// B页面
document.domain='test.com'
...
var allCookie = document.cookie; // B页面可以读取到A页面的cookie
...
```

### iframe
- 使用iframe嵌套页面时，iframe的src不受同源策略的影响，可以加载任意地址的文档；当父页面和iframe存在同源限制时，将不能相互访问`window`和`DOM`。例如在A页面嵌套了百度，则在A页面是无法访问到百度首页的`window`和`DOM`的。父子页面同源时，能可以相互访问`window`、`DOM`
- 父子页面，一级域名相同，二级域名不同时，可以使用和cookie相同的办法（设置`documen.domain`）来解决，进而可以相互访问`window`和`DOM`
```javascript
// http://www.test.com/a.html
<iframe src="child.test.com/b.html"></iframe>
<script>
document.domain='test.com';
var user='cgh';
</script>

// http://child.test.com/b.html
<script>
document.domain='test.com';

console.log(window.parent.user);// 获取父窗口的变量
</script>  
```
- 完全不同源的iframe可以通过以下方法实现
    - hash+hashchange
        - 父页面将信息写入到子iframe的hash值中，子页面通过监听`hashchange`事件来接收信息(也可以采用setInterval向下兼容);子页面可以直接改写父页面的hash值(有些浏览器不允许直接修改父页面的hash值，可以通过在父页面A的同域下再创建一个中间页面C,在目标页面B嵌入C来实现，具体看下面例子)，父页面也可以监听`hashchange`来接收子页面的信息；参考:https://www.zhihu.com/question/20314348/answer/20025563
        ```html
        // http://w1.test.com/a.html，a页面中嵌入了不同源的b页面；a页面可以设置bFrame src中的hash值
        <iframe id="bFrame" src="http://ah122.cn/b.html"></iframe>
        
        // http://ah122.cn/b.html，b页面嵌套了和a页面同源的c页面
        <iframe id="cFrame" src="http://w1.test.com/c.html" style="display:none"></iframe>
        <script>
            window.onhashchange = function(){// b页面监听a页面改变bFrame Hash值的事件，并将值传递给和a同源的c页面
                document.getElementById('cFrame').src = 'http://w1.test.com/c.html' + location.hash; 
            }
        </script>
        
        // http://w1.test.com/c.html，和a页面同源，ac页面可以相互访问变量和DOM
        <script>
            window.onhashchange = function(){// c页面监听到hash值改变，将值传递到同源的a页面
                parent.parent._bHash = location.hash; 
            }
        </script>
        
        // 总结
        A->B    AB不同源，但A可以修改bFrame中src的hash值，B可以通过hashchange事件获取到传递过来的hash值
        B->A    因为B为子页面，而且不同源，无法直接传递数据或者修改A的window、DOM，必须找个中间桥梁来实现传递，于是，通过在B中嵌套和A同源的C页面，将数据通过修改cFrame的hash的方式传递给C页面，C页面通过hashchange事件接收到数据，并通过parent.parent将数据传递给A(AC同源，可以相互访问window、DOM)，这样就完成了子页面B像不同源的A页面传递数据的过程。
        ```
    - window.name
        - 浏览器窗口有window.name属性。这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页(iframe)设置了这个属性，后一个网页可以读取它(就是iframe即使src发生改变了，iframe对应的contentWindow.name都不会发生改变，除非重新设置了)。并且容量很大，正常2M，IE、FF可以达到32M;
        - 利用window.name跨域，原理和hash值跨域类似，都需要一个中间(代理)页面；参考:http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html
    ```html
    // a.com/a.html：应用页面。
    // a.com/c.html：代理文件，一般是一个没有任何内容的html文件，需要和应用页面在同一域下。
    // b.com/b.html：应用页面需要获取数据的页面，可称为数据页面。
    
    // a.com/a.html，a页面中创建并嵌入不同源的c页面；可以通过修改hash值来传递数据给b页面
    <script>
        var state = 0, 
        iframe = document.createElement('iframe'),
        loadfn = function() {// b.html加载完成后会执行
            if (state === 1) {
                var data = iframe.contentWindow.name;    // 读取数据
                alert(data);    //弹出'需要回传给a的数据'
            } else if (state === 0) {
                state = 1;
                iframe.contentWindow.location = "http://a.com/c.html";    // 设置的代理文件
            }  
        };
        iframe.src = 'http://b.com/b.html';
        if (iframe.attachEvent) {
            iframe.attachEvent('onload', loadfn);
        } else {
            iframe.onload  = loadfn;
        }
        document.body.appendChild(iframe); 
        // 数据获取完毕后，可以销毁iframe
        iframe.contentWindow.document.write('');
        iframe.contentWindow.close();
        document.body.removeChild(iframe);
    </script>
    
    // b.com/b.html，b页面可以监听hashchange事件来接收a页面传递来的数据
    <script>
    window.onhashchange=function(){ // 接收a传递来的数据
        var receive=window.location.hash;
    };
    
    window.name='需要回传给a的数据';
    </script>
    
    // a.com/c.html，是一个空白页面，什么都不需要处理，只需要和a页面同源
    ```
    - window.postMessage
        - 无论是hash还是window.name都属于一种hack技术；H5引入了跨文档通信的API，postMessage；可以支持IE8+(IE89仅支持在iframe中传递)；参考:http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
        ```javascript
        // a.com/a.html
        var popup = window.open('http://b.com', 'title');
        popup.postMessage('Hello World!', 'http://b.com');
        
        // b.com/b.html
        window.addEventListener('message', function(e) {
        // event.source：发送消息的窗口
        // event.origin: 消息发向的网址
        // event.data: 消息内容
          console.log(e.data);// 'Hello World!'
        },false);
        ```

### AJAX
- 同源政策规定，AJAX请求只能发给同源的网址，否则就报错
- 跨域解决
    - 架设服务器代理(浏览器请求同源服务器，再由后者请求外部服务)
        - 在vue-cli中可以通过配置express.Router.get()来实现对外接口请求
    - JSONP
        - 在html中`src、href`请求的资源是不受同源策略的限制的;img、iframe、css、script可以加载任意的资源；jsonp就是利用script标签的src不受同源限制，并在创建时自动执行内部代码来实现的。
        - 一个简单的jsonp实现；源码请查看:https://github.com/BryanAdamss/SourceSave/blob/master/Plugins/js/vendor/12_jsonp.js
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
        
        // 使用
        <script type="text/javascript" src="js/vendor/12_jsonp.js"></script>
        <script type="text/javascript">
        window.onload = function() {
            main();
        };
        
        function main() {
            getList();
        }
        
        function getList() {
            c_jsonp({
                url: 'http://192.168.23.126:8080/hfgj/app/cgh.do',
                data: {
                    id: 3,
                    name: 'cgh'
                },
                success: function(resp) {
                    console.log(resp);
                },
                error: function() {
                    console.log('出错了');
                }
            });
        }
        </script>
        ```
    - websocket
        - 双向实时全双工通信，支持跨域连接；具体参考:http://www.52im.net/forum.php?mod=viewthread&tid=331&ctid=15
    - CORS
        - 跨资源共享，需要在服务端配置`Access-Control-Allow-Origin`为`*`，代表允许任何来源的请求资源；客户端在发送请求时需要带上`origin`来标识请求来自哪个域，好让服务端做验证。具体可参考:http://www.ruanyifeng.com/blog/2016/04/cors.html

## 参考链接
> https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy#%E8%B7%A8%E6%BA%90%E7%BD%91%E7%BB%9C%E8%AE%BF%E9%97%AE
> https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
> http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html
> http://www.cnblogs.com/rainman/archive/2011/02/21/1960044.html
> http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html
> http://www.ruanyifeng.com/blog/2016/04/cors.html

---
title: websocket_chat
tags:
  - 聊天室
  - websocket
  - nodejs
  - im
categories:
  - 前端
date: 2018-01-18 14:53:58
---

> 最近在看web端即时通信方面东西，用nodejs结合websocket(socket.io)做了一个简单的web聊天室。特在此做个记录
> 完整代码可以查看(https://github.com/BryanAdamss/SourceSave/tree/master/WebSocket/ws-socketiodemo)

# 使用NodeJs、Socket.io搭建一个web聊天室

## 前置知识
Web端实现即时通信主要有四种方式：短轮询(polling)、comet、Websocket、SSE
- 短轮询
    - 前台设置个定时器不断发送请求去请求后台数据
        - 缺点:会有大量无效请求、浪费服务器资源、有延迟
- Comet
    - 其实是一种hack技术，主要是一种基于http长连接的"服务器推"的技术
    - 主要有两种实现方式
        - ajax长轮询(long-polling)
            - 客户端发出ajax请求，服务端接收到请求后，会阻塞请求直到有数据或者超时才返回，客户端在在处理信息后再次发出请求，重新建立连接。
                - 优点:相比短轮询减少了无效请求、实时性提高
                - 缺点:保持连接也会消耗服务器资源
        - 基于iframe及htmlfile的流方式
            - iframe的src属性会保持对指定服务器的长连接请求，服务器端则可以不停地返回数据
                - 缺点:ie、ff下会显示页面未加载完成
            - 利用htmlfile的ActiveX解决了IE上的加载显示问题
- WebSocket
    - Websocket是一个全新的、独立的协议，基于TCP协议，与http协议兼容。
    - Websocket在建立连接之前有一个Handshake过程，在关闭连接前也有一个Handshake过程，建立连接之后，双方即可双向通信。
        - 优点:全双工相互通信
- SSE
    - Server-Sent Event 服务器推送事件，允许服务端向客户端推送新数据。
    - 传统情况下服务端可以通过flash(Flash XMLSocket)或者Java Applet 套接口来实现推送
    - 一般说websocket和sse都能做彼此能做的事情，不过sse更多的是专注服务端向客户端推，客户端想发送消息给服务端必须通过ajax来发送，而websocket连接上后，双方就可以直接通过websocket通信了。

## 需求
- 允许同名登录
- 查看在线人数及列表
- 区分自己发言和他人发言
- 加入、退出聊天室有提示
- 尽量兼容低版本浏览器

## websocket API
- 通过分析对比几种及时通信的实现方式，客户端选择使用websocket来实现较为简单。
- 常用API，具体可参考(https://developer.mozilla.org/zh-CN/docs/Web/API/WebSocket);
    ```javascript
    var socket=new Socket('http://xxxx.com:port');
    
    socket.onopen=function(){
        // 连接上服务器时触发(此时表示客户端可以接受和发送数据)
    };
    
    socket.onerror=function(){
        // 当错误发生时用于监听error事件的事件监听器。会接受一个名为“error”的event对象。
    };
    
    socket.onmessage=function(){
        // 当有消息到达的时候该事件会触发
    };
    
    socket.close();// 主动关闭连接
    
    socket.send();// 发送信息给服务端，可接受DOMString data、ArrayBuffer data、Blob data
    
    ```

## 分析
- 原生的websocket只能支持IE10+、移动端只能支持到安卓4.4，所以最后采用socket.io来实现。
- socket.io是运行在node环境上的，可以兼顾到前后台，它可以做到优雅降级，当不支持websocket的浏览器会自动使用轮询的方式来实现即时通信。具体可参考(https://socket.io/docs/);

## 代码实现
- 完整代码可以查看(https://github.com/BryanAdamss/SourceSave/tree/master/WebSocket/ws-socketiodemo)
- 服务端
```javascript
var express = require('express');
var app = express();
var path = require('path');
var server = require('http').createServer(app);
var io = require('socket.io')(server);
var port = process.env.PORT || 3000;

server.listen(port, function() {
    console.log('服务启动在:%d', port);
});

app.use(express.static(path.join(__dirname, './public')));


var onlineUsers = {}; //在线用户

var onlineCount = 0; //当前在线人数

// 监听用户连接
io.on('connection', function(socket) {
    console.log('有用户连接!');
    // 连接后监听相应事件

    // 监听客户端的登录事件
    socket.on('login', function(obj) {

        // 保存连接的id，退出时会用到
        socket.id = obj.userId;

        // 将新的连接加入到在线列表中
        if (!onlineUsers.hasOwnProperty(obj.userId)) {
            onlineUsers[obj.userId] = obj.userName;
            onlineCount++;
        }

        // 向所有客户端广播有新用户加入，并将新用户及最新的在线人数和在线列表传过去
        io.emit('newIn', { onlineUsers: onlineUsers, onlineCount: onlineCount, user: obj });

        console.log(obj.userName + '加入了聊天室');
    });

    // 监听客户端的发送事件，将其广播给所有用户
    socket.on('message', function(obj) {
        console.log('%s说了:%s', obj.name, obj.message);
        io.emit('message', obj);
    });

    // 监听客户端的离线事件
    socket.on('disconnect', function() {
        // 在线列表中删除对应的连接
        if (onlineUsers.hasOwnProperty(socket.id)) {
            var obj = { userId: socket.id, userName: onlineUsers[socket.id] };

            delete onlineUsers[socket.id];

            onlineCount--;
            // 向所有客户端广播有用户退出    
            io.emit('logout', { onlineUsers: onlineUsers, onlineCount: onlineCount, user: obj });
            console.log(obj.userName + '退出了聊天室');
            console.log('现在聊天室里有:', onlineUsers);
        }
    });
});
```
- 客户端
```javascript
(function() {
    function addEvent(ele, type, handler) {
        if (ele.addEventListener) {
            ele.addEventListener(type, handler, false);
        } else if (ele.attachEvent) {
            ele.attachEvent("on" + type, handler);
        } else {
            ele["on" + type] = handler;
        }
    }

    function trim(str) {
        if (!str.trim) {
            return str.replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g, '');
        }
        return str.trim();
    }

    window.onload = function() {
        main();
    };

    function main() {
        var loginInput = document.querySelector('#js_loginInput'),
            loginBox = document.querySelector('#js_loginBox');

        loginInput.focus(); // 登录框默认聚焦

        var URL = 'http://192.168.23.27:3000';
        addEvent(loginInput, 'keydown', function(e) {
            var e = e || window.e;
            if (e.keyCode === 13) {
                var val = trim(loginInput.value);
                if (val !== '') {
                    // 隐藏登录框
                    loginInput.value = '';
                    loginBox.className = loginBox.className + ' is-hide';

                    // 创建一个ws连接
                    var link = new Link(URL, val);
                }
            }
        });
    }


    function Link(url, name) {
        this.url = url;
        this.userName = trim(name);
        this.init();
    }

    Link.prototype = {
        constructor: Link,
        init: function() {
            var self = this;

            this.socket = io.connect(this.url);
            this.userId = this.genUid();
            this.submitInput = document.querySelector('#js_chatInput');
            this.list = document.querySelector('#js_chatList');
            this.hd = document.querySelector('#js_chatHd');

            this.submitInput.focus();

            this.socket.emit('login', { userName: this.userName, userId: this.userId }); // 告诉服务端，当前客户端登录了

            this.socket.on('newIn', function(o) { // 监听服务端派发的用户登录事件
                var li = self.makeTips(o.user.userName);

                self.updateList(li);
                self.updateOnline(o.onlineUsers, o.onlineCount);
            });

            addEvent(this.submitInput, 'keydown', function(e) {
                var e = e || window.e;
                var text = trim(self.submitInput.value);
                if (e.keyCode === 13) {
                    if (text !== '') {
                        self.submitInput.value = '';

                        self.socket.emit('message', { message: text, name: self.userName, id: self.userId }); // 告诉服务端，当前客户端发送了一个message
                    }
                }
            });

            this.socket.on('message', function(o) { // 监听服务端派发的message事件
                var isMe = o.id === self.userId ? true : false; // 判断消息是否是当前客户端发送的

                var li = self.makeMessage(o.name, o.message, isMe);
                self.updateList(li);
            });

            this.socket.on('logout', function(o) { // 监听服务端派发的logout事件
                var li = self.makeTips(o.user.userName, true);

                self.updateList(li);
                self.updateOnline(o.onlineUsers, o.onlineCount);
            });
        },
        genUid: function() {
            return new Date().getTime() + "" + Math.floor(Math.random() * 899 + 100); // 生成唯一id，用于后面的判断
        },
        makeTips: function(name, logout) {
            if (typeof name === 'undefined') {
                return;
            }

            var li = document.createElement('li');
            li.className = 'c-ChatList-item is-tips';
            if (logout) {
                li.innerHTML = '<span class="c-ChatList-tip">' + name + '退出了聊天室</span>';
            } else {
                li.innerHTML = '<span class="c-ChatList-tip">' + name + '加入了聊天室</span>';
            }

            return li;
        },
        makeMessage: function(name, text, me) {
            if (typeof name === 'undefined' || typeof text === 'undefined') {
                return;
            }

            var li = document.createElement('li');
            if (me) {
                li.className = 'c-ChatList-item fadeInRight is-me';
                li.innerHTML = '<div class="c-ChatList-cont"><div class="c-ChatList-text">' + text + '</div><div class="c-ChatList-name">' + name + '</div></div>';
            } else {
                li.className = 'c-ChatList-item fadeInLeft';
                li.innerHTML = '<div class="c-ChatList-cont"><div class="c-ChatList-name">' + name + '</div><div class="c-ChatList-text">' + text + '</div></div>';
            }

            return li;
        },
        updateList: function(li) {
            this.list.appendChild(li);

            li.scrollIntoView();
        },
        updateOnline: function(users, count) {
            var html = '在线人数：' + count + '人；在线列表：';
            var arr = [];

            for (var key in users) {
                if (users.hasOwnProperty(key)) {
                    arr.push(users[key]);
                }
            }

            html += arr.join('、');

            this.hd.innerHTML = html;
        }
    };
})();
```

## 相关链接
> http://www.52im.net/thread-336-1-1.html
> https://segmentfault.com/a/1190000002496055
> https://socket.io
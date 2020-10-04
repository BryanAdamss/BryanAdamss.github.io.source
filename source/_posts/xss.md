---
title: xss
tags:
  - 安全
  - xss
categories:
  - 前端
date: 2020-03-23 15:18:49
---

# XSS

## 概念

- `XSS(Cross-site scripting)`是一种跨站脚本攻击；它是一种注入攻击，攻击者可以利用漏洞在网站上注入恶意的客户端代码，进而获取用户的敏感信息(cookie、session tokens 等)；
- 维基定义

```
XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java，VBScript，ActiveX，Flash或者甚至是普通的HTML。攻击成功后，攻击者可能得到更高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。
```

- 白话解释
  - 攻击者利用特殊方法将恶意代码注入客户端网站(浏览器)，受害者访问网站，进而执行了被注入的恶意代码，攻击者获取到受害者的信息
- 因为缩写`css`同层叠样式表名冲突，所以改为`xss`

## 分类

- 一般有三种`Persistent XSS(持久型/存储型)`、`Reflected XSS(反射型)`、`DOM-based XSS(基于DOM的XSS攻击)`
- 存储区：恶意代码存放的位置。
- 插入点：由谁取得恶意代码，并插入到网页上。

| 类型      | 存储区                  | 插入点          |
| --------- | ----------------------- | --------------- |
| 持久型    | 后端数据库              | HTML            |
| 反射型    | URL                     | HTML            |
| DOM-based | 后端数据库/前端存储/URL | 前端 JavaScript |

## 持久型

### 常见攻击步骤

- **攻击者将恶意代码提交到目标网站的数据库中。**
  - 恶意代码被存储在后台数据库中，这是持久型的特点
- 用户打开目标网站时，网站服务端将恶意代码从数据库取出，拼接在 HTML 中返回给浏览器。
  - **持久型常见于服务端渲染中，例如利用`jsp`技术拼接返回 html**
- 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
- 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

### 实例

- persistent.png
  - ![persistent.png](persistent.png)

1. 攻击者利用提交网站表单将一段恶意文本插入网站的数据库中
2. 受害者向网站请求页面
3. 网站从数据库中取出恶意文本把它包含进返回给受害者的页面中
4. 受害者的浏览器执行返回页面中的恶意脚本，把自己的 cookie 发送给攻击者的服务器。

### 特点

- 恶意代码被存储在后台数据库中
  - 恶意代码通过与用户交互功能被存入数据库
- 后台通过拼接 html 的形式直接返回给浏览器渲染
  - 未做过滤(转义)
- 导致所有访问页面的用户，都会执行恶意代码

### 常见场景

- 常出现在能保存用户数据的功能上，例如评论、私信等

## 反射型

### 常见攻击步骤

- 攻击者构造出特殊的 URL，其中包含恶意代码。
- 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码从 URL 中取出，拼接在 HTML 中返回给浏览器。
  - **反射型跟持久型类似，也常见于服务端渲染中，例如利用`jsp`技术拼接返回 html**
- 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
- 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作

- 注意
  - 按正常步骤，受害者亲自提交恶意代码，然后窃取自己的用户信息，一般没人会这么干:)
  - 所以，攻击者一般会用特殊的方法诱导受害者点击某个链接
  - 如果攻击者的目标是一位具体的个人用户，攻击者可以把恶意链接发送给受害者（比如通过电子邮件或者短信），并且欺骗他去访问这个链接。
  - 如果攻击者的目标是一个群体，攻击者能够发布一条指向恶意 URL 的链接（比如在他的个人网站或者社交网络上），然后等待访问者去点击它。

### 实例

- reflect.png
  - ![reflect.png](reflect.png)

1. 攻击者构造了一个包含恶意文本的 URL 发送给受害者
2. 受害者被攻击者欺骗，通过访问这个 URL 向网站发出请求
3. 网站通过拼接 html 形式给受害者的返回中包含了来自 URL 的的恶意文本
4. 受害者的浏览器执行了来自返回中的恶意脚本，把受害者的 cookie 发送给攻击者的服务器

### 特点

- 恶意代码存储在 URL 上，没有存储到数据库
- 服务端没有对恶意代码做过滤，直接输出到页面上，导致恶意脚本被执行
- 由于需要用户主动打开恶意的 URL 才能生效，攻击者往往会结合多种手段诱导用户点击
- 同持久型区别
  - 反射性 xss 恶意代码存储在 URL 中
  - 持久型 xss 恶意代码存储在后台数据库中

### 常见场景

- 一般常见于通过 URL 传递参数的功能，如网站搜索、跳转等

## DOM-based 型(前端重点关注)

- 基于 DOM 的 XSS 是属于持久型和反射型 XSS 的变种。在基于 DOM 的 XSS 的攻击中，除非网站自身的合法脚本被执行，否则恶意文本不会被受害者的浏览器解析
  - 恶意代码是在网站合法代码执行时触发
- DOM-based 型，严格来说应该是前端的一个漏洞
  - 因为取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞
  - 前端 JavaScript 代码本身不够严谨，把不可信的数据当作代码执行了。

### 常见攻击步骤

- 攻击者构造出特殊的 URL，其中包含恶意代码。
- 用户打开带有恶意代码的 URL。
- **用户浏览器接收到响应后解析执行，前端 JavaScript 取出 URL 中的恶意代码并执行**。
  - 服务端没有返回恶意代码(拼接好的 html)
- 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

### 实例

- dom.png
  - ![dom.png](dom.png)

1. 攻击者构造一个包含恶意文本的 URL 发送受害者
2. 受害者被攻击者欺骗，通过访问这个 URL 向网站发出请求
3. **网站收到请求，但是恶意文本并没有包含在给受害者的返回页面中**
4. 受害者的浏览器执行来自网站返回页面里的合法脚本，导致恶意脚本被插入进页面中
5. 受害者的浏览器执行插入进页面的恶意脚本，把自己的 cookie 发送到攻击者的服务器

### 特点

- 持久型、反射型的变种，恶意代码可以存储在服务端数据库也可以存储在 URL 中
- 触发时机是在网站合法代码执行时
  - 在传统的 XSS 攻击中，恶意脚本作为服务器传回的一部分，在页面加载时就被执行
  - 在基于 DOM 的 XSS 攻击中，恶意脚本在页面加载之后的某个时间点才执行，是合法脚本以非安全的方式处理用户输入的结果

## 防止

- 输入侧
  - 攻击者提交恶意代码。
- 输出侧
  - 浏览器执行恶意代码。

### 输入侧

- 不要相信任何用户输入(用户能修改的任何数据)，做转义(编码)、过滤(校验)
  - 来自用户的 UGC 信息
    - 表单输入
  - URL
  - 客户端存储
    - localStorage
    - sessionStorage
    - indexDB
- 转义是指将用户输入转义的行为，以确保浏览器把输入当作数据而不是代码对待。在 web 开发中最知名的一类编码莫过于 HTML 转义，该方法将`<`和`>`分别转义为`&lt;`和`&gt;`
  - **大部分情况下，只要用户的输入会被包含进页面，就应该执行转义**
  - 前端可以借助`xss`这个 npm 包来完成转义
- 过滤是指对用户输入的内容做过滤，可以建立白名单，只有白名单中的内容才允许输入
  - 例如允许输入`<em>、<strong>`，不允许输入`<script>`
- 转义、过滤需要同时在服务端、前端同时进行

### 输出侧

- 防止 html 中出现注入(针对持久型、反射型)
- 防止 JavaScript 执行时，执行恶意代码(针对 DOM-based 型)

#### 防止 html 中出现注入

- 针对持久型、反射型，这两种是由于服务端返回的 html 携带恶意代码导致恶意代码被执行
  - 所以可以改为前端渲染，将代码和数据分离
    - 前端通过 ajax 请求数据然后渲染
    - 纯前端渲染，还需要预防 DOM-based xss 攻击
  - 或者服务端对返回的 HTML 做充分转义
    - 需要针对不同的输入点位置做不同的转义

```html
<!-- HTML 标签内文字内容 -->
<div><%= Encode.forHtml(UNTRUSTED) %></div>

<!-- HTML 标签属性值 -->
<input value="<%= Encode.forHtml(UNTRUSTED) %>" />

<!-- CSS 属性值 -->
<div style="width:<= Encode.forCssString(UNTRUSTED) %>">
  <!-- CSS URL -->
  <div style="background:<= Encode.forCssUrl(UNTRUSTED) %>">
    <!-- JavaScript 内联代码块 -->
    <script>
      var msg = "<%= Encode.forJavaScript(UNTRUSTED) %>";
      alert(msg);
    </script>

    <!-- JavaScript 内联代码块内嵌 JSON -->
    <script>
      var __INITIAL_STATE__ = JSON.parse(
        "<%= Encoder.forJavaScript(data.to_json) %>"
      );
    </script>

    <!-- HTML 标签内联监听器 -->
    <button onclick="alert('<%= Encode.forJavaScript(UNTRUSTED) %>');">
      click me
    </button>

    <!-- URL 参数 -->
    <a
      href="/search?value=<%= Encode.forUriComponent(UNTRUSTED) %>&order=1#top"
    >
      <!-- URL 路径 -->
      <a href="/page/<%= Encode.forUriComponent(UNTRUSTED) %>">
        <!--
  URL.
  注意：要根据项目情况进行过滤，禁止掉 "javascript:" 链接、非法 scheme 等
-->
        <a
          href='<%=
  urlValidator.isValid(UNTRUSTED) ?
    Encode.forHtml(UNTRUSTED) :
    "/404"
%>'
        >
          link
        </a></a
      ></a
    >
  </div>
</div>
```

- 针对不同的上下文进行转义是很复杂的，可以考虑谷歌的[Automatic Context-Aware Escaping](https://security.googleblog.com/2009/03/reducing-xss-by-way-of-automatic.html)

#### 防止 JavaScript 执行时，执行恶意代码(这其实主要是预防 DOM-based xss 型攻击)

- 原生 js
  - 避免使用 .innerHTML、.outerHTML、document.write()  将不可信的数据做为 html 插到页面上，而应尽量使用  .textContent、.setAttribute()  等
  - DOM 中的内联事件监听器，如 location、onclick、onerror、onload、onmouseover 等，<a> 标签的 href 属性，JavaScript 的 eval()、setTimeout()、setInterval() 等，都能把字符串作为代码运行。如果不可信的数据拼接到字符串中传递给这些 API，很容易产生安全隐患，请务必避免。
- Vue/React
  - 避免使用  v-html/dangerouslySetInnerHTML

```html
<!-- 内联事件监听器中包含恶意代码 -->
<img onclick="UNTRUSTED" onerror="UNTRUSTED" src="data:image/png," />
<!-- 链接内包含恶意代码 -->
<a href="UNTRUSTED">1</a>

<script>
  // setTimeout()/setInterval() 中调用恶意代码
  setTimeout("UNTRUSTED");
  setInterval("UNTRUSTED");
  // location 调用恶意代码
  location.href = "UNTRUSTED";
  // eval() 中调用恶意代码
  eval("UNTRUSTED");
</script>
```

### 其他方法

- 攻击者最终目的是获取受害者的敏感信息
  - 所以可对部分用户敏感信息做限制
    - 例如针对 cookie，可以设置 http-only，防止客户端读取 cookie 进而传递给攻击者
- 当转义和过滤都不能起到很好的作用时，可以考虑配合使用`CSP(Content Security Policy)`
  - 简单的说:`CSP`是一组安全协议策略，你可以配置你的网络服务器返回  `Content-Security-Policy`  HTTP 头部来告诉浏览器开启某些安全策略，用于检测并削弱某些特定类型的攻击
  - 具体可参考[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)

## 总结

- XSS 是一种利用用户输入的安全漏洞的代码注入攻击的行为
- 主要有三种类型
  - 持久型
    - 恶意代码存储在服务端
  - 反射型
    - 恶意代码存储在 URL
    - 需要诱导用户点击
  - DOM-based
    - 前端需要关注
    - 前端使用了危险的方法将不可信数据插入页面导致的
- 防止
  - 输入侧
    - 转义特殊字符
      - 需根据不同上下文使用不同转义方法
      - 原则上，只要用户的输入会被包含进页面，就应该执行转义
    - 过滤
      - 建立白名单，过滤危险字符的输入
  - 输出侧
    - 针对持久、反射型
      - 对服务端返回的 html 做充分转义
      - 用前端渲染
        - 需要预防 DOM-based 型 xss 攻击
    - 针对 DOM-based 型 xss 攻击
      - 前端避免使用危险的方法将不相数据插入页面
  - 其它
    - 预防 xss，需要前后端协同配合
    - 针对窃取 cookie，可以设置 http-only
    - 必要时候可以开启 CSP 策略，增加一层安全防护

## 彩蛋

- xss 小游戏
- [https://xss-game.appspot.com/](https://xss-game.appspot.com/)
- 参考答案
  - [https://blog.csdn.net/abc_12366/article/details/82054946](https://blog.csdn.net/abc_12366/article/details/82054946)

## 参考

- [https://blog.csdn.net/abc_12366/article/details/82054946](https://blog.csdn.net/abc_12366/article/details/82054946)
- [https://zhuanlan.zhihu.com/p/21308080](https://zhuanlan.zhihu.com/p/21308080)
- [https://juejin.im/post/5bad9140e51d450e935c6d64#heading-29](https://juejin.im/post/5bad9140e51d450e935c6d64#heading-29)

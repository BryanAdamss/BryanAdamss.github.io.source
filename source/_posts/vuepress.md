---
title: vuepress
tags:
  - vuepress
categories:
  - 前端
date: 2020-03-26 15:33:12
---

# vuepress

## 介绍

- `VuePress`是一个静态网站生成器
- 由`VuePress`生成的静态网站是一个由`Vue`、`vue-router`、`webpack`驱动的`SPA单页应用`
- 构建时，会采用预渲染的机制，生成所有静态页面，所以有非常好的加载性能和`SEO`
- 本文大部分内容摘抄自`vuepress`官网，加了一些自己的理解，明确了一些语义
  - 一来加深印象
  - 二来方便自己查阅

## 安装

- 确保`node >=8`
- 本地安装即可

```bash
yarn add -D vuepress # 或者：npm install -D vuepress
```

## 目录结构

```
.
├── docs // 目标目录targetDir，vuepress基于此目录工作
│   ├── .vuepress (可选的) // vuepress配置目录
│   │   ├── components (可选的) // 内部的vue组件会自动被注册为全局的异步组件
│   │   ├── theme (可选的) // 配置的自定义主题
│   │   │   └── Layout.vue // 自定义主题必须要的布局组件
│   │   ├── public (可选的) // 公用资源
│   │   ├── styles (可选的) // 自定义样式
│   │   │   ├── index.styl // 全局样式文件
│   │   │   └── palette.styl // stylus 颜色常量
│   │   ├── templates (可选的, 谨慎配置)
│   │   │   ├── dev.html // 开发环境的 HTML 模板文件
│   │   │   └── ssr.html // vue ssr 的模板文件
│   │   ├── config.js (可选的) // 配置文件
│   │   └── enhanceApp.js (可选的) // 用来增强客户端实例用，可在此文件中使用三方UI库（如element-ui）
│   │
│   ├── README.md
│   ├── guide
│   │   └── README.md
│   └── config.md
│
└── package.json
```

- `docs`是`vuepress`约定的目标目录(工作目录)`targetDir`
  - 路由的生成是相对于`targetDir`
- `.vuepress`目录是`vuepress`的配置目录
- `docs`目录下除了`.vuepress`文件夹以外的所有`.md`都将被编译为一个页面
  - 所有`README.md`、`index.md`都会被编译为`index.html`
  - 上述目录对应的默认路由为

| 文件的相对路径     | 页面路由地址   |
| ------------------ | -------------- |
| `/README.md`       | `/`            |
| `/guide/README.md` | `/guide/`      |
| `/config.md`       | `/config.html` |

- `vuepress`遵循约定大于配置原则，并且本身自带一套默认主题[@vuepress/theme-blog](https://github.com/vuepressjs/vuepress-theme-blog)(主要用来书写文档)，所以上述的目录基本都是可选的；

## scripts

- 主要提供了

```json
{
  "scripts": {
    "dev": "vuepress dev docs", // 开发，docs为targetDir
    "build": "vuepress build docs" // 编译，docs为targetDir
  }
}
```

## 配置

### 基本配置

- `targetDir/.vuepress/config.js`是`vuepress网站`必须有的一个配置文件

### 主题配置

- `vuepress`内置一个默认主题`@vuepress/plugin-blog`，就是`vue`官网使用的主题
  - 它适合用来书写技术文档

#### 配置主题

- 配置默认主题`@vuepress/plugin-blog`
- `targetDir/.vuepress/config.js`中不要指定`theme`字段(让其使用默认的主题)，直接配置`themeConfig`字段即可

```js
// .vuepress/config.js
{
    themeConfig:{
        logo: '/assets/img/logo.png',
    }
}
```

- 默认主题的配置项可以查看
  - [https://vuepress.vuejs.org/zh/theme/default-theme-config.html#%E9%A6%96%E9%A1%B5](https://vuepress.vuejs.org/zh/theme/default-theme-config.html#%E9%A6%96%E9%A1%B5)
- 配置自定义主题
  - 首先安装自定义主题
  - 然后用`theme`字段指定需要的`theme`主题
  - 配置`themeConfig`字段来控制自定义主题样式

```js
// .vuepress/config.js
{
  theme: 'modern-blog',
  themeConfig: {
    // vuepress-theme-modern-blog主题的配置
  }
}
```

### 应用级配置

- `vuepress`生成的网站其实就一个`vue`应用，所以可以对`vue`实例做一些增强
  - 例如使用 UI 框架
- 创建一个`.vuepress/enhanceApp.js`并导出一个函数，`vuepress`会读取此文件

```js
// .vurepress/enhanceApp.js

import ElementUI from "element-ui";
import "element-ui/lib/theme-chalk/index.css";

// 使用异步函数也是可以的
export default ({
  Vue, // VuePress 正在使用的 Vue 构造函数
  options, // 附加到根实例的一些选项
  router, // 当前应用的路由实例
  siteData, // 站点元数据
  isServer, // 当前应用配置是处于 服务端渲染 或 客户端
}) => {
  // ...做一些其他的应用级别的优化
  Vue.use(ElementUI);
};
```

## 静态资源

- 每个`.md`文件都会被编译成一个`vue组件`，里面引用到的资源会被`webpack的loader`处理
  - 例如图片资源，会被  `url-loader`  和  `file-loader`  处理

### 相对路径

- 所以可以通过相对路径的形式来引用资源
- 例如如下目录，可以在`a.md`中引用`assets`中的图片

```js
.
├── docs // 目标目录targetDir，vuepress基于此目录工作
│   │
│    ├──  .vuepress/
│   ├── README.md
│   ├── guide
│   │   ├── test.jpg
│   │   └── README.md
│   └── config.md
│
└── package.json

// docs/guide/README.md
// 引用test.jpg
![test.jpg](./test.jpg)

```

### 路径别名

- 可以通过配置`webpack alias别名`，然后使用`~别名`来告诉`webpack`加载某个资源

```js
// .vuepress/config.js

module.exports = {
  // 配置别名
  configureWebpack: {
    resolve: {
      alias: {
        '@alias': 'path/to/some/dir'
      }
    }
  }
}

// 在.md中通过别名引用资源
![test.png](~@alias/test.png)
```

### 加载依赖包中资源

- 引用资源时，可以通过`~`前缀来明确地指出这是一个 `webpack` 的模块请求，这将允许你通过 `webpack`来引用`npm`包中的资源

```md
![Image from dependency](~some-dependency/image.png)
```

### 公共资源

- 项目级的公共资源，例如`favicon.ico`，可以直接放在`.vuepress/public`文件夹中，它们最终会被复制到生成的静态文件夹中
  - 可以通过项目根路径来引用

```md
![favicon](/favicon.ico)
```

- 如果网站没有部署到`根路径`上，则需要设置`.vurepress/config.js`中的`base`基础路径，这个时候引用项目根路径的公共资源，就需要填写完整路径
  - 例如
  - 项目部署到网站的`/blog目录(非根路径)下`
  - 所以`favicon.ico`的引用路径需要调整成`/blog/favicon.ico`
- `vuepress`提供了一个快捷方法来引用部署基础路径；使用`$withBase`来引用，`vurepss`会保证`$withBase`返回正确的路径

```js
<img :src="$withBase('/foo.png')" alt="foo">
```

## markdown 常规能力

### 链接

- `vuepress`中每篇文章都会生成一个链接，在`md`内部可以使用常规`md`链接跳转形式来访问

```
// 有下面的目录
. ├─ README.md
  ├─ foo
  │ ├─ README.md
  │ ├─ one.md
  │ └─ two.md
  └─bar
  ├─ README.md
  ├─ three.md
  └─ four.md

// 可以使用下面的方式跳转
[Home](/)
<!-- 跳转到根部的 README.md -->

[foo](/foo/)
<!-- 跳转到 foo 文件夹的 index.html -->

[foo heading](./#heading)
<!-- 跳转到 foo/index.html 的特定标题位置 -->

[bar - three](../bar/three.md)
<!-- 具体文件可以使用 .md 结尾（推荐） -->

[bar - four](../bar/four.html)
<!-- 也可以用 .html -->
```

- 外部链接
  - 所有外部链接(`http/https`开头)，`vuepress`会自动为通过`OutboundLink`组件添加上`外部链接`标识
  - outside.png
    - ![outside.png](outside.png)

### toc、Emoji、自定义容器

- 可以生成目录(`toc`)，使用`emoji表情`、自定义容器
- `toc`

```html
// xxx.md // 目录 [[toc]] // emoji :tada: :100: // 自定义容器 ::: tip
这是一个提示 ::: ::: warning 这是一个警告 ::: ::: danger 这是一个危险警告 :::
::: details 这是一个详情块，在 IE / Edge 中不生效 :::
```

- toc.png
  - ![toc.png](toc.png)
- `emoji`

```html
// xxx.md // emoji :tada: :100:
```

- emoji.png
  - ![emoji.png](emoji.png)
- 自定义容器
  - `vuepress`使用`vuepress-plugin-container`插件实现自定义容器

```html
// xxx.md // 自定义容器 ::: tip 这是一个提示 ::: ::: warning 这是一个警告 :::
::: danger 这是一个危险警告 ::: ::: details 这是一个详情块，在 IE / Edge
中不生效 :::
```

- tips.png
  - ![tips.png](tips.png)
- 其他常规功能可以参考[https://vuepress.vuejs.org/zh/guide/markdown.html](https://vuepress.vuejs.org/zh/guide/markdown.html)

## markdown 增强

- `vuepress`驱动的网站，`md`文件除了可以用常规的`md`语法和特性，还能使用一些`vue`赋予的能力

### 基本原理

- `vuepress`使用`markdown-it`来渲染`markdown`，将其编译成`html`,然后通过`vue-loader`，接着做为一个`Vue组件`传给`vue-loader`
- 例子
  - `test.md`通过`markdown-it`编译成 html 字符串
  - `vuepress`将其组装成一个`vue`组件
  - 组装好的`vue组件`传递给`vue-loader`、结合`SSR`生成最终的`test.html`
- 所以在`vuepress`驱动的网站中，可以`md`文件中使用`vue`的一些特性，例如

### 插值表达式

- 可以在`md`中使用`vue的插值表达式`

```html
// xxx.md 表达式的值为{{ 1 + 1 }} // 输出 表达式的值为2
```

### 转义

- 由于`md`中可以使用`vue`的插值表达式，所以在特定情况下，如果需要展示`左右大括号{ }`，则需要使用转义

```html
::: v-pre `{{ This will be displayed as-is }}` :::
```

### 内置指令

- 可以在`md`中使用`vue的内置指令`

```html
// xxx.md
<span v-for="i in 3">{{ i }} </span>

// 输出 1 2 3
```

### 访问网站的元数据

- 可以在`md`中访问网站的元数据

```js
// xxx.md
{{ $page }}

// 输出
{
  "path": "/using-vue.html",
  "title": "Using Vue in Markdown",
  "frontmatter": {}
 }
```

### 使用组件

- `.vuepress/components`下的组件会被注册为全局的异步组件，可以在`md`文件中使用

```js
.
└─ .vuepress
   └─ components
      ├─ demo-1.vue
      ├─ OtherComponent.vue
      └─ Foo
         └─ Bar.vue

// xxx.md
<demo-1/>
<OtherComponent/>
<Foo-Bar/>

```

- 使用组件时，始终保持`PascalCase`命名风格

### 在标题中使用 vue 组件

- 可以在`md`的标题中使用`vue组件`
- markdown-vue.png
  - ![markdown-vue.png](markdown-vue.png)
- 输出的 HTML 由  `markdown-it`  完成。而解析后的标题由 `VuePress` 完成，用于侧边栏以及文档的标题。

### 注意

- 由于`vuepress`会使用`SSR`生成静态页面
- 所以` Vue` 相关代码，需要确保只在  `beforeMount`  或者  `mounted`  访问浏览器 / `DOM` 的 `API`。
  - 参考[https://ssr.vuejs.org/zh/guide/universal.html](https://ssr.vuejs.org/zh/guide/universal.html)
- 必要时，可以使用`ClientOnly`内置组件，包裹对`SSR`不友好的组件，或者**在合适的生命周期钩子中动态载入相关组件，保证他们能访问到 DOM API**

```

// 使用ClientOnly组件
<ClientOnly>
  <NonSSRFriendlyComponent />
</ClientOnly>

// 或者在能访问到DOM API的钩子中导入组件，保证他们能访问到DOM API
<script>
  export default {
    mounted() {
      import("./lib-that-access-window-on-import").then((module) => {
        // use code
        document.querySelector(".test").className = "hello";
      });
    },
  };
</script>
```

### markdown 插槽

- `<Content/>`组件会保存`md`文件中的普通内容
- `markdown`插槽可以分发`md`内容，方便灵活布局

```
// .vuepress/theme/Layout.vue

<template>
  <div class="container">
    <header>
      <content slot-key="header" />
    </header>
    <main>
      <content />
    </main>
    <footer>
      <content slot-key="footer" />
    </footer>
  </div>
</template>

// xxx.md
// 内容会被分发到<content slot-key="header" />
::: slot header
# Here might be a page title
:::

- A Paragraph
- Another Paragraph

// 内容会被分发到<content slot-key="footer" />
::: slot footer
 Here's some contact info
:::


// xxx.html
<div class="container">
  <header>
    <div class="content header">
      <h1>Here might be a page title</h1>
    </div>
  </header>
  <main>
    <div class="content default">
      <ul>
        <li>A Paragraph</li>
        <li>Another Paragraph</li>
      </ul>
    </div>
  </main>
  <footer>
    <div class="content footer">
      <p>Here's some contact info</p>
    </div>
  </footer>
</div>
```

- 注意
  - 每个被分发的内容都会被包裹在`div`中，并且`div`的`class`为`content`和`slot`的名字

## 样式

### 预处理器

- `vuepress`项目默认采用`stylus`处理样式，所以内置了`stylus`及`stylus-loader`
- 如果需要使用其它预处理器(`sass、scss、less、pug`)，可直接安装相关依赖包，`vuepress`已经包含相关配置，开箱即用
- `yarn add -D sass-loader node-sass`

```
<style lang="sass">
  .title
    font-size: 20px
</style>
```

### 在页面使用页面级样式及脚本

- 可以直接在`md`文件中添加`<script>`、`<style>`标签，这两个标签会被`vuepress`提取出来并当成`vue`单文件组件中的`<style>`、`<script>`使用，所以可能利用这个特性定义页面级样式及逻辑

```html
// xxx.md

<style lang="scss">
  .test {
    color: red;
  }
</style>
```

## 内置组件

- 内置组件可以直接在`md`文件中使用
- `OutboundLink`
  - 标识外部链接用，`vuepress`会自动在外部链接后面生成这个标识
- `ClientOnly`
  - 标识内部组件需要访问 DOM API，不需要在服务端进行渲染
- `Content`
  - 包含页面 md 的所有渲染内容
  - 自定义布局(主题)时，很有用
  - 具体可参考[https://vuepress.vuejs.org/zh/guide/using-vue.html#content](https://vuepress.vuejs.org/zh/guide/using-vue.html#content)

```html
<template>
    
  <div class="c-Container">
        
    <div class="c-Container-hd">自定义头部</div>
        
    <div class="c-Container-bd">
            <content />

          
    </div>
             
    <div class="c-Container-ft">自定义页脚</div>
      
  </div>
</template>
```

- `Badge`
  - 徽章，主要用来给标题添加一些标识
  - 用法可见[https://vuepress.vuejs.org/zh/guide/using-vue.html#%E5%9C%A8%E6%A0%87%E9%A2%98%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%BB%84%E4%BB%B6](https://vuepress.vuejs.org/zh/guide/using-vue.html#%E5%9C%A8%E6%A0%87%E9%A2%98%E4%B8%AD%E4%BD%BF%E7%94%A8%E7%BB%84%E4%BB%B6)

```html
### Badge <Badge text="beta" type="warning" /> <Badge text="默认主题" />
```

- badge.png
  - ![badge.png](badge.png)

## 多语言

- 要支持多语言，首先要有翻译好的其他语言版本`md`文件
- 不同语言的`md`文件目录、文件名要保持完全一致

```
docs
├─ README.md
├─ foo.md
├─ nested
│  └─ README.md
└─ zh
   ├─ README.md
   ├─ foo.md
   └─ nested
      └─ README.md

// 外层为原版md目录，zh目录下为中文版本，二者目录结构、文件名完全一致
```

- 其次，需要配置`vuepress`的`locales`字段，来告诉不同语言环境，使用哪个目录下的`md`文件

```js
// .vuepress/config.js

module.exports = {
  locales: {
    // 键名是该语言所属的子路径
    // 作为特例，默认语言可以使用 '/' 作为其路径。
    "/": {
      lang: "en-US", // 将会被设置为 <html> 的 lang 属性
      title: "VuePress",
      description: "Vue-powered Static Site Generator",
    },
    "/zh/": {
      lang: "zh-CN",
      title: "VuePress",
      description: "Vue 驱动的静态网站生成器",
    },
  },
};
```

- 还可以为不同的语言版本使用不同的配置(需要看主题是否提供了此能力)；默认主题支持对不同语言使用不同配置

```js
// .vuepress/config.js

module.exports = {
  locales: {
    /* ... */
  },
  themeConfig: {
    locales: {
      "/": {
        selectText: "Languages",
        label: "English",
        ariaLabel: "Languages",
        editLinkText: "Edit this page on GitHub",
        serviceWorker: {
          updatePopup: {
            message: "New content is available.",
            buttonText: "Refresh",
          },
        },
        algolia: {},
        nav: [{ text: "Nested", link: "/nested/", ariaLabel: "Nested" }],
        sidebar: {
          "/": [
            /* ... */
          ],
          "/nested/": [
            /* ... */
          ],
        },
      },
      "/zh/": {
        // 多语言下拉菜单的标题
        selectText: "选择语言",
        // 该语言在下拉菜单中的标签
        label: "简体中文",
        // 编辑链接文字
        editLinkText: "在 GitHub 上编辑此页",
        // Service Worker 的配置
        serviceWorker: {
          updatePopup: {
            message: "发现新内容可用.",
            buttonText: "刷新",
          },
        },
        // 当前 locale 的 algolia docsearch 选项
        algolia: {},
        nav: [{ text: "嵌套", link: "/zh/nested/" }],
        sidebar: {
          "/zh/": [
            /* ... */
          ],
          "/zh/nested/": [
            /* ... */
          ],
        },
      },
    },
  },
};
```

## 全局计算属性

- `vuepress`将一些站点、页面信息注入到`Vue.prototype`上，所以我们可以直接在`md`文件、`自定义vue组件`中直接使用

### \$site

- 包含站点的信息

```js

{
  "title": "VuePress",
  "description": "Vue 驱动的静态网站生成器",
  "base": "/",
  "pages": [
    {
      "lastUpdated": 1524027677000,
      "path": "/",
      "title": "VuePress",
      "frontmatter": {}
    },
    ...
  ]
}

```

### \$page

- 包含当前页面的信息

```js

{
  "title": "Global Computed",
  "frontmatter": {
    "sidebar": "auto"
  },
  "regularPath": "/zh/miscellaneous/global-computed.html",
  "key": "v-bc9a3e3f9692d",
  "path": "/zh/miscellaneous/global-computed.html",
  "headers": [
    {
      "level": 2,
      "title": "$site",
      "slug": "site"
    },
    {
      "level": 2,
      "title": "$page",
      "slug": "page"
    },
  ]
 }

```

### \$frontmatter

- `$page.frontmatter`的一个引用

```
$page.frontmatter === $frontmatter  // true
```

### \$localePath

- 当前页面的 `locale` 路径前缀，默认值为  `/`
- 返回`config`中的`base`字段值？待确认

### $lang、$title、$description、$themeConfig

- `$lang`当前页面的语言，默认`en-US`
- `$title`当前页面`<title>`标签的值
- `$description`用于当前页面的  `<meta name="description" content="..."`>  的  `content`  值
- `$themeConfig`是`.vupress/config.js`中的`themeConfig`
  - `.vupress/config.js`导出的内容称为`siteConfig`

## Front Matter

- `vuepress`支持[YAML front matter](https://jekyllrb.com/docs/front-matter/)
  - `Front matter`可以理解为对`md`文件添加一些特殊字段(元信息)，用来自定义当前页面；用来增强`md`的表现力
- 在`md`文件中，一般开头`---`之间的内容为`Front matter`
- `vuepress`预定义了一些`front matter`，也可以自定义，然后使用`$frontmatter`来引用
- front-matter.png
  - ![front-matter.png](front-matter.png)

### 预定义 front matter

- `title`
  - 当前页面标题
- `lang`
  - 当前页面语言
- `description`
  - 当前页面描述
- `layout`
  - 当前页面的布局组件(默认为`Layout.vue`)
- `permalink`
  - 当前页面永久链接格式
- `metaTitle`
  - 重写页面的 title
- `meta`
  - 当前页面额外的`meta`标签
- meta.png
  - ![meta.png](meta.png)

### 默认主题预定义变量

- 默认主题也提供了很多预定义变量用来控制当前页面一些布局组件的展示
- 例如`navbar`、`sidebar`等等
- 具体可参考[https://vuepress.vuejs.org/zh/theme/default-theme-config.html#%E9%A6%96%E9%A1%B5](https://vuepress.vuejs.org/zh/theme/default-theme-config.html#%E9%A6%96%E9%A1%B5)

## 永久链接 permalink

- 如果利用`vuepress`生成一个博客，那永久链接是必不可少的，为每一篇文章生成一个永久链接

### 配置 permalink

- `permalink`拥有一些变量
  - `:year`
    - 文章发布的年份 (4 数字)
  - `:month`
    - 文章发布的月份 (2 数字)
  - `:i_month`
    - 文章发布的月份 (前面不带 0)
  - `:day`
    - 文章发布的日份 (2 数字)
  - `:i_day`
    - 文章发布的日份 (前面不带 0)
  - `:slug`
    - 不带扩展名的文件名
  - `:regular`
    - 默认链接生成方式
    - [https://vuepress.vuejs.org/zh/guide/directory-structure.html#%E9%BB%98%E8%AE%A4%E7%9A%84%E9%A1%B5%E9%9D%A2%E8%B7%AF%E7%94%B1](https://vuepress.vuejs.org/zh/guide/directory-structure.html#%E9%BB%98%E8%AE%A4%E7%9A%84%E9%A1%B5%E9%9D%A2%E8%B7%AF%E7%94%B1)

```js
// .vuepress/config.js
module.exports = {
  permalink: "/:year/:month/:day/:slug",
};
```

- 在页面中，也可以用`front matter`为页面单独配置一个永久链接
- link.png
  - ![link.png](link.png)

## 部署到 github pages

- 第一步确定，静态存放目录
  - 例如`https://github.com/bryanadamss/testrepo`
- 确定仓库名`repoName`
  - `testrepo`
- 配置`siteConfig.base`为`repoName`

```js
// .vuepress/config.js
module.exports = {
  base: "/testrepo/",
};
```

- 设置仓库为`github pages`
- 编译文件
  - 运行`npm run docs:build`
- 上传静态文件
  - 可以通过`shell`脚本上传，避免重复劳动

```bash

#!/usr/bin/env sh

# 确保脚本抛出遇到的错误set -e

# 生成静态文件npm run docs:build

# 进入生成的文件夹cd docs/.vuepress/dist

# 如果是发布到自定义域名# echo 'www.example.com' > CNAME

git init
git add -A
git commit -m 'deploy'

# 如果发布到 https://<USERNAME>.github.io# git push -f git@github.com:<USERNAME>/<USERNAME>.github.io.git master

# 如果发布到 https://<USERNAME>.github.io/<REPO># git push -f git@github.com:<USERNAME>/<REPO>.git master:gh-pages

cd -

```

- 具体可参考
  - [https://vuepress.vuejs.org/zh/guide/deploy.html#github-pages](https://vuepress.vuejs.org/zh/guide/deploy.html#github-pages)

## 参考

- [https://vuepress.vuejs.org/zh/guide/](https://vuepress.vuejs.org/zh/guide/)

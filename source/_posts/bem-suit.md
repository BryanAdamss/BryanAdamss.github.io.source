---
title: bem-suit
tags:
  - css
  - bem
  - suit
categories:
  - 前端
date: 2019-03-11 23:18:16
---

# 样式类的命名

## BEM

### 基本组成

- B(block)-每个模块可视为一个 block
- E(element)-代表.block 的后代，用于形成一个完整的.block 的整体
- M(modifer)-代表.block 的不同状态或不同版本。如 current active 等特殊样式

### 命名约定

```css
.block-name {
}

.block-name__element-name {
}

.block-name--modifier-name {
}
```

- 长单词采用-分隔,如 `siteName` 用 `site-name`
- `BE` 之间采用`__(双下划线)`分隔
- `M` 用`--`标识

### 注意事项

- **注意:并非所有类都必须采用 bem 命名方式**
- 当一个元素 `element` 总是固定出现在某块下面，则可采用 bem 命名方式。若只是恰巧此次出现在某块下面，则不必采用 `bem`。如 `logo` 可直接使用`.logo` 而不必采用`.header{} .header__logo{}`来标识。因为 `logo` 只是恰巧出现在 `header` 中,`logo` 还可以出现在 `footer` 等其它地方；
- 单独的`css` 样式，可不用`bem`命名方式如
  - `.fr{float:right;}`不必采用 `bem`

## SUIT

### BEM 的改进

- `BEM`采用`-`以及`__`来分隔容易造成混乱,`SUIT`对其改进，思想是一样的，但是命名约定稍有改变，`SUIT`多采用`帕斯卡,驼峰`命名方法

### 基本组成

- Utilities(公用样式)-如 clearfix
- Component(公用组件)-类似 BEM 中的 block
- Descendants(后代)-类似 BEM 中的 element
- Status(状态)-组件的各种状态，如有需要也可用在后代上，不可单独使用(添加样式)
- Modifer(修饰符)-类似 BEM 中的 M,不过它只能用在 Components 上不能用在
- Descendants 上,也不能表示组件状态，状态由 Status 标识

### 命名约定

```css
.u-utility {
}

.ComponentName {
}

.ComponentName-descendantName {
}

.ComponentName--modifierName {
}

.ComponentName.is-someState {
}
```

- 公用样式
  - 添加 u-前缀,并采用驼峰命名，如 u-clearFix{},u-floatRight{}，还可以添加 sm、md、lg 等响应式如.u-sm-floatR
- 组件
  - 采用帕斯卡命名，如.SearchForm{} 避免冲突可添加命名空间，如.cgh-SearchForm{}；**组件后只能跟一个后代、不能跟多个**
- 后代
  - 采用-分隔，驼峰命名。如.SearchForm-submitBtn{}
- 修饰符
  - 采用--标识，只能跟在组件后，不能跟在后代后面。.SearchForm--advanced{}
- 状态
  - 用驼峰命名，并在驼峰上添加.is-前缀。用在组件后面，如有需要也可以跟在后代后面；
  - 不要单独使用这些状态类(添加样式)，配合组件/后代来实现某种样式如.is-mouseOver{}
  - 使用时.SearchForm.is-mouseOver{} 注意：中间无空格，不是.SearchForm 空格.is-mouseOver{}；它是表示同时拥有.SearchForm 和.is-mouseOver{}类的标签;通过这种方式能确定处于特定状态下的标签,等于将 is-mouseOver 绑定到 SearchForm 上了

### 示例

| 好的                                                                   |             错误              |                                               原因 |
| ---------------------------------------------------------------------- | :---------------------------: | -------------------------------------------------: |
| .u-clearfix{}、.u-sm-clearfix{}、.u-floatL{}                           |                               |                                                    |
| .Tree、.cgh-Tree                                                       |       .tree、.cgh_Tree        |                组件首字母应大写；命名空间应用-连接 |
| .Tree-list、.Tree-listHd                                               | .Tree-list-item、.Tree-listhd | 组件后面只能跟一个后代，不能跟多个；后代应该用驼峰 |
| .Tree--large                                                           |       .Tree-list--large       |                             修饰符应只跟在组件后面 |
| .Tree.is-active{display:block;}、.Tree-item.is-active{display:block; } |  is-active{display:block; }   |                           不要给状态单独添加样式面 |


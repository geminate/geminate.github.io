---
layout: post
title: position:sticky 粘性定位
categories: css
---

## 一. 说明
sticky 翻译过来就是 粘性的意思，所以称之为粘性定位。MDN 对其的说明如下：

> 盒位置根据正常流计算(这称为正常流动中的位置)，然后相对于该元素在流中的 flow root（BFC）和 containing block（最近的块级祖先元素）定位。在所有情况下（即便被定位元素为 table 时），该元素定位均不对后续元素造成影响。当元素 B 被粘性定位时，后续元素的位置仍按照 B 未定位时的位置来确定。position: sticky 对 table 元素的效果与 position: relative 相同。

描述的比较复杂，从我个人的理解来讲，这是一个结合了 `position:relative` 和 `position:fixed` 的定位方式，在元素在跨越特定阈值前表现得和 `position:relative` 一致，之后和 `position:fixed` 的表现一致。

```css
#one { position: sticky; top: 10px; }
```

在视口滚动到元素 top 距离小于 10px 之前，元素为相对定位。之后，元素将固定在与顶部距离 10px 的位置(fixed)，直到 viewport 视口回滚到阈值以下。

## 二. 兼容性

![兼容性](https://geminate.github.io/assets/images/2018/sticky.png)

2018/09/26 ,大多数浏览器都已经兼容了该写法。

## 三. 使用场景

### 1. 浮动边栏

比如本人博客中，右侧的文章导航如下图

![sticky-blog.gif](https://geminate.github.io/assets/images/2018/sticky-blog.gif)

文章和导航的父级元素 使用 `display:flex` ，对右侧的导航设置 `css position: sticky; top: 100px;` ,非常方便的的实现了粘性效果。如果不使用 这个css 属性，我们往往需要 鉴定页面的滚动事件 动态修改 css 来实现，比较麻烦。

### 2. 字母索引

![sticky-n.gif](https://geminate.github.io/assets/images/2018/sticky-n.gif)

实现方法类似



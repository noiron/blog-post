---
layout: post
title: 使用 ::before 给列表添加样式
date:   2016-02-26
tags: [css]
categories: 
---

::before会创建一个**伪元素（pseudo-element）**作为选定元素的第一个子元素。这个元素默认为行内元素。

## 语法

```css
/* CSS3 语法 */
element::before { style properties }

/* CSS2 的过时语法（仅用于支持IE8） */
element:before  { style properties }

/* 在每个<p>元素前插入内容 */
p::before { content: "Hello world!"; }
```

注意CSS2和CSS3的写法有一个冒号的差别。

<!-- more -->

## 一个简单的例子

这个例子利用`::before`伪元素来插入引号。

HTML内容：

```html
<q>Some quotes</q>, he said, <q>are better than none</q>.
```

CSS内容

```css
q::before {
  content: "«";
  color: blue;
}
q::after { 
  content: "»";
  color: red;
}
```

## 给列表添加样式

HTML内容：

```html
<h1>学习中的前端书籍列表</h1>
<ul>
  <li>精通CSS</li>
  <li>JavaScript高级程序设计</li>
  <li class="done">JavaScript语言精粹</li>
  <li>jQuery基础教程</li>
  <li>JavaScript DOM 编程艺术</li>
</ul>
```

CSS内容：

```css
li {
  list-style-type: none;
  position: relative;
  margin: 2px;
  padding: 0.5em 0.5em 0.5em 2em;
  background: lightgrey;
  font-family: "Microsoft Yahei", sans-serif;
}

li.done {
  background: #CCFF99;
}

li.done::before {
  content: '';
  position: absolute;
  border-color: #009933;
  border-style: solid;
  border-width: 0 0.3em 0.25em 0;
  height: 1em;
  width: 0.5em;
  top: 1.3em;
  left: 0.6em;
  margin-top: -1em;
  transform: rotate(45deg);
}
```

JavaScript内容：

```javascript
// 单击列表项给其添加或删除一个'done'类
var list = document.querySelector('ul');
list.addEventListener('click', function(ev) {
  if (ev.target.tagName === 'LI') {
    ev.target.classList.toggle('done');
  }
}, false);
```

这里列表项前面的勾号不是使用图片实现的，而是通过给元素的右侧和下侧添加边框并旋转来实现的。

> [Codepen上的demo](https://codepen.io/noiron/pen/BKBXvq)


***

## 参考资料

> [MDN文档](https://developer.mozilla.org/en-US/docs/Web/CSS/::before)
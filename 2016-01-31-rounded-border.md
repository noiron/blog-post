---
layout: post
title: 几种实现圆角框的方法
date:   2016-01-31
tags: [css, css3]
categories: 
---

创建圆角框有几种方法，每种方法各有优缺点，对这些方法的选择主要取决于实际情况。

## 固定宽度的圆角框

固定宽度的圆角框是最容易创建的，我们只需要两个图像分别用作顶部和底部的背景。

```html
<h1>Simple, Fixed Rounded Corner Box</h1>

<div class="box">
  <h2>Lorem Ipsum</h2>
    <p>Lorem ipsum dolor sit amet</p>
</div>
```

用作背景的两张图片：

顶部 ![top.gif](http://7xqn7m.com1.z0.glb.clouddn.com/rounded-border%2Ftop.gif) 
底部 ![bottom.gif](http://7xqn7m.com1.z0.glb.clouddn.com/rounded-border%2Fbottom.gif)

将这两张图片作为顶部和底部的背景：

```css
.box {
  width: 418px;
  background: #effce7 url("bottom.gif") no-repeat left bottom;
  padding-bottom: 1px;
}

.box h2 {
  background: url("top.gif") no-repeat left top;
  margin-top: 0;
  padding: 20px 20px 0 20px;
}

.box p {
  padding: 0 20px;
}
```

> [固定宽度的圆角框](http://codepen.io/noiron/pen/VedBvK)

<!-- more -->

这种方法对于单色且没有边框的简单框是有效的，如果希望创建一个颜色更特殊的圆角框，可以不在框上设置背景颜色，而是设置一个重复显示的背景图像。

```html
<div class="box">
  <h2>Lorem Ipsum</h2>
    <p class="last">Lorem ipsum dolor sit amet</p>
</div>
```

对应的css:

```css
/* 这里将图片在y方向上进行重复 */
.box {
  width: 418px;
  background: url("tile2.gif") repeat-y;
}

.box h2 {
  background: url("top2.gif") no-repeat left top;
  padding: 20px 20px 0 20px;
}

.box .last {
  background: url("bottom2.gif") no-repeat left bottom;
  padding-bottom: 20px;
}

.box h2, .box p {
  padding-left: 20px;
  padding-right: 20px;
}
```

> [固定宽度的圆角框 2](http://codepen.io/noiron/pen/RrJBMj/)


## 灵活的圆角框

在前面的固定宽度的圆角框中，其高度会随内容而扩展，但是不会水平扩展。如果希望创建灵活的框，则不要用一个图像组成顶部和底部曲线，而是用两个相互重叠的图像。

当框尺寸增加时，大图像有更多的部分显露出来，就实现了框扩展的效果，这个方法有时候被称为**滑动门技术（sliding doors technique）**。

因为这个方法需要更多的图像，我们需要在标记中添加两个额外的无语义元素。

```html
<h1>Flexible Rounded Corner Box Example</h1>

<div class="box">
    <div class="box-outer">
        <div class="box-inner">
            <h2>Lorem Ipsum</h2>
            <p>Lorem ipsum dolor sit amet<p>
        </div>
    </div>
</div>
```

这里需要四个图像：

![flexible](http://7xqn7m.com1.z0.glb.clouddn.com/rounded-border%2Fflexible-rounded-border.jpg)

将图像应用在对应的标签上：

```css
/* 将bottom-left应用于主框div */
.box {
  width: 20em;
  background: url("bottom-left.gif") no-repeat left bottom;
}

/* 将bottom-right.gif应用于外边的div */
.box-outer {
  background: url("bottom-right.gif") no-repeat right bottom;
  padding-bottom: 1em;
}

/* 将top-left.gif应用于内部的div */
.box-inner {
  background: url("top-left.gif") no-repeat left top;
}

/* 将top-right.gif应用于标题 */
.box h2 {
  background: url("top-right.gif") no-repeat right top;
  padding-top: 1em;
}
```

> [灵活的圆角框](http://codepen.io/noiron/pen/BjVPev)


## 使用CSS3添加多个背景图像

前面的示例有些需要添加多余的非语义性标记，这是因为一个元素中只能添加一个背景图像。我们可以通过CSS3实现添加多个背景图像。

```css
.box {
    background-image: url(mtop-left.gif), 
        url(mtop-right.gif), 
        url(mbottom-left.gif), 
        url(mbottom-right.gif);
    background-repeat: no-repeat, no-repeat, no-repeat, no-repeat;
    background-position: top left, top right, bottom left, bottom right;
}
```
> [示例](http://codepen.io/noiron/pen/bEKjXJ)

## 使用 border-radius

在CSS3中我们有了`border-radius`这一个属性，使我们可以不用添加图像，而只需要设置边框角的半径，浏览器就会实现圆角边框的效果。

```css
.box {
    -moz-border-radius: 1em;
    -webkit-border-radius: 1em;
    border-radius: 1em;
}
```

## 使用 border-image

还有一种CSS3新特性是`border-image`，它允许指定一个图像作为元素的边框。可以根据一些简单的百分比规则把图像划分为9个单独的部分，浏览器会自动地使用适当的部分作为边框的对应部分。这种技术称为**九分法缩放（nine-slice scaling）**，有助于避免在调整圆角框大小是通常会出现的失真。

```css
.box {
    -webkit-border-image: url(http://7xqn7m.com1.z0.glb.clouddn.com/rounded-border%2Fcorners.gif) 
    25% 25% 25% 25% / 25px round round;
}
```

***

## 参考资料

>《精通CSS 高级Web标准解决方案（第二版）》第四章
---
layout: post
title:  利用JavaScript在canvas中画一棵树
date:   2015-10-04
tags: [canvas, javascript]
categories: 
---

看到[这个网页](http://kennethjorgensen.com/blog/2014/canvas-trees)中在canvas里绘制一棵树，感到很有趣，于是仿照他的源代码，同样也利用JavaScript生成了一棵树。

在程序中需要两个对象Branch, BranchCollection。Branch中存放当前正在绘制的这一段树枝的信息，BranchCollection中存放的是所有的树枝。

首先对canvas画布进行初始设置：
    
```javascript
var width = window.innerWidth;
var height = window.innerHeight;
var canvas = document.getElementById("canvastree");
canvas.width = width;
canvas.height = height;
```

对初始的branch的数量、半径进行设置：

```javascript
// 设置初始的数量
var n = 2 + Math.random() * 3;
// 设定初始的半径大小
var initialRadius = width / 50;
```

新建一个BranchCollection对象用于放置所有的branch：

```javascript
branches = new BranchCollection();
```

<!-- more -->

这里的BranchCollection对象有如下的几个方法：

- add()：加入一个新元素
- remove()：删除一个元素
- process()：对集合内的每一个元素，依次调用这个元素自己的处理方法

对于创建的BranchCollection对象，将初始的branch依次加入其中，并初始化。

```javascript
for (var i = 0; i < n; i++) {
    branch = new Branch();
    // 以canvas的中点为基准，左右各占一个initialRadius的宽度
    // 根据序号i算出初始x坐标
    branch.x = width/2 - initialRadius + i * 2 * initialRadius / n;
    branch.radius = initialRadius;

    // 将新的branch加入集合中去
    branches.add(branch);
}
```

Branch对象有这些属性：

- x,y：坐标值
- radius：每一条显示在屏幕上的树枝实际上都是由半径逐渐减小的圆形组合而成的，radius就是圆形的半径
- angle：树枝从底部向上延伸时的角度，初始值是PI/2
- speed：一个参数，用于控制树枝延伸时的速度
- generation：当前的树枝是第几代，当出现分叉时，generation会加一
- distance：当前的这一段树枝的长度，用于控制分叉的概率
- fillStyle, shadowColor, shadowBlur：绘图参数

Branch对象的方法有：

- precess()：主要的处理部分，调用其它几个方法
- draw()：在当前的坐标处画出一个圆形
- iterate()：将branch向上延伸，更新坐标值，减小半径，给angle增加一个随机值
- split()：根据distance值判断当前是否可以分叉，如果可以则新建若干个Branch对象加入集合，并删除当前的父代对象
- die()：判断是否需要删除当前对象

最后通过setInterval()函数来生成图像，每隔一个时间间隔对所有branch进行一次遍历处理，画出图形，更新坐标，生成子代等。

```javascript
var interval = setInterval(function() {
    // 对集合内的每个元素依次进行处理
    branches.process();
    if (branches.branches.length == 0) {
        clearInterval(interval);
    }

}, 20);
```

这样我们就完成了在canvas上绘制一棵树的工作。

![A tree picture](https://raw.githubusercontent.com/noiron/canvas-trees/master/image/tree.png)

[GitHub代码地址](https://github.com/noiron/canvas-trees)

这里是我的一个在线的[demo](http://noiron.github.io/canvas-trees)


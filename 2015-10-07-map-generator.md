---
layout: post
title:  利用JavaScript生成一张随机的城市地图
date:   2015-10-07
tags: [canvas, javascript]
categories: 
---

zz85在[这里](http://jsdo.it/zz85/7jWa)用简单的代码就生成了一张随机的城市地图。生成地图与画一棵随机的树十分相似，都是先给出初始的元素，每个元素在满足条件时会生成新的子代，最后由所有的元素共同组成了我们想要的地图或树。

地图中的每一条道路对应程序中的一个Boid对象，Boid对象中用两个向量分别表示道路的起点和终点坐标。程序中的向量是使用 [**Three.js**](http://threejs.org/) 这个库中的`Vector2`对象来表示的。

```javascript
// 使用three.js中的向量来表示
this.start = new THREE.Vector2(x, y);
this.end = new THREE.Vector2(x, y);
```

Boid对象还有这些属性：

- `x,y`：道路上距离起点最远的坐标
- `angle`：道路的角度，会在其父代角度基础上偏转一个随机的角度
- `distance`：这条道路的长度
- `dead`：对象是否已经死亡

Boid还有一个`update`方法，它有如下的几个功能：

- 更新 x,y 坐标

```javascript
this.distance += 2;
x = this.start.x + this.distance * this.dx;
y = this.start.y + this.distance * this.dy;
this.end.set(x, y);
```
<!-- more -->

- 检测相交情况，根据更新后的坐标作图。


在程序中需要创建两个数组用于保存Boid对象，boids中存放当前存活的元素，all_boids存放所有（包括存活和死亡）的元素。产生一个新元素时，会被同时放入两个数组，当元素死亡后，将其从boids中移除。

对于一条道路A，它会一直向前延伸，直到与另一条道路相交，这时将A的状态设置为dead。为了检测相交，需要对all_boids数组中的元素进行遍历。如果与其中的元素B出现了交点，可能是以下几种情况：

- A是B的子代
- B是A的子代
- B的终点在A上
- A在延伸过程中遇上了B

这最后一种情况才是我们所需要的，将交点坐标赋给A的终点，将A从boids数组中删去。以上检查交点的过程发生在update()函数中。

在程序开始时，首先创建四个元素来表示画面的边框。

```javascript
var b1 = new Boid();
var b2 = new Boid();
var b3 = new Boid();
var b4 = new Boid();

b1.dead = b2.dead = b3.dead = b4.dead = true;

b1.start.set(0, 0);
b2.start.set(width, 0);
b3.start.set(width, height);
b4.start.set(0, height);

b1.end = b2.start;
b2.end = b3.start;
b3.end = b4.start;
b4.end = b1.start;

all_boids.push(b1);
all_boids.push(b2);
all_boids.push(b3);
all_boids.push(b4);
```

然后创建第一个`boid`，它的坐标在画面的中间

```javascript
var b = new Boid(width/2, height/2, Math.random() * 2 * Math.PI);
boids.push(b);
all_boids.push(b);
```

调用`setInterval`函数进入循环，首先检查`boids.length`，如果当前没有存活的`boid`，则退出循环，程序完成。否则遍历所有存活的`Boid`，更新其状态。在满足如下的几个条件时生成子代。

1. 没有死亡
2. 只有0.1的概率产生子代
3. 当前所有存活元素的数量小于50

```javascript
    for (i = 0; i < boids.length; i++) {
        var b = boids[i];
        b.update();

        // 产生子代的几个条件：
        // 1. 没有死亡
        // 2. 只有0.1的概率产生子代
        // 3. 当前所有存活元素的数量小于50
        if (!b.dead && Math.random()>0.9 && boids.length < 50) {
            var child = new Boid(b.end.x, b.end.y,
                b.angle + Math.PI * (Math.random() > 0.5 ? 0.5 : -0.5));
            child.parent = b;
           // child.fillStyle = getRndColor();
            boids.push(child);
            all_boids.push(child);
        }
    }
```

最终当存活的Boid数量为零时，程序运行完毕，就得到了一张随机的城市道路地图。当然，现在的地图还只是 2D 的版本，想生成 3D 的城市，可以查看下面的参考资料中zz85的博客。

![一张生成的随机地图](https://raw.githubusercontent.com/noiron/map-generator/master/image/map.png)

[我在GitHub上的代码地址](https://github.com/{{site.github_username}}/map-generator)

在线的[demo](http://{{site.github_username}}.github.io/map-generator)

***

参考资料：

- [Making of Boids and Buildings](http://www.lab4games.net/zz85/blog/2012/11/19/making-of-boids-and-buildings/)
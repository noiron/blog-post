---
layout: post
title:  如何在Three.js中检测两个物体是否了发生碰撞
date:   2015-12-28
tags: [javascript, three.js]
categories: 

---

最近在试着用Three.js做一个简单的赛车游戏，里面有一个需要解决的问题是如何判断两个物体发生了碰撞，比如赛车是否碰上了障碍物或者获得了奖励物品。

## 示例

我找了一些资料，发现了两个示例程序：[第一个示例](https://stemkoski.github.io/Three.js/Collision-Detection.html)、[第二个示例](http://webmaestro.fr/collisions-detection-three-js-raycasting/)。

以上两个程序都是用`THREE.Raycaster`类来解决问题的。


## Raycaster类

[Raycaster](http://threejs.org/docs/#Reference/Core/Raycaster)应该翻译为“光线投射”，顾名思义，就是投射出去的一束光线。

Raycaster的构造函数如下

    Raycaster( origin, direction, near, far ) {
    
    origin — 射线的起点向量。
    direction — 射线的方向向量，应该归一化。
    near — 所有返回的结果应该比 near 远。Near不能为负，默认值为0。
    far — 所有返回的结果应该比 far 近。Far 不能小于 near，默认值为无穷大。

<!-- more -->

## 使用Raycaster进行碰撞检测

用Raycaster来检测碰撞的原理很简单，我们需要以物体的中心为起点，向各个顶点（vertices）发出射线，然后检查射线是否与其它的物体相交。如果出现了相交的情况，检查最近的一个交点与射线起点间的距离，如果这个距离比射线起点至物体顶点间的距离要小，则说明发生了碰撞。

这个方法有一个**缺点**，当物体的中心在另一个物体内部时，是不能够检测到碰撞的。而且当两个物体能够互相穿过，且有较大部分重合时，检测效果也不理想。

还有需要**注意**的一点是：在Three.js中创建物体时，它的顶点（veritces）数目是与它的分段数目相关的，分段越多，顶点数目越多。为了检测过程中的准确度考虑，需要适当增加物体的分段。

检测光线是否与物体相交使用的是`intersectObject`或`intersectObjects`方法:

    .intersectObject ( object, recursive )

    object — 检测该物体是否与射线相交。
    recursive — 如果设置，则会检测物体所有的子代。

相交的结果会以一个数组的形式返回，其中的元素依照距离排序，越近的排在越前：

    [ { distance, point, face, faceIndex, indices, object }, ... ]

这样通过对数组中的元素进行处理，就能得出想要的结果。

`intersectObjects`与`intersectObject`类似，除了传入的参数是一个数组之外，并无大的差别。

## 检测碰撞的代码片段
    
    /**
     *  功能：检测 movingCube 是否与数组 collideMeshList 中的元素发生了碰撞
     * 
     */
    var originPoint = movingCube.position.clone();

    for (var vertexIndex = 0; vertexIndex < movingCube.geometry.vertices.length; vertexIndex++) {
        // 顶点原始坐标
        var localVertex = movingCube.geometry.vertices[vertexIndex].clone();
        // 顶点经过变换后的坐标
        var globalVertex = localVertex.applyMatrix4(movingCube.matrix);
        // 获得由中心指向顶点的向量
        var directionVector = globalVertex.sub(movingCube.position);
        
        // 将方向向量初始化
        var ray = new THREE.Raycaster(originPoint, directionVector.clone().normalize());
        // 检测射线与多个物体的相交情况
        var collisionResults = ray.intersectObjects(collideMeshList);
        // 如果返回结果不为空，且交点与射线起点的距离小于物体中心至顶点的距离，则发生了碰撞
        if (collisionResults.length > 0 && collisionResults[0].distance < directionVector.length()) {
            crash = true;   // crash 是一个标记变量
        }
    }

在Three.js中是使用矩阵来记录3D转换的，每一个Object3D的实例都有一个矩阵，存储了位置position，旋转rotation和伸缩scale。

    var globalVertex = localVertex.applyMatrix4(movingCube.matrix);

这一句代码将物体的本地坐标乘以变换矩阵，得到了这个物体在世界坐标系中的值，处理之后的值才是我们所需要的。


## 我的示例代码

[GitHub示例代码](https://github.com/noiron/race-game-threejs/blob/master/test/cube_collision.html)

[Online Demo](http://noiron.github.io/race-game-threejs/test/cube_collision.html) 按方向键移动方块，点击鼠标左键并拖动更改视角。


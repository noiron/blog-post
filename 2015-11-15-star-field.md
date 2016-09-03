---
layout: post
title:  Three.js入门——画星空（star field）
date:   2015-11-15
tags: [javascript, three.js]
categories: 
---

Three.js是一个很流行的3D JavaScript库。[这里](http://creativejs.com/tutorials/three-js-part-1-make-a-star-field/)有一个three.js的入门教程，在浏览器窗口中画出星空。我按照教程重新实现了一遍，这里的这篇博客把教程大致翻译了一遍。[我的demo](http://noiron.github.io/star-field-threejs/)。

## 变量

定义全局变量：

```javascript
    // three.js的主要部分
    var camera, scene, renderer,
    
    // 跟踪鼠标位置
    mouseX = 0, mouseY = 0,
    
    // 一个数组，用于存储我们的粒子
    particles = [];

    // 初始化
    init();
```
<!-- more -->

## 初始化three.js

为了使用three.js，我们需要在`init()`函数中设置三个主要的对象：

- scene(场景): 场景中包含了所有的3D对象数据
- camera(相机): 相机有位置（position）,旋转（rotation）和视野属性（field of view）
- renderer(渲染器): 决定场景中的一个物体在照相机的视角看来是什么样子


## 照相机（Camera）

```javascript
    function init() {
        // 照相机参数
        camera = new THREE.PerspectiveCamera(80, window.innerWidth/window.innerHeight, 1, 4000);

        // 将相机向后（即屏幕外）移
        camera.position.z = 1000;
```

`Camera`构造器的第一个参数是视野（field of view）。这是一个角度，越大，则表示虚拟的相机镜片越宽。

第二个参数是输出的宽和高之比。这个值必须与CanvasRenderer相一致。

相机只能看见一定范围之内的物体，这个范围是由near和far来确定的，在这里分别为1和4000。因而任何比1近的物体或者比4000远的物体是不会被渲染的。

在默认情况下相机位于3D世界的起始位置（origin）0,0,0（我的上一篇博客[用CSS3绘制一个旋转的立方体](http://noiron.github.io/2015/10/22/css3-cube/)中有关于origin的介绍）。但是你创建的3D物体也会放置在这一点，因而默认值用处并不大。我们需要将相机向后移动一点，即给其一个正的z值（由屏幕内指向外侧）。在这里将其移动1000。


## 场景（Scene）

```javascript
    // the scene contains all the 3D object data
    scene = new THREE.Scene();
    scene.add(camera);
```

注意必须将相机加入到场景中。


## 渲染器（Renderer）

```javascript
    // 加入CanvasRenderer，由渲染器决定场景中的物体看起来如何，并将其画出
    renderer = new THREE.CanvasRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);

    // 将渲染器的canvas domElement加入到body中
    document.body.appendChild(renderer.domElement);
```

这里的CanvasRenderer创建了它自己的DOM元素，这是一个普通的2D canvas对象，我们需要把它加入到文件的body部分才能看见它。我们想让它充满整个浏览器窗口，所以将它的大小设置为`window.innerWidth`和`window.innerHeight`。


## 渲染循环

接下来我们需要做的是产生粒子，加入鼠标移动监听器（mousemove listener）来追踪鼠标位置，最后设定间隔每秒调用`update`函数30次。

```javascript
    makeParticles();

    // add the mouse move listener
    document.addEventListener('mousemove', onMouseMove, false);

    // 每秒渲染30次
    setInterval(update, 1000/30);

其中的`update`函数如下：

    function update() {
        updateParticles();
    
        // 从相机的视角渲染场景
        renderer.render(scene, camera);
    }
```    

`updateParticles`函数会在后面加以解释，它的作用是将粒子向前移动。

直到`renderer.render()`方法调用之前，你是不会看见任何3D物体的。渲染器（renderer）会把所有的东西画到canvas上，它决定了场景中的物体在相机的视角看来是什么样子的。


## 粒子（Particles）

Three.js有三种主要类型的3D对象：三角形（triangles），直线（lines）和粒子（particles）。粒子表示的是3D空间中的一个点，因而最容易使用。它们可以被渲染成2D的图片，比如一个简单的圆或矩形，或是位图图片。关于粒子有一点很重要，它们从任何角度看起来都是一样的，当然根据距离的远近大小会发生变化。


## 创建粒子
    
```javascript
    // creates a random field of Particle object
    function makeParticles() {
        var particle, material;
        
        // 将z坐标从-1000（最远处）逐步增加至1000（相机所在处）
        // 每一个位置加入一个随机的粒子
        for (var zpos = -1000; zpos < 1000; zpos += 20) {

            // 创建一个粒子材质，向其传入颜色及我们定义的粒子渲染函数
            material = new THREE.ParticleCanvasMaterial( {
                // color: 0xffffff,
                color:   getRandomColor(),
                program: particleRender
            });

            // 创建粒子
            particle = new THREE.Particle(material);
            
            // 赋给它一个位于-500至500之间的随机x和y值
            particle.position.x = Math.random() * 1000 - 500;
            particle.position.y = Math.random() * 1000 - 500;
    
            particle.position.z = zpos;
    
            // 将其放大一点
            particle.scale.x = particle.scale.y = 10;
    
            // 把它加入到场景中
            scene.add(particle);

            // 将粒子加入到我们的particles数组中   
            particles.push(particle);
        }
    }
```

在for循环中`zpos`以20为步长从-1000增加至1000。

在循环中，我们创建一个新的材质（material）然后新建一个粒子。粒子有一个position属性，它有x, y, z三个值。

我们给每一个粒子一个随机的x和y位置，将它的z位置设置为`zpos`。

接着将粒子加入到particles数组中，保证能够跟踪所有的粒子并在update循环中对它们进行操作。


## 创建粒子材质（particle material）

所有的3D对象都有材质对象，材质定义了它们是如何画出来的，颜色及透明度之类的属性。我们是这样定义的每个粒子的材质对象：

```javascript
    material = new THREE.ParticleCanvasMaterial( {
            color: 0xffffff,
            program: particleRender
        });     
```

three.js并没有内置圆形粒子材质，所以需要告诉它如何去画一个圆。我们是通过给`particleRender`传递必要的canvas绘图API来做到这一点的。

```javascript
    function particleRender(context) {
        context.beginPath();
        context.arc(0, 0, 1, 0, 2*Math.PI, true);
        context.fill();
    }
```

把一个函数传递给材质（material）似乎有点奇怪，但这实际上是一个非常灵活的系统。它获得了canvas上下文的引用，所以我们可以画出任何想要的形状。同时canvas的坐标系统将会根据粒子进行缩放和平移，因此我们只需要以起点（origin）为中心画图即可。

对于粒子的颜色，可以选取一个固定的颜色，也可以选取随机的颜色。

```javascript
    function getRandomColor() {
        var r = 255*Math.random()|0,
            g = 255*Math.random()|0,
            b = 255*Math.random()|0;
        // return 'rgb(' + r + ',' + g + ',' + b + ')';
        return '0x' + parseInt(r, 16) + parseInt(g, 16) + parseInt( b, 16);
    }
```

## 移动粒子

现在来看一下`updateParticles`函数，它将会在我们每个的更新周期中调用，就在我们进行渲染之前。

```javascript
    // 根据鼠标位置移动所有的粒子
    function updateParticles() {
        
        // 对每个粒子进行迭代处理
        for (var i = 0; i < particles.length; i++) {
            particle = particles[i];
    
            // 根据mouseY值进行移动
            particle.position.z += mouseY * 0.1;
    
            // 如果粒子过近，将其移至后面
            if (particle.position.z > 1000)
                particle.position.z -= 2000;
        }
    }
```

鼠标越低，mouseY值越大，粒子移动速度越快。

最后的效果如图，上下移动鼠标，粒子移动速度会发生变化。

![star-field](https://raw.githubusercontent.com/noiron/star-field-threejs/master/image/2015-11-16-star-field.png)
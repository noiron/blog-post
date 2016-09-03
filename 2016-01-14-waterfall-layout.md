---
layout: post
title: 学习笔记：瀑布流布局
date:   2016-01-14
tags: [javascript, jquery, 网页布局]
categories: 
---


本文是我在学习了[慕课网教程：瀑布流布局](http://www.imooc.com/learn/101)后的学习笔记，具体代码见[GitHub](https://github.com/noiron/learn-front-end/tree/master/waterfall-layout)。

>**瀑布流布局**适合于小数据块，每个数据块内容相近且没有侧重。通常，随着页面滚动条向下滚动，这种布局还会不断加载数据块并附加至当前尾部。

我们可以用多种方法实现瀑布流布局，如纯 JavaScript 方法实现，jQuery 方法以及使用 CSS3 实现。

## 网页结构与布局

先来看一下需要实现的效果。

![瀑布流布局](img/2016-01-14-waterfall.png)

可以看出图片之间等宽但不等高，相互之间有间隔，当页面滚动到底部时，加载更多的图片。

<!-- more -->

我们把所有的内容放在一个`div`中。

```html
    <div id="main">
```

每一张图片放在单独的`div`中。

```html
    <div class="box">
            <div class="pic">
                <img src="../images/0.jpg" />
            </div>
    </div>
    <div class="box">
        <div class="pic">
                <img src="../images/1.jpg" />
        </div>
    </div>
```

然后设置 CSS 样式。内部的图片`box`需要设置为绝对定位，相对其已定位的父元素进行定位，因此先将 `main` 容器设置为相对定位。

图片之间的间距可以用内边距（padding）或外边距（margin）来设置，由于之后我们需要用`offsetWidth`这个属性来获得元素的宽，而`offset`属性是不包括 margin 的，为了方便起见，使用`padding`来设置间距。

```css
    #main {
        position: relative;
    }
    
    .box {
        padding: 15px 0 0 15px;
        float: left;
    }
    
    /* 设置图片的边框和边距 */
    .pic {
        padding: 10px;
        border: 1px solid #cccccc;
        border-radius: 5px;
        box-shadow: 0 0 5px #cccccc;
    }
    
    /* 图片等宽不等高 */
    .pic img {
        width: 165px;
        height: auto;
    }
```
此时的效果如下：

![瀑布流布局](img/2016-01-14-waterfall2.png)

可以看出离想要的效果还有一些差距，我们希望每一行图片排满后，下一行的图片能自动在排在高度最小的一列。这个效果将用 javascript 实现。

> [offsetWidth属性](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth)



## 用纯 JavaScript 实现效果

先定义两个函数：

```javascript
    // 根据class获取元素
    function getByClass(parent, className) {
        var boxArray = [];      // 用来存储获取到的所有class为box的元素
        var oElements = parent.getElementsByTagName('*');
        for (var i = 0; i < oElements.length; i++) {
            if(oElements[i].className == className) {
                boxArray.push(oElements[i]);
            }
        }
        return boxArray;
    }
    
    // 获得数组中某个值得索引
    function getIndex(arr, val) {
        for (var i = 0; i < arr.length; i++) {
            if (arr[i] === val) {
                return i;
            }
        }
    }
```

定义一个`waterfall()`函数，设置页面的列数及居中，计算当前每一列的高度。

```javascript
    function waterfall(parent, box) {
    
        // 将main下的所有class为box的元素取出来
        var oParent = document.getElementById(parent);
        var oBoxes = getByClass(oParent, box);
    
        // 计算整个页面显示的列数（页面宽/box的宽）
        var oBoxWidth = oBoxes[0].offsetWidth;
        var cols = Math.floor(document.documentElement.clientWidth / oBoxWidth);
    
        // 设置main的宽度，并将其居中
        oParent.style.cssText = 'width:' + oBoxWidth * cols + 'px; margin:0 auto';
    
        var heightArray = [];
        for (var i = 0; i < oBoxes.length; i++) {
            if (i < cols) {
                heightArray.push(oBoxes[i].offsetHeight);
            } else {
                var minHeight = Math.min.apply(null, heightArray);
                var index = getIndex(heightArray, minHeight);
                oBoxes[i].style.position = 'absolute';
                oBoxes[i].style.top = minHeight + 'px';
                oBoxes[i].style.left = oBoxWidth * index + 'px';
    
                heightArray[index] += oBoxes[i].offsetHeight;
            }
        }
    }
```

调用`waterfall()`函数，保证每张图片排在正确的位置上。

最后实现滚动加载的功能，当最后一个数据块已经有一半出现在页面上时，进行加载。

```javascript
    // 检测是否具备了加载数据块的条件
    function checkScrollSlide() {
        var oParent = document.getElementById('main');
        var oBoxes = getByClass(oParent, 'box');
        // 最后一个数据块距离页面顶部的距离
        var lastBoxHeight = oBoxes[oBoxes.length-1].offsetTop + Math.floor(oBoxes[oBoxes.length-1].offsetHeight / 2);
        // 滚动条顶部的距离
        var scrollTop = document.body.scrollTop || document.documentElement.scrollHeight;
        //console.log(scrollTop);
        // 页面可视区域的高度
        var height = document.body.clientHeight || document.documentElement.clientHeight;
        return (lastBoxHeight < scrollTop + height);
    }

    // 绑定滚动页面的事件
    var dataInt = {"data":[{"src":'0.jpg'}, {"src":'2.jpg'}, {"src":'3.jpg'}, {"src":'4.jpg'}]};
    window.onscroll = function() {
        if(checkScrollSlide) {
            var oParent = document.getElementById('main');

            // 将数据块渲染到当前页面的尾部
            for (var i = 0; i < dataInt.data.length; i++) {
                var oBox = document.createElement('div');
                oBox.className = 'box';
                oParent.appendChild(oBox);
                var oPic = document.createElement('div');
                oPic.className = 'pic';
                oBox.appendChild(oPic);
                var oImg =document.createElement('img');
                oImg.src='../images/' + dataInt.data[i].src;
                oPic.appendChild(oImg);
            }
            waterfall('main', 'box');
        }
    }
```
>[完整代码](https://github.com/noiron/learn-front-end/tree/master/waterfall-layout/javascript)


### 用 jQuery 实现效果

用 jQuery 实现的思路与之前相同，只不过语法更简单。

>[jQuery 代码实现](https://github.com/noiron/learn-front-end/tree/master/waterfall-layout/jquery)


### 参考资料

- [瀑布流布局浅析](http://ued.taobao.org/blog/2011/09/waterfall/)

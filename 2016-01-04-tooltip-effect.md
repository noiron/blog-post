---
layout: post
title:  学习笔记：用JavaScript实现提示框效果
date:   2016-1-4
tags: [javascript]
categories: 

---

提示框效果（tooltip）是常见的网页特效，它的功能是当用户将指针放置在控件上时为用户显示提示信息。

## 实现原理

我们将需要显示提示框的部分称为：**tooltip 超链接**。

当鼠标移动到 tooltip 超链接上时，首先判断是否存在与之对应的提示框元素，如果存在就将其显示；不存在的话就用JavaScript代码创建一个新的 DOM 元素，然后将新建的 DOM 元素附在原来的 tooltip 超链接元素后面。

每个ToolTip提示框都是一个 div 元素，新建 DOM 元素的格式是类似这样的：

```html
    <div class="tooltip" id="xx"> xxxxxx </div>
```

这里将用到两个函数：

```javascript
    document.createElement();   // 创建一个 HTML 元素
    .appendChild();     // 添加元素
```

<!-- more -->

代码如下：

```javascript
    // 创建一个 div 元素
    var toolTipBox = document.createElement("div");     
    
    // 设置提示框元素的属性
    toolTipBox.className = toolTipBoxClass;
    toolTipBox.id = id;
    toolTipBox.innerHTML = html;
    
    // 将提示框元素添加在 Tooltip 超链接的后面
    obj.appendChild(toolTipBox);
```

然后设置提示框的宽、高、显示方式等属性：

```javascript
    toolTipBox.style.width = width + "px" || "auto";
    toolTipBox.style.height = height + "px" || "auto";

    toolTipBox.style.position = "absolute";
    toolTipBox.style.display = "block";
```

最后确定提示框的位置，提示框应该显示在 tooltip 超链接的下面，并且不超出浏览器窗口的范围：

```javascript
    // Tooltip 超链接左上角的坐标
    var left = obj.offsetLeft;
    var top = obj.offsetTop;

    // 保证 Tooltip 提示框不会超出浏览器的窗口
    if (left + toolTipBox.offsetWidth > document.body.clientWidth) {
        var demoLeft = document.getElementById("demo").offsetLeft;
        left = document.body.clientWidth - toolTipBox.offsetWidth - demoLeft;
        if (left < 0) {
            left = 0;
        }
    }
    toolTipBox.style.left = left + "px";
    toolTipBox.style.top = top + 20 + "px";
```

如果提示框元素已经存在了，只需要将它显示出来：

```javascript
    document.getElementById(id).style.display = "block";
```

******

## 鼠标的移入和移出

我们需要实现的效果是：当鼠标移至 ToolTip 超链接上时，显示提示框；当鼠标离开时，隐藏提示框。

这里鼠标的动作对应的事件是：

```javascript
    onmouseenter    // 鼠标进入
    onmouseleave    // 鼠标离开
```

**注意**：[mouseenter 和 onmouseover 的区别](https://developer.mozilla.org/en-US/docs/Web/Events/mouseover)

当鼠标离开时，将提示框隐藏：
    
```javascript
     obj.onmouseleave = function() {
         setTimeout(function() {
             document.getElementById(id).style.display = "none";
         }, 300);
     }
```

`setTimeout()` 表示在给定时间间隔后执行代码一次。注意 `setInterval()`会循环执行代码，而`setTimeout()`只会执行一次。

******

## 事件的处理

为了实现效果，我们可以分别获取文件中的每一个 Tooltip 超链接元素，并将我们的 `showToolTip()` 函数绑定在超链接的元素的 `onmouseenter` 对象上。以第一个 Tooltip 超链接为例：

```javascript
    var t1 = document.getElementById("tooltip1");
    t1.onmouseenter = function() {
        showToolTip(this, "t1", "hello world", 200);
    }
```

对其余的 ToolTip 超链接也进行类似的操作。

这样的写法为每一个 Tooltip 超链接元素都添加了事件处理程序，会使页面性能变差，可以使用**事件冒泡**来解决这一问题。

事件冒泡指的是在更高的层次上添加一个事件处理程序，来处理所有的 ToolTip 超链接上的事件。这里是将事件添加到了上一级 id="demo" 的 `div` 上。

```javascript
    // 添加事件函数
    function addEvent(element, event, callbackfunction) {
        if (element.addEventListener) {
            element.addEventListener(event, callbackfunction, false);
        } else if (element.attachEvent) {
            element.attachEvent('on' + event, callbackfunction);
        }
    }


    // 利用事件冒泡来处理所有的 Tooltip 超链接的鼠标移入事件
    addEvent(demo, "mouseover", function(e) {
        var event = e || window.event;
        var target = event.target || event.srcElement;

        // 处理发生在 tooltip 超链接上的事件
        if (target.className == "tooltip") {
            var _html;
            var _id;
            var _width = 200;

            switch (target.id) {
                case "tooltip1":
                    _id = "t1";
                    _html = "HyperText Markup Language";
                    break;
                case "tooltip2":
                    _id = "t2";
                    _html = "<h2>JavaScript的5种基本数据类型：</h2><p>Undefined</p><p>Null</p><p>Boolean</p><p>Number</p><p>String</p>";
                    break;
                case "tooltip3":
                    _id = "t3";
                    _html = "<img src='marvin.jpg'>";
                    _width = 220;
                    break;
                case "tooltip4":
                    _id = "t4";
                    _html = "<img src='marvin.jpg' width='150px'> <p><strong>姓名：Wu Kai</strong></p><p>简介：正在学习中的前端程序员</p>";
                    _width = 200;
                    break;
                case "tooltip5":
                    _id = "t5";
                    _html = '<iframe src="http://noiron.github.io" width="600" height="600"></iframe>';
                    _width = 600;
                    break;
                default :
                    return false;
            }
            showToolTip(target, _id, _html, _width);
        }
    });
```

至此，tooltip 提示框的效果就完成了。

******

## 参考资料与演示：

[慕课网：Tooltip浮动提示框效果](http://www.imooc.com/learn/120)

[我的代码](https://github.com/noiron/learn-front-end/tree/master/tooltip-effect)

[在线 demo](http://noiron.github.io/learn-front-end/tooltip-effect/demo2.html)
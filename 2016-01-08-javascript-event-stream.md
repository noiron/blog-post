---
layout: post
title:  《JavaScript高级程序设计》学习笔记：JavaScript中的事件流和事件处理程序
date:   2016-01-08
tags: [javascript]
categories: 
---


JavaScript和HTML之间的交互是通过事件实现的。

事件：文档或浏览器窗口中发生的一些特定的交互瞬间。

可以使用侦听器（或处理程序来预订事件），以便事件发生时执行相应的代码。

***

## 1. 事件流

事件流：从页面中接收事件的顺序。

IE和Netscape开发团队提出了差不多是完全相反的事件流的概念。

IE: 事件冒泡流
Netscape： 事件捕获流


### 1.1 事件冒泡

事件冒泡（event bubbling）：事件开始时由最具体的元素（文档中嵌套层次最深的那个节点）接收，然后逐级向上传播到较为不具体的节点（文档）。


### 1.2 事件捕获

事件捕获（event capturing）:不太具体的节点应该更早接收到事件，最具体的节点应该最后接收到事件。

事件捕获的用意在于在事件到达预定目标之前捕获它。

<!-- more -->

### 1.3 DOM事件流

“DOM2级事件”规定的事件流包括三个阶段：
    1. 事件捕获阶段
    2. 处于目标阶段
    3. 事件冒泡阶段

***

## 2. 事件处理程序
事件是用户或浏览器自身执行的某种动作。如：click，load，mouseover。
响应某个事件的函数称为**事件处理程序（或事件侦听器）**。

### 2.1 HTML事件处理程序

某个元素支持的每种事件，都可以使用一个与相应事件处理程序同名的HTML特性来指定。

    <input type="button" value="Clicke Me" onclick="alert('Clicked')" />

单击按钮，显示警告框。通过指定 onclick 特性并将一些JavaScript代码作为它的值来定义。

在HTML中定义的事件处理程序可以包含要执行的具体动作，也可以调用在页面其他地方定义的脚本。

    <script type="text/javascript">
        function showMessage() {
            alert("Hello world");
        }
    </script>
    <input type="button" value="Clicke Me" onclick="showMessage()" />

事件处理程序中的代码在执行时，有权访问全局作用域中的任何代码。

缺点：

1. 存在时差问题。用户在事件处理程序被解析之前就触发了事件。
2. 扩展事件处理程序的作用域链在不同的浏览器中会导致不同结果。
3. HTML和JavaScript代码紧密耦合。


### 2.2 DOM0 级事件处理程序

通过JavaScript指定事件处理程序的传统方式，就是将一个函数赋值给一个事件处理程序属性。

这种方法被称为**事件处理程序赋值**，出现在第四代 Web 浏览器中。

每个元素（包括 window 和 document）都有自己的事件处理程序属性，属性通常全部小写，如 onclick。将这种属性的值设置为一个函数，就可以指定事件处理程序。

        <input type="button" id="myBtn" value="Click Me" />
        
        <script type="text/javascript">
            var btn = document.getElementById("myBtn");
            btn.onclick = function() {
                alert("Clicked");
            }
        </script>

可以删除通过 DOM0 级方法指定的事件处理程序。

    btn.onclick = null;    // 删除事件处理程序


### 2.3 DOM2 级事件处理程序

“DOM2级事件”定义了两个方法，用于处理指定和删除事件处理程序的操作：

- addEventListener()
- removeEventListener()

所有 DOM 节点都包含这两个方法，它们接受3个参数：

1. 要处理的事件名
2. 作为事件处理程序的函数
3. 一个布尔值：
    - true: 捕获阶段调用事件处理程序
    - false: 冒泡阶段调用
    
为一个按钮添加 onclick 事件处理程序：

    var btn = document.getElementById("myBtn");
    btn.addEventListener("click", function() {
        alert(this.id);
    }, false);

使用 DOM2级方法添加事件处理程序的主要好处是可以添加多个事件处理程序。

用 addEventListener() 添加的事件处理程序只能使用 removeEventListener() 来移除，移除时传入的参数与添加时的参数相同。所以，用 addEventListener() 添加的匿名函数将无法移除。

大多数情况下，都是将事件处理程序添加到事件流的冒泡阶段，这样可以最大限度地兼容各种浏览器。


### 2.4 IE 事件处理程序

IE 实现了与 DOM 中类似的两个方法：
  
- attachEvent()
- detachEvent()

接受两个参数：事件处理程序名称、事件处理程序函数
通过这种方法添加的事件处理程序会被添加到冒泡阶段。

    var btn = document.getElementById("myBtn");
    btn.attachEvent("onclick", function() {
        alert("clicked");
    });

**注意**： attachEvent() 的第一个参数是“onclick”,而不是 addEventListener()方法中的“click"。

在IE中使用 attachEvent() 与使用 DOM0 级方法的主要区别在于事件处理程序的作用域。

- DOM0 级方法：事件处理程序会在其所属元素的作用域内运行
- attachEvent()方法：事件处理程序会在全局作用域内运行，this 等于 window。


### 2.5 跨浏览器的事件处理程序

创建一个方法 addHandler()，它属于名叫EventUtil的对象视情况分别使用 DOM0级方法、 DOM2级方法或IE方法来添加事件。

addHandler()方法接收3个参数：

1. 要操作的元素
2. 事件名称
3. 事件处理程序函数

与其对应的方法是 removeHandler()，接收相同的参数。

    var EventUtil = {
        addHandler:  function(element, type, handler) {
            if (element.addEventListener) {
                element.addEventListener(type, handler, false);
            } else if (element.attachEvent) {
                element.attachEvent("on" + type, handler);
            } else {
                element["on" + type] = handler;
            }
        },

        removeHandler: function(element, type, handler) {
            if (element.removeEventListener) {
                element.removeEventListener(type, handler, false);
            } else if (element.detachEvent) {
                element.detachEvent("on" + type, handler);
            } else {
                element["on" + type] = null;
            }
        }
    }

这两个方法首先都会检测传入的元素中是否存在DOM2级方法。如果存在DOM2级方法，则使用该方法。如果存在的是IE的方法，则采取第二种方案。

像下面这样使用EventUtil对象：

    var btn = document.getElementById("myBtn");
    var handler = function() {
        alert("Clicked");
    }
    EventUtil.addHandler(btn, "click", handler);
    EventUtil.removerHandler(btn, "click", handler);



    

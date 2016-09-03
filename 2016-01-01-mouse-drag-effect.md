---
layout: post
title:  学习笔记：实现登录界面上的鼠标拖拽效果
date:   2016-1-1
tags: [javascript, html, css]
categories: 
---


这是学习了[慕课网教程](http://www.imooc.com/learn/60)后的学习笔记，最终结果如下：[GitHub链接](http://noiron.github.io/learn-front-end/mouse-drag-effect/)


## 分析界面结构

因为我们需要实现的是登录浮层，所以为了方便起见，直接将网页的其余部分作为一张背景图片展现出来。

整个登录对话框（dialog）分为两个大的部分，上部的标题（dialogTitle）和下部的内容部分（dialogContent）。当我们点击在标题部分时，将会触发相应的拖拽效果。

![对话框结构分析](/img/2016-1-1-mouse-drag-dialog.png)

<!-- more -->

对应的HTML：

    <div class="ui-dialog" id="dialog">
        <div class="ui-dialog-title" id="dialogTitle">
            登录通行证
            <!-- 右上角的关闭按钮 -->
            <a href="javascript:hideDialog();" class="ui-dialog-title-closebutton"></a>
        </div>
        <div class="ui-dialog-content">
            <div class="ui-dialog-l40 ui-dialog-pt15">
                <input class="ui-dialog-input ui-dialog-input-username" type="input" value="手机/邮箱/用户名" />
            </div>
            <div class="ui-dialog-l40 ui-dialog-pt15">
                <input class="ui-dialog-input ui-dialog-input-password" type="input" value="密码" />
            </div>
            <div class="ui-dialog-l40">
                <a href="#">忘记密码</a>
            </div>
            <div>
                <a class="ui-dialog-submit" href="#">登录</a>
            </div>
            <div class="ui-dialog-l40">
                <a href="#">立即注册</a>
            </div>
        </div>
    </div>

当点击登陆按钮后，登录对话框后面的内容将会变为灰色，这是依靠蒙版来实现的。这里的蒙版就是一个简单的DIV结构，给它设置一个合适的透明度，再通过z-index属性将它放在登录对话框和网页其余部分之间。

    <div class="ui-mask" id="mask"></div>

    .ui-mask {
            width: 100%;
            height: 100%;
            background: #000;
            opacity: 0.4;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 8000;
            display: none;
        }

***

## 几个显示效果

***

对于登录浮层，我们希望它有一个自动居中的效果，而遮罩层（即蒙版）能够自动扩展到整个可视区域。我们需要用JavaScript定义几个函数来实现这些效果：

    // 获取元素对象
    function g(id) {
        return document.getElementById(id);
    }

    // 自动居中函数 -- 登录浮层
    // el {Element}
    function autoCenter(el) {
        // 获得可视区域的宽和高
        var bodyW = document.documentElement.clientWidth;
        var bodyH = document.documentElement.clientHeight;

        // 获得元素 el 的宽和高
        var elW = el.offsetWidth;
        var elH = el.offsetHeight;

        // 设置元素的 style 样式
        el.style.left = (bodyW - elW) / 2 + 'px';
        el.style.top = (bodyH - elH) / 2 + 'px';
    }

    // 扩展元素到整个可视区域 -- 遮罩层
    // el {Element}
    function fillToBody(el) {
        // 将元素的宽和高设置的和可视区域一样
        el.style.width = document.documentElement.clientWidth + 'px';
        el.style.height = document.documentElement.clientHeight + 'px';
    }

**重要的函数**：
    
    // 获得可视区域的宽和高
    document.documentElement.clientWidth;
    document.documentElement.clientHeight;
    
    // 获得元素的宽和高
    element.offsetWidth;
    element.offsetHeight;


我们还需要登录浮层能够根据需要来显示和隐藏，这一点可以通过设置`display`属性来实现。

    // 展现登录浮层
    function showDialog() {
        g('dialog').style.display = 'block';
        g('mask').style.display = 'block';
        autoCenter(g('dialog'));
        fillToBody(g('mask'));
    }

    // 隐藏登录浮层
    function hideDialog() {
        g('dialog').style.display = 'none';
        g('mask').style.display = 'none';
    }


***

## 实现原理

***

需要实现的功能是当鼠标在标题栏按下后，鼠标可以拖拽登录浮层到屏幕内的任意位置。

我们需要加入三个鼠标事件：

1. 在标题栏上按下：计算鼠标相对拖拽元素的的左上角的坐标， 并且标记元素为可拖动
2. 鼠标移动：根据鼠标位置设置登录浮层的位置
3. 鼠标松开

相应的JavaScript代码如下所示：

    // 定义全局变量
    var mouseOffsetX = 0;
    var mouseOffsetY = 0;
    var isDragging = false;

    // 鼠标事件1 -- 在标题栏上按下
    // 计算鼠标相对拖拽元素的的左上角的坐标， 并且标记元素为可拖动
    g('dialogTitle').addEventListener('mousedown', function(e) {
        var e = e || window.event;

        // 用鼠标按下时的坐标减去 dialog 的左上角坐标
        mouseOffsetX = e.pageX - g('dialog').offsetLeft;
        mouseOffsetY = e.pageY - g('dialog').offsetTop;

        isDragging = true;
    });

    // 鼠标事件2 -- 鼠标移动
    document.onmousemove = function(e) {
        var e = e || window.event;

        // 鼠标当前位置
        var mouseX = e.pageX;
        var mouseY = e.pageY;

        // 鼠标从单击时至当前时刻移动的距离
        var moveX = 0;
        var moveY = 0;

        if (isDragging === true) {
            moveX = mouseX - mouseOffsetX;
            moveY = mouseY - mouseOffsetY;

            // 范围限定
            // moveX > 0 且 moveX < (页面最大宽度 - 浮层宽度)
            // moveY > 0 且 moveY < (页面最大宽度 - 浮层高度)
            var pageWidth = document.documentElement.clientWidth;
            var pageHeight = document.documentElement.clientHeight;

            // 登录浮层的宽、高
            var dialogWidth = g('dialog').offsetWidth;
            var dialogHeight = g('dialog').offsetHeight;

            var maxX = pageWidth - dialogWidth;
            var maxY = pageHeight - dialogHeight;

            moveX = Math.min(maxX, Math.max(0, moveX));
            moveY = Math.min(maxY, Math.max(0, moveY));

            g('dialog').style.left = moveX + 'px';
            g('dialog').style.top = moveY + 'px';
        }
    };

    // 鼠标事件3 -- 鼠标松开
    document.onmouseup = function() {
        isDragging = false;
    };

最后一点，当窗口大小变化时，登录浮层仍然能够居中：

    window.onresize = function() {
        autoCenter(g('dialog'));
        fillToBody(g('mask'));
        };

***

至此，鼠标拖拽效果完成，[完整代码](https://github.com/noiron/learn-front-end/tree/master/mouse-drag-effect)。
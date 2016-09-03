---
layout: post
title: 开发一个用于屏蔽知乎网内容的Chrome扩展
date:   2016-03-25
tags: [javascript, chrome扩展]
categories: 
---

前段时间电影《疯狂动物城》上映了，然后我的知乎首页就被它刷屏了。虽然我对这部电影没有任何意见，但作为一个还没去电影院看过的人来说，每看到相关问题一次都是无情的剧透，于是我毅然屏蔽了“疯狂动物园”这个话题。本以为问题解决了，但是接下来我又被迫看到这个问题：

![](img/2016-03-25-zhihu-filter1.png)

问题上添加的五个话题无一命中，我又被剧透了一脸。算了，既然知乎的屏蔽规则靠不住，那就自己动手吧。这样我的Chrome浏览器扩展——ZhihuFilter就诞生了，[点击这里查看Github上的项目](https://github.com/noiron/ZhihuFilter)。

## 扩展功能

其实扩展的功能很简单，当打开知乎首页后，扩展会依次检查你的屏蔽关键词列表是否出现在了某一个答案中，如果出现了，就会把这个答案隐藏，取而代之的是提示信息和一个展开答案的按钮。效果如下图所示：

![](img/2016-03-25-zhihu-filter-demo1.png)

你可以点击图中的按钮来查看答案，之后可以选择再次隐藏或展开。

<!-- more -->

当你安装了扩展后，会在地址栏的右侧显示出图标

![icon](img/2016-03-25-icon.png)

点击图标后，将会出现设置屏蔽词的界面

![popup页面](img/2016-03-25-popup.png)

你可以在这个页面中设置你想屏蔽的词语。

## 关于Chrome扩展的开发

关于Chrome扩展开发的内容，可以查看[Google的官方文档](https://developer.chrome.com/extensions/getstarted)或者是[这个教程](http://www.ituring.com.cn/minibook/950)。

>一个应用（扩展）其实是压缩在一起的一组文件，包括HTML，CSS，Javascript脚本，图片文件，还有其它任何需要的文件。

开发扩展的时候，必不可少的是一个`manifest.json`文件，这是一个配置文件，会告诉Chrome你的扩展中包含了哪些内容。`manifest.json`中包含扩展的名字、版本及各种资源文件（如图标、显示页面等）的链接。

[我的扩展的manifest.json文件](https://github.com/noiron/ZhihuFilter/blob/master/manifest.json)

### Browser_action 和 page_action

`Browser_action`和`page_action`是扩展的两种类型，它们很相似，主要的区别在于`browser_action`可以应用于所有的页面，而`page_action`只能在你设定的几个特定网站内使用。我的扩展是专门用于知乎网站的，因此选择了`page_action`来处理。

按照Google的文档描述，两者还有一个区别：`browser_action`的图标显示在地址栏的外部，`page_action`的图标显示在地址栏内部。但是，在[这里的讨论](https://groups.google.com/a/chromium.org/forum/#!searchin/chromium-extensions/upcoming/chromium-extensions/7As9MKhav5E/dNiZDoSCCQAJ)中，似乎新版本的Chrome浏览器中已经将两者都显示在了地址栏的外侧，不过`page_action`的图标只有在打开特定的网站时才不会显示为灰色。

在`manifest.json`文件中进行如下设置：

    "page_action": {
      "default_icon": "images/icon.png",
      "default_title": "知乎屏蔽扩展"
    },

### background.js

> 在Manifest中指定background域可以使扩展常驻后台。background可以包含三种属性，分别是`scripts`、`page`和`persistent`。

我在扩展中只用到了`scripts`：

    "background": {
      "scripts": ["js/jquery-2.2.1.js","js/background.js"]
    },

这样就会自动生成一个包含了列出的脚本文件的后台页面。

在我的`background.js`文件中有如下内容：

```javascript
    // 当网址改变的时候，判断当前的页面是否是知乎
    // 如果是的话，就显示出图标，并设置它的弹出页面
    chrome.tabs.onUpdated.addListener(function(id, info, tab){
        if (tab.url.toLowerCase().indexOf("zhihu.com") > -1){
            chrome.pageAction.show(id);
            chrome.pageAction.setPopup({
                tabId: id,
                popup: 'popup.html'
            });
        }
    });
```
`background.js`文件中还有一个用于和`content_script`进行通信的监听器。

> 参考资料：
> [Google的Backgorund Pages文档](https://developer.chrome.com/extensions/background_pages)
> [另一个教程](http://www.ituring.com.cn/article/60242)


### Content Scripts

> Content scripts是在Web页面内运行的javascript脚本。通过使用标准的DOM，它们可以获取浏览器所访问页面的详细信息，并可以修改这些信息。

在`manifest.json`文件中进行设置：

    "content_scripts": [
    {
      "matches": ["*://*.zhihu.com/*"],
      "js": ["js/jquery-2.2.1.js", "js/content_script.js"]
    }

在打开匹配的网站时，列出的js文件会被注入页面，这样就可以获得浏览器所访问的web页面的详细信息，并可以对页面做出修改。虽然content script和页面共享了DOM结构，但不能访问该页面或其它content script中定义的函数和变量，这样就避免了相同的函数或变量名称的干扰。

我的扩展的主要功能就是在`content_script.js`中完成的，在该脚本中通过对页面的DOM进行操作来实现功能。


## 扩展功能的实现

### 对知乎首页进行分析

查看一下知乎首页的源代码，所有的答案内容是放在一个`id=js-home-feed-list`的div中的，结构大致如下：

```html
    <!--这里省略了很多内容，只是一个大致示意-->
    <div id="js-home-feed-list">
        <div class="feed-item">
            <!--这里省略一些题目的id，url等信息-->
            <div class="avatar"></div>  <!--头像-->
            <div class="feed-main">  <!--除了头像外的其它部分-->
                <div class="source"></div>
                <div class="content">
                    <!--答案内容及按钮等部分-->
                </div>
            </div>
        </div>
        <div class="feed-item"></div>
        <div class="feed-item"></div>
    </div>
```

而在答案部分中，又分为摘要和完整的答案。

```html
    <div class="zm-item-answer-detail">
        …
        <div class="zm-item-rich-text">   <!--内容部分-->
            <div class="zm-editable-content"></div>   <!--点击显示全部后，这一部分才显示-->  
            <textarea class="content hidden"></div>   <!--包括了全部答案内容但不显示-->
            <div class="zh-summary summary clearfix"></div> <!--显示摘要-->
        </div>
    </div>
```

我们可以获取上面的节点内容，来确定是否需要屏蔽这个答案。最简单的实现方法就是查找关键词是否在节点的`outerHTML`中出现，如果出现了就给`.feed-main`加上一个名为`hidden`的class。

### 用于替换的内容

原来的答案被隐藏了之后，需要在这个地方换上点新的内容，我在这里创建了一个div，内部有一个p元素用于显示信息，及一个button用于切换答案的状态。

```javascript
    // 创建用于替换的div
    var $div = $('<div class="block-info"></div>');
    $div.append($('<p>这里有一个被屏蔽的答案<span></span></p>'));
    $div.append($('<button class="block-btn">手贱一下</button>'));
```

还需要在按钮上绑定点击事件，p元素内显示的信息也会根据实际进行修改。


### 关键词的存储

在我的扩展中，我是将需要屏蔽的关键词存放在了`localStorage`中。关键词保存在localStorage中的`keywords`键下，是一个简单的由逗号分隔开的字符串。

> 要访问同一个localStorage对象，页面必须来自同一个域名（子域名无效），使用同一种协议，在同一个端口上。

由于content script注入到了知乎页面，而localStorage存放在扩展的域下，要在content script中获得关键词的值，就必须用到页面中的通信。

Chrome提供了4个有关扩展页面间相互通信的接口，分别是`runtime.sendMessage`、`runtime.onMessage`、`runtime.connect`和`runtime.onConnect`，
这里用到了前两个。

content_script.js

```javascript
    // 从扩展的localStorage中获得存储的关键词
    chrome.runtime.sendMessage({method: "getKeywords"}, function (response) {
        str = response.keywords;
        keywords = str !== '' ? str.split(',') : [];
    });
```

对应的background.js

```javascript
    chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
        if (request.method == "getKeywords")
            sendResponse({keywords: localStorage['keywords']});
        else
            sendResponse({});
    });
```
这样就可以在content script中使用localStorage的值了。


> 参考资料：[扩展页面中的通信](http://www.ituring.com.cn/article/60272)


### 检测页面加载了更多内容

每当当知乎页面加载了更多的答案时，我们需要再进行处理。那么如何检测页面中加载了更多内容，这里需要用到`MutationObserver`对象。`MutationObserver`提供了检测页面中的DOM变化的方法。

`MutationObserver`的使用方法：

首先，创建一个mutationObserver的实例

    var observer = new MutationObserver(callback)

这里需要一个回调函数作为构造器的参数。

callback函数接受一个参数，所有被记录到的DOM变化将会组合成一个数组，作为参数传给回调函数进行处理。

mutationObserver对象有几个方法，这里只用到observe方法。

observe方法接收两个参数：

```javascript
    void observe(
      Node target,
      MutationObserverInit options
    );
```

第一个参数指的是你要观察哪一个节点的DOM变化。第二个参数是一个选项参数，

```javascript
    var option = {
        'childList': true,    // 观察子元素
        'subtree': true,      // 观察目标节点的后代元素
        'attributes': false   // 不观察目标节点属性的变化
    };
```

这里只设置了三个选项，其余属性可以看[MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver#MutationObserverInit)。

观察到的每一个变化都是一个MutationRecord对象，它有许多属性，比如：

- type，记录变化的类型
- addedNodes，由增加的节点组成的NodeList
- removeNodes，由删除的节点组成的NodeList

因为addedNodes是一个NodeList，所以可以在它上面应用jQuery的$(), `$(mutation.addedNodes)`。

其它的属性可以见[MDN文档](https://developer.mozilla.org/en-US/docs/Web/API/MutationRecord)。

所有的MutationRecord会被放进一个list中，作为参数传给上面的callback函数。通过对MutationRecord对象的属性进行检测，如果新增的节点里出现了`class=feed-main`的元素，则代表加载了新的答案，需要再一次运行程序。

> 参考资料：
> [Getting to Know Mutation Observers](https://dev.opera.com/articles/mutation-observers-tutorial/)
> [Mutation Observer](http://javascript.ruanyifeng.com/dom/mutationobserver.html)

## 扩展的使用

在我的设想中，扩展可以提供一些选项，比如正则表达式的支持，再比如除了首页外，在答案页面是否也需要提供屏蔽功能。这些选项会在之后逐步加入。

由于Chrome的设置，不能够安装Web Store中没有的程序。而发布扩展需要先付$5，我没有信用卡，也就暂时没有发布扩展。如果想尝试一下扩展的话，可以直接下载[Github中的代码](https://github.com/noiron/ZhihuFilter)到本地。在Chrome浏览器的菜单中选择 More tools -> Extensions，进入扩展页面后，勾选右上角的Developer mode，选择Load unpacked extension，选择扩展文件夹即可。 

欢迎使用后提出建议。
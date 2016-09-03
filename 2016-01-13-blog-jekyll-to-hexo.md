---
layout: post
title:  将博客从Jekyll迁移至Hexo
date:   2016-01-13
tags: [hexo]
categories: 
---

今天把我的博客从 Jekyll 转到了 Hexo 上，因为一直用的是 Windows系统，而在 Windows 系统下面安装Ruby和使用 Jekyll 的时候会出现各种奇怪的问题。虽然问题都在Google的帮助下解决了，但是也让我一直不敢修改博客主题，生怕一不小心解决了一个问题又多出来了几十个问题。

最终我决定改用 Hexo 来搭建我的博客。在 GitHub 上看了不少 Hexo 的主题后，发现了几个对我胃口的主题：

- [yilia](https://github.com/litten/hexo-theme-yilia)
- [NexT](https://github.com/iissnan/hexo-theme-next)
- [Pacman](https://github.com/A-limon/pacman)

最终我决定使用**yilia**了。

## 一、安装Node.js和Hexo

关于安装的教程已经有很多了，不需要赘述，可以参考这些：

>[使用GitHub和Hexo搭建免费静态Blog](http://wsgzao.github.io/post/hexo-guide/)
>[史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

我这里就记录一下我遇到的问题。

<!-- more -->

## 二、解决的问题

### jekyll和hexo文件名的区别

在jekyll下，我使用的文件命名格式为`2015-12-28-threejs-collision.md`，这样生成的文章地址为`/2015/12/28/threejs-collision/`。但是在hexo中，默认情况下生成的地址会是`/2015/12/28/2015-12-28-threejs-collision/`，这明显不是我们想要的，所以在 _config.yml 中修改 new_post_name 参数。

    new_post_name: :year-:month-:day-:title.md

还有要注意的一点是，文件名里的 2016-1-8 要写成 2016-01-08，前面要加数字0，才能得到预期结果。

>[从Jekyll迁移到Hexo](http://reverland.org/web/2015/11/18/notes-on-migrating-from-jeykll-to-hexo/)
>[官方文档](https://hexo.io/zh-cn/docs/migration.html)


### 标签的大小写问题

由于我之前用作 tag 的单词包含了大小写，比如"JavaScript"和"javascript"是两个不同的标签，这样会导致一些奇怪的404问题。所以我决定所有的 tag 一律使用小写，设置：

    filename_case: 1

表示生成的网址全部为小写单词。

然后为了保证 GitHub 的 tags 目录也全部为小写，需要删除本地 publish 文件夹下的 tags 目录，执行`hexo d`命令后，GitHub上的 tags 文件夹也会被删除。最后`hexo g -d`，重新生成 tags 文件夹，就能保证博客不会因为大小写的问题而出现404错误了。


### 首页语言为繁体中文

在默认情况下，首页的菜单项显示为繁体中文。

![](/img/2016-01-13-traditional-chinese.png)

修改在 themes/yilia 文件夹下的 _config.yml 中的语言项：

    language: zh-Hans

之后执行命令：

    hexo clean
    hexo server

发现问题解决。


### 开启多说

在主题的 _config.yml 文件中修改：

    duoshuo: "noironblog"

后面的值为你可以在登录多说的站点管理界面时的网址中找到：

我的网址为：`http://noironblog.duoshuo.com/admin`，对应的值为"noironblog"。

>[多说配置 #7](https://github.com/litten/hexo-theme-yilia/issues/7)
>[多说配置问题 #60](https://github.com/litten/hexo-theme-yilia/issues/60)


### 加入头像图片

在 themes/yilia/source/img 文件夹下放入头像图片 avatar.jpg，并设置：

    avatar: /img/avatar.jpg 

这样就能显示头像图片。


### 加入RSS

>[Hexo—正确添加RSS订阅](http://hanhailong.com/2015/10/08/Hexo%E2%80%94%E6%AD%A3%E7%A1%AE%E6%B7%BB%E5%8A%A0RSS%E8%AE%A2%E9%98%85/)


## 三、尚未解决的问题

### GitHub上原有的commit记录没了

在使用`hexo g -d`命令后，hexo 直接将新的博客部署到了 GitHub 上，并清空了原先我使用 jekyll 博客时的记录。虽然我在本地保存了整个项目文件，但是不知道还能不能恢复到GitHub上。

### 博客的markdown文件没有同步

hexo 部署文件时，只是将生成的网页资源放在了GitHub上，但是 _posts文件夹下 .md 文件没有同步，现在我使用 Dropbox 进行保存。也许之后可以新建一个 repo 或分支来保存这些文件。

### 没有在首页建立项目页

原先使用的主题在首页有一个 Demo 页，需要在这里也建立一个。

***

最后，修改一些配置项，就完成了将博客从 Jelyll 迁移至 Hexo 的任务。
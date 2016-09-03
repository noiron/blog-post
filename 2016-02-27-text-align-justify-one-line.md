---
layout: post
title: 实现单行文本的左右对齐
date:   2016-02-27
tags: [css]
categories: 
---

今天碰到了一个问题，如何让只有一行的文本进行左右对齐。对于英文文本，左右对齐的效果如下所示：

![左右对齐的段落](img/2016-02-27-justify-text.png)

上述效果是使用`text-align: justify`来实现的。可以看到除了最后一行之外，段落的其余部分都是左右对齐的。如果我们将`text-align: justify`样式直接应用在单行的文本上时，将会像上面段落的最后一行，仅仅是向左对齐。可以看一下下面的例子：

**代码：**

```html
<p id="justify-para">这是一行没有左右对齐的文本</p>

<style type="text/css">
#justify-para {
    background-color: #ccc;
    width: 400px;
    text-align: justify;
}
</style>
```

**结果：**

![一行没有左右对齐的单行文本](img/2016-02-27-one-line-text1.png)

对于汉字文本，因为汉字的等宽特点，文本左右对齐的效果就是每个汉字间的间隔相等。为了实现这一效果，需要利用`:after`伪元素，在文本之后添加一个不可见的元素，如下所示。

```css
#justify-para:after {
  content: "";
  display: inline-block;
  width: 100%;
}
```
**最终效果：**

![左右对齐的单行文本](img/2016-02-27-one-line-text.png)

这样就实现了单行文本的左右对齐效果。

***

## 参考资料

>[CSS – One Line Justify](http://blog.vjeux.com/2011/css/css-one-line-justify.html)

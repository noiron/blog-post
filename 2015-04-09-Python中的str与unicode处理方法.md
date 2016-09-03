---
layout: post
title: Python中的str与unicode处理方法
tags: [python, unicode]
categories:
---

> http://python.jobbole.com/81244/
> http://www.pulpcode.cn/program-language/2014/12/23/python-encode/

## str与字节码

```python
	s = "人生苦短"
```

s是个字符串，它本身存储的就是字节码。
（字节码（英语：Bytecode）通常指的是已经经过编译，但与特定机器码无关，需要直译器转译后才能成为机器码的中间代码。）

如果这段代码是在解释器上输入的，那么这个s的格式就是解释器的编码格式，对于windows的cmd而言，就是gbk。

如果将段代码是保存后才执行的，比如存储为utf-8，那么在解释器载入这段程序的时候，就会将s初始化为utf-8编码。


<!-- more -->

## unicode与str

unicode是一种编码标准，具体的实现标准可能是utf-8，utf-16，gbk ......

python 在内部使用两个字节来存储一个unicode，使用unicode对象而不是str的好处，就是unicode方便于跨平台。

两种方式定义一个unicode：

```python
	s1 = u"人生苦短"
	s2 = unicode("人生苦短", "utf-8") # 在IDLE中会提示UnicodeDecodeError，可以改为gbk
```

## encode与decode

某种编码如GBK、UTF-8通过解码（decode）得到Unicode

Unicode通过编码（encode）得到某种编码

也能对str进行编码：

```python
	s = "人生苦短"
	s.encode('gbk')
```

当对str进行编码时，会先用**默认编码**将自己解码为unicode，然后再将unicode编码为你指定编码。

**这就引出了python2.x中在处理中文时，大多数出现错误的原因所在：python的默认编码，defaultencoding是ascii**

```python
	# -*- coding: utf-8 -*-
	s = "人生苦短"
	s.encode('gbk')
```


上面的代码会报错，错误信息：UnicodeDecodeError: 'ascii' codec can't decode byte ......

因为你没有指定defaultencoding,所以它其实在做这样的事情:

```python
	# -*- coding: utf-8 -*-
	s = "人生苦短"
	s.decode('ascii').encode('gbk')
```

## 设置defaultencoding

```python
	reload(sys)
	sys.setdefaultencoding('utf-8')
```

如果你在python中进行编码和解码的时候，不指定编码方式，那么python就会使用defaultencoding。

使用str创建unicode对象时，如果不说明这个str的编码格式，那么程序会使用defaultencoding。

u = unicode("人生苦短") 等价于 u = unicode("人生苦短",defaultencoding)

默认的defaultcoding：ascii是许多错误的原因，所以早早的设置defaultencoding是一个好习惯。


## 文件头声明编码的作用

参考： [python中文乱码问题深入分析](http://blog.csdn.net/kiki113/article/details/4062063)

顶部的:# -*- coding: utf-8 -*-目前看来有三个作用。

1. 如果代码中有中文注释，就需要此声明
2. 比较高级的编辑器，会根据头部声明，将此作为代码文件的格式。
3. 程序会通过头部声明，解码初始化 u"人生苦短"，这样的unicode对象，（所以头部声明和代码的存储格式要一致）


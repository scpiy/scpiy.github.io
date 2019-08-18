---
layout:		post
title:		python中的import与循环import
subtitle:	python 模块 循环import
date:		2019-08-18
author:		scpi
header-img:	img/post-2019-08-18.jpg
catalog:	true
tags:
    - python
---



## python中的import与循环import

这段时间一直在看一个flask的教程，老师时不时在教程上提到"**该导入位于底部以避免循环依赖**"，一直以来不明白是什么意思，今天又一次看到这句话终于忍不住了，就去网上查了查相关的东西，现在脑袋里清晰了很多，做一下笔记，同时希望对你也能有所帮助。

> 我看的是[Importing Python Modules](http://effbot.org/zone/import-confusion.htm#import-gothas)这篇文章，所以这篇博客其实也可以算是那篇博客的翻译与总结，那篇博客写于1999年，最后一次修改似乎是在2001年，并且还是用英文写的，所以如果我写的有什么错误的地方，希望你可以指出，~~我也看了些在CSDN上的博文，应该不会有错误的才对吧......大概~~

#### 当你import一个模块的时候，发生了什么

要想知道为什么“该导入位于底部以避免循环依赖”，首先要知道“What Does Python Do to Import a Module”，那篇博客的原文写的很详细，分了四个步骤

1. 创建了一个新的，空的模块对象(本质上是一个字典)
2. 把创建好的模块对象插入到”*sys.modules* dictionary“
3. 在创建好的模块中加载代码，也就是复制粘贴被import的模块的代码到第一步创建的模块中(如果有必要，先编译，~~大概是调用其它静态语言的模块吧，不清楚，但反正人家是这么说的~~)
4. 把模块中的代码在一个新的模块命名空间中运行一遍，在代码中定义的所有的对象都可以在这个模块对象中获得

其实简单来说的话，就是在你导入(import)一个模块的时候，这个模块的代码都会被执行一次，~~只要知道这个应该就够用了，对新手来说，至少对今天我要说的东西来说是这样。~~

#### 循环import

首先假设存在这么两个文件**X.py**与**Y.py**

```python
# module X
import Y

def spam():
    print "function in module x"

def test():
	Y.Y_spam()
```

```python
# model Y
from X import spam #这个spam是没有定义的，是不能用的

def Y_spam():
	print(span())	
```

```python
# mian.py
import X

X.test()
#ImportError: cannot import name 'spam' from 'X'
```

解释一下，

1. 在main.py中import X
2. 程序会停下来并且开始执行X中的代码
3. 程序在X中执行到了import Y
4. 程序又停下来并且开始执行Y中的代码
5. 执行到from X import spam这行代码
6. **spam在X中还没有定义，引发错误**

~~看起来很简单的样子。~~

#### 循环import的解决方法

那篇博文的作者提出的两种方法

1. 改善组织结构，避免循环import，看起来似乎是废话，但真的很重要，这么做有太多好处了。
2. 把导致循环import发生的import语句放到文件的末尾，这样就可以在循环import发生的时候确保模块中的函数、类、总而言之，所有东西都已经被定义并且可用，这也就是我在学的课程的老师说的那句话的意思。

最后说一下，我参考的那篇博客虽然相当地有年头了，但是写的很好，里面也提到了很多我没有说的东西，有兴趣的可以去看一下。
---
layout:		post
title:		使用python与opencv实现黑白图片与彩色图片的辨别
subtitle:	python opencv
date:		2019-08-08
author:		scpi
header-img:	img/post-2019-08-08.jpg
catalog:	true
tags:
    - python
---



## 使用python的opencv实现黑白图片与彩色图片的辨别

首先大概说一下实现逻辑。

一张正常的彩色图片，里面有各种颜色，他的RGB(不知道RGB的自行百度吧)三通道的值在色彩鲜艳的地方差距是很大的，而一张合格的黑白图片，也叫灰度图片，他如果使用RGB三通道，那么他的三个通道的值必定是相等的，即R=G=B，但是因为有很多黑白图片在其上方有一层或蓝或绿的四类蒙版的东西，所以不能严格按照这个标准来判断一张图片的身份

> 为啥说如果灰度图片采用RGB三通道呢，这是因为很多时候，我们去读取一章灰度图片，得到的只有一个通道，就是灰色通道，但opencv这个库，如果你不指定参数，他是不会给你按照灰度图片去读取的，所以，正常情况下opencv读取出来的灰色图片也至少有三个通道
>
> 为啥说至少呢，因为有些图片格式，比如png，它不仅仅有RGB三个通道，还有一个叫做不透明度通道的通道，用过ps等图片编辑软件的同学大概知道这是个什么，我也不细说，你只要知道个别情况下他会多一个通道就好，这也不影响我们今天要做的事

继续刚才的话题，不能按照R=G=B的标准，那该怎么办呢？

这个解决方案很久以前就有人想出来了，因为就算蒙上了一层带颜色的蒙版，这个蒙版的颜色也不会太重，不然就不是黑白图片了，我们可以利用这个机制，设置一个阈值，当一张图片的所有像素点的RGB三个通道的值相差都不超过这个阈值的时候，我们就认为这是一张灰度图片，按照前人的经验，这个阈值设置在50就很好用。

> opencv库读取出来的通道的顺序不是现在常见的RGB，而是很多年以前的标准BGR，这个可以留意一下，虽然今天用不到就是了

知道了上面这些，就很好办了，我先简单介绍几个opencv库的函数

```python
import cv2
#这是opencv在python中的名字
img = cv2.imdecode(np.fromfile(path_and_name, dtype=np.uint8),-1)
#这是opencv读取一张图片的方式，看起来很麻烦对不对，因为这不是正常的方式，正常的方式是没法读取带有中文的文件名的文件的，至于具体原因我最后会说一下，你只要复制粘贴就好了
#path_and_name是图片的路径与名字
img.shape
#获取img对象的高、宽(以像素为单位)以及通道的数量
img[i, j]
#可以返回img对象i， j坐标上的像素点的各个通道的值
#但是又不知道这个通道数是几个，带不带不透明度通道，你是不是想到上面在img.shape那里获取到的通道数量了？使用判断语句进行判断当然是可以的，但实际上还有更简单的办法
img[i, j, num]
#根据num的值，返回对应的通道的值，0返回的就是B，1返回的就是G，2返回的就是R
```

就这么多，是不是很简单，当然很简单了。但要自己做还真的有不少坑。。

还有这个程序还需要依赖numpy模块，通过pip安装就好

> numpy是为图片的读取提供支持

```python
import cv2
import numpy as np

def indentify_picture():
    img = cv2.imdecode(np.fromfile("G:\\picture\\test.jpg", dtype=np.uint8), -1)
    #演示用，只展示识别的方法，真要写一个能用的程序的话我还用到了os(用于获取路径与文件名)和shutil(用于文件的复制)
    #这里不那么麻烦了，如果有想要相关代码的可以联系我
    #我也没有用"[4783]空の呼び声-64616604.jpg"这种文件名，只是为了方便
    height, wide, gallery = img.shape
    max_dif = 0
    #定义BGR通道的值最大差值，如果最后这个差值大于50，就认为这是彩色图片
    for i in range(height):
        for j in range(wide):
            #遍历每一个像素点
            v = max(abs(img[i, j, 0] - img[i, j, 1]), abs(img[i, j, 0]- img[i, j, 2]), abs(img[i, j, 1]- img[i, j, 2]))
            #找到每个像素点的最大差值
            if v > max_dif:
                max_dif = v
            if max_dif > 50:
                #一旦某个像素点的差值大于50， 直接退出，
                return "这是一张彩色图片"
    return "这是一张灰度图片"

print(indentify_picture())
#运行结果
temp> py.exe .\indetify_picture.py
这是一张彩色图片
```

emmmmm，那么今天就到这里了。

***

感谢你的观看，希望能对你有帮助。

如果有自己解决不了的问题可以联系我。
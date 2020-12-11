---
title: "关于树莓派上使用HyperLPR(pip版)"
subtitle: ""
date: 2020-07-11T17:29:09+08:00
lastmod: 2020-07-11T17:29:09+08:00
draft: false
author: ""
authorLink: ""
description: ""
images: ""
tags:
  - RaspberryPi
categories: []
twemoji: true
ruby: true
fraction: true
fontawesome: true
linkToMarkdown: ""
hiddenFromHomePage: false
rssFullText: true
hiddenFromSearch: false
featuredImage: ""
featuredImagePreview: ""
toc:
  enable: false
math:
  enable: true
comment:
  enable: true
lightgallery: false
license: ""
---

> [https://github.com/zeusees/HyperLPR](https://github.com/zeusees/HyperLPR)

首先树莓派使用的是官方镜像
![](https://libget.com/gkirito/blog/image/190422/2019-04-22-15556058592094.jpg)

> [https://www.raspberrypi.org/downloads/](官方网站)

之后需要开启vnc功能，可以参照照片博客

> http://shumeipai.nxez.com/2018/08/31/raspberry-pi-vnc-viewer-configuration-tutorial.html

在官方原本镜像中，已经包含了Python2和3的版本，所以我们直接用pip安装所需要的库就行，这里我们使用Python3的版本。

官方安装方式

![](https://libget.com/gkirito/blog/image/190422/2019-04-22-15556060832745.jpg)

由于我已经安装完成，这里就直接贴出结果

![](https://libget.com/gkirito/blog/image/190422/2019-04-22-15556062196900.jpg)

```
Collecting hyperlpr
  Using cached https://files.pythonhosted.org/packages/d7/97/cb06bb3b1a6505c376fd8df605cb05407ffafdb595ef9243c6fb183e1fcb/hyperlpr-0.0.1-py2.py3-none-any.whl
Collecting opencv-python (from hyperlpr)
  Using cached https://www.piwheels.org/simple/opencv-python/opencv_python-3.4.4.19-cp35-cp35m-linux_armv7l.whl
Collecting numpy>=1.12.1 (from opencv-python->hyperlpr)
Installing collected packages: numpy, opencv-python, hyperlpr
Successfully installed hyperlpr-0.0.1 numpy-1.16.2 opencv-python-3.4.4.19
```

可以看出依赖包和版本，分别是：

1. hyperlpr-0.0.1-py2.py3
2. opencv-python
3. numpy>=1.12.1

由于cv2链接包源都在国外的服务器原因，可以先通过自己可以科学上网的电脑下载好包，然后传给树莓派，再手动安装
安装命令：
`pip3 install XXX.whl`
当然提前是你要先进入到该包所在目录
这里说明一下，从
![](https://libget.com/gkirito/blog/image/190422/2019-04-22-15556065547210.jpg)图中可以看出，树莓派上的cv2和普通的cv2所在地址不一样，所以直接复制粘贴这个url下载就好
*[https://www.piwheels.org/simple/opencv-python/opencv_python-3.4.4.19-cp35-cp35m-linux_armv7l.whl](https://www.piwheels.org/simple/opencv-python/opencv_python-3.4.4.19-cp35-cp35m-linux_armv7l.whl)*

但是像`numpy`这类包，树莓派版本还是是使用官方原镜像的，所以可以通过pip安装时指定国内源方式快速安装
`pip3 install numpy -i https://pypi.tuna.tsinghua.edu.cn/simple`
`-i`之后的参数就是国内的源仓库，有好几种：

* 阿里云 https://mirrors.aliyun.com/pypi/simple/
* 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
* 豆瓣(douban) http://pypi.douban.com/simple/
* 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
* 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

之后的`hyperlpr`也可以通过
`pip3 install hyperlpr-i https://pypi.tuna.tsinghua.edu.cn/simple `
来安装，之后我们终于在Python3环境中安装好了`hyperlpr`

接下来可以写个Python3的demo跑一下试试了

```Python
#导入包
from hyperlpr import *
#导入OpenCV库
import cv2
#读入图片
image = cv2.imread("demo.jpg")
#识别结果
print(HyperLPR_PlateRecogntion(image))
```

但是一运行，发现了

```
ImportError: libcblas.so.3: cannot open shared object file: No such file or directory
```

此类问题
这是我们系统环境缺少一些运行opencv的依赖
这一引用`stackoverflow`上的一个提问
[https://stackoverflow.com/questions/53347759/importerror-libcblas-so-3-cannot-open-shared-object-file-no-such-file-or-dire](https://stackoverflow.com/questions/53347759/importerror-libcblas-so-3-cannot-open-shared-object-file-no-such-file-or-dire)


```
sudo apt-get install libcblas-dev
sudo apt-get install libhdf5-dev
sudo apt-get install libhdf5-serial-dev
sudo apt-get install libatlas-base-dev
sudo apt-get install libjasper-dev 
sudo apt-get install libqtgui4 
sudo apt-get install libqt4-test
```

通过执行这些命令，将相应的OpenCV环境依赖安装好了就能运行了！！
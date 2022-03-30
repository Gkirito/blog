---
title: "SSH登陆使用本地RSA"
subtitle: '一个密码登录多个服务器'
date: 2020-12-16T19:57:36+08:00
lastmod: 2020-12-16T19:57:36+08:00
draft: false
author: ""
authorLink: ""
description: ""
images: ""
tags: []
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
toc: true
math: true
comment: true
lightgallery: false
license: ""
---

> 平时ssh过程中针对每台机子都有不同的密码（为了安全性🔒，大多数为随机密码），这样导致了密码繁多，而且难记，保存于🗒️上容易泄露，可以采用`SSH 公钥`进行认证，这样只需要一个密码

## 1. 创建RSA密钥
``` shell
ssh-keygen -o
```
运行命令后会需要输入两次一样的密钥口令，`需要记住这个密码`
然后会在当前用户目录下发现`.ssh`文件夹，里面包含如下：

![image-20201216icXRehb0@2x](https://libget.com/gkirito/blog/image/2020/image-20201216icXRehb0@2x.png)

## 2. 复制公钥到目标服务器
复制`id_rsa.pub`中的公钥内容，在目标服务器上的`~/.ssh/authorized_keys`中填入内容，如果没有`.ssh`文件夹，就在对应目录下创建：
``` shell
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
```
然后填入内容，并保存退出

## 3. 退出ssh，重新登录验证
退出ssh，然后重新登录，这时需要你输入刚刚创建的ssh key的密码，输入之前记住的密码即可

这个方法可以多次复用，多台服务器只需一个密码就可以管理
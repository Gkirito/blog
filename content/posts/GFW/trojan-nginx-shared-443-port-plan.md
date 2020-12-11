---
title: 'Trojan和Nginx共用443端口方案'
subtitle: ''
date: 2020-11-12T11:53:03+08:00
lastmod: 2020-11-12T11:53:03+08:00
draft: false
author: ""
authorLink: ""
description: ""
images: ""
tags:
  - gfw
  - Trojan
  - Nginx
  - VLESS
  - HTTPS
categories: 
  - GFW
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
  enable: true
math:
  enable: true
comment:
  enable: true
lightgallery: false
license: ""
---

> 本文选用 Trojan 为 V2ray 中的 Trojan 版本，支持的是 V2ray 中的 fallback
>
> 观看本文前需申请好 TLS 证书，可以参考我的博客的这篇文章
>
> 以下 `采用Nginx基于SNI的4层转发`参考于[程小白](https://www.chengxiaobai.cn/)的这篇文章[Trojan 共用 443 端口方案](https://www.chengxiaobai.cn/record/trojan-shared-443-port-scheme.html)

## 简介

在使用`Trojan`和`VLESS`之类代理时，需要占用`443`端口，但是如果同服务器上部署有其他 Web 服务或者其他需要使用 443 端口（HTTPS）的业务，就需要共用 443 端口了，这里将从一下几个方案入手:

[1. 采用 Nginx 基于 SNI 的 4 层转发](#1-采用nginx基于sni的4层转发)

[2. 使用 Nginx 的 stream 针对不同域名转发，并配置 Trojan 的回落](#1-采用-nginx-基于-sni-的-4-层转发)

[3. 使用其他 TLS 流量分流器（如： ](#1-采用-nginx-基于-sni-的-4-层转发)[tls-shunt-proxy](https://github.com/liberal-boy/tls-shunt-proxy)

以下为配置方案：

## 1. 采用Nginx基于SNI的4层转发

成品架构：

> ![image-202011151iaLRNz9@2x](https://libget.com/gkirito/blog/image/2020/image-202011151iaLRNz9@2x.png '图片来自程小白')

---
title: "Trojan和Nginx共用443端口方案"
date: 2020-11-12T11:53:03+08:00
draft: false
slug: trojan-nginx-shared-443-port-plan
categories: 
lightgallery: true
toc: true
images:
tags:
  - gfw
  - Trojan
  - Nginx
  - VLESS
  - HTTPS
---

>  本文选用Trojan为V2ray中的Trojan版本，支持的是V2ray中的fallback
>
> 观看本文前需申请好TLS证书，可以参考我的博客的这篇文章
>
> 一下 ``采用Nginx基于SNI的4层转发``参考于[程小白](https://www.chengxiaobai.cn/)的这篇文章[Trojan 共用 443 端口方案](https://www.chengxiaobai.cn/record/trojan-shared-443-port-scheme.html)

## 简介

在使用`Trojan`和`VLESS`之类代理时，需要占用`443`端口，但是如果同服务器上部署有其他Web服务或者其他需要使用443端口（HTTPS）的业务，就需要共用443端口了，这里将从一下几个方案入手:

1. [采用Nginx基于SNI的4层转发](#1-采用Nginx基于SNI的4层转发)

2. 使用Nginx的stream针对不同域名转发，并配置Trojan的回落
3. 使用其他TLS流量分流器（如：[tls-shunt-proxy](https://github.com/liberal-boy/tls-shunt-proxy)）

一下为配置方案

## 1. 采用Nginx基于SNI的4层转发

成品架构：

> ![image-202011151iaLRNz9@2x](https://libget.com/gkirito/blog/image/2020/image-202011151iaLRNz9@2x.png "图片来自程小白")


---
title: "链网几大业务场景分析及解决方案"
subtitle: ''
date: 2021-03-23T18:16:08+08:00
lastmod: 2021-03-23T18:16:08+08:00
draft: false
author: "gkirito"
authorLink: "https://www.gkirito.com"
description: ""
images: ""
tags: 
  - work
  - chainx.org
categories: 
  - work
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

# 链网几大业务场景分析及解决方案

## 主要访问有严重问题
1. [polkaworld](https://polkaworld.org)

    ![picture 11](https://libget.com/gkirito/blog/image/2021/image-20210305ZJP9R1fZ%402x.png)  


2. [区块链浏览器](https://scan.chainx.org)

    ![picture 12](https://libget.com/gkirito/blog/image/2021/image-202103059Ma04QVF%402x.png)  

分析以上加载时间较久的网页，可以发现主要长时间加载的有两种：

1. 后端传回数据
   
    ![picture 13](https://libget.com/gkirito/blog/image/2021/image-20210305JtKEAGFz%402x.png)  

2. 前端部分静态资源

    ![picture 14](https://libget.com/gkirito/blog/image/2021/image-20210305WVZpRgsN%402x.png)  
    ![picture 15](https://libget.com/gkirito/blog/image/2021/image-20210305ECIXCucT%402x.png)  
    ![picture 16](https://libget.com/gkirito/blog/image/2021/image-20210305KYaWb0fp%402x.png)  
    ![picture 17](https://libget.com/gkirito/blog/image/2021/image-20210305zQ99W60n%402x.png)  


## 解决方案
### 后端数据传输

后端这边目前在搭建支持多语言的分布式框架，可以把这些后端一起集成入分布式框架，方便解决弹性扩容等问题，还有后端的代码优化，sql优化等。

### 前端静态资源问题
1. 对于静态资源加载慢问题，可以采取上CDN解析或者边缘计算方案
2. 考虑有DDoS攻击经历，需要采取一些DDoS防御方案，如能够弹性扩容，使用AWS的DDoS防御或者风险转移方案等
   
**解决方案**：  
1. 使用AWS的DDoS防御方案，前端静态文件放入AWS的CDN方案
   DDoS上，AWS的防御流量和支持能力对比阿里云较为完善，AWS上也有边缘计|CDN算方案，可以加速对静态资源文件的访问速度，且为自动调配。

   **优点**：
   能加速全球的静态资源访问，同时支持DDoS防御强

   **缺点**：
   在DDoS防御上，采用扩容和增加防御容量的模式，是一种增加机器性能和带宽暴力解决DDoS攻击时暴增的流量，本身是一种设备上的对抗

2. 使用Cloudflare方案
   Cloudflare在CDN上支持多项功能，在配置上比较方便，无需已部署应用的再次迁移服务器，只需将域名交给Cloudflare，并开启CDN代理后，Cloudflare能解决DNS解析、SSL、网络(HTTP/2等)、流量负载均衡等。  
   
   在DDoS防御上，Cloudflare采用一种风险转移方案，当一个请求正常发起时，由Cloudflare先对该请求进行审核，确定请求和IP是否为正常行为，再进行转发，转发到应用所属服务器。
   
   ![picture 18](https://libget.com/gkirito/blog/image/2021/image-202103052Amoqrhl%402x.png)  


   **优点**：
   能加速全球的静态资源访问，同时能帮忙解决DNS解析、TLS、负载均衡等方面的功能，能让我们更专注于开发和服务部署；在DDoS防御上，采取风险转移手段，由Cloudflare去审查和承受DDoS的大流量请求，对我们本身部署的机器来讲更加安全稳定

   **缺点**：
   国区的CDN需要Enterprise方案，需要企业沟通
   Business方案不支持国区CDN，价格为200$/月
   WAF上Business方案只支持25个规则，有些功能需要另外收费



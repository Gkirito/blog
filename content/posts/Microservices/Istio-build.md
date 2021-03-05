---
title: "Istio搭建记录📝"
subtitle: ''
date: 2021-03-05T18:41:07+08:00
lastmod: 2021-03-05T18:41:07+08:00
draft: false
author: "gkirito"
authorLink: "https://www.gkirito.com"
description: ""
images: ""
tags: 
  - Istio
  - microservices
categories: 
  - 微服务
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

# Istio搭建记录📝

## 先前准备
1. 部署好一个Kubernetes(1.17, 1.18, 1.19, 1.20)
2. 需要有个cluster

## 本地搭建
### Minikube
> 要求：
> 1. 安装好虚拟机，可以是docker或者VM之类
> 2. 安装好Minikube
> 3. 设置Minikube的虚拟机驱动(如果不使用默认驱动的话)
>   ``` shell
>       minikube config set vm-driver kvm2
>   ```
> 4. 保证本机有足够的性能(例如能支持Minikube的4CPUs和8GB启动)

1. 根据虚拟机情况设置相应cpu和内存资源，并启动Minikube
    ``` shell
    minikube start --memory=6144 --cpus=4 --kubernetes-version=v1.20.2
    ```

    ![picture 7](https://libget.com/gkirito/blog/image/2021/image-20210305GM1ygnne%402x.png)  

2. （可选，推荐）如果你希望 minikube 提供一个负载均衡给 Istio，你可以使用 `minikube tunnel`。 在另一个终端运行这个命令，因为 minikube tunnel 会阻塞的你的终端用于显示网络诊断信息。删除可用`minikube tunnel --cleanup`

3. 下载Istio 
    ``` shell
    curl -L https://istio.io/downloadIstio | sh -
    ```

4. 设置Istio临时环境
    ``` shell
    cd istio-1.9.1
    export PATH=$PWD/bin:$PATH
    ```
5. 安装Istio
   ``` shell
   istioctl install
   ```
   或者可以设定安装时的配置文件
    如：
    ``` shell
    istioctl install --set profile=demo -y
    # Or
    istioctl install -f my-config.yaml
    ```
    ``` yml
    # my-config.yaml
    apiVersion: install.istio.io/v1alpha1
    kind: IstioOperator
    spec:
      meshConfig:
        accessLogFile: /dev/stdout
    ```
6. 给命名空间添加标签，指示 Istio 在部署应用的时候，自动的注入 Envoy 边车代理：
    ``` shell
    kubectl label namespace default istio-injection=enabled
    ```                  

    ![picture 9](https://libget.com/gkirito/blog/image/2021/image-20210305FY0MYLLD%402x.png)  


## AWS搭建
> 要求：   
> 1. AWS搭建好EKS集群
> 2. 配置好本地当前kubectl的config

在确定好kubectl的config为aws的集群后，执行之前本地搭建的2-6步骤，最后可在aws上看到Istio相应基础Delpyment已经加载完成

![picture 8](https://libget.com/gkirito/blog/image/2021/image-20210305AK8yuUJU%402x.png)  

![picture 10](https://libget.com/gkirito/blog/image/2021/image-20210305ie5M0EaE%402x.png)  

---
title: 'Djang-sspanel与vscar搭建trojan'
subtitle: ''
date: 2020-11-12T08:51:23+08:00
lastmod: 2020-11-12T08:51:23+08:00
draft: false
slug: sspanel-vscar-trojan-build
author: ""
authorLink: ""
description: ""
images: ""
tags:
  - gfw
  - Trojan
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

> 本文选用框架：
>
> 来自于[Ehco](https://github.com/Ehco1996)的[django-sspanel](https://github.com/Ehco1996/django-sspanel)和[v2scar](https://github.com/Ehco1996/v2scar)
>
> 来自于[千影](https://www.94ish.me/)的[Linux 网络优化加速一键脚本](https://www.94ish.me/1635.html)（内函 BBR、BBR 膜改、锐速）

## 1. 首先安装 Nginx 并配置域名和 HTTPS

关于这块可以参考我的另一篇博客 [使用 HTTPS——SSL 申请](https://www.gkirito.com/usehttps/)

## 2. 安装环境

这里参照了[菜鸟教程](https://www.runoob.com/docker/centos-docker-install.html)，不同系统按侧边栏选择

**docker 安装**

```shell
curl -fsSL https://get.docker.com | bash -s docker
sudo systemctl start docker
sudo systemctl enable docker.service
```

docker-compose 安装

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.27.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

```shell
sudo yum install git
```

## 3. 关闭 Nginx 并且配置 Trojan

由于 Trojan 需要使用 443 端口(TLS)，所以关闭 Nginx

```shell
sudo systemctl stop nginx.service
# 或者
nginx -s quit
```

然后用命令检查一下 `443`端口是否还有被占用

```shell
netstat -anp | grep 443
```

从 nginx 的配置文件中获取 cert 和 key 的文件位置：

```shell
cat /etc/nginx/conf.d/web.conf
```

**注意 ⚠️：** `web.cof` 为我此次测试机上配置文件名，请以自己实际为准

找到这两条：
![image-20201112i8i2833R@2x](https://libget.com/gkirito/blog/image/2020/image-20201112i8i2833R@2x.png)

- `ssl_certificate` : 为 cert 地址

- `ssl_certificate_key` : 为 key 地址

分别填入[django-sspanel](https://github.com/Ehco1996/django-sspanel) 的配置文件中

- `服务端端口`: 如果 v2scar 不采用 docker 形式，填写 443，如果采用 docker 形式随意

- `客户端端口`: 443

- `连接方式` : tcp （目前只支持 tcp）

- `加密方式` : tls

- `是否允许不安全连接(跳过tls验证)` : 否（采用真 tls）

- `alpn` : http/1.1

在服务器上 git 下最新的[v2scar](https://github.com/Ehco1996/v2scar)

```shell
git clone https://github.com/Ehco1996/v2scar.git
cd v2scar/
```

修改`docker-compose.yml`文件

```yaml
services:
  v2ray:
    image: v2fly/v2fly-core:latest
    container_name: v2ray
    restart: always
    volumes:
      - ./v2ray-config.json:/etc/v2ray/config.json
      - /etc/letsencrypt/live/XXX.XXX.com/fullchain.pem:/etc/letsencrypt/live/XXX.XXX.com/fullchain.pem
      - /etc/letsencrypt/live/XXX.XXX.com/privkey.pem:/etc/letsencrypt/live/XXX.XXX.com/privkey.pem
    ports:
      - '10086:10086'
      # command: ["v2ray","-config=https://xxx.com"]   # 取消command前的注释，并填入django-sspanel的对接连接，这里填入vmess_server_config

  v2scar:
    container_name: v2scar
    image: ehco1996/v2scar
    restart: always
    depends_on:
      - v2ray
    links:
      - v2ray
    environment:
      V2SCAR_SYNC_TIME: 60
      V2SCAR_API_ENDPOINT: '' # 填入django-sspanel的对接连接，这里填入user_vmess_config
      V2SCAR_GRPC_ENDPOINT: 'v2ray:8080'
```

在**v2ray**的**volumes**下新增两条，分别是之前填入[django-sspanel](https://github.com/Ehco1996/django-sspanel) 中的 cert 和 key 的地址，中间`冒号`之后再复制一遍。

v2ray 的端口`ports`:

冒号前改为 443，冒号后改为之前[django-sspanel](https://github.com/Ehco1996/django-sspanel) 所填端口

## 4. 启动 v2scar

```shell
docker-compose up -d
```

启动后在 v2ray 的日志中看到这个：

`unknown config id: trojan > exit status 255`

定位到是 docker 的 images 镜像太老，导致使用 v2scar 是旧版本，更新一下镜像就好

```shell
docker pull ehco1996/v2scar
docker pull v2fly/v2fly-core
```

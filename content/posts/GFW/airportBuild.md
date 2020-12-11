---
title: '机场搭建 django-sspanel+v2scar v2ray+ws+tls'
subtitle: ''
date: 2020-07-11T01:13:03+08:00
lastmod: 2020-12-11T01:13:03+08:00
draft: false
author: ""
authorLink: ""
description: ""
images: ""
tags:
  - gfw
  - Vmess
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

## 1.准备 VPS

先去买个 vps，买完后最好测速一下延迟什么的，当然位置离得越近越好，这里在[vultr](https://my.vultr.com/)上买了个 vps，选了日本。

![image-20200407063903471](https://libget.com/gkirito/blog/image/2020/image-20200407063903471.png)

然后用[ping 查询网站](https://tools.ipip.net/newping.php)测试了一下 ping 延迟，当然了得等服务器开启后一会儿再测。

## 2.上服务器进行 BBR 优化

> 一下摘自千影博客

### 1）ssh 登录服务器

### 2）下载一键脚本

```shell
wget -N --no-check-certificate "https://raw.githubusercontent.com/chiakge/Linux-NetSpeed/master/tcp.sh"
chmod +x tcp.sh
./tcp.sh
```

### 3）根据需求安装 BBR，这里使用了 BBRplus

安装内核需要重启系统，安装完内核，再安装模块，最后可以进行一下系统优化

![image-20200407083100256](https://libget.com/gkirito/blog/image/2020/image-20200407083100256.png)

> **脚本说明**
>
> 支持系统
> Centos 6+ / Debian 7+ / Ubuntu 14+
> BBR 魔改版不支持 Debian 8
>
> 如果在删除内核环节出现这样一张图
>
> ![3363374172.png](https://www.94ish.me/usr/uploads/2017/10/4150223835.png)
>
> 注意选**NO**

## 3. docker 和 docker-componse

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

确保都安装完成

![image-20200407084436507](https://libget.com/gkirito/blog/image/2020/image-20200407084436507.png)

## 4. 安装 git

```shell
sudo yum install git
```

## 5. 域名和安装配置 Nginx 的 HTTPS

关于这块可以参考我的另一篇博客 [使用 HTTPS——SSL 申请](https://www.gkirito.com/usehttps/)

## 6. 配置 v2scar

```shell
git clone https://github.com/Ehco1996/v2scar.git
cd v2scar/
```

修改 docker-compose.yml

```yaml
version: '3'

services:
  v2ray:
    image: v2fly/v2fly-core:latest
    container_name: v2ray
    restart: always
    volumes:
      - ./v2ray-config.json:/etc/v2ray/config.json
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

启动 v2scar

```shell
docker-compose up -d
```

## 7.开始 ws+tls 配置

1. django-sspanel 修改配置

   ![image-20200407095407431](https://libget.com/gkirito/blog/image/2020/image-20200407095407431.png)

> 上图本地监听地址为了安全改为 127.0.0.1，Grpc 地址可以改为 django-sspanel 服务器所配置的地址
>
> WS_path 中，uuid 之前记得添加“/”
>
> **图片中忘了一步，端口改为 443**

2.  去之前的服务器修改 Nginx 的配置

```shell
vi /etc/nginx/conf.d/web.conf
```

会发现配置文件已经改变为

```config
server{
    server_name xjp.gkirito.monster ;
    access_log  /var/log/nginx/access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/xjp.gkirito.monster/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/xjp.gkirito.monster/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server{
    if ($host = xjp.gkirito.monster) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name xjp.gkirito.monster ;
    return 404; # managed by Certbot
}
```

我们修改部分，添加代理

```config
server{
    server_name XXX.com ;
    access_log  /var/log/nginx/access.log  main;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
    location /XXXX-XXXX-XXX... {   # XXX部分填写之前生成的uuid
        proxy_redirect off;
        proxy_pass http://127.0.0.1:10086;#之前监听地址改成了本地，端口根据服务器上v2scar开的端口填
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;

        # Show realip in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
.......
一下为ssl配置信息，不用修改
```

然后检查 Nginx 配置，并重载

好了现在所有配置都已经完成，测试连通就行

如果测试过程中连不上网，抓取一下 Nginx 日志

```shell
cat /var/log/nginx/error.log
```

如果发现**Permission denied**字样说明是**SELinux 造成的**

```shell
setsebool -P httpd_can_network_connect 1
```

{{< admonition type=bug title="注意" open=false >}}

这有个`BUG`[#304](https://github.com/Ehco1996/django-sspanel/issues/304)暂时给出一下解决：

~~原本到了这里是应该结束了，但是目前 django-sspanel 的客户端有点小问题，由于节点端口设置会和 docker 中开启端口还有订阅配置中的端口一致，在使用 ws+tls 后，客户端订阅端口应该为 443，所以 django-sspanel 的端口应该改为 443，同时 v2scar 的 docker-compose 中 v2ray 的端口映射应改为 10086:443~~
{{< /admonition>}}

{{< admonition type=success title="问题已修复" open=true >}}

此 PR[#324](https://github.com/Ehco1996/django-sspanel/pull/324)后修复问题，以后只需服务端设置 10086 端口，客户端设置 443 端口即可

{{</admonition>}}

---
title: "使用HTTPS——SSL申请"
subtitle: ""
date: 2020-11-11T22:04:27+08:00
lastmod: 2020-12-11T22:04:27+08:00
draft: false
author: ""
authorLink: ""
description: ""
images: ""
tags:
  - TLS
categories: 
  - Web
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

> 本文选用：`Let’s Encrypt`作为证书颁发机构
>
> 采用软件：[Certbot](https://certbot.eff.org/) 、[snapd](https://snapcraft.io/) 、Nginx

## 开头杂谈

`HTTPS`作为目前网站的常用要求，除了在访问网站上，能让浏览器出现个小🔒，还能在某些科学事业上用户真`TLS`伪装。


## 1. HTTP服务验证获得证书
### 1.1 域名

对于配置`HTTPS`我们首先需要一个域名

当然了为了省去备案什么的麻烦，建议在国外的域名网站上购买（毕竟有看到网友说套路云好像解析不了vultr）这里我在[namesilo](https://www.namesilo.com/)上购买了一个域名，然后点击这里

![image-20200407075506372](https://libget.com/gkirito/blog/image/2020/image-20200407075506372.png)

把原先给你自动生成的都删除，然后点击上方的**A**

![image-20200407075726881](https://libget.com/gkirito/blog/image/2020/image-20200407075726881.png)

添加一个域名解析，然后就是等待了，他们的dns刷新提交大概每15分钟一次，所以这一步拿到前面做。15分钟后建议去[ping查询网站](https://tools.ipip.net/newping.php)用域名ping一下效果

当然也可以买了域名交给[cloudflare.com](cloudflare.com)来解析，cf的DNS解析速度比较快。

### 1.2 Certbot安装

由于使用[Let’s Encrypt](https://letsencrypt.org/)作为证书颁发机构，所以根据官网文档，我们直接采用[certbot](https://certbot.eff.org/)作为申请工具

针对不同系统的安装方式，可在[certbot](https://certbot.eff.org/)官网查看，一下以`CentOS 7`为例

#### 1.2.1 首先安装snapd

``` shell
sudo yum install epel-release
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
```

在`yum`安装过程中出现了`error`：

![image-20201112qXZerCQl@2x](https://libget.com/gkirito/blog/image/2020/image-20201112qXZerCQl@2x.png)

搜了一下，需要开启``CR(Continuous Release)`repository，命令如下：

``` shell
sudo yum-config-manager --enable cr
```

之后继续 `sudo yum install snapd`即可

#### 1.2.2 确保安装了最新的snapd

``` shell
sudo snap install core
sudo snap refresh core
```

#### 1.2.3 删除服务器上多余的Certbot

```shell
# Ubuntu
sudo apt-get remove certbot
# Fedora
sudo dnf remove certbot
# CentOS
sudo yum remove certbot
```

#### 1.2.4 安装Certbot

```shell
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 1.3 安装Nginx

```shell
# 安装 Nginx
sudo yum install nginx

# 启动 Nginx
sudo systemctl start nginx.service

# 设置开机自启 Nginx
sudo systemctl enable nginx.service
```

这个时候去访问一下网页，如果发现不能访问，检查一下80端口是否已经在了。

检查的工具可能需要装一下

``` shell
sudo yum install net-tools
```

然后检查一下80端口

``` shell
netstat -lnp | grep 80
```

![image-20200407090516883](https://libget.com/gkirito/blog/image/2020/image-20200407090516883.png)

确定80端口已经由Nginx开放，但是仍旧无法访问，应该是iptables或者firewall防火墙的问题了

**解决方案：**

1. 关闭iptables和firewall，一劳永逸

    ``` shell
    # iptables
    systemctl stop iptables.service
    # firewall
    systemctl stop firewalld.service
    ```

   

2. 为80端口添加通行

    ``` shell
    # iptables
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    iptables -A INPUT -p tcp --dport 443 -j ACCEPT
    service iptables save
    systemctl restart iptables.service
    # firewall
    firewall-cmd --zone=public --add-port=80/tcp --permanent
    firewall-cmd --zone=public --add-port=443/tcp --permanent
    # --zone #作用域
    # --add-port=80/tcp  #添加端口，格式为：端口/通讯协议
    # --permanent   #永久生效，没有此参数重启后失效
    systemctl restart firewalld.service
    ```

当然了为了方便后面配置ws+tls，这里先把443端口也开放

这时候再访问自己的ip，就能看到Nginx的欢迎页面，或者是CentOS的介绍页面

### 1.4 配置Nginx和ssl

先去Nginx的conf.d目录下建一个conf文件，然后参照一下我一下这个文件

``` shell
vi /etc/nginx/conf.d/web.conf
```

XXX.com改为你刚刚绑定好的域名

``` 
server{
    listen 80;
    server_name XXX.com;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

然后检查一下Nginx的配置，并重载配置

``` shell
# 检查配置
nginx -t
# 重载
nginx -s reload
```

![image-20200407092907557](https://libget.com/gkirito/blog/image/2020/image-20200407092907557.png)

再去浏览器用域名访问一下，如能正确访问，准备开始ssl申请

``` shell
# 执行配置，中途会询问你的邮箱，如实填写即可
sudo certbot --nginx

# 自动续约检查
sudo certbot renew --dry-run
```

完成后再次访问网页，注意用https访问，可以正常访问

## 2. 域名DNS验证方式获取证书

> 以下Shell为Ubuntu使用
> 
> 此教程对接cloudflare的API
> 
> certbot支持的plugins有这些
> 
> ![image-20201220Yn3xIkmB@2x](https://libget.com/gkirito/blog/image/2020/image-20201220Yn3xIkmB@2x.png)
### 2.1 域名和Certbot安装
与方法1一样，先准备域名和Certbot安装，域名这里交给了[cloudflare](https://dash.cloudflare.com/)

#### 2.1.1 安装snapd
``` shell 
sudo apt install -y snapd
sudo systemctl start snapd.service
sudo snap install core; sudo snap refresh core
```
#### 2.1.2 卸载多余Certbot并安装Certbot

``` shell 
 # Ubuntu
sudo apt-get remove certbot
# install
sudo snap install --classic certbot
# prepare the command
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
如果遇到这个问题
![image-20210101DM1ioGe1@2x](https://libget.com/gkirito/blog/image/2020/image-20210101DM1ioGe1@2x.png)
输入以下解决：

``` shell
sudo ln -s /var/lib/snapd/snap /snap
```

### 2.2 certbot-dns-cloudflare插件安装

``` shell
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```
然后运行
``` shell
certbot plugins
```
检查certbot-dns-cloudflare插件是否安装成功
![](https://libget.com/gkirito/blog/image/2020/image-20201220mgzMZEym@2x.png)

### 2.3 创建Cloudflare的API的key

登录Cloudflare，点击右上角 头像->我的个人资料->API令牌->创建令牌->编辑区域 DNS->使用模板
![image-20201217QKdVLj8k@2x.png](https://libget.com/gkirito/blog/image/2020/image-20201217QKdVLj8k@2x.png)
在上图位置选择好对应域名，然后
继续以显示摘要->创建令牌


### 2.4 创建存储API调用凭证目录
``` sehll
mkdir -p ~/.secrets/certbot
chmod 0700 ~/.secrets
vim ~/.secrets/certbot/cloudflare.ini
```
在`cloudflare.ini`文件内填入[2.3 创建Cloudflare的API的key](#23-创建cloudflare的api的key)中创建的key
``` text
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
```
**注意⚠️**如果`cloudflare python module`版本过低，只能使用`Global API Key`，所以[2.3 创建Cloudflare的API的key](#23-创建cloudflare的api的key)创建的key需为`Global API Key`，然后按以下格式填入
``` text
# Cloudflare API credentials used by Certbot
dns_cloudflare_email = cloudflare@example.com
dns_cloudflare_api_key = 0123456789abcdef0123456789abcdef01234
```
最后文件赋予权限
``` shell
chmod 0400 ~/.secrets/certbot/cloudflare.ini
```

### 2.5 申请ssl证书

``` shell
sudo certbot certonly --email XXX@XX --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini  -d XXX.com,*.XXX.com
```
`XXX.com`为你的域名

`*.XXX.com`为申请根域

## 注意

1. Let’s Encrypt的免费证书只有`90`天的有效期。
2. certbot具有到期自动续的功能，但是也要留意邮箱的里的到期通知，注意证书防止过期



## 补充

关于`certbot`的命令这里补充一些：

```shell
# 查看已经申请的证书
sudo certbot certificates

# 撤销证书（revoking certificates）
certbot revoke --cert-path /etc/letsencrypt/live/{你的域名}/cert.pem --reason keycompromise
```

撤销证书后面的`--reason`有这些选项(可不带)：

unspecified（默认）， keycompromise, affiliationchanged, superseded, 和 cessationofoperation

一旦证书被revoke后，可以使用delete命令删除证书

```shell
sudo certbot delete --cert-name example.com
```

**注意**：如果你revoke一个证书，那么如果不delete的话，当renew的时候该证书仍然会被更新。
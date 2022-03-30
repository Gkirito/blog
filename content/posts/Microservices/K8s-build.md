---
title: "Kubernetes搭建记录📝"
subtitle: ''
date: 2021-03-05T18:38:16+08:00
lastmod: 2021-03-05T18:38:16+08:00
draft: false
author: "gkirito"
authorLink: "https://www.gkirito.com"
description: ""
images: ""
tags: 
  - k8s
  - Kubernetes
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
toc: true
math: true
comment: true
lightgallery: false
license: ""
---

# Kubernetes搭建记录📝

## 工具🔧准备
### 安装kubectl
Kubernetes 命令行工具，kubectl，使得你可以对 Kubernetes 集群运行命令。 你可以使用 kubectl 来部署应用、监测和管理集群资源以及查看日志。

话不多说上包管理器  
Snap
``` shell
snap install kubectl --classic
```
``` shell
brew install kubectl
```
考虑到后期经常需要敲`kubectl`，可以设置一个别名，同时配置shell补全功能
``` shell
echo 'alias k=kubectl' >>~/.zshrc
echo 'complete -F __start_kubectl k' >>~/.zshrc
```

## 本地Kubernetes搭建
### Docker for Desktop内K8s
通过`Preferences`开启Kubernetes，在Kubernetes中设置`enable`即可，
之后会pull images，但有极大可能遇到images下载失败导致一直处于`starting`状态，建议移除相关镜像重下，不过本次尝试下来似乎移除重启Docker没有效果。  

![picture 1](https://libget.com/gkirito/blog/image/2021/image-20210304sbKUz3ye%402x.png)  

出现情况：一直处于这个状态  

![picture 2](https://libget.com/gkirito/blog/image/2021/image-20210304OFCpQ5Fz%402x.png)  

重试几次未好在Issues中找到一位大佬操作：  

![picture 3](https://libget.com/gkirito/blog/image/2021/image-20210304IrjrX6b9%402x.png)  

直接暴力点，移除`~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/`
下的`Docker.raw`文件  
**警告**：  
`Docker.raw`为docker image文件存放位置，删除会导致所有其他镜像的删除，但是能解决问题(至少我是这样最后成功run了Kubernetes)

### minikube
安装
``` shell
brew install minikube
```
开启
``` shell
minikube start
```
等待镜像pull完，一个小型k8s集群就搭建好了，
启动完成后，kubectl的config就会自动切到minikube，这时我们便可以进行正常操作了

![picture 4](https://libget.com/gkirito/blog/image/2021/image-20210304d9GXAqPS%402x.png)  

可以通过该命令来查看k8s dashboard
``` shell 
minikube dashboard
```

![picture 10](https://libget.com/gkirito/blog/image/2021/image-20210304qLyTQJrr%402x.png)  


可以尝试创建一个`Deployment`,可以通过指定`replicas`来设置pod
``` shell
kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4 --replicas=2
```
然后可以通过查看Deployment和pod看到结果
``` shell
kubectl get deployments,pods
```

![picture 5](https://libget.com/gkirito/blog/image/2021/image-20210304LlCgMxEh%402x.png)  

默认情况下，Pod 只能通过 Kubernetes 集群中的内部 IP 地址访问。 要使得 hello-node 容器可以从 Kubernetes 虚拟网络的外部访问，你必须将 Pod 暴露为 Kubernetes Service。
``` shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

这时查看Service，便能看到端口暴露信息
``` shell
kubectl get services
```

![picture 6](https://libget.com/gkirito/blog/image/2021/image-202103049uzDZEGo%402x.png)  

对于支持负载均衡器的云服务平台而言，平台将提供一个外部 IP 来访问该服务。 在 Minikube 上，LoadBalancer 使得服务可以通过命令 minikube service 访问。
``` shell
minikube service hello-node
```
![picture 7](https://libget.com/gkirito/blog/image/2021/image-20210304WBVSE96J%402x.png)  

![picture 8](https://libget.com/gkirito/blog/image/2021/image-20210304wROeku1e%402x.png)  

> 补充：  
> Minikube 有一组内置的 插件， 可以在本地 Kubernetes 环境中启用、禁用和打开
> ``` shell
> minikube addons list
> ```
> ![picture 9](https://libget.com/gkirito/blog/image/2021/image-20210304O9ZrdZyu%402x.png)  
> 启用与禁用插件
> ``` shell
> minikube addons enable metrics-server
> minikube addons disable metrics-server
> ```

最后可以用k8s删除service和deployment,暂停minikube
``` shell
kubectl delete service hello-node
kubectl delete deployment hello-node

minikube stop
```

## AWS上Kubernetes搭建(EKS方案)
### AWS平台工具🔧安装

#### AWS CLI 版本 2
1. 下载对应系统的AWS CLI 
    [https://awscli.amazonaws.com/AWSCLIV2.pkg](https://awscli.amazonaws.com/AWSCLIV2.pkg)

2. 安装完以后，在Terminal终端验证
    ``` shell
    which aws
    aws --version
    ```
    ![picture 11](https://libget.com/gkirito/blog/image/2021/image-20210304MKrh1STt%402x.png)  

3. 配置 AWS CLI  
    > 配置前准备：  
    > 首先配置AWS CLI需要`aws access key`和`aws secret access key`,由于`root`账号没法获得这个，所以需要去IAM建立user
    1. 首先去`Groups`创建一个新的Group，在`Permissions`上，可以把EKS(Elastic Kubernetes Service)相关的都选上
    2. 在`Users`添加一个新的user，填入名字；把`Programmatic access`和`AWS Management Console access`都选上，可做如下配置：  

        ![picture 12](https://libget.com/gkirito/blog/image/2021/image-20210305MR2Hy31F%402x.png)  

    3. 下一步勾选之前创建的Group；之后再往下一步，设置Tags(随意)
    4. 确定之后点击创建  
    
    **注意**：创建成功后需要记住`Secret access key`和`Password`(特别是password自动生成情况)，这两种之后便无法再次查看，同时`下载 .csv 文件`备份;点击完成后，还要再去Users下，点击创建的user，获得`User ARN`并保存

    ![picture 13](https://libget.com/gkirito/blog/image/2021/image-20210305s1GkRwsw%402x.png)  

    根据上面方式获得`aws access key`和`aws secret access key`后，在Terminal上执行
    ``` shell
    aws configure
    ```
    ![picture 14](https://libget.com/gkirito/blog/image/2021/image-20210305pAYmjSiO%402x.png)  
    
    根据要求填入即可

#### eksctl

安装
``` shell
brew install weaveworks/tap/eksctl
```
并通过
``` shell
eksctl version
```
确定安装成功

### 通过平台面板创建EKS
1. 先登录之前创建的子账号(IAM用户)

   ![picture 15](https://libget.com/gkirito/blog/image/2021/image-20210305MaEbyGnk%402x.png)  

    *账户ID(12位)即之前保存的`User ARN`中间的12位数字*

2. 登录以后选择`IAM`，在`Roles`下添加新的role，选择EKS --> EKS-Cluster,点击下一步添加权限，权限默认即可，之后添加Tags，最后为这个role添加名字

3. 创建`VPC`，`IPv4 CIDR block`可以填：`192.168.0.0/16`;之后在VPC下创建`Subnets`，选择之前创建的VPC，在`IPv4 CIDR block`中选择一个192.168下的段地址，然后重复创建多个，分别用不同的段，如：`192.168.0.0/24、192.168.10.0/24、192.168.20.0/24`等   

    ![picture 1](https://libget.com/gkirito/blog/image/2021/image-20210305izQM4JKA%402x.png)  

4. 创建`Security Groups`，先创建EKS需要的基础，`Outbound rules`中默认即可，`Inbound rules`中随机选一个，然后点击创建，再点击`Edit inbound rules`，修改`source`选择刚刚创建的**Security Groups**，保存

5. 选择`Elastic Kubernetes Service`，输入`Cluster name`后点击下一步，选择Kubernetes版本、`Cluster Service Role`选择刚刚创建的role,然后下一步；选择之前创建的VPC相关内容，`Cluster endpoint access`选为`public`，之后继续下一步，`Control Plane Logging`上可选都打开；最后`review`确定没有问题后选择创建

    ![picture 2](https://libget.com/gkirito/blog/image/2021/image-20210305czZUfRqP%402x.png)  


### 通过eksctl创建EKS

eksctl创建会比之前手动面板创建方便不少，根据需求填入相应参数，就会自动创建，可以不用再关注于VPC、IAM等role等方面。  
可以通过
``` shell
eksctl create cluster -h
```
获知一些相关参数信息

```
Create a cluster

Usage: eksctl create cluster [flags]

General flags:
  -n, --name string               EKS cluster name (generated if unspecified, e.g. "hilarious-wardrobe-1614914845")
      --tags stringToString       Used to tag the AWS resources. List of comma separated KV pairs "k1=v1,k2=v2" (default [])
  -r, --region string             AWS region
      --with-oidc                 Enable the IAM OIDC provider
      --zones strings             (auto-select if unspecified)
      --version string            Kubernetes version (valid options: 1.15, 1.16, 1.17, 1.18, 1.19) (default "1.18")
  -f, --config-file string        load configuration from a file (or stdin if set to '-')
      --timeout duration          maximum waiting time for any long-running operation (default 25m0s)
      --install-vpc-controllers   Install VPC controller that's required for Windows workloads
      --fargate                   Create a Fargate profile scheduling pods in the default and kube-system namespaces onto Fargate

Initial nodegroup flags:
      --nodegroup-name string          name of the nodegroup (generated if unspecified, e.g. "ng-87f0eb81")
      --without-nodegroup              if set, initial nodegroup will not be created
  -t, --node-type string               node instance type
  -N, --nodes int                      total number of nodes (for a static ASG) (default 2)
  -m, --nodes-min int                  minimum nodes in ASG (default 2)
  -M, --nodes-max int                  maximum nodes in ASG (default 2)
      --node-volume-size int           node volume size in GB (default 80)
      --node-volume-type string        node volume type (valid options: gp2, gp3, io1, sc1, st1) (default "gp3")
      --max-pods-per-node int          maximum number of pods per node (set automatically if unspecified)
      --ssh-access                     control SSH access for nodes. Uses ~/.ssh/id_rsa.pub as default key path if enabled
      --ssh-public-key string          SSH public key to use for nodes (import from local path, or use existing EC2 key pair)
      --enable-ssm                     Enable AWS Systems Manager (SSM)
      --node-ami string                'auto-ssm', 'auto', 'static' (deprecated) or an AMI id (advanced use)
      --node-ami-family string         'AmazonLinux2' for the Amazon EKS optimized AMI, or use 'Ubuntu2004' or 'Ubuntu1804' for the official Canonical EKS AMIs (default "AmazonLinux2")
  -P, --node-private-networking        whether to make nodegroup networking private
      --node-security-groups strings   attach additional security groups to nodes
      --node-labels stringToString     extra labels to add when registering the nodes in the nodegroup. List of comma separated KV pairs "k1=v1,k2=v2" (default [])
      --node-zones strings             (inherited from the cluster if unspecified)
      --instance-prefix string         add a prefix value in front of the instance's name
      --instance-name string           overrides the default instance's name
      --disable-pod-imds               Blocks IMDS requests from non host networking pods
      --managed                        Create EKS-managed nodegroup
      --spot                           Create a spot nodegroup (managed nodegroups only)
      --instance-types strings         Comma-separated list of instance types (e.g., --instance-types=c3.large,c4.large,c5.large

Cluster and nodegroup add-ons flags:
      --install-neuron-plugin    install Neuron plugin for Inferentia nodes (default true)
      --install-nvidia-plugin    install Nvidia plugin for GPU nodes (default true)
      --asg-access               enable IAM policy for cluster-autoscaler
      --external-dns-access      enable IAM policy for external-dns
      --full-ecr-access          enable full access to ECR
      --appmesh-access           enable full access to AppMesh
      --appmesh-preview-access   enable full access to AppMesh Preview
      --alb-ingress-access       enable full access for alb-ingress-controller

VPC networking flags:
      --vpc-cidr ipNet                 global CIDR to use for VPC (default 192.168.0.0/16)
      --vpc-private-subnets strings    re-use private subnets of an existing VPC
      --vpc-public-subnets strings     re-use public subnets of an existing VPC
      --vpc-from-kops-cluster string   re-use VPC from a given kops cluster
      --vpc-nat-mode string            VPC NAT mode, valid options: HighlyAvailable, Single, Disable (default "Single")

AWS client flags:
  -p, --profile string         AWS credentials profile to use (overrides the AWS_PROFILE environment variable)
      --cfn-role-arn string    IAM role used by CloudFormation to call AWS API on your behalf
      --cfn-disable-rollback   for debugging: If a stack fails, do not roll it back. Be careful, this may lead to unintentional resource consumption!

Output kubeconfig flags:
      --kubeconfig string               path to write kubeconfig (incompatible with --auto-kubeconfig) (default "/Users/gkirito/.kube/config")
      --authenticator-role-arn string   AWS IAM role to assume for authenticator
      --set-kubeconfig-context          if true then current-context will be set in kubeconfig; if a context is already set then it will be overwritten (default true)
      --auto-kubeconfig                 save kubeconfig file by cluster name, e.g. "/Users/gkirito/.kube/eksctl/clusters/hilarious-wardrobe-1614914845"
      --write-kubeconfig                toggle writing of kubeconfig (default true)

Common flags:
  -C, --color string   toggle colorized logs (valid options: true, false, fabulous) (default "true")
  -h, --help           help for this command
  -v, --verbose int    set log level, use 0 to silence, 4 for debugging and 5 for debugging with AWS debug logging (default 3)

Use 'eksctl create cluster [command] --help' for more information about a command.
```
我们可以简单创建一个4个node的集群
``` shell
eksctl create cluster \
 --name g4-test \
 --version 1.19 \
 --with-oidc \
 --region ap-southeast-1 \
 --nodes 4 \
 --nodegroup-name g4-node-test \
 --ssh-access
```
由于创建需要十几分钟，大概接下来一段时间都是waiting状态

 ![picture 5](https://libget.com/gkirito/blog/image/2021/image-20210305o7nlJwQq%402x.png)  


### 创建后kube config配置
可以通过AWS CLI快速切换集群配置
``` shell
aws eks --region ap-southeast-1 update-kubeconfig --name t
```
![picture 6](https://libget.com/gkirito/blog/image/2021/image-20210305Dfiwbap0%402x.png)  

## AWS上Kubernetes搭建(EC2手动搭建)
通过自建EC2服务器，设置搭建K8s集群

这部分内容过多，还未整理，到目前为止还未完全搭建完成
---
title: "微服务架构分析"
subtitle: ''
date: 2021-03-05T18:28:24+08:00
lastmod: 2021-03-05T18:28:24+08:00
draft: false
author: "gkirito"
authorLink: "https://www.gkirito.com"
description: ""
images: ""
tags: 
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

# 微服务架构分析

## 公司业务分析
### 目前涵盖业务

从Tower上看，目前公司业务范围有这些：     
  - PokaWorld
  - Coming
  - ChainX
  - Dapp钱包
  - ParaMarketCap  
.......  
涵盖业务多，且复杂
部分业务已经出现扩容复杂，例如[polkaworld.org](https://polkaworld.org/)打开加载在1000ms以上。

### 技术栈类型(涉及语言)
- Rust(ChainX链)
- NodeJs
- Python(爬虫)
- Java(Coming、ParaMarket等)
- PHP

### 服务量要求
1. 支持多语言环境，可接入公司所有业务
2. 支持快速简易弹性扩容
3. 支持多云服务商，例如阿里云、亚马逊等
4. 支持后续业务接入，如直播等

## 微服务   
微服务是指开发一个单个小型的但有业务功能的服务，每个服务都有自己的处理和轻量通讯机制，可以部署在单个或多个服务器上。微服务也指一种种松耦合的、有一定的有界上下文的面向服务架构。  
微服务的主要架构： 

![模块](https://libget.com/gkirito/blog/image/2020/module.png)

### Severless 
#### Severless技术介绍
Serverless是由事件（event）驱动（e.g. http,pub/sub）的全托管计算服务。用户无需管理服务器等基础设施，只需编写代码和选择触发器(trigger)，（比如rpc请求，定时器等）并上传。其余的工作（实例选择、 扩缩容、部署、容灾、监控、日志、安全补丁等）全部由serverless系统托管。用户只需要为代码实际运行消耗的资源付费——代码未运行则不产生费用。
Serverless 是云原生技术发展的高级阶段，使开发者更聚焦在业务逻辑，而减少对基础架构的关注。

![](https://libget.com/gkirito/blog/image/2020/p5.jpg)   

图中 serverless 就是构建在虚拟机和容器之上的一层，与应用本身的关系更加密切。   
serverless通常包含了两个领域BaaS(Backend as a Service)和FaaS(Function as a Service)   
- **BaaS**  
BaaS(Backend as a Service)后端即服务，一般是一个个的API调用后端或别人已经实现好的程序逻辑，比如身份验证服务Auth0，这些BaaS通常会用来管理数据，还有很多公有云上提供的我们常用的开源软件的商用服务，比如亚马逊的RDS可以替代我们自己部署的MySQL，还有各种其它数据库和存储服务。   
- **FaaS**  
FaaS(Functions as a Service)函数即服务，FaaS是无服务器计算的一种形式，当前使用最广泛的是AWS的Lambada。

#### Severless支持云厂家
目前Amazon、微软、Google、IBM、阿里云、腾讯云、华为云等都支持Severless服务，但是各家之间还是会有不同的操作过程。可能对于迁移不大友好

#### Severless使用场景
- 异步的并发，组件可独立部署和扩展
- 应对突发或服务使用量不可预测（主要是为了节约成本，因为 Serverless 应用在不运行时不收费）
- 短暂、无状态的应用，对冷启动时间不敏感
- 需要快速开发迭代的业务（因为无需提前申请资源，因此可以加快业务上线速度）      
  

根据以上，serverless可以用于类似这些场景：
- 机器学习及 AI 模型处理
- 图片处理
- IoT 传感器数据分析
- 流处理
- 聊天机器人

#### Severless优点
- 降低运营成本
- 降低开发成本
- 扩展能力     
- 更简单的管理
- 降低人力成本
- 降低风险
- 减少资源开销
- 增加缩放的灵活性
- 缩短创新周期
  
#### Severless缺点
- 状态管理  
  要想实现自由的缩放，无状态是必须的，而对于有状态的服务，使用serverless这就丧失了灵活性，有状态服务需要与存储交互就不可避免的增加了延迟和复杂性。
- 延迟  
  应用程序中不同组件的访问延迟是一个大问题，我们可以通过使用专有的网络协议、RPC调用、数据格式来优化，或者是将实例放在同一个机架内或同一个主机实例上来优化以减少延迟。而serverless应用程序是高度分布式、低耦合的，这就意味着延迟将始终是一个问题，单纯使用serverless的应用程序是不太现实的。
- 本地测试  
  Serverless应用的本地测试困难是一个很棘手的问题。
- 高度绑定  
  Serverless一般和云服务商高度绑定，日后切换是一个比较大的问题

### Service Mesh
#### Service Mesh技术介绍
Service Mesh 是一个专门处理服务通讯的基础设施层。它的职责是在由云原生应用组成服务的复杂拓扑结构下进行可靠的请求传送。在实践中，它是一组和应用服务部署在一起的轻量级的网络代理，并且对应用服务透明。

**Service Mesh发展过程：**  
#####  服务发现和负载均衡
- 模式一：传统集中式代理  
    在服务消费者和生产者之间，代理作为独立一层集中部署，由独立团队(一般是运维或框架)负责治理和运维。常用的集中式代理有硬件负载均衡器(如F5)，或者软件负载均衡器(如Nginx)，F5(4层负载)+Nginx(7层负载)这种软硬结合两层代理也是业内常见做法，兼顾配置的灵活性(Nginx比F5易于配置)。  

  ![](https://libget.com/gkirito/blog/image/2020/p3.png)
  
    **优点** ：模式一的最大好处是集中治理，应用不侵入，语言栈无关，另外因为模式一是集中部署的  
    **缺点**：单点问题，配置麻烦，多一跳性能开销大  
    案例：eBay、携程
- 模式二：客户端的嵌入式代理  
  这是很多互联网公司比较流行的一种做法，代理(包括服务发现和负载均衡逻辑)以客户库的形式嵌入在应用程序中。这种模式一般需要独立的服务注册中心组件配合，服务启动时自动注册到注册中心并定期报心跳，客户端代理则发现服务并做负载均衡。  

  ![](https://libget.com/gkirito/blog/image/2020/p2.png)

  **优点**：无单点，性能好  
  **缺点**：客户端复杂，多语言支持是个大问题  
  案例：Netflix开源的Eureka(注册中心)和Ribbon(客户端代理)、Dubbo等
- 模式三：主机独立进程代理(Sidecar)  
  是一种模式1和2的折中方案  

  ![](https://libget.com/gkirito/blog/image/2020/p4.png)

  **缺点**：运维较复杂  
  案例：Istio等  

服务网格从总体架构上来讲比较简单，不过是一堆紧挨着各项服务的用户代理，外加一组任务管理流程组成。代理在服务网格中被称为数据层或数据平面（data plane），管理流程被称为控制层或控制平面（control plane）。数据层截获不同服务之间的调用并对其进行“处理”；控制层协调代理的行为，并为运维人员提供 API，用来操控和测量整个网络。  

![](https://www.servicemesher.com/istio-handbook/images/service-mesh-schematic-diagram.png)

总而言之，Service Mesh 的基础设施层主要分为两部分：控制平面与数据平面。  
控制平面的特点：  
- 不直接解析数据包。
- 与控制平面中的代理通信，下发策略和配置。
- 负责网络行为的可视化。
- 通常提供 API 或者命令行工具可用于配置版本化管理，便于持续集成和部署。  
  

数据平面的特点：  
- 通常是按照无状态目标设计的，但实际上为了提高流量转发性能，需要缓存一些数据，因此无状态也是有争议的。
- 直接处理入站和出站数据包，转发、路由、健康检查、负载均衡、认证、鉴权、产生监控数据等。
- 对应用来说透明，即可以做到无感知部署。

#### RPC
##### RPC简介
RPC是指远程过程调用，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。
RPC框架要做到的最基本的三件事：  
1. 服务端如何确定客户端要调用的函数
2. 如何进行序列化和反序列化
3. 如何进行网络传输（选择何种网络协议） 
   

与REST对比  

|          | REST | RPC         |
| -------- | ---- | ----------- |
| 通信协议 | HTTP | 一般使用TCP |
| 性能     | 低   | 高          |
| 灵活度   | 高   | 低          |

RPC在微服务中的优势：  
能大大降低架构微服务化的成本，提高调用方与服务提供方的研发效率，屏蔽跨进程调用函数（服务）的各类复杂细节。让调用方感觉就像调用本地函数一样调用远端函数、让服务提供方感觉就像实现一个本地函数一样来实现服务。

##### RPC框架
目前开源的RPC框架有：
1. Dubbo(单语言带服务治理)  
   国内最早开源的RPC框架，有阿里巴巴公司开发并于2011年对外开源，只支持Java语言。  

   ![](https://libget.com/gkirito/blog/image/2020/image-20210225uUFreagC@2x.png)

    通信框架方面，默认采用Netty作为通信框架。  
    通信协议，出了支持私有的Dubbo协议外，还支持RMI协议，Hession协议，HTTP协议，Thrift协议等。  
    序列化，支持多种序列化格式，包括Dubbo、Hession、JSON、Kryo，FST等。

2. Motan(单语言带服务治理)  
   Motan 是国内另外一个比较有名的开源的 RPC 框架，同样也只支持 Java 语言实现  

    ![](https://libget.com/gkirito/blog/image/2020/image-20210225eJSoNG8q@2x.png)

    Motan与Dubbo类似,通信框架方面，默认使用 Netty NIO.  
    序列化协议支持 Hessian2 和Java序列化  
    通讯协议支持Motan、http、tcp、mc等

3. Tars(带多语言以及服务治理能力)  
   Tars 是腾讯根据内部多年使用微服务架构的实践，总结而成的开源项目，仅支持 C++ 语言  

    ![](https://libget.com/gkirito/blog/image/2020/image-20210225Nxi5KUTG@2x.png)

    tars协议采用接口描述语言（Interface description language，缩写IDL）来实现，它是一种二进制、可扩展、代码自动生成、支持多平台的协议，使得在不同平台上运行的对象和用不同语言编写的程序可以用PRC远程调用的方式相互通信交流， 主要应用在后台服务之间的网络传输协议，以及对象的序列化和反序列化等方面。


4. Spring Cloud(单语言带服务治理)  
   Spring Cloud 利用 Spring Boot 特性整合了开源行业中优秀的组件，整体对外提供了一套在微服务架构中服务治理的解决方案,只支持 Java 语言平台。  

   ![](https://libget.com/gkirito/blog/image/2020/image-20210226yATvEx7Q@2x.png)

   Spring Cloud微服务间通信一般采用两种方案  
   - **RestTemplate**  
   是spring中方便使用rest资源的一个对象，交互访问的资源通过URL进行识别和定位，每次调用都使用模板方法的设计模式，模板方法依赖于具体接口的调用，从而实现了资源的交互和调用；  它的交互方法有30多种，大多数都是基于HTTP的方法，比如：delete()、getForEntity()、getForObject()、put()、headForHeaders()等； 
   - **Feign**  
    一个声明式的伪HTTP客户端，使得编写HTTP客户端更加容易；它只需要创建一个接口，并且使用注解的方式去配置，即可完成对服务提供方的接口绑定，大大简化了代码的开发量；同时，它还具有可拔插的注解特性，而且支持feign自定义的注解和springMvc的注解(默认)；

5. gRPC(无服务治理类)  
   一个现代化的、高性能、开源和通用的RPC框架，可以在任何地方运行。它使客户端和服务端应用能够透明地通信，并更加容易的构建连接系统。  
   默认情况下gRPC使用[Protocol Buffers](https://developers.google.com/protocol-buffers/)。根据Google官方的介绍，Protocol Buffers是一种与语言和平台无关的、高效的、可扩展的自动序列化结构化的数据的机制，以便在通信协议、数据存储等方面使用。Protocol Buffers比XML小3到10倍，并且快20到100倍。使用生成数据访问类编译的.proto源文件很容易以编程方式使用。  

   ![](https://libget.com/gkirito/blog/image/2020/image-20210226SlcWF355@2x.png)

   主要特性:  
   1. 通信协议采用了 HTTP/2，因为 HTTP/2 提供了连接复用、双向流、服务器推送、请求优先级、首部压缩等机制
   2. IDL 使用了ProtoBuf，ProtoBuf 是由 Google 开发的一种数据序列化协议，它的压缩和传输效率极高，语法也简单
   3. 多语言支持，能够基于多种语言自动生成对应语言的客户端和服务端的代码。  
   
   支持语言：
   C# C++ Dart Go Java Kotlin/JVM Node Objective-C PHP Python Ruby

6. Thrift(无服务治理类)  
   Thrift 是一种轻量级的跨语言 RPC 通信方案，支持多达 25 种编程语言。为了支持多种语言，跟 gRPC 一样，Thrift 也有一套自己的接口定义语言 IDL，可以通过代码生成器，生成各种编程语言的 Client 端和 Server 端的 SDK 代码，这样就保证了不同语言之间可以相互通信。  

    ![](https://libget.com/gkirito/blog/image/2020/image-2021022604LmiUmj@2x.png)

    Thrift RPC 框架的特性:
    1. 支持多种序列化格式：如 Binary、Compact、JSON、Multiplexed 等
    2. 支持多种通信方式：如 Socket、Framed、File、Memory、zlib 等
    3. 服务端支持多种处理方式：如 Simple 、Thread Pool、Non-Blocking 等

   Thrift作为一款较老的RPC框架，在性能测试方面似乎一直是最好的（从19年左右的各类测试博客看）

    根据公司目前环境来看，RPC的选型上必定会选择gRPC或Thrift。从成熟度上来讲，Thrift 因为诞生的时间要早于 gRPC，所以使用的范围要高于 gRPC，在 HBase、Hadoop、Scribe、Cassandra 等许多开源组件中都得到了广泛地应用。而且 Thrift 支持多达 25 种语言，这要比 gRPC 支持的语言更多，所以如果遇到 gRPC 不支持的语言场景下，选择 Thrift 更合适。

    但 gRPC 作为后起之秀，因为采用了 HTTP/2 作为通信协议、ProtoBuf 作为数据序列化格式，在移动端设备的应用以及对传输带宽比较敏感的场景下具有很大的优势，而且开发文档丰富，根据 ProtoBuf 文件生成的代码要比 Thrift 更简洁一些，从使用难易程度上更占优势，所以如果使用的语言平台 gRPC 支持的话，建议还是采用 gRPC 比较好。

目前开源的RPC框架对比

|            | Dubbo | Motan | Tars | Spring Cloud | gRPC            | Thrift |
| ---------- | ----- | ----- | ---- | ------------ | --------------- | ------ |
| 多种序列化 | ✔️     | ✔️     | ✔️    | ✔️            | protocol buffer | thrift |
| 支持语言   | Java  | Java  | 5种  | Java         | 11种            | 25种   |

#### 优点
1. 微服务治理与业务逻辑的解耦。服务网格把 SDK 中的大部分能力从应用中剥离出来，拆解为独立进程，以 sidecar 的模式进行部署。服务网格通过将服务通信及相关管控功能从业务程序中分离并下沉到基础设施层，使其和业务系统完全解耦，使开发人员更加专注于业务本身。
2. 异构系统的统一治理。随着新技术的发展和人员更替，在同一家公司中往往会出现不同语言、不同框架的应用和服务，为了能够统一管控这些服务，以往的做法是为每种语言、每种框架都开发一套完整的 SDK，维护成本非常之高，而且给公司的中间件团队带来了很大的挑战。有了服务网格之后，通过将主体的服务治理能力下沉到基础设施，多语言的支持就轻松很多了。只需要提供一个非常轻量级的 SDK，甚至很多情况下都不需要一个单独的 SDK，就可以方便地实现多语言、多协议的统一流量管控、监控等需求。
3. 可观察性
4. 流量控制
5. 安全

#### 缺点
1. 对开发技术要求高，需要懂得更多的运维知识
2. 前期部署比较麻烦

#### Service Mesh框架介绍
目前开源的Service Mesh框架有：
   Istio  
   Istio 是由 Google、IBM 和 Lyft 开源的微服务管理、保护和监控框架。Istio 为希腊语，意思是”起航“。  

  ![](https://libget.com/gkirito/blog/image/2020/istio-mindmap.png)

  Istio的前身是IBM开源的Amalgam8
  Amalgam8是一款基于内容和版本的路由布局，用于集成多语言异构体微服务。 其control plane API可用于动态编程规则，用于在正在运行的应用程序中跨微服务进行路由和操作请求。


  由于Istio和k8s深度结合，所以我们先了解一下k8s

##### Kubernetes 
  Kubernetes架构  

  ![](https://libget.com/gkirito/blog/image/2020/architecture.png)

  Kubernetes主要由以下几个核心组件组成：
- etcd保存了整个集群的状态；
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制；
- controller manager负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上；
- kubelet负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）=的管理；
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）；
- kube-proxy负责为Service提供cluster内部的服务发现和负载均衡；
- 除了核心组件，还有一些推荐的插件，其中有的已经成为CNCF中的托管项目：

CoreDNS负责为整个集群提供DNS服务
Ingress Controller为服务提供外网入口
Prometheus提供资源监控
Dashboard提供GUI
Federation提供跨可用区的集群


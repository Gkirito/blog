---
title: "Arduino Pro mini 刷入 uno系统"
slug: Arduino-Pro-mini-Brush-in-Uno-System
date: 2020-07-11T08:16:55+08:00
draft: false
toc: false
images:
tags:
  - Arduino
---

1. 按照图片链接2块arduino板，其中原来的arduino版是programmer，需要刷入的是target。

   ![p1](https://libget.com/gkirito/blog/image/190429/15565024783863.jpg)

2. 将programmer板链接PC。

3. 在PC上，打开arduino IDE，依次点击文件->示例->Arduino ISP->ArduinoISP
   
   ![p2](https://libget.com/gkirito/blog/image/190429/15565024937650.jpg)

4. 工具->开发板->Arduino/Genuino Uno
   
   ![p3](https://libget.com/gkirito/blog/image/190429/15565025353696.jpg)
   
5. 工具->编程器->Arduino as ISP

   ![p4](https://libget.com/gkirito/blog/image/190429/15565025632785.jpg)

6. 确定好工具->端口中选择的单片机

7. 先点击编译上传

   ![p5](https://libget.com/gkirito/blog/image/190429/15565025898082.jpg)

8. 等待上传完成，点击工具->烧录引导程序

   ![p6](https://libget.com/gkirito/blog/image/190429/15565026170693.jpg)
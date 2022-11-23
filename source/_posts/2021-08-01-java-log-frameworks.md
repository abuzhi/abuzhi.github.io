---
title: Java程序Log框架梳理及冲突解决
author: abuzhi
date: 2021-08-01 18:32:00
categories: [Java, LogFramework]
tags: [Java Log, Log4j, Slf4j, logback]
---
* content
{:toc}


Java程序Log框架梳理及冲突解决

![](/images/2021-08-01/image2021-3-10_18_52_25.png)



## 一、Log框架简介

java的log框架分为两类：

记录型日志框架: 具体实现log功能的框架，也就是代码运行时，真正append log数据的框架，目前有四个

Log4j  ： 最初的log框架
J.U.L ：jdk自带的log类，(java.util.logging) 也常称为JDKLog、jdk-logging，自Java1.4以来的官方日志实现。
Log4j2-core ：log4j的升级版，有很多
Logback：log4j2诞生前做的相对log4j的大优化框架，性能比log4j高很多，与log4j2差不多


门面型日志框架:  为了能在大项目中，合并各log实现框架的冲突问题，开发出的桥接类型的日志框架，里面无具体日志append实现，只有接口。即面向接口编程的框架，目前有两个：

SLF4J：用的比较多的桥接日志框架，桥类比较多，也是比较头疼的一个,全称Simple Logging Facade for Java。官网 http://www.slf4j.org/
J.C.L：在slf4j诞生前，apache开发的桥类，以前叫 (Jakarta Commons Logging)，也叫 Apache Common logging，现在叫 (commons-logging)
Log4j-api: Log4j 2.x 后，将log4j的接口独立的包

## 二、Log框架的演化

要明白log框架目前的混乱问题，我们得先理一下其发展过程，整明白各日志框架间的关系

![](/images/2021-08-01/image2021-3-10_18_52_25.png)

### 2.1 蛮荒时代：System.out和System.err

在java 1.4之前，java自己没有自带的log实现，所有的日志统计都是以System.out和System.err进行输出的。麻烦，不可配置，不灵活。

此时间应该是在jdk1.4发布之前的情况，时间上应该是2002年之前

### 2.2 最初统一：log4j 的诞生

在1996年初，E.U.SEMPER（欧洲安全电子市场）项目决定编写自己的跟踪API，最后该API演变为Log4j，Log4j日志软件包一经推出就备受欢迎，这里有一个主要贡献者：Ceki Gülcü，记住这个人，一切才刚刚开始。。。

后来Log4j成为了Apache基金会项目中的一员，同时Log4j的火爆，让Log4j一度成为业内日志标杆。（据说Apache基金会还曾经建议Sun引入Log4j到java的标准库中，但是sun拒绝了）

此日志一直在大规模使用，目前也还有好多框架是用的这个，虽然Log4j项目已经在2015年 End Of Life了。。但是，挡不住一直有人用哇。。。

![](/images/2021-08-01/image2021-3-10_18_27_25.png) 

### 2.3 Java跟风：JUL（java.util.logging）发布

Sun没有直接将log4j放入jdk组件中，而是自己仿照log4j 实现了一套自己的日志库，即java.util.logging。。。

从2002年2月发布jdk1.4开始，java中就自带有自己的log框架了。

真的是仿照（或许叫抄袭？），毕竟log4j流行那么久了。但是使用起来，还是没有log4j好用。

这就尴尬了，自带的不行，但是起码有人在用。

### 2.4 混乱初阶：JCL （Jakarta Commons Logging）发布

Sun自带了jul日志，之前的log4j还归属于apache，这里就有问题了，用户开发程序引入不同的日志类型时，就容易冲突或者导致打日志混乱的问题。所以apache推出了第一个接口类型日志框架：JCL。

当然初始目的是好的，让用户可以在Log4j和JUL日志框架间进行自由切换。这里面还有个坑，JCL自带了个默认实现，就是Simple Log ，这个日志实现导致实际上有了三种日志实现框架。

看第一版发布日期，应该是apache想和Sun对干或者想统一日志江湖。与Sun JUL发布相差6个月。

这个框架应该是没有Ceki 参与，
S
![](/images/2021-08-01/image2021-3-10_18_15_39.png) 

实际使用中，虽然JCL能实现日志框架的统一，比较优雅，但是使用中遇到的问题还是很多。

有个吐槽，说明一切
![](/images/2021-08-01/image2021-3-10_18_24_3.png) 

jcl 推出后，log的统一可以如下：
![](/images/2021-08-01/image2021-3-10_18_24_3.png) 

### 2.5 争霸开始：Slf4j 接口框架和桥的诞生

大神来了

2006年，Log4j的作者Ceki Gülcü离开Apache后，觉得J.C.L这套接口设计的不好，容易让开发者写出有性能问题的代码。

也是仿照J.C.L的接口类，自己搞出了一套接口框架，就是Slf4j。

但是Slf4j只有接口，没有实现，大神也没有能力去推动apache log4j 和sun log 去实现Slf4j的接口，这里就体现出大神的牛逼了，自己写各适配器。。。

适配器可解决一切，桥接是万能的。。哈哈哈。。。这里就开始出现混乱的东西了：

大神先写了桥接jul和log4j的包，结构如下
---

title: spring boot 指南
date: 2018-08-22
categories: spring boot
author: Phillip Webb, Dave Syer, Josh Long, Stéphane Nicoll, Rob Winch, Andy Wilkinson, Marcel Overdijk, Christian Dupuis, Sébastien Deleuze, Michael Simons, Vedran Pavić, Jay Bryant
description: spring boot 参考指南，翻译自官网

---


[官网原版点这里](https://docs.spring.io/spring-boot/docs/2.0.0.M7/reference/htmlsingle/#howto-initialize-a-spring-batch-database)

2.0.0.M7

Copyright © 2012-2017

Copies of this document may be made for your own use and for distribution to others, provided that you do not charge any fee for such copies and further provided that each copy contains this Copyright Notice, whether distributed in print or electronically.



目录
1. Spring Boot文档
   1. 关于本文档
   2. 获取帮助
   3. 第一步
   4. 使用Spring Boot开始工作
   5. 学习关于Spring Boot的功能
   6. Spring Boot项目上线
   7. 高级应用
2. 正式开始
   8. Spring Boot简介
   9. 

### 第一部分 Spring Boot文档

这部分是关于Spring Boot 参考文档的简单概要，相当于整个文档的内容地图，描述了文档的各个组成部分。

1. 关于本文档

`Spring Boot参考指南`有三种文档格式，内容是一致的，但是格式不一样，不同的人按自己的阅读习惯选择哪一种文档格式。

* [HTML](https://docs.spring.io/spring-boot/docs/2.0.0.M7/reference/html/)
* [PDF](https://docs.spring.io/spring-boot/docs/2.0.0.M7/reference/pdf/spring-boot-reference.pdf)
* [EPUB](https://docs.spring.io/spring-boot/docs/2.0.0.M7/reference/epub/spring-boot-reference.epub)

随着Spring Boot相关的技术不断的演进，文档经常被更新，点击这里查看当前最新的版本：[docs.spring.io/spring-boot/docs/current/reference.](https://docs.spring.io/spring-boot/docs/current/reference/)

你可以复拷本文档自己使用，或者分发给其他人使用，无论是电子版或者打印版，在传播时，每一份拷贝都需要包括原文的 Copyright Notice,同时不能把本文档作为收费的依据。

2. 获得帮助

如果你在实践Spring Boot时遇到困难，希望你能通过以下途径获得帮助
* 尝试到这里看看 [How-to doucments](https://docs.spring.io/spring-boot/docs/2.0.0.M7/reference/htmlsingle/#howto) . 这里提供了大多数问题的解决方法。
* 学习Spring的基础知识，Spring Boot是基于很多其他的Spring 家族的其他Spring Projects来构建的。查看[Spring.io](https://spring.io/)官网首页来查看大量丰富的参考文档。如果你已经开始使用spring，遇到问题，尝试到这里的[实践指南](https://spring.io/guides)找找看有没有你所需要的。
* 到stackoverflow去提问，Spring官方会监控[stackoverflow.com](https://stackoverflow.com/)上tag为`spring-boot`的问题。
* 在这里报告你发现的bugs [ github.com/spring-projects/spring-boot/issues.](https://github.com/spring-projects/spring-boot/issues)

*所有的Spring Boot相关的技术都是开源的，包括文档，如果你在这些文档中发现有问题或者你想改进提高这些文档，请[get involved](https://github.com/spring-projects/spring-boot/tree/v2.0.0.M7)*

3. 第一步

如果你要遵从一般的步骤开始学习`Spring Boot` 、 `Spring`,从下面几个方面入手开始学习。

* 从动手开始：概要、配置要求、安装
* 指南:第一部分、第二部分
* 从例子开始:例子1、例子2

4. `Spring Boot`的工作环境

* 构建系统：Maven、Gradle、Ant、Starters
* 最佳实践:Code Structure、@Configuration、@EnableAutoConfiguration、Beans and Dependency Injection
* 代码开始环境:IDE、Packaged、Maven、Gradle
* 应用打包:上线Jars包
* `Spring Boot`命令行接口(CLI):使用CLI

5. `Spring Boot`主要功能

* 核心功能:SpringApplication、External Configuration 、 Profiles 、 Logging
* Web应用:MVC 、 内嵌容器
* 持久层：SQL 、 NO-SQL
* 消息处理: 概述 、 JMS
* 测试: 概述 、 `Boot Applications` 、 实用工具
* 扩展: Auto-configuration 、 @Conditions

6. 上线 

当你的`Spring Boot`应用要上线时，有一些技术或技巧你可能会用到：

* Management endpoints:概述 、 客制化
* 连接选项: HTTP 、 JMX
* 监控:Metrucs 、 Auditing 、 Tracing 、 Process

7. 高级应用

最后，这些有些高级应用是给技术能力强的高级用户进一步探索

*  `Spring Boot` 应用部署：云部署 、 OS Service
*  构建工具插件: Maven 、 Gradle
*  附录: Application Properties 、 Auto-configuration 、 Executable Jars

### 第二部分，从这里开始
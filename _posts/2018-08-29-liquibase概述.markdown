---

layout: post
title: Liquibase概述
date: 2018-08-29
categories: liquibase 
description: Liquibase概述，列举了Liquibase的通常使用场景，翻译国外技术文章

---

[原文点这里](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)
[参考文章](https://github.com/liquibase/liquibase-hibernate/wiki)
[Liquibase官网](http://www.liquibase.org/)

## 概述

这是一篇简略的使用指南，我们使用`Liquibase`来持续开发演进java应用程序的数据库的结构。我首先来看下java应用是怎么整合使用`Liquibase`，接着是来看关于Spring,Hibernate与`Liquibase`的整合。

总的来说，`Liquibase`的核心是`changeLog`文件-一个xml文件，用来跟踪管理数据库的演化过程。

首先，我们需要把`liquibase-core`加入到pom.xml文件
```xml
<dependency>
    <groupId>org.liquibase</groupId>
     <artifactId>liquibase-core</artifactId>
      <version>3.4.1</version>
</dependency>
```

更新的版本，请点[这里](http://mvnrepository.com/artifact/org.liquibase/liquibase-core)查看

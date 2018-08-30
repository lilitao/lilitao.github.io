---

layout: post
title: Liquibase in practice
date: 2018-08-29
categories: liquibase 
description: Liquibase概述，列举了Liquibase的通常使用场景，翻译国外技术文章

---

[原文点这里](https://www.baeldung.com/liquibase-refactor-schema-of-java-app)

[liquibase-hibernate插件](https://github.com/liquibase/liquibase-hibernate/wiki)

[Liquibase官网](http://www.liquibase.org/)

1. 概述

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

> 更多关于Liquibase的功能简介
  * [Introduction to Liquibase Rollback](https://www.baeldung.com/liquibase-rollback)
  * [Database Migrations with Flyway](https://www.baeldung.com/database-migrations-with-flyway)
  * [Quick guide on Loading Inital Data With Spring Boot](https://www.baeldung.com/spring-boot-data-sql-and-schema-sql)

2. Change Log文件

我们先看个简单的 *changeLog* 文件--这个 `changeLog`文件只是简单在在table `user`里新增一个字段`address`

```xml
<databaseChangeLog 
  xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
  xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog-ext
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd 
   http://www.liquibase.org/xml/ns/dbchangelog 
   http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
     
    <changeSet author="John" id="someUniqueId">
        <addColumn tableName="users">
            <column name="address" type="varchar(255)" />
        </addColumn>
    </changeSet>
     
</databaseChangeLog>

```

> 注意思考liquibase是怎么通过`id`和`author`来惟一识别一个`change set`--识别`change set`是为了确保每一个`change set`只能被执行一次，否则，执行多次会报错或破坏db的结构。

让我们看下怎么把上面的`change log`加入到我们的应用里并确保它在应用起动时被执行

3. 使用Spring Bean配置并运行Liquibase

第一次执行方法，配置一个Spring Bean在Spring Boot应用启动时执行`change log`。当然还有其他方法，但是如果我们使用Spring开发我们的应用时，这是一种简单实用的方法。

```java
@Bean
public SpringLiquibase liquibase() {
    SpringLiquibase liquibase = new SpringLiquibase();
    liquibase.setChangeLog("classpath:liquibase-changeLog.xml");
    liquibase.setDataSource(dataSource());
    return liquibase;
}
```
> 注意：我们引用的`change log`文件`liquibase-changeLog.xml`要在运行前被加入到我们的class path里。

4. Spring Boot整合并运行Liquibase

如果你正在使用Spring Boot作为开发框架，只需要简单的配置，甚至不需要为Liquibase定义一个Bean

你只需要把你所有的`change log`文件都包含在这个文件内`src/resources/db/changelog/db.changelog-master.yaml`，那么在Spring Boot启动时，这些`change log`就行被应用到db。

* 首先你需要把`liquibase-code`依赖加入到你的pom.xml文件里
* 你可以通过`liquibase.change-log`这个Spring Boot配置项来修改默认的`change log `文件，如下：
```properties
liquibase.change-log=classpath:liquibase-changeLog.xml
```

5. 使用Maven plugin来生成`change log`文件




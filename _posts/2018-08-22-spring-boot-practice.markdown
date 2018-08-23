---

title: spring boot in practice
date: 2018-08-22
layout: post
categories: spring boot
description: demonstrate how to  build up a spring boot application
author: AndyLi

--- 

## Spring Boot实践

目录

* 建立项目结构
* application.properties
* logback配置
* ehcache配置
* Swagger2配置
* MessageResources配置
* liquibase
* JPA data access
* FeignClient
* actuator
* jacoco
* javadoc
* plantuml
* 发布到私有maven仓库
* spring test & Mockito


### 建立项目结构

一切从`spring-boot-starter-parent`开始,建立一个maven的pom项目，项目名称:`parent` ,这个项目只包括一个`pom.xml`文件，用于管理整个项目的maven dependency。

![parent project]({{ site.url }}/assets/images/spring-construct.PNG)

#### 关于pom.xml的内容

* pom文件的`packaging`类型必须是`pom`

```xml
...
<packaging>pom</packaging>
...
```
*  `spring-boot-starter-parent`,因为这是一个SpringBoot项目，所以项目必须继承`spring-boot-starter-parent`

```xml
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.1.RELEASE</version>
</parent>
```

* 在`dependencyManagement`里加入`spring-cloud-dependencies`,在这个项目中，我们使用的`FeignClient`是依赖 `spring-clound-dependencies`的

```xml
<properties>
...
<spring.cloud.version>Finchley.RELEASE</spring.cloud.version>
</properties>
...
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-dependencies</artifactId>
	<version>${spring.cloud.version}</version>
    <type>pom</type>
	<scope>import</scope>
</dependency>
...
```

* 我们有使用流的方式读取excel,在`dependencyManagement`中加入关于excel的dependency

```xml
...
<org.apache.poi.version>3.16</org.apache.poi.version>
<poi.streaming.version>1.2.0</poi.streaming.version>
...
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi</artifactId>
	<version>${org.apache.poi.version}</version>
</dependency>
<dependency>
	<groupId>org.apache.poi</groupId>
	<artifactId>poi-ooxml</artifactId>
	<version>${org.apache.poi.version}</version>
</dependency>

<dependency>
	<groupId>com.monitorjbl</groupId>
	<artifactId>xlsx-streamer</artifactId>
	<version>${poi.streaming.version}</version>
</dependency>
...
```

* 关于JPA使用的读取database的依赖，apache commons依赖,web appliation依赖加入到`dependencyManager`

```xml
...
<jdbc.sqlserver.version>5.0</jdbc.sqlserver.version>
<jdbc.mysql.version>6.0.6</jdbc.mysql.version>
<commons.lang3.version>3.2.1</commons.lang3.version>
<commons.collections.version>3.2.1</commons.collections.version>
<servlet.api.version>2.5</servlet.api.version>
...
<dependency>
	<groupId>com.microsoft.sqlserver</groupId>
	<artifactId>sqljdbc4</artifactId>
	<version>${jdbc.sqlserver.version}</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>${jdbc.mysql.version}</version>
</dependency>

<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-lang3</artifactId>
	<version>${commons.lang3.version}</version>
</dependency>
<dependency>
	<groupId>commons-collections</groupId>
	<artifactId>commons-collections</artifactId>
	<version>${commons.collections.version}</version>
</dependency>
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>servlet-api</artifactId>
	<version>${servlet.api.version}</version>
</dependency>

<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>${servlet.api.version}</version>
</dependency>
```

* 在`dependencies`里加入所有子项目都需要用到的依赖

项目用到`spring-test` , `mockito` , `junit` 来构建测试框架，加入以下相关依赖包

```xml
...
<mockito.version>1.10.19</mockito.version>
...
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-test</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>org.mockito</groupId>
    <artifactId>mockito-all</artifactId>
    <version>${mockito.version}</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>junit</groupId>
	<artifactId>junit</artifactId>
	<scope>test</scope>
</dependency>
```

* 使用`sonarQue`扫描代码缺陷，作为质量内建的工具之一,加入`sonarQue`的依赖

```xml
...
<sonar.maven.plugin.version>3.4.0.905</sonar.maven.plugin.version>
...
<dependency>
    <groupId>org.sonarsource.scanner.maven</groupId>
    <artifactId>sonar-maven-plugin</artifactId>
    <version>${sonar.maven.plugin.version}</version>
</dependency>
```
* 子项目作为jar包的方式提供组件服务，在`build` `plugins`加入 `maven-jar-plugin`,同时在jar包打包时包括源代码，加入`maven-source-plugin`

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-jar-plugin</artifactId>
</plugin>
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-source-plugin</artifactId>
	<executions>
     	<execution>
				<id>attach-sources</id>
				<goals>
					<goal>jar</goal>
				</goals>
		</execution>
	</executions>
</plugin>
```
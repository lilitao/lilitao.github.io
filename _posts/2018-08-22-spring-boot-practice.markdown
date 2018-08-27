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
* 统一异常处理
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

#### `Parent`项目

关于pom.xml的内容

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

* 使用`sonarQue`扫描代码缺陷，作为质量内建的工具之一,在`dependenies`里加入`sonarQue`的依赖

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

* 在`build` `plugins`加入`maven-surefire-plugin`，用来运行系统的test case,时同加入`jacoco-maven-plugin`，用来生成代码覆盖率统计数据

```xml
...
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<version>2.18.1</version>
	<configuration>
			<failIfNoTests>false</failIfNoTests>
			<argLine>${surefireArgLine}</argLine>
			<includes>
				<include>**/*Test.java</include>
				<include>**/Test*.java</include>
			</includes>
			<excludes>
				<exclude>**/*IT.java</exclude>
				<exclude>**/IT*.java</exclude>
				<exclude>**/*ITCase.java</exclude>
			</excludes>
			<skip>${maven.test.skip}</skip>
			<skipTests>${skipTests}</skipTests>
	</configuration>
	<executions>
			<execution>
				<id>integration-tests</id>
				<goals>
					<goal>test</goal>
				</goals>
            	<configuration>
					<skip>${skipITs}</skip>
					<includes>
						<include>**/*IT.java</include>
						<include>**/IT*.java</include>
						<include>**/*ITCase.java</include>
					</includes>
					<excludes>
						<exclude>**/*Test.java</exclude>
						<exclude>**/Test*.java</exclude>
					</excludes>
				</configuration>
			</execution>
	</executions>
</plugin>
<plugin>
	<groupId>org.jacoco</groupId>
	<artifactId>jacoco-maven-plugin</artifactId>
	<version>0.8.1</version>
	<configuration>
		<destFile>${project.build.directory}/coverage-reports/jacoco-ut.exec</destFile>
		<dataFile>${project.build.directory}/coverage-reports/jacoco-ut.exec</dataFile>
		<skip>${skipTests}</skip>
		<output>file</output>
		<append>true</append>
	</configuration>
	<executions>
		<execution>
			<id>pre-unit-test</id>
			<goals>
				<goal>prepare-agent</goal>
			</goals>
			<configuration>
				<propertyName>surefireArgLine</propertyName>
		    </configuration>
		</execution>
			<execution>
				<id>post-unit-test</id>
				<phase>test</phase>
				<goals>
		        	<goal>report</goal>
				</goals>
				<configuration>
					<outputDirectory>${project.reporting.outputDirectory}/jacoco-ut</outputDirectory>
				</configuration>
			</execution>
		</executions>
</plugin>
```

> `surefireArgLine`由`jacoco-maven-plugin`设置值，提供`maven-surefire-plugin`使用，使`jacoco`可以抓取test case执行时的统计数据


以上为parent项目的基本结构完成，但是一些其他应用的plugin还没有加入，比如`maven-javadoc-plugin`  等，我在用到这些插件再添加。
开始下一步之前，首先在`parent`项目里运行`mvn install`命令，下载相关依赖和插件。

#### `ERP`项目

新建 `ERP`项目 ,这是一个聚合项目`ERP`包括`CRM`子项目,项目结构如下：

![erp]({{ site.url }}/assets/images/spring-construct-erp.PNG "erp")

pom.xml文件内容

maven项目类型
```xml
<packaging>pom</packaging>
```

`ERP`继承`Parent`项目

```xml
<parent>
      <groupId>com.ay</groupId>
      <artifactId>Ay_Parent</artifactId>
      <version>1.0-SNAPSHOT</version>
</parent>
```

`erp`作为一个web application,为每个子项目引入`spring-boot-starter-web`,同时使用`logback`日志，引用`logback-classic`。
我们使用前后面后离的开发模式，后端应用引用swagger UI管理rest-API,引入`swagger-annotations`。
在`dependencies`加入以下依赖。
```xml
<dependency>
	<groupId>ch.qos.logback</groupId>
	<artifactId>logback-classic</artifactId>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
	<groupId>io.swagger</groupId>
	<artifactId>swagger-annotations</artifactId>
	<version>1.5.3</version>
</dependency>
```

* 默认情况下,引入`spring-boot-starter-web`后，`Spring Boot`还会为我们引入以下几个包
  > org.springframework.boot:spring-boot-starter
    |--org.springframework.boot:spring-boot
    |--org.springframework.boot:spring-boot-autoconfigure
    |--org.springframework.boot:spring-boot-starter-logging
    |--org.springframework.boot:spring-core
    |--org.yaml:snakeyaml

### 新建`ERP`子模块

按功能职责分成以下几个模块

* `Ay_Crm`核心业务逻辑,通过restfull-API对外提供服务
* `CRM_Client` 提供`FeignClient`接口
* `CRM_DataAccess` 提供持久层服务
* `CRM_Web`负责打包运行web application服务

#### `CRM_Web`模块




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
<surefireArgLine></surefireArgLine>
<maven.test.skip>false</maven.test.skip>
<skipTests>false</skipTests>
<skipITs>false</skipITs>
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
            <phase>compile</phase>
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

在`pluginManagement` `plugins`加入`liquibase-maven-plugin`用来管理database的持续开发与集成

```xml
...
<pluginManagement>
     <plugins>
          <plugin>
              <groupId>org.liquibase</groupId>
              <artifactId>liquibase-maven-plugin</artifactId>
          </plugin>
     </plugins>
....
</pluginManagement>
...
```

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
    * --org.springframework.boot:spring-boot
    * --org.springframework.boot:spring-boot-autoconfigure
    * --org.springframework.boot:spring-boot-starter-logging
    * --org.springframework.boot:spring-core
    * --org.yaml:snakeyaml

### 新建`ERP`子模块

![erp-module]({{ "/assets/images/spring-construct-erp-module.png" | absolute_url }})

按功能职责分成以下几个模块

* `Ay_CRM`核心CRM业务逻辑,通过restfull-API对外提供服务
* `ERP_Client` 提供`FeignClient`接口
* `ERP_DataAccess` 提供持久层服务
* `ERP_Web`负责打包运行web application服务

#### `ERP_DataAccess`模块

负责`ERP`项目的持久层服务,`DataAccess`使用 `Spring JAP`抽象关系统数据库的访问。首先作为`Spring Boot`项目，先引入依赖:`spring-boot-starter`,同时因为maven的传递依赖，同时也会引入`Spring Boot`的几个核心包：`spring-boot`,`spring-boot-autoconfugure`,`spring-boot-starter-loggin`,`spring-core`,`snakeyaml`
```xml
...
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter</artifactId>
 </dependency>
...
```

引入`spring-boot-starter-data-jpa`包与`sqljdbc4`包，基于sql server关系数据库开发
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>sqljdbc4</artifactId>
</dependency>
```
database unit test使用liquibase与h2内存数据库,引入相关的包
```xml
 <dependency>
      <groupId>org.liquibase</groupId>
      <artifactId>liquibase-core</artifactId>
      <scope>test</scope>
 </dependency>
 <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>test</scope>
 </dependency>
```
因为只在test阶段使用，所以`liuqibase-core`, `h2`的`scope`是test

引入`liquibase-maven-plugin`插件，管理database的持续集成与开发

```xml
 <properties>
        <liquibase.action>updateSQL</liquibase.action>
    </properties>
    <profiles>
        <profile>
            <id>updateSQL</id>
            <properties>
                <liquibase.action>updateSQL</liquibase.action>
            </properties>
        </profile>
        <profile>
            <id>update</id>
            <properties>
                <liquibase.action>update</liquibase.action>
            </properties>
        </profile>
    </profiles>
    <build>
        <plugins>
            <plugin>
                <groupId>org.liquibase</groupId>
                <artifactId>liquibase-maven-plugin</artifactId>
                <configuration>
                    <propertyFileWillOverride>true</propertyFileWillOverride>
                    <propertyFile>src/main/resources/liquibase.properties</propertyFile>
                    <promptOnNonLocalDatabase>false</promptOnNonLocalDatabase>
                </configuration>
                <executions>
                    <execution>
                        <phase>process-resources</phase>
                        <goals>
                            <goal>${liquibase.action}</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

```
运行`mvn package -P update`就可以将liuqibase的change set执行并应用到db；`mvn package -P updateSQL`就可以对比change set与db之间的差异并生成sql 脚本，以便检查并手动执行chnage set。

通过以上的配置，我们就可以使用liquibase管理和续持集成开发database。但在实际开过程中，我们会发现：当我们在java persistence object里定义了db的结构，再把这种定义用xml的形式翻译成liquibase的changelog,liquibase再把changelog应用到db。在java persistence object，xml,db之间，用三种不同的形式描述相同的db结构。我们期望只维护一种表达形式（java），其他两种表达形式可以由工具帮我们自动维护，xml与db之间由liquibase自动帮我们管理，但是java与xml之间到目前为止，还没有自动管理手段。liquibase提供了一个插件帮我们从java persistence object生成changelog:`liquibase-hibernate5`,我们修改`liquibase-maven-plugin`配置加入相关依赖，如下：

```xml
....
			<liquibase-hibernate5.version>3.6</liquibase-hibernate5.version>
        	<spring.jpa.version>2.0.6.RELEASE</spring.jpa.version>	
			<plugin>
                <groupId>org.liquibase</groupId>
                <artifactId>liquibase-maven-plugin</artifactId>
                <configuration>
                    <propertyFileWillOverride>true</propertyFileWillOverride>
                    <propertyFile>src/main/resources/liquibase.properties</propertyFile>
                    <promptOnNonLocalDatabase>false</promptOnNonLocalDatabase>
                </configuration>
                <executions>
                    <execution>
                        <phase>process-resources</phase>
                        <goals>
                            <goal>${liquibase.action}</goal>
                        </goals>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>org.liquibase.ext</groupId>
                        <artifactId>liquibase-hibernate5</artifactId>
                        <version>${liquibase-hibernate5.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring-beans</artifactId>
                        <version>${spring.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.springframework.data</groupId>
                        <artifactId>spring-data-jpa</artifactId>
                        <version>${spring.jpa.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring-core</artifactId>
                        <version>${spring.version}</version>
                    </dependency>
                    <dependency>
                        <groupId>org.springframework</groupId>
                        <artifactId>spring-context</artifactId>
                        <version>${spring.version}</version>
                    </dependency>
                </dependencies>
            </plugin>
```

在`liquibase.properties`配置文件加入配置：

```properties
referenceUrl: hibernate:spring:com.ay.erp.dao.persistence?dialect=org.hibernate.dialect.SQLServerDialect
diffChangeLogFile: src/main/resources/liquibase-diff-changeLog.xml
```

运行maven命令`mvn liquibase:diff`，即可以`liquibase-diff-changeLog.xml`文件中生成关于`spring:com.ay.erp.dao.persistence`包下的java persistence object与db之间的差异了，最后打开文件`liquibase-diff-changeLog.xml`作出必要的修改，整合成我们需要的changelog格式。

java persistence object的格式大概如下：

```java
@Entity()
@Table(name = "ayErpCustomer"
,indexes = {
        @Index(name = "pk_ayErpCustomer_id",columnList = "id asc",unique = true),
        @Index(name = "index_ayErpCustomer_id_name_birthday",columnList = "id desc,name desc,birthday desc",unique = false)
}
,uniqueConstraints = {
        @UniqueConstraint(columnNames = {"id"},name ="constraint_ayErpCustomer_id")
}
)
@Setter
@Getter
public class ErpCustomerPo extends BasePo {

    @TableGenerator(name = "ayErpCustomerGenerator", table = TABLE_NAME, pkColumnName = PK_COLUMN_NAME, valueColumnName = VALUE_COLUMN_NAME, initialValue = 0, allocationSize = 100)
    @GeneratedValue(generator = "ayErpCustomerGenerator",strategy = GenerationType.TABLE)
    @Id
    @Column(name = "id",columnDefinition = DaoConstant.BIGINT,nullable = false)
    private Long id;

    @Column(name = "name", columnDefinition = DaoConstant.VARCHAR, length = 100, nullable = false)
    private String name;

    @Column(name = "sex",columnDefinition = DaoConstant.CHAR,length = 1,nullable = false)
    private String sex;

    @Column(name = "birthday",columnDefinition = TIMESTAMP,nullable = true)
    private Date birthday;

    @Column(name = "personalPhone",columnDefinition = VARCHAR,nullable = true,length = 20)
    private String personalPhone;

    @Column(name = "busniessPhone",columnDefinition = VARCHAR,nullable = true,length = 20)
    private String busniessPhone;

    @Column(name = "officeAddress",columnDefinition = VARCHAR,nullable = true,length = 150)
    private String officeAddress;

    @Column(name = "homeAddress",columnDefinition = VARCHAR,nullable = true,length = 150)
    private String homeAddress;
}
```

* DB unit test

DB unit test是基于Spring Boot使用liquibase 和h2内存数据库构建不依赖外部的测试环境，首先在`test->java`包下建`Spring Boot`的启动类仅用于DB unit test

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.ay.erp.dao",useDefaultFilters = false,
        includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Repository.class)
        })
@EntityScan(basePackages = "com.ay.erp.dao")
public class DbUnitConfig {
}
```

![DbUnitConfig]({{ "assets/images/spring-dbUnitConfig.png" | absolute_url }})

DB unit test需要依赖`DbUnitConfig`的配置启动

添加Spring Boot和liquibase的启动配置文件`test/java/resources/application.yml`,如上图所示,内容如下,

```properties
logging:
  level:
    org.springframework: DEBUG
    com.ay: DEBUG
    liquibase: DEBUG
    org.h2: DEBUG

spring:
  liquibase:
    change-log: classpath:db.changeog.erp.test.master.xml
    contexts: UT
    enabled: true
    drop-first: false
  jpa:
    generate-ddl: false
    show-sql: true
    hibernate:
      naming:
#      This is needed, otherwise hibernate will change the table name in SQL. Such as: change TblAddress to tbl_address
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

建立liquibase changelog配置文件`db.changeog.erp.test.master.xml`,这个文件直接引用`main/resources/db.changelog.erp.master.xml`文件，使已经配置好的changelog文件集合

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
    <include file="db.changelog.erp.master.xml"></include>
</databaseChangeLog>
```
配置完成，开始第一个test case
```java
@RunWith(SpringRunner.class)
@DataJpaTest
@Transactional
public class CustomerPoRepositoryTest {

    @Autowired
    CustomerPoRepository repository;

    @Test
    public void should_save_and_find_success() {
        //given
        ErpCustomerPo po = createPersistenceObject();

        //when
        repository.save(po);
        Optional<ErpCustomerPo> result = repository.findById(po.getId());
        //then
        Assertions.assertThat(repository.count()).isEqualTo(1);
        Assertions.assertThat(result.get().getId()).isEqualTo(po.getId());
    }

    @Test
    public void should_delete_success() {
        //given
        ErpCustomerPo po = createPersistenceObject();
        repository.save(po);
        //when
        repository.delete(po);
        //then
        Assertions.assertThat(repository.count()).isEqualTo(0);
    }

    private ErpCustomerPo createPersistenceObject() {
        ErpCustomerPo po = new ErpCustomerPo();
        po.setBirthday(new Date());
        po.setBusniessPhone("123");
        po.setHomeAddress("11");
        po.setName("AndyLi");
        po.setOfficeAddress("chinase");
        po.setSex("M");
        po.setRecordStatus("A");
        return po;
    }
}
```

#### `ERP_Web`模块

`Web`模块负责配置打包运行`ERP`应用

* application.propertes配置文件

在maven的`resources`目录下新建`application.properties`配置文件。

关于`application.properties`的详细配置项信息，[请点这里查看](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

![properties]({{ "assets/immages/spring-properties.png" | absolute_url }})

完成Spring Boot配置文件后，新建Spring Boot启动类`@SpringBootApplication`

```java
@SpringBootApplication(scanBasePackages = "com.ay")
@EnableCaching
@EnableTransactionManagement
@Slf4j
public class ErpApp {
    public static void main( String[] args ){
        SpringApplication.run(ErpApp.class, args);
    }
    /**
     *  create a CommandLineRunner to initiate application during application context starting
     * @param context spring application context
     * @return CommandLineRunner using to take action of initiation
     */
    @Bean
    public CommandLineRunner commandLineRunner(ApplicationContext context) {
        return args -> {
            log.info("inspect the beans provided by Spring Boot:");

            String[] beanNames = context.getBeanDefinitionNames();
            Arrays.sort(beanNames);
            for (String beanName : beanNames) {
                log.info(beanName);
            }

        };
    }
}

```

使用`SwaggerUI`管理restful API

加`Swagger`相关依赖包

```xml
...
		<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>

        <dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.6-jre</version>
        </dependency>
...
``` 

新增`Swagger`关于Spring Boot的配置

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
	
	@Bean
	public Docket createDocket() {
		return new Docket(DocumentationType.SWAGGER_2)
				.apiInfo(createApiInfo())
				.select()
				//select()method return back  ApiSelectorBuilder instance to control which API should be shown using Swagger , in this configuration we will scan the the class underlyying specifical package
				//，Swagger will scan all the controller under package:com.aiatss.coast to create document except for method annotated @ApiIgnore
				.apis(RequestHandlerSelectors.basePackage("com.ay"))
				.paths(PathSelectors.any())
				.build();
	}

	private ApiInfo createApiInfo() {
		return  new ApiInfoBuilder()
				.title("restful API")
				//.description("")
				.termsOfServiceUrl("http://google.com")
				//.contact("COAST NFR")
				.version("0.0.1-SNAPSHOT")
				.build();
	}
}
```

通过Spring Boot的启动类启动`ErpApp`,并在浏览器输入地址：http://localhost:8080/swagger-ui.html 即可访问swagger ui主界面


#### Ay_Aggregate 项目

`Ay_Aggreate`项目的作用与`Ay_Parent`类似，是`<packaging>pom</packaging>`类型项目，不负责具体的功能代码。主要作为项目的聚合根，聚合所有的项目模块，负责一些跟聚合相关的操作和配置。

例如下面要介绍的关于生成所有项目的测试覆盖率报告。我们通过`jacoco-maven-plugin`生成所有项目的测试报告，但是最早期时`jacoco-maven-plugin`只能在单个项目的target目录下生成单个项目的测试报告，但是随着复杂的多模块项目的出现，为了方面管理，我们需要把所有模块或者子项目的测试覆盖数据都聚合在同一份报告里生成，所以`jacoc-maven-plugin`插件又新增了一个`gola`：`report-aggregate` 用来把所有子项目（子模块）测试数据聚合在同一份报告里。`jacoco-maven-plugin`插件使用的测试数据是依赖插件:`maven-surefire-plugin`生成的,所以`report-aggregate`需要依赖每个单独项目下，由`jacoco-maven-plugin`集成`maven-surefire-plugin`生成的测试数据文件：`jacoco.exec`。所以关于这个数据文件的路径和名称的配置，如果不太清楚的话，就使用默认配置，如果`report-aggregate`没有找到对应的`exec`文件，生成的测试数据都错误的。

 > 关于`jacoco-maven-plugin`中`report-aggregate`的官方文档，[请查看这里](https://www.eclemma.org/jacoco/trunk/doc/report-aggregate-mojo.html)
 

 > 关于`report-aggregate`在多模块项目中的演化过程，[请看这里](https://github.com/jacoco/jacoco/wiki/MavenMultiModule)

`Ay_Aggregate`项目结构如下：
![aggregate]({{ "assets/images/spring-boot-ay-aggregate.png" | absolute_url }})


`Ay_Aggreate`通过继承`Ay_Parent`获得`Ay_Parent`的所有特性，同时把所有项目配置为`Ay_Aggregate`的子模块

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ay</groupId>
    <artifactId>Ay_Aggregate</artifactId>
    <packaging>pom</packaging>

    <parent>
        <groupId>com.ay</groupId>
        <artifactId>Ay_Parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>Ay_CRM</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>ERP_DataAccess</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>ERP_Client</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <modules>
        <module>../Ay_ERP</module>
        <module>../Ay_Security</module>
    </modules>

    <build>
        <plugins>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>report-aggregate</id>
                        <phase>verify</phase>
                        <goals>
                            <goal>report-aggregate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

</project>
```

在上面的代码中，需要注意的是：
1. `packaging`类型为`pom`
2. 通过以下配置继承`Ay_Parent`项目

```xml
<parent>
    <groupId>com.ay</groupId>
    <artifactId>Ay_Parent</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>
```
3. 配置测试覆盖率报告生成插件,

```xml
<plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <executions>
            <execution>
                  <id>report-aggregate</id>
                  <phase>verify</phase>
                   <goals>
                       <goal>report-aggregate</goal>
                   </goals>
            </execution>
       </executions>
</plugin>
```

4. 只配置了`jacoco-maven-plugin`,暂时是不能生效生成覆盖率报告，还需要添加以下配置
  * 添加项目到将要生成的覆盖率报告中

```xml
<dependencies>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>Ay_CRM</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>ERP_DataAccess</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <dependency>
            <groupId>com.ay</groupId>
            <artifactId>ERP_Client</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

```

  * 把需要生成覆盖率的项目作为子模块引入

```xml
  <modules>
        <module>../Ay_ERP</module>
        <module>../Ay_Security</module>
    </modules>
```

5. 上面提到`report-aggregate`需要集成`maven-surefire-plugin`来给每个单独子模块（子项目)生成测试数据:`jacoco.exec` 。所在，在`Ay_Parent`配置好`maven-surefire-plugin`和`jacoco-maven-plugin`确保每个单独的项目可以正确生成测试数据文件

```xml
 <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.18.1</version>
                <configuration>
                    <failIfNoTests>false</failIfNoTests>
                    <argLine>@{surefireArgLine}</argLine>
                    <skip>${maven.test.skip}</skip>
                    <skipTests>${skipTests}</skipTests>
                </configuration>
                <executions>
                    <execution>
                        <id>unit-tests</id>
                        <phase>test</phase>
                        <goals>
                            <goal>test</goal>
                        </goals>
                       
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.jacoco</groupId>
                <artifactId>jacoco-maven-plugin</artifactId>
                <version>0.8.1</version>
                <configuration>
                    <skip>${skipTests}</skip>
                    <output>file</output>
                    <append>true</append>
                </configuration>
                <executions>
                    <execution>
                        <id>pre-unit-test</id>
                        <phase>initialize</phase>
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

6. 生成的测试覆盖率大概如下：

![coverage-coverage]({{"assets/images/spring-boot-coverage-report-1.png" | absolute_url}})
![coverage-coverage]({{"assets/images/spring-boot-coverage-report-2.png" | absolute_url}})

---

layout: post
title: 软件质量内建
date: 2018-10-23
categories: software quality
description: 介绍软件质量内建的工具与手段，提高我们软件开发的质量，降低软件开发过程中的复杂度。
author: Andy Li

---

软件质量，是我们软件开发的核心，我们大部分的开发活动(软件架构的设计)，管理活动都是围绕着软件质量展开的。

在大规模软件开发过程中，我们希望我们的软件最终是低耦合，高内聚的。我们需要引入一些成熟的工作、手段、最佳开发实践来辅助我们提高软件开发的质量，降低软件开发过程中的复杂度。

在这篇文章里，我将介绍我们在实际开发过程用到的最佳实践、开发工具。参考我们的实际项目代码:

https://cangzpwsvn01.aia.biz/svn/Group_Pension/Project_Coast/branches/COAST_BE_DDD_POC

## 业务建模与分析
在项目开始，使用DDD辅助我们设计系统架构，使用`plantUml`这个工具帮我理清思路.

1. 系统分层模型：
  * 表示层(front end)
  * 应用层
  * 领域层
  * 基础架构层

在以上的分层模式下，再对代码类型进一步分类，即：我们在软件中的每一行代码都应该归属以下分类，把每一行代码都分别分到相关对应的包中。

* form : 用于不同层次之前传递数据，比如，我们使用form来接收由front end传递过来的数据；
* view : 用于传递业务处理的返回数据，这些数据往往是业务处理结果需要返回到表现层用以展示，或者模块对外接口调用的结果数据； 
* application controller : 专门用来处理front end发送过来的业务处理请求，并路由到对应的业务处理器上。
* application service:为了支撑领域业务的技术实现
* domain entity : 处理核心问题领域业务，在业务领域中业务操作的对象，这些对象保持，维护着业务中产品或者服务的状态，在业务操作中需要区别这些对象的身份，有完整的生命周期定义。
* domain value object: 在业务处理当中的一些对象，业务处理过程中不需要关心这些对象之间的区别，只需要关心这些对象的值，当成一个值使用。
* domain service : 在业务处理过程中，有一些方法或者服务很难与任何一领域对象建立明确的关系，但是这些服务或者行为却又会操作领域对象，为了不产生耦合度过度的领域对象，我们把这些服务或者行为添加服务对象中。
* domain repository : 提供统一的实体对象的持久化服，持久化到文件系统或者数据库
* domain factory:负责领域对象的创建工作，与domain repository关联，因为创建工作可能会从repository里取相关的领域对象
* domain event : 业务事件
* domain listener: 监听业务事件，实现业务逻辑，解耦复杂的业务逻辑
* data access persistence object : 持续久化对象
* data access repository : domain repository的技术实现，与具体数据库实现相关


2. 使用DDD的思想设计业务系统,在这个过程中使用plantUml来理清思路
*  plantUml是一个面象开发人员的，建模工具；特点是使用过程式的代码来构建业务图形，比如以下这段过程式的代码

```java
@startuml member-module-domain-diagram.svg

 together {
 class Client << (E, YellowGreen) >>
 class Policy << (E, YellowGreen) >>
 }

 ...

 Member "1" *-- "*" Dependent
 Individual <|-- Member
 Individual <|-- Dependent
 Individual "1" *-- "*" Email
 Individual "1" *-- "*" Phone
 Individual "1"  *-- "*" Address
 Individual "1" *-- "*" PartyAccount
 PartyAccount <|-- BankAccount
 PartyAccount <|-- CreditCard
 Member "1" *-- "1" Membership
 Dependent "1" *-- "1" Membership

 Policy "n" --* "1" Client
 Member "n" --* "1" Client
 @enduml

```

通上以上的代码我们生成以下这幅建模图形：

![mfu]( "./mfu-domain.png" )

关于PlantUml与maven、开发环境(IntelliJ)的整合，请参考我们DDD COAST POC的示例代码和官方说明文档: http://plantuml.com/

## TDD开发模式

当我们完成对业务的建模后，我们分解业务场景，使用TDD的开发模式(TDD工发模式的应用与实践请参考TDD相关的课程与文档)。在TDD的开发模式下我们会产生大量的unit test,我们使用`maven-surefire-plugin`和`jacoco-maven-plugin`来生成测试覆盖报告来监控我们unit test并辅助我检查业务场景的测试情况。我们生成的测试报告如下所示：

![]("./coast-unit-test-coverage.png")

以上的图展示了我们整个项目的unit test的情况,我们点击进去，可以细看每个类的生产代码的覆盖情况，如下图：

![]("coast-unit-test-coverage-class.PNG")

如上图所示，绿色部分生产代码显示已经被测试覆盖；红色部分显示这部分代码还没有测试覆盖，需要分析是否正常（注意，不是所有的生产代码都应该被测试覆盖）。

## 数据库演化模式

数据库结构是业务逻辑在持久化层的表现方式。在理想的状态下，开发人员期望在开始写业务代码之前，数据库的结构已经被SA预先设计师完成确定并固化下来，开发人员期望在确定并正确的数据库结构下按需求保存或查询数据。但是实际的项目开发经常告诉我们，这种理想的状态实际上并不存在，需求的变化，复杂的业务使得预先设计变得十分脆弱，需求的变化常常会带来数据库结构的变化，复杂的业务使得预先设计会出估计不足的情况。在实际开发过程中，往往是我们首先产生一分关于数据库的初步设计，但是这种设计并不是百分百确定不变，在开发过程中不断按实际情况演化数据库结构。在这个过程中，我们引入工具`Liquibase`帮助我们简化复杂度，管理演化过程，消除演化过程中的重复。

`Liquibase`是一个跨平台的数据库持续开发，集成工具。`Liquibase`帮我们解决开发过程中的以下问题：
1. 跨平台数据库结构开发；
2. 数据库开发过程中的版本控制；
3. 提供一致的数据库结构规范开发规范与流程，例如所有开发人员都按`Liquibase`提供的规范开发数据库，不需要人工编写sql执行，或者直接使用数据库工具变更数据库结构；
4. 配合CI服务器检查开发过程中的错误，比如开发人员是否按规范编写数据库脚本直接修改旧版本的脚本；
5. 配合数据库unit test，检查数当前提交的数据库开发脚本是否会影响到已有的功通模块；
6. 生成数据库上线脚本;
7. 对比两个数据库之间的差异；

为了降件持久化工作复杂度，解耦持久化代码，我们同时引入Spring Data JPA数据库中间件负责持久化操作。

`Liquibase`的核心是使用`changeLog`来管理数据库开发工作，以下演示一个标准的`changeLog`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
        xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
         http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
    <changeSet author="liquibase-docs" id="createTable-com.aiatss.coast.pmm.pm.infrastructure.persistence.po.TblTeam">
        <createTable tableName="TblTeam">
            <column name="teamId" type="java.sql.Types.BIGINT">
                <constraints nullable="false"/>
            </column>
            <column name="TeamCode" type="java.sql.Types.CHAR(100)"/>
            <column name="TeamName" type="java.sql.Types.CHAR(100)"/>
            <column name="RecordStatus" type="java.sql.Types.CHAR(1)"/>
            <column name="VersionNo" type="java.sql.Types.BIGINT"/>
            <column name="RecordChangedBy" type="java.sql.Types.VARCHAR(20)"/>
            <column name="RecordChangedTime" type="java.sql.Types.TIMESTAMP"/>
        </createTable>
    </changeSet>
    <changeSet author="liquibase-docs" id="createTablePk-com.aiatss.coast.pmm.pm.infrastructure.persistence.po.TblTeam">
        <addPrimaryKey constraintName="PK_TblTeam" columnNames="teamId" tableName="TblTeam" />
    </changeSet>
    <changeSet author="liquibase-docs" id="createTableIndex-com.aiatss.coast.pmm.pm.infrastructure.persistence.po.TblTeam">
        <createIndex indexName="ak_TblTeam" tableName="TblTeam" unique="true">
            <column name="TeamId"/>
            <column name="TeamCode"/>
        </createIndex>
    </changeSet>
</databaseChangeLog>
```
在`changeLog`文件里以`changeSet`为单元开发数据库，以上面的代码片段为例，包括三个`chnageSet`：
1. 第一个`changeSet`是使用`createTable`标签创建表；
2. 第二个`changeSet`使用`addPrimaryKey`标签创建表主键；
3. 第三个`changeSet`使用`createIndex`标签创建索引； 

Spring Data JPA的核心是使用`Entity`来完成数据库与java对象之间的映射。以下是一个标准的`Entity`配置：

```java
@Entity()
@Table(
        name = "TblCountry",
        indexes = {
                @Index(name = "ak_TblCountry", columnList = "CountryCode ASC", unique = true)
        }
)
public class CountryPo  extends BasePo {
    private Long CountryId;
    private Long RegionId;
    private Long CurrencyId;
    private String CountryCode;
    private String CountryName;

    @TableGenerator(name = "TblCountry",
            table = TABLE_NAME,
            pkColumnName = PK_COLUMN_NAME,
            valueColumnName = VALUE_COLUMN_NAME,
            initialValue = 1,
            allocationSize = 100)
    @GeneratedValue(strategy = TABLE, generator = "TblCountry")
    @Id
    @Column(name = "CountryId", nullable = false, columnDefinition = BIGINT)
    public Long getCountryId() { return CountryId; }
    @Column(name = "RegionId", nullable = false, columnDefinition = BIGINT)
    public Long getRegionId() { return RegionId;  }
    @Column(name = "CurrencyId", nullable = false, columnDefinition = BIGINT)
    public Long getCurrencyId() { return CurrencyId;   }
    @Column(name = "CountryCode", length = 3, nullable = false, columnDefinition = CHAR)
    public String getCountryCode() {  return CountryCode;  }
    @Column(name = "CountryName", length = 200, nullable = false, columnDefinition = NVARCHAR)
    public String getCountryName() {  return CountryName;  }
}
```

以上的`CountryPo` entity以java的方式完成地表达了一个表`TblCountry`在数据库的结构。

引入以上的关于数据库的开发模式后，我们引入了一个新的问题：在开发过程中，我们发现对于数据库的实现，我们在三个不同的环境下用了三种不同的方式完全地表达了同样的数据库结构；分别是:
* Spring Data JPA 的Entity
* Liquibase 的changeLog
* 数据库的对象（包括具体数据库内的表结构，视图，函数等其他数据库对象）

重复是开发过程中代码的坏味道，重构的一个重要目标就是消除重复，所以我们引入新的工具消除这三种环境下的重复。我们期望引入的工具使我们在开发过程中只维护三种环境下Spring Data JPA的entity，使用工具帮我们维护其他两种环境的数据库结构
1. 我们使用一个自开发的maven工具`COAST_BuildTool`扫描项目中所有Entity并生成对应的`changeLog`
2. `liquibase-maven-plugin`使用这个插件帮我们对比Entity与数据库之间的差异，并生成`changeLog`

`liquibase`还提供了其他的丰富的插件帮我们完成数据库持续集成开发工作。我们使用entity来驱动我们的数据库开发，我们期望从entity可以了解到我们数据库的总体结构来了解我们数据库设计是否健康，我们使用`COAST_BuildTool`来生成数据库结构的E-R图：

![]("er.PNG")

## Continuous Integeration

我们使用`Jenkins`来帮助我们完成持续开发集成，持续交付的工作。在开发阶段`Jenkins`帮我们完成以下三件事:
1. 每次提交代码后自动构建项目，并执行unit test检查本次提交是否引入新的问题；
2. 每次提交代码后自动构建数据库结构，数据库结构的演化是否有引入新的风险
3. 自动触发sonarQube检查静态代码质量。

![]("jenkins-ddd-poc.PNG")

### 监控项目状态
使用`Jenkins`插件实时监控项目状态：`Build Monitor View` ： https://plugins.jenkins.io/build-monitor-plugin
![]("jenkins-monitor-view.PNG")

每当开发人员提交代码时，`Jenkins`都会自动触发软件质量检查相关的任务，通过上面的插件实时监控，如果出现了代码质量问题，我们应该在10到15分钟内修复完成，否则应该回滚我们的代码，使其他的开发人员可以继续工作，回滚代码解决后再提交，保证开发工作持续不断的集成。

### SonarQube
使用`SonarQube`检查静态代码；示例: http://cangzdlcoa02:9002/
1. 关于`SonarQube`在mavne setting.xml的配置
```xml

		<profile>
			<id>sonar_poc</id>
			<properties>
				<sonar.jdbc.url>jdbc:sqlserver://cangzdwats01:1433;databaseName=sonarPOC</sonar.jdbc.url>
				<sonar.jdbc.driver>com.microsoft.sqlserver.jdbc.SQLServerDriver</sonar.jdbc.driver>
				<sonar.jdbc.username>COASTSONARQUBE</sonar.jdbc.username>
				<sonar.jdbc.password>Aiait12345</sonar.jdbc.password>
				<sonar.host.url>http://CANGZDLCOA02:9002</sonar.host.url>
			</properties>
		</profile>
```
2. 项目pom文件引入`SonarQube`
3. 配置`Jenkins`扫描任务
```bat
cd ../POC_BE_UT/sources
set SONAR_SCAN=-Psonar_poc org.jacoco:jacoco-maven-plugin:prepare-agent -Dmaven.test.failure.ignore=false sonar:sonar -Dsonar.host.url=http://cangzdlcoa02:9002 -Dsonar.scm.disabled=True 
call mvn %SONAR_SCAN%
```
以上这段脚本是启动扫描静态代码
```bat
cd ../POC_BE_UT/sources
set SONAR_Coverage=-Psonar_poc sonar:sonar -Dsonar.jacoco.reportPath=target/jacoco.exec -Dsonar.jacoco.reportMissing.force.zero=true
call mvn %SONAR_Coverage%
```
这一段脚本是关于在`SonarQube`集成 `Jacoco`扫描代码的unit test coverage

![](sonarQube-quality.PNG)

### 数据展示Grafana GuideLine
为了方便监控更多项目的健康指示，我们引入一个新的工具：数据展示Grafana GuideLine : http://cangzdlcoa01:3000 ；为以展示其他项目指标，比如以下指标：
* BUGS
* Function Complexity
* Duplicate Line
* Coverage
* Technical Debt


![](granfana.PNG)

## API自动化测试

## UI自动化测试




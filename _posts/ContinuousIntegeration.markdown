---

layout: post
title: Continuous Integration
date: 2018-11-06
categories: CI Server
description: Continuous Integration Server的常用配置 

---

## Jenkins

示例代码请参考虑`DDD POC COAST POC`项目

定时触发svn构建，因为svn没有设定勾子的功能，只能是jenkins定时查询是否有代码提交

![](jenkins-task-triiger.PNG)

使用用jenkins打包测试，并生成javadoc

```bat
cd ./sources/COAST_Aggregate
set PATH=D:\Software\apache-maven-3.5.2\bin;%PATH%
set GRAPHVIZ_DOT=D:\Software\graphviz-2.38\release\bin\dot.exe
mvn clean install site 

```
`GRAPHVIZ_DOT` 变量使用用graphvize组件在java doc中生成plantUml图片

生成PlantUml ER图并生成svg图下

```bat
cd ./sources/COAST_PMM/PMM_DataAccess  
set PATH=D:\Software\apache-maven-3.5.2\bin;%PATH%
set GRAPHVIZ_DOT=D:\Software\graphviz-2.38\release\bin\dot.exe
mvn javadoc:javadoc@gen-er-diagram plantuml:generate
```
注意：这种语法`javadoc:javadoc@gen-er-diagram`，需要在maven3.5以上版本才支持



使用jenkins作为一个静态web服务器展示文档，作为持续集成过程中生成的文档的展示服务器。在Jenkins安装目录下/home/userContent目录作为jenkins web服务器的根目录，在这个目录下的所有文件都通过jenkins的URL浏览： http://ip:9527/jenkins/userContent/。所以当我们在自动化构建过程中生成javadoc,coveragereport,er图等文档，可以通过bat或者sh命令copy到userContent目录下，然后通过jenkins服务器展示，在自动化构建完成后，加入以下类似脚本

```bat
cd ../../userContent
::rmdir /s/q COAST
md COAST
md COAST\apidocs
md COAST\coverageReport
md COAST\ER
xcopy ..\workspace\POC_BE_UT\sources\COAST_Aggregate\target\site\apidocs  COAST\apidocs /s /e /h /d /y
xcopy  ..\workspace\POC_BE_UT\sources\COAST_Aggregate\target\site\jacoco-aggregate  COAST\coverageReport /s /e /h /d /y
xcopy ..\workspace\POC_BE_UT\sources\COAST_PMM\PMM_DataAccess\target\site\er  COAST\ER /s /e /h /d /y
```

因为我们的jenkins服务器安装在windows下面，所以以上这段脚本是bat命令，主要作用是将在项目中生成的javadoc，coverage report,er图复制到userContent目录下,然后我们可以通过jenkins服务器直接浏览:http://cangzdwats01:9527/jenkins/userContent/


jenkins执行SonarQube扫描

```bat
cd ../POC_BE_UT/sources

set SONAR_SCAN=-Psonar_poc org.jacoco:jacoco-maven-plugin:prepare-agent -Dmaven.test.failure.ignore=false sonar:sonar -Dsonar.host.url=http://cangzdlcoa02:9002 -Dsonar.scm.disabled=True 

call mvn %SONAR_SCAN%
```

jenkins集成jacoco生成测试coverager report

```bat
cd ../POC_BE_UT/sources

set SONAR_Coverage=-Psonar_poc sonar:sonar -Dsonar.jacoco.reportPath=target/jacoco.exec -Dsonar.jacoco.reportMissing.force.zero=true


call mvn %SONAR_Coverage%
```

使用Liquibase持续集成database开发

```bat
cd ../POC_BE_UT/sources/COAST_PMM/PMM_DataAccess

mvn liquibase:dropAll  liquibase:update
```

使用jenkins执行API test

```bat
cd ../POC_BE_UT/sources/COAST_PMM/PMM_Web

mvn install -Dspring.boot.start=false -Dspring.boot.stop=false -Dapi.test=false
```

把api test report复制到usercontent目录

```bat 

cd ../../userContent
rmdir /s/q COAST\APITest
md COAST\APITest
xcopy ..\workspace\POC_BE_UT\sources\COAST_ApiTest\target\robotframework-reports   COAST\APITest /s /e /h /d /y

```

## 使用pipeline 改造jenkins任务

```script
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout'
                checkout()
            }
        }        
        stage('Build') {
            steps {
                echo 'Building'
                bat '''cd ./sources/ 
                        set PATH=D:/Software/apache-maven-3.5.2/bin;%PATH%  
                        set GRAPHVIZ_DOT=D:/Software/graphviz-2.38/release/bin/dot.exe  
                        mvn clean install site'''
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
               
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying'
                
            }
        }
    }
}


```


Granfana 配置

配置Function complexity

```sql

 select $__time(createTime), project_name AS meas_name,countVal AS meas_value from 
(
SELECT dbo.[projects].[name] as project_name,
	  [project_measures].[id]
	  ,[metrics].name as metricsName
      ,[value]
      ,[metric_id] 
      ,[text_value]
	  ,[dbo].[f_count]([text_value],';') as countVal,
	  dbo.[snapshots].[created_at]/1000 as createTime
	  --into #tbl_andy_temp
  FROM [dbo].[project_measures]
  JOIN dbo.[projects] on dbo.[project_measures].[component_uuid] = dbo.[projects].[uuid]
  JOIN dbo.[metrics] on dbo.[project_measures].[metric_id] = dbo.[metrics].[id]
  Join dbo.[snapshots] on  dbo.[snapshots].[uuid] = dbo.[project_measures].[analysis_uuid]
  
 where dbo.[metrics].name ='function_complexity_distribution'
 and dbo.[projects].[name] in ('COASTBP_PMM','COASTBP_NFR','COASTBP_OverAllCoverage')
 
 )t 

 order by project_name,createTime 


```

Duplicate line 

```sql

	SELECT
		 dbo.[projects].[name]                 AS meas_name,
		 $__time(dbo.[snapshots].[created_at] / 1000 )   ,
		--from_unixtime(floor((dbo.[snapshots].[created_at] / 1000))) AS created_at,
		cast(dbo.[project_measures].[value] as decimal(17,2))         AS meas_value
		-- dbo.[metrics].[name]                  AS meas_name
     
	   FROM dbo.[snapshots] 
	   JOIN dbo.[project_measures]  on dbo.[snapshots].[uuid] = dbo.[project_measures].[analysis_uuid]
	   JOIN dbo.[projects] on dbo.[project_measures].[component_uuid] = dbo.[projects].[uuid]
	   JOIN dbo.[metrics] on dbo.[project_measures].[metric_id] = dbo.[metrics].[id]
	   WHERE dbo.[metrics].[name] ='duplicated_lines_density'
	   AND dbo.[projects].[name] in  ('COASTBP_PMM','COASTBP_NFR','COASTBP_OverAllCoverage')
	   ORDER BY dbo.[projects].[name],  dbo.[snapshots].[created_at]

```

coverage 

```sql 
SELECT
		 dbo.[projects].[name]                 AS meas_name,
		$__time(dbo.[snapshots].[created_at] / 1000) ,  -- AS time,
		--from_unixtime(floor((dbo.[snapshots].[created_at] / 1000))) AS created_at,
		 dbo.[project_measures].[value]       AS meas_value
	   FROM dbo.[snapshots] 
	   JOIN dbo.[project_measures]  on dbo.[snapshots].[uuid] = dbo.[project_measures].[analysis_uuid]
	   JOIN dbo.[projects] on dbo.[project_measures].[component_uuid] = dbo.[projects].[uuid]
	   JOIN dbo.[metrics] on dbo.[project_measures].[metric_id] = dbo.[metrics].[id]
	   WHERE dbo.[metrics].[name] ='coverage'
	   AND dbo.[projects].[name] in  ('COASTBP_PMM','COASTBP_NFR','COASTBP_OverAllCoverage')
	   ORDER BY dbo.[projects].[name],  dbo.[snapshots].[created_at]

```


BUGS

```sql 
	SELECT
		 dbo.[projects].[name]                 AS meas_name,
		$__time(dbo.[snapshots].[created_at] / 1000) ,  -- AS time,
		--from_unixtime(floor((dbo.[snapshots].[created_at] / 1000))) AS created_at,
		cast(dbo.[project_measures].[value] as decimal(17,2))         AS meas_value
		-- dbo.[metrics].[name]                  AS meas_name
     
	   FROM dbo.[snapshots] 
	   JOIN dbo.[project_measures]  on dbo.[snapshots].[uuid] = dbo.[project_measures].[analysis_uuid]
	   JOIN dbo.[projects] on dbo.[project_measures].[component_uuid] = dbo.[projects].[uuid]
	   JOIN dbo.[metrics] on dbo.[project_measures].[metric_id] = dbo.[metrics].[id]
	   WHERE dbo.[metrics].[name] ='bugs'
	   AND dbo.[projects].[name] in  ('COASTBP_PMM','COASTBP_NFR','COASTBP_OverAllCoverage')
	   ORDER BY dbo.[projects].[name], dbo.[snapshots].[created_at]

```


Technical debt

```sql 
	   	SELECT
		 dbo.[projects].[name]                 AS meas_name,
		 $__time(dbo.[snapshots].[created_at] / 1000) ,  -- AS time,
		--from_unixtime(floor((dbo.[snapshots].[created_at] / 1000))) AS created_at,
		cast( (dbo.[project_measures].[value]/480) as int)        AS meas_value
		-- dbo.[metrics].[name]                  AS meas_name
			
	   FROM dbo.[snapshots] 
	   JOIN dbo.[project_measures]  on dbo.[snapshots].[uuid] = dbo.[project_measures].[analysis_uuid]
	   JOIN dbo.[projects] on dbo.[project_measures].[component_uuid] = dbo.[projects].[project_uuid]
	   JOIN dbo.[metrics] on dbo.[project_measures].[metric_id] = dbo.[metrics].[id]
	   WHERE dbo.[metrics].[name] ='sqale_index'
	   AND dbo.[projects].[name] in  ('COASTBP_PMM','COASTBP_NFR')
	   ORDER BY dbo.[projects].[name],  dbo.[snapshots].[created_at]

```
---

layout: post
title: Continuous Integration
date: 2018-11-06
categories: CI Server
description: Continuous Integration Server的常用配置 

---

## Jenkins
使用jenkins作为一个静态web服务器展示文档，作为持续集成过程中生成的文档的展示服务器。在Jenkins安装目录下/home/userContent目录作为jenkins web服务器的根目录，在这个目录下的所有文件都通过jenkins的URL浏览： http://ip:9527/jenkins/userContent/。所以当我们在自动化构建过程中生成javadoc,coveragereport,er图等文档，可以通过bat或者sh命令copy到userContent目录下，然后通过jenkins服务器展示，在自动化构建完成后，加入以下类似脚本

```bat
cd ../../userContent
rmdir /s/q COAST
md COAST
md COAST\apidocs
md COAST\coverageReport
md COAST\ER
xcopy ..\workspace\POC_BE_UT\sources\COAST_Aggregate\target\site\apidocs  COAST\apidocs /s /e /h /d /y
xcopy  ..\workspace\POC_BE_UT\sources\COAST_Aggregate\target\site\jacoco-aggregate  COAST\coverageReport /s /e /h /d /y
xcopy ..\workspace\POC_BE_UT\sources\COAST_PMM\PMM_DataAccess\target\site\er  COAST\ER /s /e /h /d /y
```

因为我们的jenkins服务器安装在windows下面，所以以上这段脚本是bat命令，主要作用是将在项目中生成的javadoc，coverage report,er图复制到userContent目录下

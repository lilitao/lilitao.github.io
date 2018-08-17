---
layout: post
title: spring batch introduction
date: 2018-08-15
categories: spring batch
---

## table of content
* summary of meta-data table
* summary of spring batch architecture
* creation of parallel batch job
   * thread pool
* creation of synchronous batch job
* job status
   * job success
   * job fail
* scheduling task
* transaction 
* re-run strategy 
* distributed deployment


## summary of meta-data table

Meta-Data table design details refer to : [spring docs](https://docs.spring.io/spring-batch/3.0.x/reference/html/metaDataSchema.html)  [spring docs](https://docs.spring.io/spring-batch/trunk/reference/html/configureJob.html)

Use above link to find and digging into spring batch data module in database . In here , this article ,we will only summarize something must be aware for developer . 
1. The initial DDL scripts placed in the packages : org.springfreamework.batch.core of spring-batch-core.xx.jar
2. The spring batch configuration file reside in spring-batch-core.xx.jar named as : batch-xx.properties . you can change the configuration optional according to this file optional 
3. optimistic locking stategy
4. table structure below

![meta-data-erd.png]({{ site.url }}/assets/images/meta-data-erd.png)

the primary table are : batch_job_instance , batch_job_execution , batch_setp_excution make up the overall hierarchy of spring batch data module . 

### summary of architecture

refer to : [spring batch docs](http://www.baeldung.com/introduction-to-spring-batch)
Use above link for find and digging into design architecture of spring batch , in this sections , will walk you through the summary of spring batch .

### creation of parallel batch job 

[Go and find the details of parallel job](https://docs.spring.io/spring-batch/trunk/reference/html/scalability.html#multithreadedStep)

 
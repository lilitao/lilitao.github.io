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
* example with spring boot


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

### example with spring boot

This example demonstrate how to upload data into database from excel .

#### First , configure spring to enable batch processing 

```java

@Configuration
@EnableBatchProcessing
@EnableScheduling
public class BatchConfiguration 

```
1. `@EnableBatchProcessing` annotation on class to enable batch job
2. `@EnableScheduling` enable our use spring schedule component to schedule our batch job , beside launch batch job by web application online

In addition , we should also enable spring transaction management for batch job transaction.

```java

@SpringBootApplication(scanBasePackages = {"com.a**s.c**t"})
@EnableCaching
@EnableTransactionManagement
public class StarterWithSpringBoot {

```
Now , let to configure `JobLauncher` `TaskExeuctor` `JobRepository` `Scheduled` in `BatchConfiguration` class for global configuration. 

Using `JobLauncher` to launcher our batch job

```java
@Bean
public JobLauncher jobLauncher(JobRepository jobRepository,TaskExecutor taskExecutor) {
    SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
    jobLauncher.setTaskExecutor(taskExecutor);
    jobLauncher.setJobRepository(jobRepository);
    return jobLauncher;
}
```

1. `TaskExecutor` we using `ThreadPoolTaskExecutor` implements of  `TaskExecutor` for parallelism batch job executions

```java
@Bean
public TaskExecutor taskExecutor() {
    ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
   taskExecutor.setCorePoolSize(10);
   taskExecutor.setMaxPoolSize(50);
   taskExecutor.setThreadNamePrefix("***-business");
   taskExecutor.initialize();
   return taskExecutor;
}
```   

2. `JobRepository` database repository for batch persisence

```java
 @Bean
public JobRepository jobRepository(DataSource dataSource, PlatformTransactionManager transactionManager) throws Exception {
    JobRepositoryFactoryBean factoryBean  = new JobRepositoryFactoryBean();
    factoryBean.setDataSource(dataSource);
    factoryBean.setTransactionManager(transactionManager);

    JobRepository jobRepository =  factoryBean.getObject();
    return  jobRepository;
}
```

We can use spring scheduler component to schedule batch job , for example,

```java 
@Scheduled(cron = "*/5 * * * * ?")
public void medicalCardDayEndTask() {
    // logger.info("*******************spring batch scheduler running ***************");
}

```

Throughout About step ,global configuration complete , follow by job configuration


#### second , job configuration

Create a component `JobConfiguration` for job configuration

```java
@Component
public class UploadFileJob 
```

create a job in `JobConfiguration` 

```java
@Autowired
private JobBuilderFactory jobBuilderFactory;
@Autowired
private JobCompletionNotificationListener jobCompletionNotificationListener;

 public void createUploadFileJob(String path) throws JobParametersInvalidException, JobExecutionAlreadyRunningException, JobRestartException, JobInstanceAlreadyCompleteException {
        Job job = jobBuilderFactory.get("UploadJob")
                .incrementer(new RunIdIncrementer())
                .listener(jobCompletionNotificationListener)
                .flow(uploadStep(path))
                .next(validationStep())
                .end()
                .build();

        runJob(job);
 }
```
1. `JobBuilderFactory` privoded by Spring Batch , be used to build a job
2. `JobCompletionNotificationListener` a implements of `JobExecutionListenerSupport` , listen to job exeuction events , like completed , failed and so on.

```java
@Component
public class JobCompletionNotificationListener extends JobExecutionListenerSupport {

    @Override
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            JobParameters jobParameters = jobExecution.getJobParameters();
            Date time = jobParameters.getDate("time");
            logger.info("************ time:take time:{}MS;start time:{}********", System.currentTimeMillis() - time.getTime(),time );
        } else if (jobExecution.getStatus() == BatchStatus.FAILED) {

        }
    }

    @Override
    public void beforeJob(JobExecution jobExecution) {
        super.beforeJob(jobExecution);
    }
}

```

3. `uploadStep(path)` a method to create a step to upload data into database from excel .

```java
@Autowired
private StreamerItemReaderBuilder streamerItemReaderBuilder ;
@Autowired
private FileUploadProcessor FileUploadProcessor;
@Autowired
FileUploadWriter<FileUploadPo> writer;
@Autowired
ExcelDataSupplementListener excelDataSupplementListener;
@Autowired
FileUploadValidatorListener FileUploadValidatorListener;
@Autowired
ValidationErrorEventListener errorEventListener;
private Step fileUploadStep(String path) {
        return stepBuilderFactory.get("uploadFileStep1")
                .<FileUploadExcel, FileUploadPo> chunk(1000)
                .reader(streamerItemReaderBuilder.reader(path))
                .processor(FileUploadProcessor)
                .writer(writer)
                .listener(excelDataSupplementListener)
                .listener(FileUploadValidatorListener)
                .listener(errorEventListener)
                .build();
}
```

* `FileUploadExcel` java bean used by `ItemReader` for extract data from excel , one field in java bean take responsibility for one column in excel.
* `FileUploadPo` a persistence object used by repository to store data into database
* `StreamerItemReaderBuilder` using to build a `ItemReader`

```java
   @Component
   public class StreamerItemReaderBuilder {
    
    public StreamerItemReader<FileUploadExcel> build(String path) {
        StreamerItemReader<FileUploadExcel> reader = new StreamerItemReader<FileUploadExcel>();
        reader.setResource(new FileSystemResource(path));
        reader.setLinesToSkip(2);
        reader.setMaxNumberOfSheets(0);

        reader.setRowMapper(rowMapper());

        DefaultRowSetFactory rowSetFactory = new DefaultRowSetFactory();
        RowNumberColumnNameExtractor columnNameExtractor = new RowNumberColumnNameExtractor();
        columnNameExtractor.setRowNumberOfColumnNames(1);

        rowSetFactory.setColumnNameExtractor(columnNameExtractor);
        reader.setRowSetFactory(rowSetFactory);

        return reader;
    }

    private RowMapper<FileUploadExcel> rowMapper() {
        BeanWrapperRowMapper<FileUploadExcel> rowMapper = new BeanWrapperRowMapper<FileUploadExcel>();
        rowMapper.setDistanceLimit(1);
        rowMapper.setStrict(false);
        rowMapper.setTargetType(FileUploadExcel.class);

        ColumnGroup products = new ColumnGroup("products");
        policyProducts.addColumnName("pl**o", "cover*ode", "status", "propo*A", "ne*mt");
        rowMapper.addColumnGroup(products);
        return rowMapper;
    }
}
```

* `FileUploadProcessor` using for data transform  from `FileUploadExcel` to `FileUploadPo` for persistence

```java
@Component
public class FileUploadProcessor implements ItemProcessor<FileUploadExcel,FileUploadPo> {
    @Override
    public FileUploadPo process(FileUploadExcel fileUploadExcel) throws Exception {
        FileUploadPo po = new FileUploadPo();
        IBeanUtils.copyProperties(fileUploadExcel,po);
        return po;
    }
}
```

* `FileUploadWriter<FileUploadPo>` a imp be using for data persistence

```java

``` 
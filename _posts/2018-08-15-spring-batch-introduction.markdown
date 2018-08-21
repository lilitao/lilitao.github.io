---
layout: post
title: spring batch in practice
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
Use this link for find and digging into design architecture of spring batch , in this sections , will walk you through the summary of spring batch .

The following diagram for overall hierarchy 

![spring-batch-reference-model]({{ site.url }}/assets/images/spring-batch-reference-model.png)

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
2. `@EnableScheduling` enable  spring schedule component to schedule our batch job , beside launch batch job by web application online

beside above configuration , in `application.properties` , batch job global configuration are needed

```properties
spring.batch.table-prefix=BATCH_
spring.batch.initialize-schema=always
spring.batch.job.enabled=false
```

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
                .next(dataSqlValidationStep())
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

* `writer` is a instance of `FileUploadWriter<FileUploadPo>` that is  implments of `ItemWriter` be using for data persistence

```java
@Component
public class FileUploadWriter<T> implements ItemWriter<FileUploadPo> {
    @Autowired
    FileUploadDataPoRepository repository;
    @Override
    public void write(List<? extends FileUploadPo > items) throws Exception {
        logger.info("upload writer:{}",items);
        long time = System.currentTimeMillis();
        repository.saveAll(items);        
        logger.info("using time:{}",System.currentTimeMillis() - time);
    }
}

``` 

* `excelDataSupplementListener`  is a instance of job event listener be used to supplement data after read before process.

```java
@Component
public class ExcelDataSupplementListener {
    static final SimpleDateFormat  DATE_FORMAT =new SimpleDateFormat("MM-dd-yyyy");
    @AfterRead
    public void supplement(FileUploadExcel excel) {
        if (excel.getDate() != null) {
            Date date = new Date();
            date.setTime(Long.valueOf(excel.getDate()));
            excel.setDate(DATE_FORMAT.format(date));
        }
    }

}

```

* `FileUploadValidatorListener` is a instance of job event listener be used to validate data before process

```java
@Component
public class FileUploadValidatorListener {
    @Autowired
    ApplicationContext applicationContext;

    @BeforeProcess
    public void verify(FileUploadExcel excelBean) {
        doVerify(new ClientValidator(excelBean));

        doVerify(new DateValidator(excelBean));
    }

    private void doVerify(Validator validator) {
        if (!validator.verify())
            handleErrorMessage(validator);
    }
    private void handleErrorMessage(Validator validator) {
        ValidationErrorMessageEvent event = new ValidationErrorMessageEvent(this, validator.getErrorMessage());
        applicationContext.publishEvent(event);
    }
}
```

* `applicationContext` used to publish a validation error event when validation error
* `ValidationErrorMessageEvent` a validation error even implemented  `ApplicationEvent` within spring

```java
public class ValidationErrorMessageEvent extends ApplicationEvent {
    List<ValidationMessageOfFileUpload> errorMessage = new LinkedList<>();

    public List<ValidationMessageOfFileUpload> getErrorMessage() {
        return errorMessage;
    }
    /**
     * Create a new ApplicationEvent.
     *
     * @param source the object on which the event initially occurred (never {@code null})
     */
    public ValidationErrorMessageEvent(Object source,List<ValidationMessageOfFileUpload> errorMessage) {
        super(source);
        this.errorMessage = errorMessage;
    }
}
``` 

* `ValidationMessageOfFileUpload` a java bean for erorr message
* `ValidationErrorEventListener` this event listener will listen to `ValidationErrorMessageEvent` aways 

```java
@Component
@Slf4j
public class ValidationErrorEventListener {

    List<ValidationMessageOfFileUpload> errorMessages = new LinkedList<>();

    @Autowired
    FileUploadErrorMessagePoRepository repository;

    @EventListener(classes = ValidationErrorMessageEvent.class)
    public void handleErrorMessage(MfuValidationErrorMessageEvent event) {
        errorMessages.addAll(event.getErrorMessage());
        if (errorMessages.size() >= 1000) {
            log.info("handle error message,the number of error message is :{}", errorMessages.size());
            persist(errorMessages);
            errorMessages.clear();
        }
    }

    private void persist(List<ValidationMessageOfFileUpload> errorMessages) {
        List<FileUploadErrorMessagePo> pos = errorMessages.stream().map(error -> createPo(error)).collect(Collectors.toList());
        repository.saveAll(pos);
    }

    private FileUploadErrorMessagePo createPo(ValidationMessageOfFileUpload error) {
        FileUploadErrorMessagePo po = new FileUploadErrorMessagePo();
        po.setRowNumber(error.getRowNumber());
        po.setMessage(error.getMessage());
        //TODO other property should be supplemented
        return po;
    }

    @AfterStep
    public void handleErrorMessageLastBatch(StepExecution stepExecution) {
        if (errorMessages.size() > 0) {
            log.info("handle  file upload error message last batch ,the number of last batch is {}:", errorMessages.size());
            persist(errorMessages);
            errorMessages.clear();
        }
    }
}

```

* `@AfterStep` on method `handleErrorMessageLastBatch` to listen to the job event , so that system can handle last batch error message to persist .

4. `dataSqlValidationStep()` create validaton setp

```java
   @Autowired
   SqlValidatorReaderBuilder sqlValidatorReaderBuilder;

   @Autowired
   SqlValidatorProcessor sqlValidatorProcessor;

   @Autowired
   SqlValidatorWriter sqlValidatorWriter;

   private Step dataSqlValidationStep() {
        return stepBuilderFactory.get("dataSqlValidationStep")
                .<SqlValidator,SqlValidator>chunk(1)
                .reader(sqlValidatorReaderBuilder.build())
                .processor(sqlValidatorProcessor)
                .writer(sqlValidatorWriter)
                .build();
   }
``` 

* `sqlValidatorReaderBuilder` build a `ItemReader` to read item like this

```java
@Component
public class SqlValidatorReaderBuilder {

    @Autowired
    ApplicationContext context;

    public ListItemReader<SqlValidator> build() {
        Map<String, SqlValidator> validators  = context.getBeansOfType(SqlValidator.class);
        ListItemReader<SqlValidator> reader = new ListItemReader<>(validators.values().stream().collect(Collectors.toList()));
        return reader;
    }
}

```

* `sqlValidatorProcessor` a validation processor like below,

```java
@Component
public class SqlValidatorProcessor implements ItemProcessor<SqlValidator, SqlValidator> {
    @Autowired
    ApplicationContext context;
    @Override
    public SqlValidator process(SqlValidator item) throws Exception {
        if (item != null) {
            item.verify();
        }
        return item;
    }
}
```

* `sqlValidatorWriter` a writer to persist data

```java
@Component
public class SqlValidatorWriter implements ItemWriter<SqlValidator> {
    @Autowired
    UploadDataPoRepository repository;
    @Override
    public void write(List<? extends SqlValidator> items) throws Exception {

    }
}
```

5. spring JPA batch insert configuration

Configure batch insert in `application.properties`

```properties
hibernate.jdbc.batch_size=1000
hibernate.order_inserts=true
hibernate.order_updates=true
hibernate.jdbc.batch_versioned_data=true
spring.jpa.hibernate.jdbc.batch_size=1000
spring.jpa.hibernate.order_inserts=true
spring.jpa.hibernate.order_updates=true
spring.jpa.hibernate.jdbc.batch_versioned_data=true
spring.jpa.properties.hibernate.jdbc.batch_size=1000
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
spring.jpa.properties.hibernate.jdbc.batch_versioned_data=true

```


Throughout above step by step , we build up a spring batch example. and walked you through the process of batch job
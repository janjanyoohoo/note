# 定时任务

\#定时任务/Quartz#

## 入门案例

所有的定时任务都需要实现Job接口重写方法

引入spring-boot-starter-quartz依赖

```java
// 从调度器工厂获得一个默认的调度器
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
// 启动
scheduler.start(); //从线程池中获取线程执行
//业务代码 ... 参考Quratz官网
//Job详情
JobDetail job = JobBuilder.newJob(实现了Job接口的任务类.class)
            .usingJobData("key","value");//传递值
            .whithIdentity("name","group")//当前JobDetail的名称与分组;
            .build();//构建JobDetail 
//触发器
Trigger trigger = TriggerBuilder.newTrigger()
            .withIdentity("name","group")
            .usingJobData("key","value")//传参
            .startNow()
            //调度策略
            .withSchedule(CronScheduleBuilder.cronSchedule("* * * * * ? *"))
            .build();

scheduler.schedulerJob(job,trigger);   //执行任务 触发器
// 停止
scheduler.shutdown();//因为存在线程池, 不调用此方法 程序不会停止



//---->>>>在任务类中可以获取参数,通过方法参数context获取;
Trigger trigger = content.getTrigger();
JobDetail job = context.getJobDetail();
trigger.getJobDataMap("key");
job.getJobDataMap("key");
//通用获取方法 key重复时Trigger优先级更高 
context.getMergedJobDataMap("key");
//通过定义全局变量,指定set方法, 变量名与key的名称相同,Quartz在创建job时会自动注入
```

**Quartz中所有的定时任务都必须实现Job接口,重写excute()方法**

**一个Job可以被多个Trigger调度,一对多. 一个触发器只能调度一个任务**

- JobDetail对象 = JobBuilder.newJob(parm ).withIdentity(name,group).build() -> parm Job实现类.class //任务
- Trigger对象 = TriggerBuilder.newTrigger().withIdentity(name,group).startNow().withSchedule(SimpleScheduleBuilder.simplerSchedule()....) -> //触发器
- SimpleScheduleBuilder.simplerSchedule().withIntervalSeconds(5).repeatForever() //调度策略 5秒一次 永远执行

JobDetail与Trigger的name,group

 name: 用来标记(不指定将指定一个随机数的值)

 group:用来标记与方便管理(可以不指定,会指定为默认值DEFAULT_GROUP_)

![image-20210417164213864](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210417164213.png)

### 任务详情JobDetail

用JobBuilder.newJob()构建Job为JobDetail

### 触发器Trigger

通过TriggerBuilder.newTrigger()构建; 继承关系 Trigger -> CoreTrigger 下有四个触发器实现类

- CalendarIntervalTriggerImpl
- **SimpleTriggerImpl**
- DaliyTimerIntervalTriggerImpl
- **CornTriggerImpl**
  - 支持Cron表达式, 主要使用

### 调度策略

- **CornScheduleBuilder.cronCronSchedule()**

### 传参

传参:通过Builder的JobDetail或Trigger是调用useJobData(key,value)设置参数;

获取参数:通过重写的execute方法参数context中获取 JobDetail或Trigger对象,调用方法getJobData(k)获取; Trigger的优先级更高;

在Job类中定义全局变量, 变量名与key名相同时,给定set方法,Quartz在构建Job时会自动注入值;

## Quartz参数配置

参考配置文档[Quartz文档](quartz/configuration.adoc at master · quartz-scheduler/quartz · GitHub)

不指定配置文件时,使用默认的Quartz配置文件在Jar包中;

指定Quartz配置文件只需**在Resource下新建quartz.properties文件**根据指定选项进行配置即可;

## 集成SpringBoot

 [spring.io简介](https://docs.spring.io/spring-boot/docs/2.2.13.RELEASE/reference/html/spring-boot-features.html#boot-features-quartz)

```
Spring Boot offers several conveniences for working with the Quartz scheduler, including the spring-boot-starter-quartz “Starter”. If Quartz is available, a Scheduler is auto-configured (through the SchedulerFactoryBean abstraction).

Beans of the following types are automatically picked up and associated with the Scheduler:

JobDetail: defines a particular Job. JobDetail instances can be built with the JobBuilder API.

Calendar.

Trigger: defines when a particular job is triggered.
```

### 相关配置

在`application.yml`中配置quartz。相关配置的作用已经写在注释上。

```yaml
# spring的datasource等配置未贴出
spring:
  quartz:
      # 将任务等保存化到数据库
      job-store-type: jdbc
      # 程序结束时会等待quartz相关的内容结束
      wait-for-jobs-to-complete-on-shutdown: true
      # QuartzScheduler启动时更新己存在的Job,这样就不用每次修改targetObject后删除qrtz_job_details表对应记录
      overwrite-existing-jobs: true
      # 这里居然是个map，搞得智能提示都没有，佛了
      properties:
        org:
          quartz:
              # scheduler相关
            scheduler:
              # scheduler的实例名
              instanceName: scheduler
              instanceId: AUTO
            # 持久化相关
            jobStore:
              class: org.quartz.impl.jdbcjobstore.JobStoreTX
              driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
              # 表示数据库中相关表是QRTZ_开头的
              tablePrefix: QRTZ_
              useProperties: false
            # 线程池相关
            threadPool:
              class: org.quartz.simpl.SimpleThreadPool
              # 线程数
              threadCount: 10
              # 线程优先级
              threadPriority: 5
              threadsInheritContextClassLoaderOfInitializingThread: true
```

### Spring Bean方式

- 通过@Bean 将Trigger与JobDetai对象放入容器中,Scheduler进行组合即可.

此方式会产生大量bean,比较零散

### 原生代码方式

- 通过原生代码方式实现,编写JobDetail与Trigger, 传入Scheduler; 可以将JobDetail设置为持久化后通过Scheduler对象的addJob()方法添加; 调用scheduleJob时,Triigger指定Job的name与Group后可以不传入JobDetail对象.

### SpringBoot方式

- 引入Quratz的start后自动配置Scheduler注入到容器
- 暴露以下Bean会被SpringBoot自动拾取关联Scheduler
  - JobDetail
  - Trigger
  - Calendar
- SpringBoot提供了QuartzJobBean抽象类,该类继承Job重写了execute()方法获取SpringBoot上下文, 然后调用了executeInternal()方法

#### SpringBoot方式1

- 通过SpringBoot使用时,任务类需要继承QuartzJobBean类重写executeInternal()方法;
- 通过此方式重写的任务,可以在任务中使用Spring容器中的bean调用方法.
- 然后通过原生代码的方式实现定时任务

#### SpringBoot方式2

- 通过自动拾取的方式,对容器中放入Bean,
- 在JobDetail构建时指定withIdentity("xxx"),并指定为持久化storeDurably()
- 构建Trigger时指定forJob("xxx") 指定JobDetail的名字

### 新增任务代码示例

#### 周期性任务

```java
/**
 * Quartz的相关配置，注册JobDetail和Trigger
 * 注意JobDetail和Trigger是org.quartz包下的，不是spring包下的，不要导入错误
 */
@Configuration
public class QuartzConfig {
	/**
	 *像IOC中注册一个JobDetail 会被自动拾取关联
	 */
    @Bean
    public JobDetail jobDetail() {
        						//Job任务类
        JobDetail jobDetail = JobBuilder.newJob(StartOfDayJob.class)
            	//任务名 / 分组名
                .withIdentity("start_of_day", "start_of_day")
            	//持久化
                .storeDurably()
                .build();
        return jobDetail;
    }

    @Bean
    public Trigger trigger() {
        Trigger trigger = TriggerBuilder.newTrigger()
            	//关联Job
                .forJob(jobDetail())
            	//触发器名 / 分组名
                .withIdentity("start_of_day", "start_of_day")
            	// 立即执行(非必选)
                .startNow()
                // 每天0点执行, 使用Cron调度器
                .withSchedule(CronScheduleBuilder.cronSchedule("0 0 0 * * ?"))
                .build();
        return trigger;
    }
}

/**
 * Job实现类
 */
@Component
public class StartOfDayJob extends QuartzJobBean {
    private StudentService studentService;

    @Autowired
    public StartOfDayJob(StudentService studentService) {
        this.studentService = studentService;
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext)
            throws JobExecutionException {
        // 任务的具体逻辑
    }
}
```

#### 非周期性任务

```java
/**
 * 实体类
 */ 
public class LeaveApplication {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private Long proposerUsername;
    @JsonFormat( pattern = "yyyy-MM-dd HH:mm",timezone="GMT+8")
    private LocalDateTime startTime;
    @JsonFormat( pattern = "yyyy-MM-dd HH:mm",timezone="GMT+8")
    private LocalDateTime endTime;
    private String reason;
    private String state;
    private String disapprovedReason;
    private Long checkerUsername;
    private LocalDateTime checkTime;

    // 省略getter、setter
}

/**
 * 业务逻辑
 */ 
@Service
public class LeaveApplicationServiceImpl implements LeaveApplicationService {
    @Autowired
    private Scheduler scheduler;
    
    // 省略其他方法与其他依赖

    /**
     * 动态的添加job和trigger到scheduler
     */
    private void addJobAndTrigger(LeaveApplication leaveApplication) {
        Long proposerUsername = leaveApplication.getProposerUsername();
        // 创建请假开始Job
        LocalDateTime startTime = leaveApplication.getStartTime();
        JobDetail startJobDetail = JobBuilder.newJob(LeaveStartJob.class)
                // 指定任务组名和任务名
                .withIdentity(leaveApplication.getStartTime().toString(),
                        proposerUsername + "_start")
                // 添加一些参数，执行的时候用
                .usingJobData("username", proposerUsername)
                .usingJobData("time", startTime.toString())
                .build();
        // 创建请假开始任务的触发器
        // 创建cron表达式指定任务执行的时间，由于请假时间是确定的，所以年月日时分秒都是确定的，这也符合任务只执行一次的要求。
        String startCron = String.format("%d %d %d %d %d ? %d",
                startTime.getSecond(),
                startTime.getMinute(),
                startTime.getHour(),
                startTime.getDayOfMonth(),
                startTime.getMonth().getValue(),
                startTime.getYear());
        CronTrigger startCronTrigger = TriggerBuilder.newTrigger()
                // 指定触发器组名和触发器名
                .withIdentity(leaveApplication.getStartTime().toString(),
                        proposerUsername + "_start")
                .withSchedule(CronScheduleBuilder.cronSchedule(startCron))
                .build();

        // 将job和trigger添加到scheduler里
        try {
            scheduler.scheduleJob(startJobDetail, startCronTrigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new CustomizedException("添加请假任务失败");
        }
    }
}


/**
 * Job逻辑
 */ 
@Component
public class LeaveStartJob extends QuartzJobBean {
    private Scheduler scheduler;
    private SystemUserMapperPlus systemUserMapperPlus;

    @Autowired
    public LeaveStartJob(Scheduler scheduler,
                         SystemUserMapperPlus systemUserMapperPlus) {
        this.scheduler = scheduler;
        this.systemUserMapperPlus = systemUserMapperPlus;
    }

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext)
            throws JobExecutionException {
        Trigger trigger = jobExecutionContext.getTrigger();
        JobDetail jobDetail = jobExecutionContext.getJobDetail();
        JobDataMap jobDataMap = jobDetail.getJobDataMap();
        // 将添加任务的时候存进去的数据拿出来
        long username = jobDataMap.getLongValue("username");
        LocalDateTime time = LocalDateTime.parse(jobDataMap.getString("time"));

        // 编写任务的逻辑
		// ...
        
        // 执行之后删除任务
        try {
            // 暂停触发器的计时
            scheduler.pauseTrigger(trigger.getKey());
            // 移除触发器中的任务
            scheduler.unscheduleJob(trigger.getKey());
            // 删除任务
            scheduler.deleteJob(jobDetail.getKey());
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
    }
}

```

### 动态控制任务

Quartz中,新增任务是通过Scheduler对象进行关联Job与Trigger启动, 删除时逻辑也大致相同,通过Scheduler对象进行控制

首先需要注入`Scheduler`对象,此对象SringBoot会自动进行配置

> If Quartz is available, a `Scheduler` is auto-configured (through the `SchedulerFactoryBean` abstraction).

- pauseJob(jobKey)
  - 停止任务
- resumeJob(jobKey)
  - 启动任务
- getJobDetail(jobKey)
  - 获取jobDetail
- deleteJob(jobKey)
  - 删除任务
- scheduleJob(job,trigger)
  - 启动任务

```java
@Autowired
private Scheduler scheduler;
rm-uf606gal5e48n965l0o.mysql.rds.aliyuncs.com

```

#### 停止

```java
public String pause() throws Exception {
    //创建一个JobKey对象, (开启关闭都通过这个对象),构造方法中传入Job的名称
    JobKey key = new JobKey("job1");
    //停止任务
    scheduler.pauseJob(key);

    return "ok";
}
```

#### 启动

```java
public String start() throws Exception {
    //创建JobKey
    JobKey key = new JobKey("job1");
    //关闭任务
    scheduler.resumeJob(key);

    return "ok";
}
```

#### 修改执行时间

```java
public String trigger() throws Exception {
    // 通过Job的name生成JobKey
    JobKey jobKey = new JobKey("job1");
    // 通过JobKey先获取JobDetail
    JobDetail jobDetail = scheduler.getJobDetail(jobKey);
    // 生成新的 trigger
    Trigger trigger = TriggerBuilder
        .newTrigger()
        .withSchedule(CronScheduleBuilder.cronSchedule("0/1 * * * * ?"))
        .build();
    // 通过Jobkey删除任务，不删除会报错。报任务已存在
    scheduler.deleteJob(jobKey);
    // 使用新的Trigger启动任务
    scheduler.scheduleJob(jobDetail, trigger);

    return "ok";
}
```



## 持久化

- 被Scheduler调度的任务会被自动持久化到数据库(未被调度的则不会持久化)
  - Scheduler调用schedulerJob()后才会持久化任务信息
  - 被初始化后的任务,不用再次初始化
- 数据库中存在任务的信息,SpringBoot启动时会从数据库捞取任务进行执行.
  - SpringBoot并不需要再次调用启动任务, 表中存在记录则会自动捞取任务执行
- 集群环境下,任务多次初始化会产生时间线错误
  - 初始化一次即可, 不需要将持久化设置为always, 若设置为never也不可初始化第二次

### SpringBoot整合持久化

[参考SpringBoot文档]([42. Quartz Scheduler](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-quartz.html))

```properties
spring.quartz.auto-startup=true # Whether to automatically start the scheduler after initialization.
spring.quartz.jdbc.comment-prefix=-- # Prefix for single-line comments in SQL initialization scripts.
spring.quartz.jdbc.initialize-schema=embedded # Database schema initialization mode.
spring.quartz.jdbc.schema=classpath:org/quartz/impl/jdbcjobstore/tables_@@platform@@.sql # Path to the SQL file to use to initialize the database schema.
spring.quartz.job-store-type=memory # Quartz job store type.
spring.quartz.overwrite-existing-jobs=false # Whether configured jobs should overwrite existing job definitions.
spring.quartz.properties.*= # Quatz中的其他配置属性,参考Quartz文档配置在此属性下即可 Additional Quartz Scheduler properties.
spring.quartz.scheduler-name=quartzScheduler # Name of the scheduler.
spring.quartz.startup-delay=0s # Delay after which the scheduler is started once initialization completes.
spring.quartz.wait-for-jobs-to-complete-on-shutdown=false # Whether to wait for running jobs to complete on shutdown.
```

#### 1.与当前服务使用同一个库

```yml
spring:
 quartz:   
  job-store-type: jdbc  # 指定数据源为JDBC
 jdbc: 
  initialize-schema: always  # 指定每次启动SSpringBoot都会重建Quartz的相关表
                             # naver不进行重建
```

#### 2.Quartz使用独立的库

声明一个Bean,类型是DataSource,并且在生成Bean的方法上标志`@QuartzDataSource`

```java
@QuartzDataSource
@Bean
public DataSource createQuartzDataSource(){
    // 声明返回一个数据源即可, 此处使用的是最简单的方式示例
    return new DataSource().....;
}
```

![img](D:\Typora\wiz:\088b68f0-2099-11e9-8f40-8f77bbfda04f\088ffcd0-2099-11e9-8f40-8f77bbfda04f\e3088b79-dba6-46d9-b137-c6d6b5799856\index_files\e7759fea-1e07-40e9-9bfd-207e93102543.png)

### Quartz表

启动时会自动建立Quartz相关的表,12张表具有不同的功能

| 表名                     | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| QRTZ_CALENDARS           | 以 Blob 类型存储 Quartz 的 Calendar 信息                     |
| QRTZ_CRON_TRIGGERS       | 存储 Cron Trigger，包括 Cron 表达式和时区信息                |
| QRTZ_FIRED_TRIGGERS      | 存储与已触发的 Trigger 相关的状态信息，以及相联 Job 的执行信息 |
| QRTZ_PAUSED_TRIGGER_GRPS | 存储已暂停的 Trigger 组的信息                                |
| QRTZ_SCHEDULER_STATE     | 存储少量的有关 Scheduler 的状态信息，和别的 Scheduler 实例(假如是用于一个集群中) |
| QRTZ_LOCKS               | 存储程序的悲观锁的信息(假如使用了悲观锁)                     |
| QRTZ_JOB_DETAILS         | 存储每一个已配置的 Job 的详细信息                            |
| QRTZ_JOB_LISTENERS       | 存储有关已配置的 JobListener 的信息                          |
| QRTZ_SIMPLE_TRIGGERS     | 存储简单的 Trigger，包括重复次数，间隔，以及已触的次数       |
| QRTZ_BLOG_TRIGGERS       | Trigger 作为 Blob 类型存储(用于 Quartz 用户用 JDBC 创建他们自己定制的 Trigger 类型，JobStore 并不知道如何存储实例的时候) |
| QRTZ_TRIGGER_LISTENERS   | 存储已配置的 TriggerListener 的信息                          |
| QRTZ_TRIGGERS            | 存储已配置的 Trigger 的信息                                  |

## Quartz集群

[Quartz集群文档]([Configuration Reference](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/configuration/ConfigJDBCJobStoreClustering.html))

存在的问题:一个任务在多台机器下,会出现重复执行的情况; 例如本地锁在分布式情况下的问题.

### 解决方案

1. 只让集群中一台及其去跑任务(浪费集群环境的机器性能)
2. 每台机器都会执行定时任务,但是一个任务只会被分配到一台机器上.

### 使用JDBC-jobStore配置集群

Quartz通过故障转移和负载平衡功能为调度程序带来高可用性和可扩展性.需要通过数据库实现;

群集当前仅适用于JDBC-Jobstore（JobStoreTX或JobStoreCMT），并且实质上是通过使群集的每个节点**共享同一数据库来工作的**

通过将“ org.quartz.jobStore.isClustered”属性设置为“ true”来启用集群。集群中的每个实例都应使用quartz.properties文件的相同副本。允许以下例外：不同的线程池大小和“ org.quartz.scheduler.instanceId”属性的不同值。

instanceId是唯一的.设置为AUTO 会在实例启动时自动分配一个.

**集群依赖每个节点的时钟,不能讲集群运行在不同的时钟下**

#### SpringBoot配置

```yml
# 将Quartz的配置,配置在spring.quartz.properties.下
spring:
 quarta: 
  org.quartz.jobStore.isClustered: true;  # 开启集群
  org.quartz.scheduler.instanceName: cluster1 # 属于哪一个集群
  org.quartz.scheduler.instanceId: AUTO  # 自动分配一个
```

#### Quartz配置

```properties
＃================================================= ==========================
＃配置主调度程序属性  配置在Spring.Quartz.properties 下
＃================================================= ==========================

org.quartz.scheduler.instanceName = MyClusteredScheduler
org.quartz.scheduler.instanceId =AUTO  # 自动分配一个

＃================================================= ==========================
＃配置ThreadPool  
＃================================================= ==========================

org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 25
org.quartz.threadPool.threadPriority = 5

＃================================================= ==========================
＃配置JobStore  
＃================================================= ==========================

org.quartz.jobStore.misfireThreshold = 60000

org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.oracle.OracleDelegate
org.quartz.jobStore.useProperties =false
org.quartz.jobStore.dataSource = myDS
org.quartz.jobStore.tablePrefix = QRTZ_

org.quartz.jobStore.isClustered = true
org.quartz.jobStore.clusterCheckinInterval = 20000

＃================================================= ==========================
＃配置数据源  
＃================================================= ==========================

org.quartz.dataSource.myDS.driver = oracle.jdbc.driver.OracleDriver
org.quartz.dataSource.myDS.URL = jdbc：oracle：thin：@polarbear：1521：dev
org.quartz.dataSource.myDS.user =
org.quartz.dataSource.myDS.password =
org.quartz.dataSource.myDS.maxConnections = 5
org.quartz.dataSource.myDS.validationQuery =select 0 from dual
```

### 集群下多数据库

1. 当Quartz任务过多数据库压力过大时,可将Quartz分成多个库,将任务存储到不同的库中
   - 配置多个数据源
   - 配置实例的配置文件为不同的集群`instanceName`
   - 在配置任务时,分配任务一个data, usingJobData(key,value),value对应instanceName
   - 在初始化任务时,判断任务的key对应的value,与当前实例配置文件中`instanceName`一致时才进行调度

### 集群下产生的问题

- 初始化任务后,多个实例启动若都指定`initialize-schema: always`则会导致多台实例会在启动时删除数据库表重建,从而任务调度时间线出错
- 设置`initialize-schema: never`任务初始化到数据库后,再次启动第二个实例,会尝试再次初始化到数据库,从而会导致报错. 初始化完成后不应当再次初始化
  - 可以将初始化写成接口,系统上线后调用接口进行初始化一次
- scheduler调用jobdetail与trigger时,若使用多参数的方法,指定任务key在数据库已存在是是否替换, 若为true则替换,false不替换若冲突则报错.
- 集群节点多可能会导致性能下降,集群下的锁是对整个集群上锁.

## 分布式环境

Quartz主要强调集群的使用,几乎没有分布式相关的配置.

Quartz依赖instanceName集群名称进行划分,不同的集群名称下的任务,不会被其他集群调用,分布式情况下,可以通过不同集群的名称来防止服务调用其他服务的定时任务;

## Job的状态

1. 默认状态下同一个任务是可以被并发调用执行的
   - 当任务的调度时间小于执行时间
   - 当任务被多个Trigger调度
2. 默认状态下同一个任务每次执行的实例是不同的
3. 默认情况下,在Job中对JobData的修改,不会被记录下来,下次执行Job时,JobData不会被修改.

### @DisallowConcurrentExecution注解

- **将此注解声明在Job类上,可以让任务按照调度顺序执行,不会再调度1未执行完时并行执行调度2导致并发执行任务**
- 只要有相同的job的key与group,则会任务试试相同的任务.

### @PersistJobDataAfterExecution注解

- **将此注解声明在Job类上,可以让Job在任务中修改JobData后持久化(RAM/JDBC)保存修改,被下次Job执行时获取到修改后的JobData**
  - 修改JobData是通过context.getJobDetail().getJobDataMap().put("相同的key","不同的value");实现.
- 如果使用Mysql做了持久化,那么持久化的JobData会被表的Job_Data字段中
- **使用到了这个注解时,建议配合@DisallowConcurrentExecution使用,防止并发问题**

## Misfire失火

**到了任务触发时间,但是任务并没有触发**

- 失火的原因
  1. 使用了@DisallowConcurrentExecution注解,并且任务执行时间>任务调度时间
  2. 线程池满了,没有空余线程执行任务
  3. 机器宕机,或者人为停止.过段时间恢复运行
- 失火策略 分为两种Trigger
  - **SimpleTrigger**
    - 默认
      - 执行时间为启动时间之前,不指定失火策略,会以项目启动时间为开始时间开始执行
    - **withMisFireHandlingInstructionFireNow**()
      - 执行时间为启动时间之前,会以项目启动时间为开始时间开始执行,并忽略掉启动时间之前所需要执行的次数.
    - **withMisFireHandlingInstructionNowWithExistingCount**()
      - 在执行固定次数的情况下与默认相同, 执行一次或无限次等情况不一定相同
    - **withMisFireHandlingInstructionIgnoreMisFire**()
      - 忽略失火策略,在启动是立即补齐之前未执行的次数,然后继续正常执行.
    - **withMisFireHandlingInstructionNextWithExistingCount**()
      - 忽略之前未执行的次数,然后正常执行
    - **withMisFireHandlingInstructionNextWithRemainingCount**()
      - 项目启动后补一次,然后正常执行
    
  - **其他Trigger**
  
  - CronTrigger Misfire
  
    - CronTrigger的Misfire有三种如下，可以通过CronSchedulerBuilder设置
      - `MISFIRE_INSTRUCTION_DO_NOTHING`
        设置方法：`withMisfireHandlingInstructionDoNothing`
        含义：不进行补齐,等下次时间到了开始执行
        - 不触发立即执行
        - 等待下次Cron触发频率到达时刻开始按照Cron频率依次执行
  
      - `MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY`
        设置方法：`withMisfireHandlingInstructionIgnoreMisfires`
        含义： 补齐错过的所有次数
        - 以错过的第一个频率时间立刻开始执行
        - 重做错过的所有频率周期后
        - 当下一次触发频率发生时间大于当前时间后，再按照正常的Cron频率依次执行
  
      - `MISFIRE_INSTRUCTION_FIRE_ONCE_NOW`
        设置方法：`withMisfireHandlingInstructionFireAndProceed`
        含义：立即补一次,然后正常执行
        - 以当前时间为触发频率立刻触发一次执行
        - 然后按照Cron频率依次执行

> 默认策略
> 在SimpleTrigger中已经提到所有trigger的默认Misfire策略都是MISFIRE_INSTRUCTION_SMART_POLICY，SimpleTrigger会根据tirgger的状态来调整具体的Misfire策略
>
> 而CronTrigger的默认Misfire策略会被CronTrigger解释为MISFIRE_INSTRUCTION_FIRE_NOW，具体可以参照CronTrigger实现类的源码

## 任务异常

### JobExecutionException

任务中抛出异常时,不影响任务被继续调度执行.不会影响后面的调度

通过JobExecutionException与try...catch实现任务出现异常后重新执行一次

### 当任务出现异常后,再次让任务执行一次

- 当任务失败时自动重新执行的方案,任务上需要标记PersistJobDataAfterExecution
  - 手动try异常,然后通过context再次调度任务执行,在try中判断JobDataMap中标记key是否存在,存在说明当前是重试. (不使用)
    - 每次调度的context是不同的对象
    - JobDataMap中的数据,只有在任务执行完才会持久化到数据库,所以可能出现执行多次的情况,
  - 使用Quartz提供的`JobExecutionException`,new 一个异常,构造方法传入指定的异常对象,在try中抛出JobExecutionException对象.
    - 同样需要设置JobDataMap一个标记key,在try中代码判断key是否存在来判断是否为重试方法
    - 使用JobExecutionException对象调用方法setRefireImmediately(true)表示立即重试执行
    - 需要注意,此方法使用的是同一个context,JobDataMap不会再次初始化,所以key设置在JobDetai或Trigger中,则需要从对应的JobDetai或Trigger获取.才能获取到.

### 当任务出现异常后,不再继续执行任务调度

- 当任务失败需要停调度时,可通过try catch使用`JobExecutionException`对象调用方法setUnscheduleAllTriggers(true),来指定全部触发器(包含当前未调度该任务的Scheduler)都停止调度该任务,然后抛出JobExecutionException
- 也可以指定当前正在调度的Scheduler不再调度此任务,通过setUnscheduleFiringTrigger(true)即可

## 任务中断

类似于线程的中断,在执行逻辑中可以打断任务的执行.不让任务继续执行下去,可以在某些步骤停止,不继续执行.

```java
// 普通代码的中断逻辑
    初始化资源
    if(中断条件){

    }
    执行逻辑1

    if(中断条件){

    }
    执行逻辑2

    if(中断条件){

    }
    执行逻辑3
return;
```

### InterruptedJob接口

此接口继承Job接口,多了一个方法interrupt() 打断任务;

#### InterruptedJob. interrupt()

原理: 通过一个全局变量作为flag, 调用interrupt()方法时,改变全局变量的值,在任务内通过if判断全局变量 的值,符合条件则 return; 即可;

注:如果是在集群的环境下打断,则只会中断当前实例的任务, interrupt只是提供了一种中断方式.

返回值: 打断一个或多个任务则返回true,否则返回false;

#### Scheduler.interrupt()

调用此方法,传入JobDetail.getKey(); 即可打断对应任务;

## 日期排除

日期排除可以让任务在指定日期不执行;

1. 声明一个AnnualCalendar对象(此对象以年为单位)
2. 声明一个GregorianCalendar(year,month,day)对象,此类继承Calendar(此处年不生效,但是构建Calendar对象需要)
3. 调用AnnualCalendar对象方法 setDayExcluded(GregorianCalendar对象.true);
   1. 构造参数第一个传入需要排除的日期
   2. 构造参数第二个传入是否生效
4. 调用SScheduler方法addCalendar(AnnualCalendar对象)生效

```java
AnnualCalendar holiday = new AnnualCalendar();
//可以添加多个Calendar
GregorianCalendar calendar = new GregorianCalendar(2021,12,10);
holiday.setDayExcluded(calendar,true);
//此处的calendar name在被调度时指定生效
scheduler.addCalendar("holiday",holiday,false,false); 
// ...省略 JobDetail...
//构建触发器时指定Calendar,
Trigger trigger = newTrigger()...modifieByCalendar("holiday").build();
```

### BaseCalendar实现类

- CronCalendar
  - 用来排除给定Cron表达式表示的时间集
- AnnualCalendar
  - 用来排除年中的某一天,以年为单位循环,不区别年份
- HolidayCalendar
  - 用来排除某年的某一天,与AnnualCalendar不同的是区别年份
- MonthlyCalendar
  - 用来排除月中的天
- WeeklyCalendar
  - 用来排除星期中的某一天
- DaliyCalendar
  - 用来排除一天中的某个时间段,不能跨天(可以反转时间段作用)
  - 通过setInvertTimeRange(true);设置时间段反转

**排除的日历是可以相互叠加的,可以直接将一个Calendar传入另一个Calendar的构造方法**

## 监听器

- 监听的场景
  - 统计一个月内,某任务的平均执行时间和执行次数
  - 只有在任务A执行完的情况下,才执行任务B
  - 想要任务在misfire的时候收到通知
  - 想要密切关注系统中任务的情况,比如调度器什么时候启动/停止,什么时候添加/移除了任务.
- 监听器类型
  - JobListener
  - TriggerListener
  - SchedulerListener
  - 监听谁,则实现对应的Listener即可
- 监听器方法
  - 在之前执行
  - 在之后执行
  - ..其他方法
- 匹配器
  - Matcher 接口
    - KeyMatcher
    - EverythingMatcher
    - StringMatcher
      - NameMatch
      - GroupMatcher
    - AndMatcher
    - NotMacher
    - OrMatcher
  - 每个匹配器下都有对应的方法可以添加Job / Trigger / Scheduler的匹配规则
  - Or,And中可以传入其他匹配器对象,形成复杂条件

```java
//前提-- 自定义实现类实现对应的监听器接口


JobDetail ....

Trigger ....

//新建一个监听器,自己实现对应的Listener接口,重写方法
JobListener listener = new MyJobListener(); 
//新建一个匹配器,匹配器类型是在各种监听器间通用的
//泛型是锁匹配的类型
Matcher<JobKey> matcher = KeyMatcher.keyEquals(job.getKey());
//在调度器中添加监听器可匹配器, 方法选择对应类型监听器的add方法
scheduler.getListenerManager().addJobListener(listener,matcher);

scheduler.start();
```

## 多Scheduler

默认情况下,一个项目中只有一个Scherduler,当需要多个Scherduler时,可以通过指定配置文件生成新的Scherduler;

`new ScherdulerFactory(props).getScherduler()` 获取一个新的Scherduler

* 例如在业务任务/系统任务间使用两个Scherduler进行控制

## 插件

不常用,TODO
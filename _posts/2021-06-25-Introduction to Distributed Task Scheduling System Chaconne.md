---
layout: post
title: Introduction of distributed task scheduling system Chaconne
date: 2021-06-25 08:30:00.000000000 +09:00
---

**chaconne**（音译夏空），是一款用Java写的，基于SpringBoot框架的轻量级的分布式任务调度框架。引入chaconne相关组件，可以非常快捷地帮助你搭建一个分布式任务集群

### chaconne 特性列表
-----------------
  1. 完美支持spring-boot框架（2.2.0+）
  2. 支持多种方式设置的定时任务（Cron表达式，参数设置等）
  3. 支持动态保存和删除任务
  4. 支持注解配置定时任务
  5. 支持两种集群模式（主备模式和负载均衡模式）
  6. 内置多种负载均衡算法，支持自定义负载均衡算法
  7. 支持失败重试和失败转移
  8. 支持日志追踪
  9. 支持任务参数分片处理
10. 支持任务依赖（串行依赖和并行依赖）
 11. 支持DAG模拟工作流
 12. 支持任务自定义终止策略
 13. 支持任务超时冷却和重置
 14. 支持邮件告警

### chaconne两种集群部署模式：
----------------------------
1. 去中心化部署模式
    * 没有固定的调度中心节点，chaconne集群会选举其中一个应用作为Leader, 进行任务指挥调度
   *  参与调度和参与执行的应用通过Tcp协议交互
2. 中心化部署模式
   * 分为调度中心和任务执行节点两个角色，且调度中心和任务执行节点都支持集群模式
   * 调度中心和任务执行节点通过Http协议交互

**说明：**
这里的集群是指参与任务执行的应用所组成的集群(chaconne集群)，它和基于SpringCloud框架组成的集群是两个独立的概念
如果chaconne集群规模较小，推荐使用去中心化部署模式，若该集群规模较大，依据实际情况，两者模式都可以使用。

### chaconne框架由两部分组成：
--------------------------------------
1. chaconne-spring-boot-starter  
    核心jar包，包含了chaconne全部核心功能（包括自定义你的Web管理界面）
2. chaconne-console
    Chaconne Web管理界面，进行任务管理和查看任务运行状态
3. chaconne-manager
    如果是中心化部署，这里提供了个chaconne的调度中心demo

### 安装：
-------------------------
``` xml
<dependency>
	  <groupId>com.github.paganini2008.atlantis</groupId>
     <artifactId>chaconne-spring-boot-starter</artifactId>
     <version>1.0-RC3</version>
</dependency>
```
（确保是最新版）

### 兼容性：
-------------------------
* Jdk 1.8 (or later)
* SpringBoot 2.2.0 (or later)
* Redis 3.0 (or later)
* MySQL 5.0 (or later)

**说明：**
Redis用于存取集群信息和做消息广播
MySQL用于存取任务定义和运行时相关数据，目前只支持MySQL, 关于表结构，应用程序启动时自动建表

### 必要的配置：
------------------------
如果是中心化配置，如下：
``` properties
spring.application.cluster.name=jobtester-cluster  # set chaconne cluster name
spring.application.name=jobtester

#Jdbc Configuration
atlantis.framework.chaconne.datasource.jdbcUrl=jdbc:mysql://localhost:3306/demo?userUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF8&useSSL=false&autoReconnect=true&zeroDateTimeBehavior=convertToNull
atlantis.framework.chaconne.datasource.username=fengy
atlantis.framework.chaconne.datasource.password=123456
atlantis.framework.chaconne.datasource.driverClassName=com.mysql.cj.jdbc.Driver

#Redis Configuration
atlantis.framework.redis.host=localhost
atlantis.framework.redis.port=6379
atlantis.framework.redis.password=123456
atlantis.framework.redis.database=0

spring.redis.messager.pubsub.channel=chaconne-management-messager-pubsub
```

如果是去中心化配置，只要确保有Redis配置和DataSource配置即可

### chaconne实现原理简述
------------------------
chaconne的底层是依赖tridenter-spring-boot-starter组件来实现任务集群模式的（主备模式和负载均衡模式），利用消息单播机制（通过Redis PubSub模拟）来实现任务分发和负载均衡，分片处理等高级特性。需要指出的是，chaconne框架中关于集群的定义和tridenter关于集群的定义是一致的，对于集群的概念，等同于用来区别不同的产品组或公司，同时chaconne也支持任务组的概念，它是可选配置，默认情况下，组名就是当前应用名称（${spring.application.name}），即当起了多个相同应用名的应用，那这些应用就成为了一个任务组。chaconne不仅支持跨组的任务调用，更支持跨集群的任务调用。

### 如何定义任务？
---------------------------------
1. 使用注解@ChacJob
2. 继承ManagedJob类
3. 实现Job接口
4. 实现NotManagedJob接口
说明：
* 前3种定义任务的方式属于声明式（编程式）定义任务，即通过代码方式定义一个任务，随之Spring框架上下文的启动而自动加载
* 第4种定义的方式用来定义动态任务，用户可以在Web界面上（Chaconne-Console）提交来创建任务或直接通过调用Http API/SDK来创建任务，需要说明的是，通过该方式创建的任务对象不属于Spring上下文托管的Bean对象。

**示例代码：**
**1. 用注解方式创建任务**
``` java
@ChacJob
@ChacTrigger(cron = "*/5 * * * * ?")
public class DemoCronJob {

	@Run
	public Object execute(JobKey jobKey, Object attachment, Logger log) throws Exception {
		log.info("DemoCronJob is running at: {}", DateUtils.format(System.currentTimeMillis()));
		return RandomUtils.randomLong(1000000L, 1000000000L);
	}

	@OnSuccess
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		log.info("DemoCronJob's return value is: {}", result);
	}

	@OnFailure
	public void onFailure(JobKey jobKey, Throwable e, Logger log) {
		log.error("DemoCronJob is failed by cause: {}", e.getMessage(), e);
	}

}
```

**2. 接口方式创建任务**
* 实现Job接口：
``` java
@Component
public class HelloWorldJob implements Job {

	@Override
	public String getClusterName() {
		return "your_cluster_name";
	}

	@Override
	public String getGroupName() {
		return "your_group_name";
	}

	@Override
	public int getRetries() {
		return 3;
	}

	@Override
	public long getTimeout() {
		return 60 * 1000L;
	}

	@Override
	public String getEmail() {
		return "your_email@helloworld.com";
	}

	@Override
	public Trigger getTrigger() {
		return GenericTrigger.Builder.newTrigger(1L, SchedulingUnit.MINUTES, false).build();
	}

	@Override
	public Object execute(JobKey jobKey, Object result, Logger logger) {
		return "Hello World!";
	}

	@Override
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		if (log.isInfoEnabled()) {
			log.info(result.toString());
		}
	}

}
```
* 或继承ManagedJob类：
``` java
@Component
public class HealthCheckJob extends ManagedJob {

	@Override
	public long getTimeout() {
		return 60L * 1000;
	}

	@Override
	public Trigger getTrigger() {
		return GenericTrigger.Builder.newTrigger("*/5 * * * * ?").setStartDate(DateUtils.addSeconds(new Date(), 30)).build();
	}

	@Override
	public Object execute(JobKey jobKey, Object arg, Logger log) {
		if (log.isInfoEnabled()) {
			log.info(info());
		}
		return UUID.randomUUID().toString();
	}

	@Override
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		if (log.isInfoEnabled()) {
			log.info(result.toString());
		}
	}

	private String info() {
		long totalMemory = Runtime.getRuntime().totalMemory();
		long usedMemory = totalMemory - Runtime.getRuntime().freeMemory();
		return FileUtils.formatSize(usedMemory) + "/" + FileUtils.formatSize(totalMemory);
	}

}
```
**如何动态任务？**
**第一种方式：**在页面上创建（后面会讲到）
略
**第二种方式：**通过SDK创建
* 首先实现NotManagedJob 接口定义任务
``` java
public class EtlJob implements NotManagedJob {

	@Override
	public Object execute(JobKey jobKey, Object attachment, Logger log) {
		log.info("JobKey:{}, Parameter: {}", jobKey, attachment);
		return null;
	}

}
```
* 使用Http API调用的方式
POST  http://localhost:6543/job/admin/persistJob
``` json
{
    "jobKey": {
        "clusterName": "yourCluster",
        "groupName": "yourGroup",
        "jobName": "yourJob",
        "jobClassName": "com.yourcompany.yourapp.YourJob"
    },
    "description": "Describe your job shortly",
    "email": "YourEmail@yourcompany.com",
    "retries": 0,
    "timeout": -1,
    "weight": 100,
    "dependentKeys": null,
    "forkKeys": null,
    "completionRate": -1,
    "trigger": {
        "triggerType": 1,
        "triggerDescription": {
            "cron": {
                "expression": "*/5 * * * * ?"
            }
        },
        "startDate": null,
        "endDate": null,
        "repeatCount": -1
    },
    "attachment": "{\"initialParameter\": \"test\"}"
}
```
* 或者使用SDK:
``` java
@Component
public class TestService {
	
	@Autowired
	private JobManager jobManager;

	public void createJob() throws Exception {
		final JobKey jobKey = JobKey.by("yourCluster", "yourGroup", "yourJob", "com.yourcompany.yourapp.YourJob");
		GenericJobDefinition.Builder builder = GenericJobDefinition.newJob(jobKey)
				.setDescription("Describe your job shortly")
				.setEmail("YourEmail@yourcompany.com")
				.setRetries(3)
				.setTimeout(60000L);
		GenericTrigger.Builder triggerBuilder = GenericTrigger.Builder.newTrigger("*/5 * * * * ?");
		builder.setTrigger(triggerBuilder.build());
		GenericJobDefinition jobDefinition = builder.build();
		
		jobManager.persistJob(jobDefinition, "{\"initialParameter\": \"test\"}");
	}

}
```
**注意：** 任务初始化参数建议是json格式的

### 任务依赖
----------------------------
任务依赖是**chaconne**框架的重要特性之一，任务依赖分为**串行依赖**和**并行依赖**，
所谓串行依赖是指任务A做完接着执行任务B,  即任务B依赖任务A。
并行依赖是指，比如有3个任务，分别为任务A, 任务B, 任务C, 任务A和任务B都做完才能执行任务C, 类似会签的业务场景。
串行依赖和并行依赖都可以共享任务上下文参数和运行结果，并且支持自定义判断策略来决定要不要触发下游任务。
##### DAG(有向无环图)
而在结合串行依赖和并行依赖的基础上，chaconne框架又提供了DAG功能并提供了友好的API，来模拟类似工作流的业务场景，更加丰富了任务依赖的使用场景。
（这里为了方便举例，都通过注解的方式配置任务)
##### 串行依赖示例：
``` java
@ChacJob
@ChacTrigger(triggerType = TriggerType.DEPENDENT)
@ChacDependency({ @ChacJobKey(className = "com.chinapex.test.chaconne.job.DemoSchedJob", name = "demoSchedJob") })
public class DemoDependentJob {

	@Run
	public Object execute(JobKey jobKey, Object attachment, Logger log) throws Exception {
		log.info("DemoDependentJob is running at: {}", DateUtils.format(System.currentTimeMillis()));
		return RandomUtils.randomLong(1000000L, 1000000000L);
	}

	@OnSuccess
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		log.info("DemoDependentJob's return value is: {}", result);
	}

	@OnFailure
	public void onFailure(JobKey jobKey, Throwable e, Logger log) {
		log.error("DemoDependentJob is failed by cause: {}", e.getMessage(), e);
	}

}
```
##### 并行依赖示例：
有3个任务，DemoTask, DemoTaskOne, DemoTaskTwo
让DemoTaskOne, DemoTaskTwo都做完再执行DemoTask，且DemoTask可以获得DemoTaskOne, DemoTaskTwo执行后的值
###### DemoTaskOne: 
``` java
@ChacJob
@ChacTrigger(triggerType = TriggerType.SIMPLE)
public class DemoTaskOne {

	@Run
	public Object execute(JobKey jobKey, Object attachment, Logger log) throws Exception {
		log.info("DemoTaskOne is running at: {}", DateUtils.format(System.currentTimeMillis()));
		return RandomUtils.randomLong(1000000L, 1000000000L);
	}

	@OnSuccess
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		log.info("DemoTaskOne return value is: {}", result);
	}

	@OnFailure
	public void onFailure(JobKey jobKey, Throwable e, Logger log) {
		log.error("DemoTaskOne is failed by cause: {}", e.getMessage(), e);
	}

}
```
###### DemoTaskTwo:
``` java
@ChacJob
@ChacTrigger(triggerType = TriggerType.SIMPLE)
public class DemoTaskTwo {

	@Run
	public Object execute(JobKey jobKey, Object attachment, Logger log) throws Exception {
		log.info("DemoTaskTwo is running at: {}", DateUtils.format(System.currentTimeMillis()));
		return RandomUtils.randomLong(1000000L, 1000000000L);
	}

	@OnSuccess
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		log.info("DemoTaskTwo return value is: {}", result);
	}

	@OnFailure
	public void onFailure(JobKey jobKey, Throwable e, Logger log) {
		log.error("DemoTaskTwo is failed by cause: {}", e.getMessage(), e);
	}
	
}

```
###### DemoTask:
``` java
@ChacJob
@ChacTrigger(cron = "0 0/1 * * * ?", triggerType = TriggerType.CRON)
@ChacFork({ @ChacJobKey(className = "com.chinapex.test.chaconne.job.DemoTaskOne", name = "demoTaskOne"),
		@ChacJobKey(className = "com.chinapex.test.chaconne.job.DemoTaskTwo", name = "demoTaskTwo") })
public class DemoTask {

	@Run
	public Object execute(JobKey jobKey, Object attachment, Logger log) throws Exception {
		log.info("DemoTask is running at: {}", DateUtils.format(System.currentTimeMillis()));
		TaskJoinResult joinResult = (TaskJoinResult) attachment;
		TaskForkResult[] forkResults = joinResult.getTaskForkResults();
		long max = 0;
		for (TaskForkResult forkResult : forkResults) {
			max = Long.max(max, (Long) forkResult.getResult());
		}
		return max;
	}

	@OnSuccess
	public void onSuccess(JobKey jobKey, Object result, Logger log) {
		log.info("DemoTask return max value is: {}", result);
	}

	@OnFailure
	public void onFailure(JobKey jobKey, Throwable e, Logger log) {
		log.error("DemoTask is failed by cause: {}", e.getMessage(), e);
	}

}
```
##### DAG任务示例
DAG任务目前只支持API创建, 后续会持续改进，增加界面方式创建DAG任务
``` java
@RequestMapping("/dag")
@RestController
public class DagJobController {

	@Value("${spring.application.cluster.name}")
	private String clusterName;

	@Value("${spring.application.name}")
	private String applicationName;

	@Autowired
	private JobManager jobManager;

	@GetMapping("/create")
	public Map<String, Object> createDagTask() throws Exception {
		Dag dag = new Dag(clusterName, applicationName, "testDag");// 创建一个Dag任务，并指定集群名，应用名，和任务名称
		dag.setTrigger(new CronTrigger("0 0/1 * * * ?"));// 设置Cron表达式
		dag.setDescription("This is only a demo of dag job");// 任务描述
		DagFlow first = dag.startWith(clusterName, applicationName, "demoDagStart", DemoDagStart.class.getName());// 定义第一个节点
		DagFlow second = first.flow(clusterName, applicationName, "demoDag", DemoDag.class.getName());// 定义第二个节点
                // 第二个节点fork两个子进程处理
		second.fork(clusterName, applicationName, "demoDagOne", DemoDagOne.class.getName());
		second.fork(clusterName, applicationName, "demoDagTwo", DemoDagTwo.class.getName());
		jobManager.persistJob(dag, "123");
		return Collections.singletonMap("ok", 1);
	}

}
```
上面的DAG示例说明一下，chaconne框架提供的DAG模型支持串行流入，即flow模式，也提供了fork模式进行并行处理，上例中，任务demoDag fork了两个子进程（“demoDagOne”和“demoDagTwo”），即demoDagOne和demoDagTwo同时处理完了再触发demoDag任务。

### Chaconne部署说明
-------------------------------
chaconne除了依托SpringBoot框架外，默认用MySQL存储任务信息(目前仅支持MySQL，后续会支持更多类型的数据库),  用Redis保存集群元数据和进行消息广播。
所以无论使用哪种部署方式，你都需要在你的应用中设置DataSource和RedisConnectionFactory
**示例代码：**
``` java
@Slf4j
@Configuration
public class ResourceConfig {

	@Setter
	@Configuration(proxyBeanMethods = false)
	@ConfigurationProperties(prefix = "spring.datasource")
        @ConditionalOnMissingBean(DataSource.class)
	public static class DataSourceConfig {

		private String jdbcUrl;
		private String username;
		private String password;
		private String driverClassName;

		private HikariConfig getDbConfig() {
			if (log.isTraceEnabled()) {
				log.trace("DataSourceConfig JdbcUrl: " + jdbcUrl);
				log.trace("DataSourceConfig Username: " + username);
				log.trace("DataSourceConfig Password: " + password);
				log.trace("DataSourceConfig DriverClassName: " + driverClassName);
			}
			final HikariConfig config = new HikariConfig();
			config.setDriverClassName(driverClassName);
			config.setJdbcUrl(jdbcUrl);
			config.setUsername(username);
			config.setPassword(password);
			config.setMinimumIdle(5);
			config.setMaximumPoolSize(50);
			config.setMaxLifetime(60 * 1000);
			config.setIdleTimeout(60 * 1000);
			config.setValidationTimeout(3000);
			config.setReadOnly(false);
			config.setConnectionInitSql("SELECT UUID()");
			config.setConnectionTestQuery("SELECT 1");
			config.setConnectionTimeout(60 * 1000);
			config.setTransactionIsolation("TRANSACTION_READ_COMMITTED");

			config.addDataSourceProperty("cachePrepStmts", "true");
			config.addDataSourceProperty("prepStmtCacheSize", "250");
			config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
			return config;
		}

		@Primary
		@Bean
		public DataSource dataSource() {
			return new HikariDataSource(getDbConfig());
		}

	}

	@Setter
	@Configuration(proxyBeanMethods = false)
	@ConfigurationProperties(prefix = "spring.redis")
        @ConditionalOnMissingBean(RedisConnectionFactory.class)
	public static class RedisConfig {

		private String host;
		private String password;
		private int port;
		private int dbIndex;

		@Bean
		public RedisConnectionFactory redisConnectionFactory() {
			RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
			redisStandaloneConfiguration.setHostName(host);
			redisStandaloneConfiguration.setPort(port);
			redisStandaloneConfiguration.setDatabase(dbIndex);
			redisStandaloneConfiguration.setPassword(RedisPassword.of(password));
			JedisClientConfiguration.JedisClientConfigurationBuilder jedisClientConfiguration = JedisClientConfiguration.builder();
			jedisClientConfiguration.connectTimeout(Duration.ofMillis(60000)).readTimeout(Duration.ofMillis(60000)).usePooling()
					.poolConfig(jedisPoolConfig());
			JedisConnectionFactory factory = new JedisConnectionFactory(redisStandaloneConfiguration, jedisClientConfiguration.build());
			return factory;
		}

		@Bean
		public JedisPoolConfig jedisPoolConfig() {
			JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
			jedisPoolConfig.setMinIdle(1);
			jedisPoolConfig.setMaxIdle(10);
			jedisPoolConfig.setMaxTotal(200);
			jedisPoolConfig.setMaxWaitMillis(-1);
			jedisPoolConfig.setTestWhileIdle(true);
			return jedisPoolConfig;
		}

	}

}
```
你也可以自定义DataSource和RedisConnectionFactory 

##### Chaconne去中心化部署
在你的Spring应用程序的主函数上加上@EnableChaconneEmbeddedMode注解，然后启动
示例代码：
``` java
@EnableChaconneEmbeddedMode
@SpringBootApplication
@ComponentScan
public class YourApplicationMain {

	public static void main(String[] args) {
		final int port = 8088;
		System.setProperty("server.port", String.valueOf(port));
		SpringApplication.run(YourApplicationMain.class, args);
	}

}
```
##### Chaconne中心化部署
* 启动调度中心，这需要你新建一个SpringBoot项目，在主函数上加上@EnableChaconneDetachedMode注解，并指定为生产端
示例代码：
``` java
@EnableChaconneDetachedMode(DetachedMode.PRODUCER)
@SpringBootApplication
public class ChaconneManagementMain {

	public static void main(String[] args) {
		SpringApplication.run(ChaconneManagementMain.class, args);
	}
}
```
（同时需要配置DataSource和RedisConnectionFactory）

* 或者直接使用注解@ChaconneAdmin即可，示例代码：
``` java
@ChaconneAdmin
@SpringBootApplication
public class ChaconneManagerApplication {

	static {
		System.setProperty("spring.devtools.restart.enabled", "false");
		File logDir = FileUtils.getFile(FileUtils.getUserDirectory(), "logs", "indi", "atlantis", "framework", "chaconne", "management");
		if (!logDir.exists()) {
			logDir.mkdirs();
		}
		System.setProperty("LOG_BASE", logDir.getAbsolutePath());
	}

	public static void main(String[] args) {
		SpringApplication.run(ChaconneManagerApplication.class, args);
		System.out.println(Env.getPid());
	}
}
```

2. 在你的Spring应用程序的主函数上加上@EnableChaconneDetachedMode注解（默认为消费端），然后启动
``` java
@EnableChaconneDetachedMode
@SpringBootApplication
@ComponentScan
public class YourApplicationMain {

	public static void main(String[] args) {
		final int port = 8088;
		System.setProperty("server.port", String.valueOf(port));
		SpringApplication.run(YourApplicationMain.class, args);
	}

}
```

### Chaconne Console使用说明
-----------------------------------
Chaconne Console是**chaconne**框架提供的任务管理和查看的Web项目，它也支持去中心化部署和中心化部署模式，默认端口6140
提供了如下功能：
1. 保存任务和查看任务信息
2. 暂停和继续任务
3. 删除任务
4. 手动运行任务
5. 查看任务统计（按天）
6. 查看任务运行时日志

目前Chaconne Console项目还在不断的维护中，有些功能略显粗糙（任务JSON编辑器），有些功能暂未开放。
同样，Chaconne Console也是一个SpringBoot的工程
源码：
``` java
@EnableChaconneClientMode
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class ChaconneConsoleMain {

	static {
		System.setProperty("spring.devtools.restart.enabled", "false");
		File logDir = FileUtils.getFile(FileUtils.getUserDirectory(), "logs", "indi", "atlantis", "framework", "chaconne", "console");
		if (!logDir.exists()) {
			logDir.mkdirs();
		}
		System.setProperty("DEFAULT_LOG_BASE", logDir.getAbsolutePath());
	}

	public static void main(String[] args) {
		SpringApplication.run(ChaconneConsoleMain.class, args);
		System.out.println(Env.getPid());
	}

}
```
注解@EnableChaconneClientMode表示启用一个任务管理客户端
启动后，输入首页地址：http://localhost:6140/chaconne/index
**首先进入的overview页面:**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-24e16386c32483c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**任务列表：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-9e9284d981e924ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**创建任务：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-7fb05512f9eb38b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击ShowJson可以展示Json格式的数据：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-22915828fc6d8666.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**任务详情：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-fc40433bb28e0044.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**任务日志：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-2403eb9291183071.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**任务统计：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-53ac71febc3b8b97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可查看每一个任务的统计（按天）
![image.png](https://upload-images.jianshu.io/upload_images/26217505-99d131f3cc9d4920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，附上分布式任务调度系统chaconne的源码地址：https://github.com/paganini2008/chaconne.git
有兴趣的朋友可以研究一下它的源码
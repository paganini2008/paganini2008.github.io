---
layout: post
title: Introduction to Greenfinger - High Performance Distributed WebSpider Framework
date: 2021-07-01 08:30:00.000000000 +09:00
---

Greenfinger is a high-performance, extension-oriented distributed web spider framework written in Java. It is based on the SprinBoot framework. Through some parameters settings, it can easily build a distributed web spider microservice and even a microservice cluster. In addition, Greenfinger framework also provides rich APIs to customize your application system.

### Feature: 
------------------------------
1. Perfectly compatible with SpringBoot 2.2.0 (or later)

2. Support universal and vertical crawlers

3. Adopt depth first crawling strategy

4. It is designed as a multi process and highly available crawler architecture to support dynamic horizontal expansion and load balancing

5. Built in multiple load balancing algorithms or custom load balancing algorithms

6. Support full index and incremental index

7. Support scheduled tasks to update indexes

8. Support a variety of mainstream HTTP client parsing technologies

9. Support 100 million URL uniqueness

10. Built in multiple conditional interrupt policies or custom conditional interrupt policies

11. Multiple version index query mechanism

### Compatibility:
-----------------------------------
1. jdk8 (or later)
2. SpringBoot Framework 2.2.x (or later)
3. Redis 3.x (or later)
4. PostgreSQL 9.x (or later)
5. ElasticSearch 6.x (or later)

**Notice**

  - Redis is used to save cluster information
  - PostgreSQL is used to save the crawled URL information
  - Elasticsearch is used to create indexes and search indexes


### Install:
-----------------------------
* Git Repository：
  https://github.com/paganini2008/greenfinger.git
* Directory Structure：
``` shell
├── greenfinger
|  ├── greenfinger-console
|  |  ├── pom.xml
|  |  └── src
|  ├── greenfinger-spring-boot-starter
|  |  ├── pom.xml
|  |  └── src
|  ├── LICENSE
|  ├── pom.xml
|  └── README.md
```
* Directory Description：
  + **greenfinger-console**：
     The web UI of Greenfinger, an independent SpringBoot application with its own management interface, which can add, modify, start and stop web spider tasks, and provide a search interface for real-time query
  + **greenfinger-spring-boot-starter**：
     Core jar of Greenfinger, which implements all the above framework features, provides open APIs such as spider program management and search and even customize your own system with these APIs.

###  Install greenfinger-console：
----------------------------
  **Step1:** enter directory: greenfinger-console 
  **Step2:** execute maven command：mvn clean install
  **Step3:** After successful execution, there will be a directory named 'run'. Move this directory to your working directory (the directory specified by yourself)
  **Step4:** run jar: java -jar greenfinger-console-1.0-RC2.jar --spring.config.location=config/  (Just reference)

* Directory Structure of Run：
``` shell
├── config
|  ├── application-dev.properties
|  └── application.properties
├── db
|  └── crawler.sql
├── greenfinger-console-1.0-RC2.jar
├── lib
|  ├── aggs-matrix-stats-client-6.8.6.jar
|  ├── aspectjweaver-1.9.5.jar
|  ├── chaconne-spring-boot-starter-1.0-RC2.jar
|  ├── checker-compat-qual-2.5.5.jar
|  ├── classmate-1.5.1.jar
|  ├── commons-codec-1.13.jar
|  ├── commons-io-2.6.jar
|  ├── ...
└── logs
   └── atlantis
```
* reference configuration：
  The Web UI of greenfinger-console uses freemaker. At present, there are two configuration files, application.properties and application-dev.properties

  Following is the default configuration of greenfinger-console (which can be extended according to the actual situation). The application.properties configuration mainly stores global configurations:
``` properties
spring.application.name=greenfinger-console
spring.application.cluster.name=greenfinger-console-cluster

#Freemarker Configuration
spring.freemarker.enabled=true
spring.freemarker.suffix=.ftl
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.template-loader-path=classpath:/META-INF/templates/
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.setting.number_format=#
spring.freemarker.setting.locale=en_US
spring.freemarker.setting.url_escaping_charset=UTF-8

server.port=21212
server.servlet.context-path=/atlantis/greenfinger

spring.profiles.active=dev
```
application-dev.properties：
``` properties
#Jdbc Configuration
atlantis.framework.greenfinger.datasource.jdbcUrl=jdbc:postgresql://localhost:5432/db_webanchor
atlantis.framework.greenfinger.datasource.username=fengy
atlantis.framework.greenfinger.datasource.password=123456
atlantis.framework.greenfinger.datasource.driverClassName=org.postgresql.Driver

#Redis Configuration
atlantis.framework.redis.host=localhost
atlantis.framework.redis.port=6379
atlantis.framework.redis.password=123456
atlantis.framework.redis.database=0

spring.redis.messager.pubsub.channel=greenfinger-console-messager-pubsub

#Vortex Configuration
atlantis.framework.vortex.bufferzone.collectionName=MyGarden
atlantis.framework.vortex.bufferzone.pullSize=100

#Elasticsearch Configuration
spring.data.elasticsearch.cluster-name=es
spring.data.elasticsearch.cluster-nodes=localhost:9300
spring.data.elasticsearch.repositories.enabled=true
spring.data.elasticsearch.properties.transport.tcp.connect_timeout=60s

#Chaconne Configuration
#atlantis.framework.chaconne.producer.location=http://localhost:6543
#atlantis.framework.chaconne.mail.host=smtp.your_company.com
#atlantis.framework.chaconne.mail.username=your_email@your_company.com
#atlantis.framework.chaconne.mail.password=0123456789
#atlantis.framework.chaconne.mail.default-encoding=UTF-8

webcrawler.pagesource.selenium.webdriverExecutionPath=D:\\software\\chromedriver_win32\\chromedriver.exe

logging.level.indi.atlantis.framework.greenfinger=INFO

```
**Description：**
application-dev.properties configures some external resources that Greenfinger framework depends on. By default, Greenfinger  application stores the crawled link information in PostgreSQL. Of course, you can also store it in other places (such as NoSQL database or other file system). As mentioned earlier, Greenfinger is an extension-oriented web spider, which provides rich APIs for extension, I will explain in detail in the next article about the implementation principle of Greenfinger later.

The information in the above configuration such database location, redis location should be modified according to your own situation.

### How to customize your application？
-------------------------
**Step1:  add maven**:
``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
	<artifactId>greenfinger-spring-boot-starter</artifactId>
	<version>1.0-RC3</version>
</dependency>
```
**Step2: Reference Configuration**:
``` java
@EnableGreenFingerServer
@SpringBootApplication
public class GreenFingerServerConsoleMain {

	public static void main(String[] args) {
		SpringApplication.run(GreenFingerServerConsoleMain.class, args);
	}
}
```
**Step3: Reference Configuration**:
``` properties
spring.application.name=cool-crawler
spring.application.cluster.name=cool-crawler-cluster

#Jdbc Configuration
spring.datasource.jdbcUrl=jdbc:postgresql://localhost:5432/db_webanchor
spring.datasource.username=fengy
spring.datasource.password=123456
spring.datasource.driverClassName=org.postgresql.Driver

#Redis Configuration
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.password=123456
spring.redis.messager.pubsub.channel=greenfinger-console-messager-pubsub

#Vortex Configuration
atlantis.framework.vortex.bufferzone.collectionName=MyGarden
atlantis.framework.vortex.bufferzone.pullSize=100

#Elasticsearch Configuration
spring.data.elasticsearch.cluster-name=es
spring.data.elasticsearch.cluster-nodes=localhost:9300
spring.data.elasticsearch.repositories.enabled=true
spring.data.elasticsearch.properties.transport.tcp.connect_timeout=60s

#Chaconne Configuration
#atlantis.framework.chaconne.producer.location=http://localhost:6543
#atlantis.framework.chaconne.mail.host=smtp.your_company.com
#atlantis.framework.chaconne.mail.username=your_email@your_company.com
#atlantis.framework.chaconne.mail.password=0123456789
#atlantis.framework.chaconne.mail.default-encoding=UTF-8

#webcrawler.pagesource.selenium.webdriverExecutionPath=D:\\software\\chromedriver_win32\\chromedriver.exe

#logging.level.indi.atlantis.framework.greenfinger=INFO
```
You can modify the above configuration according to your own situation.

###  Greenfinger-Console Using Guide：
-------------------------
* 首先，说一下**目录**和**资源**概念：
在Greenfinger框架中，对于每一个目标网站（待爬取的网站），都被称之为**Catalog(目录)**，而对于每一个从它上面爬取下来的URL, 则代表为一个**Resource(资源)**
* 目前Greenfinger的Web版界面还在持续改进中，所以看上去比较朴素

​        Home Page：http://localhost:21212/atlantis/greenfinger/catalog/

* **查看目录列表**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-de1d3f7bfc91edde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**操作说明：**
  - 【Edit】 编辑目录
  -  【Delete】删除目录（包括目录下的资源和索引）
  - 【Clean】 清理目录（包括目录下的资源和索引，但目录还在，版本号归 0）
  - 【Rebuild】重构目录（即开启一个爬虫，重新爬取该目录，并建索引，版本号递增）
  -  【Update】更新目录（即开启一个爬虫，接着最近的一次爬取地址继续爬取和更新目录，并建索引，版本号不变）
*当爬虫运行的时候，你还可以：*
  - 【Stop】停止爬虫运行
  - 【Realtime】监控爬虫运行统计等

* **新建或保存目录：**
![image.png](/assets/images/greenfinger/1.png)
**说明：**
+ Name: 目录名称
+ Cat: 分类名称
+ URL: 初始地址
+ Page Encoding: 页面编码
+ Path Pattern: URL匹配模式，可以多个，逗号分隔
+ Excluded Path Pattern: 排除的URL匹配模式，可以多个，逗号分隔
+ Max Fetch Size: 最大爬取的链接数量（默认100000）
+ Duration: 爬虫的运行时间，输入毫秒值（默认20分钟），即超过此时间，爬虫就自动结束爬取工作

* **监控爬虫运行情况：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-bfb4c25f909a9751.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **爬虫一边在爬，你也可以实时地用关键字搜索：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-dd71cb01cface83c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* **不输关键字，则查询全部：**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-f9a8e1d36bc5b837.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后，由于Greenfinger框架复杂度较高，它综合运用了微服务分布式协作框架[tridenter](https://www.jianshu.com/p/fdd0e962d7a4)、分布式流式处理框架[vortex](https://www.jianshu.com/p/c6eb9a1e850c)、分布式任务调度框架[chaconne](https://www.jianshu.com/p/9895bab1b682) 3个框架的核心内容，所以本篇主要讲述的是如何操作Greenfinger-Console界面来创建爬虫任务，运行爬虫，最后关键字搜索爬取到的内容，限于篇幅，后面会着重写一篇关于Greenfinger实现原理的文章。

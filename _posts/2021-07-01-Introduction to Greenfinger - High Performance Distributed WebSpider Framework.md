---
layout: post
title: Introduction to Greenfinger - A High Performance Distributed Web Spider Framework
date: 2021-07-01 08:30:00.000000000 +09:00
---

**Greenfinger** is a high-performance, extension-oriented distributed web spider framework written in Java. It is based on the SpringBoot framework. Through some parameters settings, it can easily build a distributed web spider microservice and even a microservice cluster. In addition, **Greenfinger** framework also provides rich APIs to customize your application system.

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

  - **Redis** is used to save cluster information
  - **PostgreSQL** is used to save the crawled URL information
  - **Elasticsearch** is used to create indexes and search indexes


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
     The Web UI of Greenfinger application, an independent SpringBoot application with its own management interface, which can add, modify, start and stop web spider tasks, and provide a search interface for real-time query
  + **greenfinger-spring-boot-starter**：
     Core jar of Greenfinger application, which implements all the above framework features, provides open APIs such as spider program management and search and even customize your own system with these APIs.

###  Install Greenfinger Console：
----------------------------
  1. enter directory: greenfinger-console 
  2. execute maven with <code>mvn clean install</code>
  3. After successful execution, there will be a directory named 'run'. Move this directory to your working directory (the directory specified by yourself)
  4. run jar with <code>java -jar greenfinger-console-1.0-RC2.jar --spring.config.location=config/ </code> (Just reference)

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
  

Here are configuration files, <code>application.properties</code> and <code>application-dev.properties</code>, which can be extended according to the actual situation. 

  - <code>application.properties</code> mainly configure some global settings:

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
Greenfinger Console uses freemarker as Web UI by default. 

<code>application-dev.properties</code> mainly configure some settings that might be changed:

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
<code>application-dev.properties</code> configures some external resources that Greenfinger framework depends on. By default, Greenfinger  application stores the crawled link information in PostgreSQL. Of course, you can also store it in other places (such as NoSQL database or other file system). As mentioned earlier, Greenfinger is an extension-oriented web spider framework, which provides rich APIs for extension, I will explain in detail in the next article about the implementation principle of Greenfinger later.

The information in the above configuration such as database location, redis location should be modified according to your own situation.

### How to customize your application？
-------------------------
**Step1**:  add dependency in your pom.xml:

``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
	<artifactId>greenfinger-spring-boot-starter</artifactId>
	<version>1.0-RC3</version>
</dependency>
```
**Step2**: add <code>@EnableGreenFingerServer</code> on the main.java:

``` java
@EnableGreenFingerServer
@SpringBootApplication
public class GreenFingerServerConsoleMain {

	public static void main(String[] args) {
		SpringApplication.run(GreenFingerServerConsoleMain.class, args);
	}
}
```
**Step3**:  modify reference configuration if required:

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

###  Greenfinger Console Using Guide：
-------------------------
Understanding the meaning of **Catalog** and **Resource:**
* **Catalog**: a website that will be crawled
* *Resource**: a URL that is crawled from a catalog

Home Page：http://localhost:21212/atlantis/greenfinger/catalog/

* **Catalog List**
  ![image.png](/assets/images/greenfinger/1.png)
  **Description：**
  
- 【Edit】 Edit Catalog
  
- 【Delete】Delete directory (including resources and indexes under the directory)
  
- 【Clean】 Clean up the directory (including the resources and indexes under the directory, but the directory is still there, and the version number is set to 0)
  
- 【Rebuild】Reconstruct the directory (start a web spider, crawl the directory again, build an index, and increase the version number)
  
- 【Update】Update the directory (start a web spider, then continue to crawl and update the directory from the latest resource, and build an index, with the version number unchanged)
  
  *When the crawler is running, you can also:*
  
- 【Stop】Stop running a web spider
  
- 【Realtime】Watching real-time statistics when a web spider works
  
* **Save Catalog：**
  ![image.png](/assets/images/greenfinger/2.png)
  **说明：**

  + **Name**: catalog name
  + **Cat**: category of catalogs
  + **URL**: the started url      (e.g.  http://www.example.com)
  + **Page Encoding**: encoding of page content    (e.g.  UTF-8, ISO-8859-1)
  + **Path Pattern**: URL matching pattern, more patterns are separated by ","           (e.g. http://www/example.com/foo/\*\*, http://www/example.com/bar/\*\*)
  + **Excluded Path Pattern**: Excluded URL matching pattern，more patterns are separated by ","
  + **Max Fetch Size**: Maximum number of links crawled（100000 by default）
  + **Duration**: The running time (milliseconds) of a web spider  (20 minutes by default), that means  beyond this time, the web spider will automatically end the crawling work.

* **Observe the statistics of a running web spider：**
![image.png](/assets/images/greenfinger/3.png)
* **While the crawler is crawling, you can also search with keywords in real-time:**
![image.png](/assets/images/greenfinger/4.png)
* **If no keyword is entered, all items will be queried：**
![image.png](/assets/images/greenfinger/5.png)

Finally, due to the high complexity of Greenfinger Framework, it comprehensively uses the core functions of three frameworks: Micro service distributed collaboration framework [tridenter](https://paganini2008.github.io/2021/06/Introduction-to-Tridenter-Microservice-Distribution-Collaboration-Framework/), distributed streaming processing framework [vortex](https://paganini2008.github.io/2021/06/Introduction-to-Vortex-Distributed-Streaming-Computing-Framework/) and distributed task scheduling framework [chaconne](https://paganini2008.github.io/2021/06/Introduction-to-Chaconne-Distributed-Task-Scheduling-Framework/). Therefore, this paper mainly describes how to operate Greenfinger Console Web UI to create web spider tasks, run them, and search content by keyword. I will write an article about the implementation principle of Greenfinger Framework later.


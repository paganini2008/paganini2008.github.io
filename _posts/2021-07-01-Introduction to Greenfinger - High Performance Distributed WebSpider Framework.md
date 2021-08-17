---
layout: post
title: Introduction to Greenfinger - High Performance Distributed WebSpider Framework
date: 2021-07-01 08:30:00.000000000 +09:00
---

**Greenfinger**是一款用Java编写的，高性能的，面向扩展的分布式网络爬虫框架，它基于SpringBoot框架，通过一些配置参数，就可以轻松地搭建一个分布式网络爬虫微服务并且可以组建集群。此外，**Greenfinger**框架还提供了大量丰富的API去定制你的应用系统。

### 框架特性
------------------------------
1. 完美兼容 SpringBoot2.2.0(or later)
2. 支持通用型和垂直型爬虫
3. 采用深度优先爬取策略
4. 设计成多进程高可用的爬虫架构，支持动态水平扩展和负载均衡
5. 内置多种负载均衡算法或自定义负载均衡算法
6. 支持全量索引和增量索引
7. 支持定时任务来更新索引
8. 支持多种主流的Http客户端解析技术
9. 支持亿级URL去重
10. 内置多种条件中断策略或自定义条件中断策略
11. 多版本索引查询机制

### 兼容性
-----------------------------------
1. jdk8 (or later)
2. SpringBoot Framework 2.2.x (or later)
3. Redis 3.x (or later)
4. PostgreSQL 9.x (or later)
5. ElasticSearch 6.x (or later)
**说明：**
   - Redis用来存取集群信息
   - PostgreSQL用来存取爬取到的URL信息
   - ElasticSearch用来创建索引和提供检索功能


### 如何安装
-----------------------------
* Git地址：
  https://github.com/paganini2008/greenfinger.git
* 目录结构：
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
* 软件说明：
  + **greenfinger-console**：
     Greenfinger的Web版，独立的SpringBoot应用程序, 自带管理界面，可以新增、修改、启动、停止爬虫任务等, 并提供搜索界面实时查询
  + **greenfinger-spring-boot-starter**：
     Greenfinger 核心jar，实现了上述所有的框架特性，对外提供了爬虫管理和搜索等Rest API, 引入jar包，可以定制你自己的系统

#### 安装 greenfinger-console：
----------------------------
  **Step1:** 进入greenfinger-console目录
  **Step2:** 执行命令：mvn clean install
  **Step3:** 执行成功后会多出一个目录run, 把此目录移动到你的工作目录（自己指定的目录）下即可
  **Step4:** 运行jar: java -jar greenfinger-console-1.0-RC2.jar --spring.config.location=config/  (命令仅供参考)
* 生成的run目录结构：
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
* 参考配置：
greenfinger-console界面用的是freemarker，目前有两个配置文件，application.properties和application-dev.properties

下面是greenfinger-console的默认配置（可以根据实际情况扩展）：
application.properties 配置，主要存放一下全局配置：
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
application-dev.properties 配置：
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
**说明：**
application-dev.properties配置了greenfinger依赖的一些外部资源，默认情况，greenfinger将爬取到的链接信息存储在PostgesSQL，当然，你也可以存储在其他地方（比如Nosql数据库或文件格式），前面说过，greenfinger是面向扩展的网络爬虫，它提供了丰富的API去做扩展，我会在后面关于讲述greenfinger实现原理一文中详细讲解。
上述配置中的地址信息等，你要根据自己的情况做修改
**注意**：在jdk8下，启动greenfinger-console可能会报错（提示你jdk版本过低），所以你可能需要用jdk11的环境，本人在jdk11下可以运行成功，其他版本暂未试过。

#### 如何自定义你的爬虫应用程序？
-------------------------
Step1:  **添加 maven**:
``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
	<artifactId>greenfinger-spring-boot-starter</artifactId>
	<version>1.0-RC3</version>
</dependency>
```
Step2: **参考代码**:
``` java
@EnableGreenFingerServer
@SpringBootApplication
public class GreenFingerServerConsoleMain {

	public static void main(String[] args) {
		SpringApplication.run(GreenFingerServerConsoleMain.class, args);
	}
}
```
Step3: **参考配置**:
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
上述配置你可以根据自己的情况做修改

###  Greenfinger-Console 使用介绍：
-------------------------
* 首先，说一下**目录**和**资源**概念：
在Greenfinger框架中，对于每一个目标网站（待爬取的网站），都被称之为**Catalog(目录)**，而对于每一个从它上面爬取下来的URL, 则代表为一个**Resource(资源)**
* 目前Greenfinger的Web版界面还在持续改进中，所以看上去比较朴素

输入首页地址：http://localhost:21212/atlantis/greenfinger/catalog/

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
![image.png](https://upload-images.jianshu.io/upload_images/26217505-be889fe4cccc5ed6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
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

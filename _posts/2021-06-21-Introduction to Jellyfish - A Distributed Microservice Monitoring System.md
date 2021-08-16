---
layout: post
title: Introduction to Jellyfish - A Distributed Microservice Monitoring System
date: 2021-06-21 08:30:00.000000000 +09:00
---

Jellyfish是一款轻量级的, 用Java写的, 分布式微服务实时监控系统，可无缝对接Spring Boot或Spring Cloud工程

**Jellyfish提供的监控功能主要分为两个部分：**
1. 应用程序日志统一收集和查询
2. 统计和监控SpringBoot应用程序HTTP接口3个指标：请求耗时，错误率，并发度

**Jellyfish部署方式**
Jellyfish分为服务端和agent端
**服务端**通常是一个独立的SpringBoot应用或集群，集群模式是通过另一个微服务分布式协作框架[tridenter](https://www.jianshu.com/writer#/notebooks/49992486/notes/88482536)实现的，Jellyfish服务端还需要部署Redis和ElasticSearch
**Agent端** 通常是加入了jellyfish相关jar包的另一组SpringBoot应用或集群，它们向服务端实时发送数据包。

**Jellyfish的特性：**
1. Jellyfish服务端基于流式计算框架vortex（底层是Netty4），低延迟，高并发，且支持TCP和HTTP两种传输协议
2. Jellyfish服务端的网络通信层依赖vortex框架，支持动态水平扩展，支持数据最终一致性
3. Jellyfish的应用接口统计是准实时的，纯内存计算，默认的统计时间窗口为1分钟（可自定义），但统计的接口数量变多可能会影响实时性，经测试，最大延迟保持在1分钟内，平均延迟在秒级。

**Jellyfish系列一共有4个子工程：**
1. jellyfish-console  
    Jellyfish Web控制台，可独立运行，通常作为服务端，当然你也可以自定义服务 端，通过使用注解@EnableJellyfishServer
2. jellyfish-http-spring-boot-starter  
    jar包，Jellyfish监控应用程序接口进行指标统计的Agent程序包
3. jellyfish-slf4j   
    jar包，Jellyfish对接slf4j（目前只实现了logback对接），统一应用收集日志的 
   agent程序包
4. jellyfish-spring-boot-starter  Jellyfish核心jar包, 实现了全部的核心功能

### Install
``` xml
<dependency>
      <artifactId>jellyfish-spring-boot-starter</artifactId>
     <groupId>com.github.paganini2008.atlantis</groupId>
     <version>1.0-RC3</version>
</dependency>
```

###  Jellyfish-console使用说明
**如何对接slf4j, 统一收集日志：**

**你的应用程序需要：**
1. pom.xml 添加依赖
``` xml
<dependency>
      <artifactId>jellyfish-slf4j</artifactId>
     <groupId>com.github.paganini2008.atlantis</groupId>
     <version>1.0-RC3</version>
</dependency>
```
2. 修改logback.xml
``` xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
<appender name="logTracker" class="indi.atlantis.framework.jellyfish.slf4j.logback.HttpTransportClientAppender">
<applicationName>tester5</applicationName> <!-- 修改成你的applicationName -->
<brokerUrl>http://192.168.159.1:10010</brokerUrl> <!-- 修改Jellyfish console地址 -->
</appender>
<!-- omit other configuration -->
<root level="info">
<appender-ref ref="STDOUT" />
<appender-ref ref="INFO" />
<appender-ref ref="ERROR" />
<appender-ref ref="logTracker" />  <!-- 引用你定义的appender -->
</root>
</configuration>
```
目前只支持logback, 未来会支持更多

3. 先启动Jellyfish-console
启动命令：java -jar jellyfish-console-1.0-RC1.jar     
默认端口：6100   地址：http://localhost:6100/jellyfish/log/
![image.png](https://upload-images.jianshu.io/upload_images/26217505-f2dbd84436a2eac3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 再启动你的应用程序，此时日志会源源不断的流入Jellyfish Console：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-078a6fac7e627493.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

说明：
jellyfish服务端从agent端接受来的每行日志数据会保存在es里。





**如何对接Spring Boot或Spring Cloud项目，统计HTTP接口指标？**
**你的应用需要：**
1. 修改你的pom.xml, 添加：
``` xml
<dependency>
<artifactId>jellyfish-http-spring-boot-starter</artifactId>
<groupId>com.github.paganini2008.atlantis</groupId>
<version>1.0-RC3</version>
</dependency>
```

2. 在application.properties加入：
``` properties
atlantis.framework.jellyfish.brokerUrl=http://localhost:6100  # Jellyfish console地址
```
3. 使用压测工具测试某一个接口，如图：
接口列表：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-cfea866870129232.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击【View】，会打开一直统计详情页，页面比较长
**首先**在在顶部，会看到接口概览和一些统计
![image.png](https://upload-images.jianshu.io/upload_images/26217505-3d2844b7a9a8d696.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**接着**会看到一些仪表盘：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-8fee847532ef9dd4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上图中仪表盘分别显示当前的并发度，QPS, 平均响应时间，接着是两个饼图，分别监控http状态码和接口执行结果
**然后**往下翻，会看到几个大图，都是按分钟统计的
**响应时间统计和QPS统计**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-b10103c83f7f0149.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后显示的是**接口调用次数统计**和**Http状态码统计**
![image.png](https://upload-images.jianshu.io/upload_images/26217505-5883d2f4326970d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


按F12，查看返回的json数据：
``` json
{
    "msg": "ok",
    "data": {
        "20:05:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623585900737
            }
        },
        "20:06:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623585960737
            }
        },
        "20:07:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586020737
            }
        },
        "20:08:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586080737
            }
        },
        "20:09:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586140737
            }
        },
        "20:10:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586200737
            }
        },
        "20:11:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586260737
            }
        },
        "20:12:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586320737
            }
        },
        "20:13:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586380737
            }
        },
        "20:14:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586440737
            }
        },
        "20:15:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586500737
            }
        },
        "20:16:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586560737
            }
        },
        "20:17:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586620737
            }
        },
        "20:18:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586680737
            }
        },
        "20:19:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586740737
            }
        },
        "20:20:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586800737
            }
        },
        "20:21:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586860737
            }
        },
        "20:22:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586920737
            }
        },
        "20:23:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623586980737
            }
        },
        "20:24:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587040737
            }
        },
        "20:25:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587100737
            }
        },
        "20:26:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587160737
            }
        },
        "20:27:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587220737
            }
        },
        "20:28:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587280737
            }
        },
        "20:29:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587340737
            }
        },
        "20:30:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587400737
            }
        },
        "20:31:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587460737
            }
        },
        "20:32:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587520737
            }
        },
        "20:33:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587580737
            }
        },
        "20:34:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587640737
            }
        },
        "20:35:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587700737
            }
        },
        "20:36:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587760737
            }
        },
        "20:37:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587820737
            }
        },
        "20:38:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587880737
            }
        },
        "20:39:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623587940737
            }
        },
        "20:40:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588000737
            }
        },
        "20:41:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588060737
            }
        },
        "20:42:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588120737
            }
        },
        "20:43:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588180737
            }
        },
        "20:44:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588240737
            }
        },
        "20:45:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588300737
            }
        },
        "20:46:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588360737
            }
        },
        "20:47:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588420737
            }
        },
        "20:48:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588480737
            }
        },
        "20:49:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588540737
            }
        },
        "20:50:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588600737
            }
        },
        "20:51:00": {
            "rt": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1623588660737
            }
        },
        "20:52:00": {
            "rt": {
                "middleValue": 2158,
                "count": 68,
                "highestValue": 4941,
                "lowestValue": 1,
                "timestamp": 1623588778333
            }
        },
        "20:53:00": {
            "rt": {
                "middleValue": 2252,
                "count": 1815,
                "highestValue": 4996,
                "lowestValue": 0,
                "timestamp": 1623588839757
            }
        },
        "20:54:00": {
            "rt": {
                "middleValue": 2233,
                "count": 2680,
                "highestValue": 5000,
                "lowestValue": 0,
                "timestamp": 1623588899641
            }
        },
        "20:55:00": {
            "rt": {
                "middleValue": 2306,
                "count": 2605,
                "highestValue": 4994,
                "lowestValue": 0,
                "timestamp": 1623588959700
            }
        },
        "20:56:00": {
            "rt": {
                "middleValue": 2224,
                "count": 2669,
                "highestValue": 5422,
                "lowestValue": 0,
                "timestamp": 1623589019949
            }
        },
        "20:57:00": {
            "rt": {
                "middleValue": 2239,
                "count": 2678,
                "highestValue": 4997,
                "lowestValue": 0,
                "timestamp": 1623589079640
            }
        },
        "20:58:00": {
            "rt": {
                "middleValue": 2233,
                "count": 2678,
                "highestValue": 4998,
                "lowestValue": 0,
                "timestamp": 1623589139980
            }
        },
        "20:59:00": {
            "rt": {
                "middleValue": 2305,
                "count": 2601,
                "highestValue": 4998,
                "lowestValue": 0,
                "timestamp": 1623589199745
            }
        },
        "21:00:00": {
            "rt": {
                "middleValue": 2227,
                "count": 2678,
                "highestValue": 5000,
                "lowestValue": 0,
                "timestamp": 1623589259579
            }
        },
        "21:01:00": {
            "rt": {
                "middleValue": 2243,
                "count": 2677,
                "highestValue": 4998,
                "lowestValue": 0,
                "timestamp": 1623589319569
            }
        },
        "21:02:00": {
            "rt": {
                "middleValue": 2255,
                "count": 2656,
                "highestValue": 5000,
                "lowestValue": 0,
                "timestamp": 1623589378556
            }
        },
        "21:03:00": {
            "rt": {
                "middleValue": 2288,
                "count": 2616,
                "highestValue": 5000,
                "lowestValue": 0,
                "timestamp": 1623589439666
            }
        },
        "21:04:00": {
            "rt": {
                "middleValue": 1272,
                "count": 152,
                "highestValue": 4271,
                "lowestValue": 0,
                "timestamp": 1623589441456
            }
        }
    },
    "success": true,
    "statusCode": 200
}
```
Jellyfish接口统计部分调用了vortex的metric模块的sdk,，所以是基于时间序列的，默认统计时间窗口是1分钟，滚动保存前60条数据，历史数据默认不落地，但可以通过实现MetricEvictionHandler和MetricSequencerFactory接口进行扩展，参考[vortex metrics](https://www.jianshu.com/writer#/notebooks/49992486/notes/88751021)

最后，Jellyfish服务端的参考配置：
``` properties
spring.application.name=jellyfish-console
spring.application.cluster.name=jellyfish-console-cluster

#management.endpoints.web.exposure.include=*
#management.endpoint.health.show-details=always
#management.security.enabled=false
#spring.boot.admin.client.url=http://localhost:10081

atlantis.framework.vortex.bufferzone.collectionName=jellyfish
atlantis.framework.vortex.bufferzone.pullSize=100

#Elasticsearch Configuration
spring.data.elasticsearch.cluster-name=es
spring.data.elasticsearch.cluster-nodes=localhost:9300
spring.data.elasticsearch.repositories.enabled=true
spring.data.elasticsearch.properties.transport.tcp.connect_timeout=60s

#Redis Configuration
atlantis.framework.redis.host=localhost
atlantis.framework.redis.port=6379
atlantis.framework.redis.password=123456
atlantis.framework.redis.database=0

#Freemarker Configuration
spring.freemarker.enabled=true
spring.freemarker.suffix=.ftl
spring.freemarker.cache=false
spring.freemarker.charset=UTF-8
spring.freemarker.template-loader-path=classpath:/META-INF/templates/
spring.freemarker.expose-request-attributes=true
spring.freemarker.expose-session-attributes=true
spring.freemarker.setting.number_format=#
spring.freemarker.setting.url_escaping_charset=UTF-8
```
具体更详细的参数设置请参考：https://github.com/paganini2008/jellyfish

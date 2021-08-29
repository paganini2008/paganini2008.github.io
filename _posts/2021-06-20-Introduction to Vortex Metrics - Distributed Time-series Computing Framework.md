---
layout: post
title: Vortex Metrics - A High Performance Distributed Time-series Computing Framework
date: 2021-06-20 08:30:00.000000000 +09:00
---

**vortex metrics**是一款用Java写的，轻量级的，高性能的分布式时序计算框架。
**vortex metrics**框架源自作者的另一个分布式流式计算框架[vortex](https://www.jianshu.com/p/c6eb9a1e850c)，可以看作是[vortex](https://www.jianshu.com/p/c6eb9a1e850c)关于时序计算场景的一个具体实现

### 框架特性：
------------------------------
1. 去中心化模式管理应用集群
2. 支持应用动态水平扩展
3. 并发性能极高且稳定(网络层可适配Netty4,Mina,Grizzly等框架）
4. CP分布式架构，支持数据的最终一致性
5. 数据存储在内存中，访问快捷。

### 数据存储：
--------------------------
**vortex metrics**本身不提供外部存储（但可以扩展去实现），相比另一个时序数据库OpenTSDB，vortex metrics更偏向实时计算，将延迟尽可能保持在秒级，但由于是内存计算，计算结果是存储在内存中的，有限的存储空间让它并不适用于存储大量指标的业务场景。但同时，对于一个指标的窗口数据，若需要保留的时间跨度较长时，vortex metrics提供了定制接口，可以保存到外部存储，比如数据库或缓存，但这需要开发人员做一些额外的开发工作。此外，**vortex metrics**提供了一个简单的Web查询界面来查看效果统计，并且这个Web项目也正在持续改进中。

实际上，**vortex metrics**的功能是很单一的，因为它只做一件事，**那就是根据客户端源源不断的上报数据实时地计算并输出单位时间窗口内的统计数据**，记住这点很重要。

### 安装
---------------------------
**Maven：**
``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
    <artifactId>vortex-spring-boot-starter</artifactId>
	<version>1.0-RC3</version>
</dependency>
```
**兼容性：**
* Jdk 1.8 (or later)
* SpringBoot 2.2.0 (or later)
* Redis 3.0 (or later)
* MySQL 5.0 (or later)

**vortex metrics**是一个基于SpringBoot的Web应用程序，前面说过，它依赖vortex框架, 而vortex是通过另一个微服务分布式协调框架[tridenter](https://www.jianshu.com/p/fdd0e962d7a4)来实现集群模式的，并支持动态水平扩展。

**参考配置**
``` properties
# 集群配置
spring.application.name=vortex-metrics
spring.application.cluster.name=vortex-metrics-cluster

# vortex配置
atlantis.framework.vortex.bufferzone.collectionName=metric  # 缓冲区集合名称
atlantis.framework.vortex.bufferzone.pullSize=1000  # 缓冲区每次拉取的消息数量
atlantis.framework.vortex.processor.threads=200  # 数据消费者线程数
```
服务的默认端口为6150，输入地址：http://localhost:6150/metric/  可以看到这个界面
![image.png](https://upload-images.jianshu.io/upload_images/26217505-cd03631d9403f5be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**vortex metrics**将server数据接收端和监控界面做在同一个应用中的，高并发场景下，可能会出现数据计算完成并已同步，但界面会滞后更新的情况。

### HTTP API说明
-----------------------------
**vortex metrics**目前只提供了2个接口：
1. 上报数据接口：
POST  http://localhost:6150/metrics/sequence/{dataType}
比如，要上报小汽车的时速，请求体示例：
``` javascript
{
"name": "car",
"metric": "speed",
"value": 120,
"timestamp": 1613200609281
}
```
参数说明：
| 参数名    | 类型   | 描述                                    | 举例          |
| --------- | ------ | --------------------------------------- | ------------- |
| dateType  | 字符串 | 数据类型，取值范围：bigint,numeric,bool | bigint        |
| name      | 字符串 | 应用名称                                | car           |
| metric    | 字符串 | 上报指标名称                            | speed         |
| value     | 数值   | 上报数值                                | 120           |
| timestamp | 长整型 | 上报时间戳                              | 1613200609281 |

2. 查看效果统计数据接口：
GET  http://localhost:6150/metrics/sequence/{dataType}/{name}/{metric}
参数同上

这里以上面上报小汽车的时速为例：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-d930c18256a10d67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对应的响应体：
``` javascript
{
    "dataType": "bigint",
    "name": "car",
    "metric": "speed",
    "data": {
        "07:13:00": {
            "speed": {
                "middleValue": 198,
                "count": 2918,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627686839790
            }
        },
        "07:14:00": {
            "speed": {
                "middleValue": 199,
                "count": 10890,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627686899924
            }
        },
        "07:15:00": {
            "speed": {
                "middleValue": 198,
                "count": 10917,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627686959928
            }
        },
        "07:16:00": {
            "speed": {
                "middleValue": 199,
                "count": 10903,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687019591
            }
        },
        "07:17:00": {
            "speed": {
                "middleValue": 199,
                "count": 10841,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687079709
            }
        },
        "07:18:00": {
            "speed": {
                "middleValue": 199,
                "count": 10901,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687139969
            }
        },
        "07:19:00": {
            "speed": {
                "middleValue": 199,
                "count": 10878,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687199267
            }
        },
        "07:20:00": {
            "speed": {
                "middleValue": 199,
                "count": 10887,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687259118
            }
        },
        "07:21:00": {
            "speed": {
                "middleValue": 199,
                "count": 10911,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687319918
            }
        },
        "07:22:00": {
            "speed": {
                "middleValue": 199,
                "count": 10866,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687379834
            }
        },
        "07:23:00": {
            "speed": {
                "middleValue": 199,
                "count": 10910,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687439577
            }
        },
        "07:24:00": {
            "speed": {
                "middleValue": 200,
                "count": 10951,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687499147
            }
        },
        "07:25:00": {
            "speed": {
                "middleValue": 200,
                "count": 10980,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687559897
            }
        },
        "07:26:00": {
            "speed": {
                "middleValue": 199,
                "count": 7811,
                "highestValue": 299,
                "lowestValue": 100,
                "timestamp": 1627687602546
            }
        },
        "07:27:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687620790
            }
        },
        "07:28:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687680790
            }
        },
        "07:29:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687740790
            }
        },
        "07:30:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687800790
            }
        },
        "07:31:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687860790
            }
        },
        "07:32:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687920790
            }
        },
        "07:33:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627687980790
            }
        },
        "07:34:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688040790
            }
        },
        "07:35:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688100790
            }
        },
        "07:36:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688160790
            }
        },
        "07:37:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688220790
            }
        },
        "07:38:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688280790
            }
        },
        "07:39:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688340790
            }
        },
        "07:40:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688400790
            }
        },
        "07:41:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688460790
            }
        },
        "07:42:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688520790
            }
        },
        "07:43:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688580790
            }
        },
        "07:44:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688640790
            }
        },
        "07:45:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688700790
            }
        },
        "07:46:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688760790
            }
        },
        "07:47:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688820790
            }
        },
        "07:48:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688880790
            }
        },
        "07:49:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627688940790
            }
        },
        "07:50:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689000790
            }
        },
        "07:51:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689060790
            }
        },
        "07:52:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689120790
            }
        },
        "07:53:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689180790
            }
        },
        "07:54:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689240790
            }
        },
        "07:55:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689300790
            }
        },
        "07:56:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689360790
            }
        },
        "07:57:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689420790
            }
        },
        "07:58:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689480790
            }
        },
        "07:59:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689540790
            }
        },
        "08:00:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689600790
            }
        },
        "08:01:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689660790
            }
        },
        "08:02:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689720790
            }
        },
        "08:03:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689780790
            }
        },
        "08:04:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689840790
            }
        },
        "08:05:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689900790
            }
        },
        "08:06:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627689960790
            }
        },
        "08:07:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690020790
            }
        },
        "08:08:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690080790
            }
        },
        "08:09:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690140790
            }
        },
        "08:10:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690200790
            }
        },
        "08:11:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690260790
            }
        },
        "08:12:00": {
            "speed": {
                "middleValue": 0,
                "count": 0,
                "highestValue": 0,
                "lowestValue": 0,
                "timestamp": 1627690320790
            }
        }
    }
}
```
可以看到，响应体json中的data为时序数据，**vortex metrics**默认的统计窗口为1分钟，默认**滚动保留**前60条记录，也就是用户可以通过界面看到前60分钟的统计数据。其中，highestValue，lowestValue，middleValue分别代表最大值，最小值，平均值，count是 这1分钟内的数据条数，timestamp是时间戳，准确点讲，是这1分钟内最后一条数据的时间戳。
说明一下，vortex metrics默认的统计窗口，保留前多少条记录，可以通过配置文件设置的，但系统需要重启才能生效。

### 如何监控指标
-----------------------------
vortex metrics框架为了方便用户内置了3个不同数据类型的测试接口：
1. http://localhost:6150/metrics/test/bigint/{name}/{metric}
2. http://localhost:6150/metrics/test/numeric/{name}/{metric}
3. http://localhost:6150/metrics/test/bool/{name}/{metric}
都是GET方式，设置name和metric参数即可

这里还是以统计小汽车的平均速度为例，数据上报地址为：
http://localhost:6150/metrics/test/bigint/car/speed

**打开压测工具，这里以JMeter为例**
   * 设置线程组
![image.png](https://upload-images.jianshu.io/upload_images/26217505-ccf0d8735ea0c2ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
   * 设置HTTP请求
![image.png](https://upload-images.jianshu.io/upload_images/26217505-0f1c5fa5700eab82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
（关于Jmeter如何使用，大家可以自行百度，这里仅供测试）
  * 点击绿色箭头运行

**然后在界面中，location输入框里查询接口地址：**
    http://localhost:6150/metrics/sequence/bigint/car/speed
   点击search, 可以看到：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-308f67aeaafba12f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### vortex metrics集群如何保证数据一致性？
----------------------------------
vortex metrics集群内的应用节点通过内部网络通讯（默认Netty4实现）实现相互复制，保证数据的最终一致性，即每个节点的数据最终都是一致的，全量的。
比如现在再起2台服务，端口分别为6151和6152
这样的话，
http://localhost:6151/metrics/sequence/bigint/car/speed
http://localhost:6152/metrics/sequence/bigint/car/speed
http://localhost:6150/metrics/sequence/bigint/car/speed
这3个不同地址的接口返回的数据是应该是最终一致的
localhost:6151:
![image.png](https://upload-images.jianshu.io/upload_images/26217505-da3deace12add15e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
localhost:6152
![image.png](https://upload-images.jianshu.io/upload_images/26217505-9e08981f05c8c347.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图，应用启动之后加入到vortex metrics集群，应用间会互相同步数据，达到最终一致的。

然后现在对任一一台服务进行压测，比如端口为6152的服务
观察localhost:6150地址的接口数据变化：
![image.png](https://upload-images.jianshu.io/upload_images/26217505-aa1372c23d7c56f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
结果验证了vortex metrics集群中应用数据是双向同步的。

前面说过，vortex metrics是内存计算型的，即计算结果是驻留在内存中的，同步之后，多个应该同样是占据大量的内存，如果存在大量的指标数据，可能会有延迟，所以vortex metrics不适用于存储大量指标的运算场景，但是你可以部署多个vortex metrics集群来解决这个问题，目前经压测，单集群可以较好的支持1000个左右的指标数据的业务场景，高并发下延迟保持在1~5分钟内。

### 如何保存历史数据
-------------------------------
前面说到，vortex metrics默认的统计窗口为1分钟，滚动保留前60条记录，那如果你想保留之前的历史记录，需要另外做开发。
首先要实现接口：
``` java
public interface MetricEvictionHandler<I, T extends Metric<T>> {

	void onEldestMetricRemoval(I identifier, String metric, T metricUnit);

}
```
具体可参考：LoggingMetricEvictionHandler类（默认实现）
你还要实现接口：
``` java
public interface MetricSequencerFactory {

	GenericUserMetricSequencer<String, BigInt> getBigIntMetricSequencer();

	GenericUserMetricSequencer<String, Numeric> getNumericMetricSequencer();

	GenericUserMetricSequencer<String, Bool> getBoolMetricSequencer();

}
```
具体可参考：DefaultMetricSequencerFactory类（默认实现）
最后，附上vortex metrics的源码地址：
https://github.com/paganini2008/vortex.git
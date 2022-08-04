---
layout: post
title: Vortex Metrics - A High Performance Distributed time series computing framework
date: 2021-06-20 08:30:00.000000000 +09:00
---

# Vortex Metrics

## Distributed Time Series Computing Framework

### Description

Vortex Metrics is a lightweight distributed time series computing framework written in Java.

Vortex Metrics is a concrete implementation of distributed streaming computing framework vortex about time series computing scenarios

Compared with <code>OpenTSDB</code>, another time series database, Vortex Metrics is more inclined to real-time calculation, and keeps the delay in seconds as far as possible. However, because it is memory calculation, the calculation results are stored in memory, and the limited storage space makes it not suitable for business scenarios where a large number of indicators are stored. But at the same time, when the window data of an indicator needs to be retained for a long time span, vortex metrics provides a custom interface, which can be saved to external storage, such as database or cache, but it needs to do some extra development work. Also, Vortex Metrics provides a web query interface, which is in continuous improvement.

In fact,  Vortex Metrics is very simple, because it only does one thing, that is to calculate and output the statistical data in the time window in real-time according to the continuous reported data of the client.

## Feature

1. High Availability
2. High Throughput
3. Data eventual consistency
4. Real-time Statistics
5. Memory Storage

## Install

``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
    <artifactId>vortex-spring-boot-starter</artifactId>
	<version>1.0-RC1</version>
</dependency>
```

### Config
``` properties
# Cluster Configuration
spring.application.name=vortex-metrics
spring.application.cluster.name=vortex-metrics-cluster

# Vortex Configuration
atlantis.framework.vortex.bufferzone.collectionName=metric
atlantis.framework.vortex.bufferzone.pullSize=1000
atlantis.framework.vortex.processor.threads=200
```



## Run

1. Open browser and input location：http://localhost:6150/metric/

![image.png](/assets/images/vortex/metrics/index.png)



2. Put Data

Example
``` shell
curl -X POST -H "Content-Type: application/json" -d '{name: "car", "metric": "speed", "value": 120, "timestamp": 1613200609281}' 'http://localhost:6150/metrics/sequence/bigint'
```

http://localhost:6150/metrics/sequence/{dataType}

**dataType:**

*  bigint
*  numeric
*  bool


**Parameter：**

| parameter name    | type   | description                                    | example          |
| --------- | ------ | --------------------------------------- | ------------- |
| name      | string | application name                 | car           |
| metric    | string | metric name                  | speed         |
| value     | string | value of metric           | 120           |
| timestamp | string | timestamp of metric           | 1613200609281 |



3. Query Data

GET  http://localhost:6150/metrics/sequence/{dataType}/{name}/{metric}

Example

``` shell
curl -X GET 'http://localhost:6150/metrics/sequence/bigint/car/speed'
```

![image.png](/assets/images/vortex/metrics/search.png)


``` json
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
        
        ...
}
```




## Test API

* http://localhost:6150/metrics/test/bigint/{name}/{metric}
* http://localhost:6150/metrics/test/numeric/{name}/{metric}
* http://localhost:6150/metrics/test/bool/{name}/{metric}

Example:
http://localhost:6150/metrics/test/bigint/car/speed



Use JMeter

* Set Thread Group
![image.png](https://upload-images.jianshu.io/upload_images/26217505-ccf0d8735ea0c2ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Set HTTP Request Parameters
![image.png](https://upload-images.jianshu.io/upload_images/26217505-0f1c5fa5700eab82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* Run it
Input url:   http://localhost:6150/metrics/sequence/bigint/car/speed

![image.png](https://upload-images.jianshu.io/upload_images/26217505-308f67aeaafba12f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


Git repository:  https://github.com/paganini2008/vortex.git
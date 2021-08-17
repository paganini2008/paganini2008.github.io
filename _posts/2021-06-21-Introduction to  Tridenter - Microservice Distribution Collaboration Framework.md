---
layout: post
title: Introduction to  Tridenter - Microservice Distribution Collaboration Framework
date: 2021-06-21 08:30:00.000000000 +09:00
---

**tridenter**是一款基于SpringBoot框架开发的微服务分布式协作框架，它可以使多个独立的SpringBoot应用轻松快捷地组成一个集群，而不依赖外部的注册中心（比如SpringCloud Eureka等）。
**tridenter**框架首先提供了丰富的分布式应用集群管理API和工具，同时也提供了一套完整的微服务治理功能

**tridenter的特性：**
1. 采用去中心化的思想管理集群
2. 支持集群间的消息多播和单播
3. 支持各种负载均衡策略
4. 支持多种Leader选举算法
5. 提供进程池/调度进程池的实现
6. 内置注册中心
7. 内置多种微服务间调用的限流降级策略
8. 内置微服务Rest客户端
9. 内置HTTP服务网关
10. 集群状态监控和告警

集群中的消息多播是tridenter非常重要的功能，tridenter底层是通过Redis（PubSub）实现多播功能从而实现应用的相互发现的，进而组成应用集群的。集群中的每个成员都支持消息多播和单播的能力。
利用tridenter支持消息单播的能力，tridenter提供了Leader选举算法接口，内置了两种Leader选举算法，快速Leader选举算法（基于Redis队列）和一致性选举算法(基于Paxos算法)
同时，利用tridenter支持消息单播的能力，tridenter提供了进程池，实现了跨进程的方法调用和方法分片的能力
另一方面，tridenter本身也提供了微服务治理的基本功能：
tridenter自带注册中心，而利用消息多播的原理，应用是相互发现，相互注册的，所以集群中的每个成员都有一份全量的成员列表，即每个应用都是注册中心，体现了去中心化的设计思想。每个成员通过命名服务，实现了应用之间HTTP接口互相调用的能力，并提供了相关各种注解和Restful配置将服务发布方和消费方解耦
tridenter自带网关功能，可以将应用独立发布成网关服务，可代理分发HTTP请求和下载任务（暂不支持上传）
tridenter还内置了多种负载均衡算法和限流降级策略，用户也可以自定义负载均衡算法或降级策略
tridenter实现了spring actuator的健康检查接口，除了监控集群状态，还自带接口的统计分析等功能，初步实现了接口的统一管理和监控
所以，基于tridenter框架，我们也可以搭建一套类似于Spring Cloud的微服务体系

tridenter采用了去中心化的设计思想，即开发人员不需要知道当前哪个是主节点，哪些节点是从节点，更不应该显式地定义某个应用为主节点，这是由tridenter采用的Leader选举算法决定的，默认的选举算法是快速选举算法。根据选举算法，集群内的任意一个应用节点都有可能成为主节点，默认第一个启动的应用就是主节点，但是如果采用的是一致性选举算法，可能就会不一样。根据作者描述，一致性选举算法目前不稳定，推荐在应用中使用快速选举算法。

**Maven:**
``` xml
		<dependency>
			<groupId>com.github.paganini2008.atlantis</groupId>
			<artifactId>tridenter-spring-boot-starter</artifactId>
			<version>1.0-RC3</version>
		</dependency>
```
tridenter的基本功能就是让不同的Spring Boot应用变成集群模式，所以配置的时候，我们要在application.properties 定义两个系统变量，否则会报错
``` properties
spring.application.name=chaconne-management  # 服务名称
spring.application.cluster.name=chaconne-management-cluster  #集群名称
```
启动之后，如果在Console看到以下信息则表示集群配置生效
``` log
2021-06-05 18:20:11 [INFO ] io.undertow - starting server: Undertow - 2.0.29.Final
2021-06-05 18:20:11 [INFO ] org.xnio - XNIO version 3.3.8.Final
2021-06-05 18:20:11 [INFO ] org.xnio.nio - XNIO NIO Implementation Version 3.3.8.Final
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationMulticastGroup - Registered candidate: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}, Proportion: 1/1
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationRegistryCenter - Register application: [{applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}] to ApplicationRegistryCenter
2021-06-05 18:20:11 [INFO ] i.a.f.c.SerialDependencyListener - SerialDependencyHandler initialize successfully.
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - This is the leader of application cluster 'chaconne-management-cluster'. Current application event type is 'indi.atlantis.framework.tridenter.election.ApplicationClusterLeaderEvent'
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - Current leader: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: true, startTime: 1622888405582, weight: 1}
2021-06-05 18:20:11 [INFO ] o.s.b.w.e.u.UndertowServletWebServer - Undertow started on port(s) 6543 (http) with context path ''
2021-06-05 18:20:12 [INFO ] i.a.f.c.m.ChaconneManagementMain - Started ChaconneManagementMain in 12.134 seconds (JVM running for 12.829)
```
首先：
``` log
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationMulticastGroup - Registered candidate: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}, Proportion: 1/1
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationRegistryCenter - Register application: [{applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}] to ApplicationRegistryCenter
```
这两行日志分别表示成功注册消息多播组和应用注册中心

``` log
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - This is the leader of application cluster 'chaconne-management-cluster'. Current application event type is 'indi.atlantis.framework.tridenter.election.ApplicationClusterLeaderEvent'
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - Current leader: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: true, startTime: 1622888405582, weight: 1}
```
这两行日志分别表示利用ApplicationLeaderElection选举算法选出当前的应用是leader，（快速选举算法默认将第一个启动的应用作为Leader, 有点类似Jgroups）

tridenter-spring-boot-starter是一个基础型的框架，提供了各种分布式能力，下面介绍一下几种能力：
### 进程池
多个同名应用（${spring.application.name}）可以组建成一个进程池，就像线程池分配不同的线程调用某个方法一样，进程池可以进行跨应用的方法调用，前提是这个方法是存在的

示例代码：
``` java
	@MultiProcessing(value = "calc", defaultValue = "11")
	public int calc(int a, int b) {
		if (a % 3 == 0) {
			throw new IllegalArgumentException("a ==> " + a);
		}
		log.info("[" + counter.incrementAndGet() + "]Port: " + port + ", Execute at: " + new Date());
		return a * b * 20;
	}

	@OnSuccess("calc")
	public void onSuccess(Object result, MethodInvocation invocation) {
		log.info("Result: " + result + ", Take: " + (System.currentTimeMillis() - invocation.getTimestamp()));
	}

	@OnFailure("calc")
	public void onFailure(ThrowableProxy info, MethodInvocation invocation) {
		log.info("========================================");
		log.error("{}", info);
	}
```
说明：
1. 注解 @MultiProcessing修饰方法calc, 表示这个方法是多进程调用的
2. onSuccess和onFailure两个方法都是异步的调用的

### 方法分片
方法分片又叫方法并行处理，其实就是将一组参数的每一个参数使用进程池分发到不同应用上运行，然后再合并输出，并需要实现分片规则接口，见源码：
``` java
public interface Parallelization {

	Object[] slice(Object argument); // 切片

	Object merge(Object[] results);  // 合并

}
```
示例代码：
``` java
    @ParallelizingCall(value = "loop-test", usingParallelization = TestCallParallelization.class)
	public long total(String arg) {// 0,1,2,3,4,5,6,7,8,9
		return 0L;
	}

	public static class TestCallParallelization implements Parallelization {

		@Override
		public Long[] slice(Object argument) {
			String[] args = ((String) argument).split(",");
			Long[] longArray = new Long[args.length];
			int i = 0;
			for (String arg : args) {
				longArray[i++] = Long.parseLong(arg);
			}
			return longArray;
		}

		@Override
		public Long merge(Object[] results) {
			long total = 0;
			for (Object o : results) {
				total += ((Long) o).longValue();
			}
			return total;
		}

	}
```
说明：
1. 注解@ParallelizingCall修饰total方法，表示这个方法要做分片处理
2. 参数arg, 比如说你可以传 0,1,2,3,4,5,6,7,8,9，分片规则会调用slice方法将参数以“,”分割，变成数组，然后将每个值转换成long型，再分发到各个应用执行，全部执行完了，再执行merge方法进行加和操作，有点MapReduce的味道
3. total方法返回的0，是指当参数为空或方法异常返回的默认值

# 微服务能力
### Rest客户端
示例代码：
``` java
@RestClient(provider = "test-service")
// @RestClient(provider = "http://192.168.159.1:5050")
public interface TestRestClient {

	@PostMapping("/metrics/sequence/{dataType}")
	Map<String, Object> sequence(@PathVariable("dataType") String dataType, @RequestBody SequenceRequest sequenceRequest);

}
```
说明：
1. 注解@RestClient修饰的接口说明这是个Http客户端
2. 注解中，provider属性表示服务提供方，可以是集群中的某个应用名（${spring.application.name}）,也可以是具体http地址
3. 支持Spring注解（GetMapping, PostMapping, PutMapping, DeleteMapping）, 此外，用注解@Api可提供更细粒度的参数设置

### 网关
``` java
@EnableGateway
@SpringBootApplication
@ComponentScan
public class GatewayMain {

	public static void main(String[] args) {
		final int port = 9000;
		System.setProperty("server.port", String.valueOf(port));
		SpringApplication.run(GatewayMain.class, args);
	}
}
```
引用注解@EnableGateway就行了，tridenter的网关底层是用Netty4实现的
##### 配置路由：
``` java
@Primary
@Component
public class MyRouterCustomizer extends DefaultRouterCustomizer {

	@Override
	public void customize(RouterManager rm) {
		super.customize(rm);
		rm.route("/my/**").provider("tester5");
		rm.route("/test/baidu").url("https://www.baidu.com").resourceType(ResourceType.REDIRECT);
		rm.route("/test/stream").url("	https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png").resourceType(ResourceType.STREAM);
	}

}
```
说明：
ResourceType的4种类型：
DEFAULT,    转发请求
REDIRECT,    跳转
STREAM,   二进制流
FILE    保存文件

### 限流降级
限流是指在客户端限流，而非服务端
限流会依赖3个指标：
1. 响应超时率
2. 错误率
3. 并发度
默认情况，当这三个指标中有任一指标超过80%，即会触发限流，调用降级服务
限流指标统计类 :  RequestStatisticIndicator
降级服务接口：FallbackProvider 
相关源码可自行研究

### 健康监控
目前tridenter提供了3个HealthIndicator的子类
1. ApplicationClusterHealthIndicator
    显示集群的整体健康状态
2. TaskExecutorHealthIndicator
    显示集群线程池的健康状态
3. RestClientHealthIndicator
    显示Rest客户端的健康状态（响应超时率，错误率，并发度）

最后，微服务分布式协作框架tridenter的源码地址：https://github.com/paganini2008/tridenter-spring-boot-starter.git

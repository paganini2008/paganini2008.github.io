---
layout: post
title: Introduction to  Tridenter - A Microservice Distribution Collaboration Framework
date: 2021-06-21 08:30:00.000000000 +09:00
---

**Tridenter** is a microservice distributed collaboration framework developed based on the SpringBoot framework. It allows multiple independent SpringBoot applications to form a cluster easily and quickly, without relying on an external registry (such as SpringCloud Eureka, etc.).
**Tridenter  Framework** first provides a wealth of distributed application cluster management APIs and tools, and also provides a complete set of microservice governance functions.

### Feature:
------------------------
1. Adopt a decentralized idea to manage the cluster
2. Support message multicast and unicast between clusters
3. Support various load balancing strategies
4. Support multiple Leader election algorithms
5. Provide the realization of process pool/scheduling process pool
6. Built-in registry
7. Built-in current limiting and downgrading strategies for calling between multiple microservices
8. Built-in microservice Rest client
9. Built-in HTTP service gateway
10. Cluster status monitoring and alarm

### Realization principle
---------------------------
The message multicast in the cluster is a very important function of tridenter. The bottom layer of tridenter realizes the multicast function through Redis (PubSub) to realize mutual discovery of applications, and then forms an application cluster. Each member in the cluster supports the capability of message multicast and unicast.
Using tridenter's ability to support message unicasting, tridenter provides a Leader election algorithm interface with two built-in Leader election algorithms, fast Leader election algorithm (based on Redis queue) and consensus election algorithm (based on Paxos algorithm)
At the same time, using tridenter's ability to support message unicasting, tridenter provides a process pool, which realizes the ability of method invocation and method fragmentation across processes
On the other hand, tridenter itself also provides the basic functions of microservice governance:
Tridenter has its own registry, and using the principle of message multicast, applications discover and register with each other, so each member in the cluster has a full list of members, that is, each application is a registry, which reflects the Centralized design thinking. Each member realizes the ability to call each other HTTP interfaces between applications through naming services, and provides various annotations and Restful configurations to decouple the service publisher and the consumer
Tridenter has its own gateway function, which can independently publish applications as gateway services, and can distribute HTTP requests and download tasks as an agent (uploading is not currently supported)
Tridenter also has a variety of built-in load balancing algorithms and current limiting and downgrading strategies. Users can also customize load balancing algorithms or downgrading strategies.
Tridenter implements the health check interface of spring actuator. In addition to monitoring the cluster status, it also comes with functions such as interface statistical analysis, and initially realizes the unified management and monitoring of the interface.
Therefore, based on the tridenter framework, we can also build a set of microservices system similar to Spring Cloud

Tridenter adopts a decentralized design idea, that is, developers do not need to know which is the current master node and which nodes are slave nodes, and should not explicitly define an application as the master node. This is the Leader election adopted by tridenter Determined by the algorithm, the default election algorithm is the fast election algorithm. According to the election algorithm, any application node in the cluster may become the master node. By default, the first application started is the master node, but if the consensus election algorithm is adopted, it may be different. According to the author's description, the consensus election algorithm is currently unstable, and it is recommended to use the fast election algorithm in applications.

### Install:
----------------------------
``` xml
		<dependency>
			<groupId>com.github.paganini2008.atlantis</groupId>
			<artifactId>tridenter-spring-boot-starter</artifactId>
			<version>1.0-RC3</version>
		</dependency>
```

### Required Configuration
------------------------------
The basic function of tridenter is to make different Spring Boot applications into cluster mode, so when configuring, we have to define two system variables in application.properties, otherwise an error will be reported

``` properties
spring.application.name=chaconne-management  # 服务名称
spring.application.cluster.name=chaconne-management-cluster  #集群名称
```

After startup, if you see the following information in the Console, it means that the cluster configuration takes effect

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

Note：

``` log
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationMulticastGroup - Registered candidate: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}, Proportion: 1/1
2021-06-05 18:20:11 [INFO ] i.a.f.t.m.ApplicationRegistryCenter - Register application: [{applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: false, startTime: 1622888405582, weight: 1}] to ApplicationRegistryCenter
```
These two lines of logs respectively indicate successful registration of the message multicast group and application registry

``` log
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - This is the leader of application cluster 'chaconne-management-cluster'. Current application event type is 'indi.atlantis.framework.tridenter.election.ApplicationClusterLeaderEvent'
2021-06-05 18:20:11 [INFO ] i.a.f.t.e.ApplicationLeaderElection - Current leader: {applicationContextPath: http://192.168.159.1:6543, applicationName: chaconne-management, clusterName: chaconne-management-cluster, id: fafdc9ada3a5d1de3836b1a0ba4ef174, leader: true, startTime: 1622888405582, weight: 1}
```
These two lines of logs respectively indicate that the current application is selected as the leader using the <code>ApplicationLeaderElection</code> election algorithm (the fast election algorithm uses the first application as the leader by default, which is similar to Jgroups)

tridenter-spring-boot-starter is a basic framework that provides various distributed capabilities. Here are several capabilities:

### Process Pool
----------------------------
Multiple applications with the same name (${spring.application.name}) can be formed into a process pool, just as the thread pool allocates different threads to call a method, the process pool can call methods across applications, provided that the method is existing

Example：
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
Description：
1. The annotation @MultiProcessing modifies the method calc, indicating that this method is called by multiple processes
2. Both onSuccess and onFailure methods are called asynchronously

### Method Slicing
-------------------------------
Method slicing is also called method parallel processing. In fact, each parameter of a set of parameters is distributed to different applications to run using the process pool, and then combined output, and needs to implement the sharding rule interface, see the source code:

``` java
public interface Parallelization {

	Object[] slice(Object argument); // 切片

	Object merge(Object[] results);  // 合并

}
```
Example：

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
Description：
1. The annotation @ParallelizingCall modifies the total method, which means that this method needs to be fragmented
2. Parameter arg, for example, you can pass 0,1,2,3,4,5,6,7,8,9, the slice rule will call the slice method to divide the parameter with "," into an array, and then Convert each value into a long type, and then distribute it to each application for execution. After all the execution is completed, the merge method is executed to add and operate, which is a bit of MapReduce.
3. The 0 returned by the total method refers to the default value returned when the parameter is empty or the method returns abnormally

# Microservice Capabilities
### Rest Client
----------------------------
Example：

``` java
@RestClient(provider = "test-service")
// @RestClient(provider = "http://192.168.159.1:5050")
public interface TestRestClient {

	@PostMapping("/metrics/sequence/{dataType}")
	Map<String, Object> sequence(@PathVariable("dataType") String dataType, @RequestBody SequenceRequest sequenceRequest);

}
```
Description：
1. Annotate the interface modified by @RestClient to indicate that this is an Http client
2. In the annotation, the provider attribute represents the service provider, which can be an application name in the cluster (${spring.application.name}) or a specific http address
3. Support Spring annotations (GetMapping, PostMapping, PutMapping, DeleteMapping), in addition, the annotation @Api can provide more fine-grained parameter settings

### Gateway
---------------------------------
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
Just quote the annotation @EnableGateway, the bottom layer of the tridenter gateway is implemented with Netty4

##### Route Configuration：
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

Description:
**4 types of ResourceType:**
DEFAULT, forward request
REDIRECT, jump
STREAM, binary stream
FILE save file

### Current Limit and Downgrade
---------------------------------
Current limiting refers to limiting the current on the client side, not the server side
The current limit depends on 3 indicators:
1. Response timeout rate
2. Error rate
3. Concurrency
By default, when any of these three indicators exceeds 80%, the current limit will be triggered and the downgrade service will be invoked
Current limit indicator statistics: <code>RequestStatisticIndicator</code>
Downgrade service interface: <code>FallbackProvider</code>
Relevant source code can be studied by yourself

### Health Monitoring
-----------------------------
Currently Tridenter Framework provides 3 subclasses of <code>HealthIndicator</code>
1. <code>ApplicationClusterHealthIndicator</code>
     Display the overall health status of the cluster
2. <code>TaskExecutorHealthIndicator</code>
     Display the health status of the cluster thread pool
3. <code>RestClientHealthIndicator</code>
     Display the health status of the Rest client (response timeout rate, error rate, concurrency)

Git repository：https://github.com/paganini2008/tridenter-spring-boot-starter.git

---
layout: post
title: Vortex - A High Performance Distributed Streaming Computing Framework
date: 2021-06-20 08:30:00.000000000 +09:00
---

**Vortex** is a lightweight distributed streaming computing framework. It means that the data stream continuously flows into this vortex and then is output steadily.
**Vortex** is a streaming framework for memory computing, suitable for high-availability, high-concurrency, and real-time computing business scenarios.
**Vortex Framework** is developed based on the SpringBoot framework. It relies on the [Tridenter Framework](https://paganini2008.github.io/2021/06/Introduction-to-Tridenter-Microservice-Distribution-Collaboration-Framework/)  to achieve cluster features. The vortex microservice is embedded with an independent TCP server (implemented by Netty4 by default). The vortex microservice cluster Applications discover each other and establish long connections through the Tridenter Framework multicast function to achieve high availability, decentralization and load balancing, making the entire Spring application cluster capable of real-time computing.

### Vortex Framework Structure:
-------------------------------

1. vortex-common
   agent side jar package of vortex framework
2. vortex-spring-boot-starter
   The core jar package of the vortex framework, added to the SpringBoot application to make it a vortex **server**
3. vortex-metrics
   Distributed timing calculation framework based on vortex, which is an important independent sub-project of vortex

### Install:
-------------------------------

**Server Side: **

``` xml
<dependency>
	 <groupId>com.github.paganini2008.atlantis</groupId>
         <artifactId>vortex-spring-boot-starter</artifactId>
	 <version>1.0-RC3</version>
</dependency>
```
**Agent Side: **
``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
	<artifactId>vortex-common</artifactId>
	<version>1.0-RC3</version>
</dependency>
```

### Current Open Source Projects Based on the Vortex Framework Are:
---------------------------------
1. Distributed microservice monitoring system:  [Jellyfish](https://paganini2008.github.io/2021/06/Introduction-to-Greenfinger-High-Performance-Distributed-WebSpider-Framework/)
2. Distributed time series computing framework:  [Vortex Metrics](https://paganini2008.github.io/2021/06/Introduction-to-Vortex-Metrics-Distributed-Time-Series-Computing-Framework/)
3. Distributed web spider framework: [Greenfinger](https://paganini2008.github.io/2021/06/Introduction-to-Greenfinger-High-Performance-Distributed-WebSpider-Framework/)

### How to Use Vortex API in Your Application?
------------------------------
1. The vortex server needs to implement the Handler interface for customization
Example: 

``` java
@Slf4j
public class TestHandler implements Handler{

	@Override
	public void onData(Tuple tuple) {
		log.info(tuple.toString());
	}

}
```
2. The agent side sends data through the TransportClient implementation class

Reference source code (From [Jellyfish](https://paganini2008.github.io/2021/06/Introduction-to-Greenfinger-High-Performance-Distributed-WebSpider-Framework/)):
**Server Side：**

``` java
public class Slf4jHandler implements Handler {

	private static final String TOPIC_NAME = "slf4j";

	@Autowired
	private IdGenerator idGenerator;

	@Autowired
	private LogEntryService logEntryService;

	@Value("${atlantis.framework.jellyfish.handler.interferedCharacter:}")
	private String interferedCharacterRegex;

	@Override
	public void onData(Tuple tuple) {
		LogEntry logEntry = new LogEntry();
		logEntry.setId(idGenerator.generateId());
		logEntry.setClusterName(tuple.getField("clusterName", String.class));
		logEntry.setApplicationName(tuple.getField("applicationName", String.class));
		logEntry.setHost(tuple.getField("host", String.class));
		logEntry.setIdentifier(tuple.getField("identifier", String.class));
		logEntry.setLoggerName(tuple.getField("loggerName", String.class));
		logEntry.setMessage(tuple.getField("message", String.class));
		logEntry.setLevel(tuple.getField("level", String.class));
		logEntry.setReason(tuple.getField("reason", String.class));
		logEntry.setMarker(tuple.getField("marker", String.class));
		logEntry.setCreateTime(tuple.getField("timestamp", Long.class));
		if (StringUtils.isNotBlank(interferedCharacterRegex)) {
			logEntry.setMessage(logEntry.getMessage().replaceAll(interferedCharacterRegex, ""));
			logEntry.setReason(logEntry.getReason().replaceAll(interferedCharacterRegex, ""));
		}
		logEntryService.bulkSaveLogEntries(logEntry);
	}

	@Override
	public String getTopic() {
		return TOPIC_NAME;
	}

}
```

**Agent Side**
You need to implement an Agent side by yourself to continuously send data to the vortex server. You can refer to the <code>TransportClientAppenderBase</code> source code(From [Jellyfish](https://paganini2008.github.io/2021/06/Introduction-to-Greenfinger-High-Performance-Distributed-WebSpider-Framework/))：
``` java
    @Override
	protected void append(ILoggingEvent eventObject) {
		if (transportClient == null) {
			return;
		}
		Tuple tuple = Tuple.newOne(GLOBAL_TOPIC_NAME);
		tuple.setField("clusterName", clusterName);
		tuple.setField("applicationName", applicationName);
		tuple.setField("host", host);
		tuple.setField("identifier", identifier);
		tuple.setField("loggerName", eventObject.getLoggerName());
		String msg = eventObject.getFormattedMessage();
		tuple.setField("message", msg);
		tuple.setField("level", eventObject.getLevel().toString());
		String reason = ThrowableProxyUtil.asString(eventObject.getThrowableProxy());
		tuple.setField("reason", reason);
		tuple.setField("marker", eventObject.getMarker() != null ? eventObject.getMarker().getName() : "");
		tuple.setField("timestamp", eventObject.getTimeStamp());
		Map<String, String> mdc = eventObject.getMDCPropertyMap();
		if (MapUtils.isNotEmpty(mdc)) {
			tuple.append(mdc);
		}
		transportClient.write(tuple);
	}
```
**Note:**
The interaction data between the vortex application server and application agent can be Map, Tuple objects or json strings, but they are ultimately packaged into Tuple Object.

Git repository：  https://github.com/paganini2008/vortex.git
---
layout: post
title: Vortex - A Distributed Streaming Computing Framework
date: 2021-06-20 08:30:00.000000000 +09:00
---

**vortex**是一款轻量级的分布式流式计算框架。vortex中文意为旋涡，代表着数据流不断地流入这个旋涡然后被平稳地输出。
**vortex**属于内存计算型的流式框架，适用于高可用，高并发，实时计算的业务场景。
**vortex**是基于SpringBoot框架之上开发的，它依赖微服务分布式协作框架tridenter实现集群特性，vortex微服务内嵌了独立的TCP服务器（默认通过Netty4实现），vortex微服务集群中的应用程序通过tridenter多播功能相互发现并建立长连接，实现高可用，去中心化和负载均衡，使得整个Spring应用程序集群具备实时计算的能力。

**vortex项目一共包含3部分：**
1. vortex-common 
    vortex框架的**agent端**jar包
2. vortex-spring-boot-starter
    vortex框架的核心jar包，添加到SpringBoot应用使其成为vortex**服务端**
3. vortex-metrics
    基于vortex的分布式时序计算框架，它是vortex重要的独立子项目

**服务端安装**
``` xml
<dependency>
	 <groupId>com.github.paganini2008.atlantis</groupId>
         <artifactId>vortex-spring-boot-starter</artifactId>
	 <version>1.0-RC3</version>
</dependency>
```
**agent端安装**
``` xml
<dependency>
	<groupId>com.github.paganini2008.atlantis</groupId>
	<artifactId>vortex-common</artifactId>
	<version>1.0-RC3</version>
</dependency>
```

**目前基于vortex框架的开源项目有3个：**
1. 分布式微服务监控系统 [Jellyfish](https://www.jianshu.com/p/3f8c7ede0d59)
2. 分布式时序计算框架 [Vortex Metrics](https://www.jianshu.com/p/c5a0e4ae2fbd)
3. 分布式网络爬虫[Greenfinger](https://www.jianshu.com/p/a31ad3f57d04)

**如何在你的应用中使用vortex的API**
前面说过，vortex服务端接收数据，vortex agent端发送数据，vortex提供了HTTP和TCP两种协议来接收和发送外部数据。
1. vortex服务端要实现Handler接口实现定制，比如：
``` java
@Slf4j
public class TestHandler implements Handler{

	@Override
	public void onData(Tuple tuple) {
		log.info(tuple.toString());
	}

}
```
2. 而agent端通过TransportClient实现类来发送数据

下面以jellyfish中日志收集模块为例，参考源码：
**服务端：**
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

**Agent端**, 你需要自己实现一个Agent端, 向vortex服务端不断发送数据，可参考jellyfish-slf4j的TransportClientAppenderBase.java源码：
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
说明一下，vortex服务端和agent端的交互数据可以是Map, Tuple对象或json字符串，但最终都被包装成Tuple对象

具体使用，可参考vortex的源码：  https://github.com/paganini2008/vortex.git
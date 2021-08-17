---
layout: post
title:  Devtools - Open Source Java Utility Tool Family
date: 2021-06-21 08:30:00.000000000 +09:00
---

介绍一款Java开源工具包系列devtools，目前由5个Java基础工具包组成。
**Maven:**
``` xml
		<dependency>
			<groupId>com.github.paganini2008</groupId>
			<artifactId>devtools</artifactId>
			<version>2.0.2</version>
		</dependency>
```
**兼容性：**
Jdk1.8+
**组成部分：**
1. devtools-lang
2. devtools-objectpool
3. devtools-cron4j
4. devtools-beans-streaming
5. devtools-db4j

下面，对每个工具包简单介绍一下：
**devtools-lang**
devtools-lang 是一款基础工具包，对JDK中关于基础数据类型，集合，日期，IO，多线程，JDBC, 日志等常用类库进行了二次封装。devtools-lang工具包提供了更高封装程度的工具方法和API, 旨在显著提高开发人员的开发效率，优化代码风格和增加可维护性。
Maven:
``` xml
<dependency>
	<groupId>com.github.paganini2008</groupId>
	<artifactId>devtools-lang</artifactId>
	<version>2.0.2</version>
</dependency>
```
**devtools-objectpool**
devtools-objectpool是一个对象池工具包，包含一个对象池和数据库连接池实现
Maven: 
``` xml
        <dependency>
			<groupId>com.github.paganini2008</groupId>
			<artifactId>devtools-objectpool</artifactId>
			<version>2.0.2</version>
		</dependency>
```
##### devtools-cron4j
cron4j是一款小巧实用的Java调度工具包，它提供了：
1. 面向API的方式来自定义cron表达式，又能将cron表达式解析为API的形式
2. 内置多种调度器可定时执行目标内容
3. 不依赖其他组件，可轻量化地定制自己的系统
Maven: 
``` xml
<dependency>
	<groupId>com.github.paganini2008</groupId>
	<artifactId>devtools-cron4j</artifactId>
	<version>2.0.2</version>
</dependency>
```
##### devtools-beans-streaming
devtools-beans-streaming 是一个对对象列表（或称为结果集）进行查询或聚合等操作的解决方案, 类似于C#中的LINQ功能
Maven:
``` xml
<dependency>
	<groupId>com.github.paganini2008</groupId>
	<artifactId>devtools-beans-streaming</artifactId>
	<version>2.0.2</version>
</dependency>
```
##### devtools-db4j
devtools-db4j 是一款简单实用的JDBC操作封装工具包, 可适用于任何项目，极大地提高了基于JDBC的开发效率
Maven:
``` xml
 <dependency>
	<groupId>com.github.paganini2008</groupId>
	<artifactId>devtools-db4j</artifactId>
	<version>2.0.2</version>
</dependency>
```
这里主要是对devtools系列做一个大致的介绍，后面会对每个工具包做一个详尽的使用说明

源码地址：https://github.com/paganini2008/devtools.git
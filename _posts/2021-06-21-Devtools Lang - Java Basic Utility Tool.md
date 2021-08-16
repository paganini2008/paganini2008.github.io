---
layout: post
title: Devtools Lang - Java Basic Utility Tool
date: 2021-06-21 08:30:00.000000000 +09:00
---

Devtools Lang是devtools系列中的一款基础工具包，是对JDK中关于基础数据类型，集合，日期，IO，多线程，JDBC, 日志等常用类库进行二次封装。devtools-lang工具包提供了更高封装程度的工具方法和API, 旨在显著提高开发人员的开发效率，优化代码风格和性能。
### 安装：
----------------------
``` xml
		<dependency>
			<groupId>com.github.paganini2008</groupId>
			<artifactId>devtools-lang</artifactId>
			<version>2.0.3</version>
		</dependency>
```
### 兼容性：
---------------------
Jdk1.8+

### 常用工具类：
-----------------------------
StringUtils
ObjectUtils
ArrayUtils
NumericUtils
RandomUtils
RandomStringUtils
ClassUtils
### 关于基础数据类型的常用工具API：
-------------------
1. Booleans
2. Chars
3. Bytes
4. Shorts
5. Ints
6. Longs
7. Floats
8. Doubles
### 关于数值计算的常用工具类：
--------------------------
1. BigDecimalUtils
2. BigIntegerUtils
### 关于日期处理的常用工具类:
-----------------------------
1. CalendarUtils
2. DateUtils
3. LocalDateUtils
### 关于集合处理的常用工具类：
--------------------------
1. CollectionUtils
2. ListUtils
3. SetUtils
4. MapUtils
5. LruMap
6. LruList
7. LruSet
### 关于IO的常用工具API：
---------------------------
1. IOUtils
2. FileUtils
3. PropertiesUtils
4. ResourceUtils
5. ImageUtils
6. SerializationUtils
7. DirectoryWalker
8. FileMonitor
9. FileComparator
### 关于多线程的常用工具类：
-----------------------
1. ExecutorUtils
2. ThreadsUtils
3. ThreadPool
4. ThreadFactoryBuilder
5. AtomicIntegerSequence
6. AtomicLongSequence
7. Latch
### 关于反射的常用工具类：
---------------------------
1. ConstructorUtils
2. FieldUtils
3. MethodUtils
### 关于Bean操作的常用工具类：
------------------------------
1. BeanUtils
2. PropertyUtils
3. EqualsBuilder
4. HashCodeBuilder
5. ToStringBuilder
### 关于数据类型转换操作的常用工具类:
------------------------
1. ConvertUtils
2. TypeConverter
### 关于JDBC操作的常用工具类：
----------------------------
1. JdbcUtils
2. ResultSetSlice
3. PageableQuery
4. JdbcDumpTemplate
### 关于日志操作的常用工具类:
--------------------------------
1. Log
2. LogFactory

源码地址：https://github.com/paganini2008/devtools.git
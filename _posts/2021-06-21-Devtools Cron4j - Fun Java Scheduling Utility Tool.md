---
layout: post
title: Devtools Cron4j - Fun Java Scheduling Utility Tool
date: 2021-06-21 20:00:00.000000000 +09:00
---

**Devtools Cron4j**是一款小巧实用的Java调度工具包，来自于devtools系列，它提供了：
1. 面向对象的方式来定义或拼装cron表达式，又能将cron表达式还原为对象形式
2. 除了支持常用的cron表达式，sdk还提供其他时间定义方式
3. 可自定义任务，内置多种调度执行器的实现
4. 不依赖其他组件，可轻量化地定制自己的系统

### 安装:
--------------------- 
``` xml
		<dependency>
			<groupId>com.github.paganini2008</groupId>
			<artifactId>devtools-cron4j</artifactId>
			<version>2.0.3</version>
		</dependency>
```
### 兼容性
-------------------------
* Jdk1.8+
### 关于如何生成cron表达式？
---------------------
参考例子：
``` java
       // */5 * * * * ?
	public static CronExpression getCron1() {
		return CronExpressionBuilder.everySecond(5);
	}

	// 0 */2 * * * ?
	public static CronExpression getCron2() {
		return CronExpressionBuilder.everyMinute(2);
	}

	// 0 26,29,33 * * * ?
	public static CronExpression getCron3() {
		return CronExpressionBuilder.everyHour().minute(26).andMinute(29).andMinute(33);
	}

	// 0 * 14 * * ?
	public static CronExpression getCron4() {
		return CronExpressionBuilder.everyDay().hour(14).everyMinute();
	}

	// 0 0-10 15 * * ?
	public static CronExpression getCron5() {
		return CronExpressionBuilder.everyDay().hour(15).minute(0).toMinute(10);
	}

	// 0 0 23 * * ?
	public static CronExpression getCron6() {
		return CronExpressionBuilder.everyDay().at(23, 0);
	}

	// 0 15 12 * * ?
	public static CronExpression getCron7() {
		return CronExpressionBuilder.everyDay().hour(12).minute(15);
	}

	// 0 0 0,13,18,21 * * ?
	public static CronExpression getCron8() {
		return CronExpressionBuilder.hour(13).andHour(18).andHour(21);
	}

	// 0 15 10 ? * 6L
	public static CronExpression getCron9() {
		return CronExpressionBuilder.everyMonth().lastWeek().Fri().at(10, 15);
	}

	// 0 15 10 ? * MON-FRI
	public static CronExpression getCron10() {
		return CronExpressionBuilder.everyWeek().Mon().toFri().at(10, 15, 0);
	}

	// 0 0/5 12,18 * * ?
	public static CronExpression getCron11() {
		return CronExpressionBuilder.hour(12).andHour(18).everyMinute(5);
	}

	// 0 30 23 L * ?
	public static CronExpression getCron12() {
		return CronExpressionBuilder.everyMonth().lastDay().at(23, 30);
	}

	// 0 10,20,30 12 ? 7-11 6L 2021-2025
	public static CronExpression getCron13() {
		return CronExpressionBuilder.year(2021).toYear(2025).Aug().toDec().lastWeek().Fri().hour(12).minute(10).andMinute(20).andMinute(30);
	}

	// 0 10 23 ? * 6#3
	public static CronExpression getCron14() {
		return CronExpressionBuilder.everyMonth().week(3).Fri().at(23, 10);
	}

	// 0 15-50/2 0-6 10-28 * ?
	public static CronExpression getCron15() {
		return CronExpressionBuilder.everyMonth().day(10).toDay(28).hour(0).toHour(6).minute(15).toMinute(50, 2);
	}
```

### 如何解析cron表达式？
---------------------
``` java
        System.out.println(CRON.parse("*/5 * * * * ?"));
		System.out.println(CRON.parse("0 */2 * * * ?"));
		System.out.println(CRON.parse("0 15 10 LW * ?"));
		System.out.println(CRON.parse("0 0 12 10W * ?"));
		System.out.println(CRON.parse("0 15 10 ? * MON-FRI"));
		System.out.println(CRON.parse("0 26,29,33 * * * ?"));
		System.out.println(CRON.parse("0 15-50/2 0-6 10-28 * ?"));
		System.out.println(CRON.parse("0 10 23 ? * 6#3"));
		System.out.println(CRON.parse("0 10,20,30 12 ? 7-11 6L 2021-2025"));
```

### 如何测试cron表达式？
------------------------
``` java
		CRON.parse("0 30 23 L * ?").forEach(date -> {
			System.out.println(DateUtils.format(date));
		}, 20);

		System.out.println("-----------------------------------------");

		CRON.parse("0 0 12 10-15 * ?").forEach(date -> {
			System.out.println(DateUtils.format(date));
		}, 20);
```

### 如何执行调度程序？
------------------------

``` java
        CronExpression expression = CronExpressionBuilder.everySecond(5);
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.schedule(new Task() {

			@Override
			public boolean execute() {
				System.out.println("Task is running at: " + new Date());
				return true;
			}

			@Override
			public Cancellable cancellable() {
				return Cancellables.cancelIfRuns(-1);
			}

			@Override
			public void onCancellation(Throwable e) {
				System.out.println("Cancelled.");
			}

		}, expression);

		System.in.read();
		executor.close();
```
源码地址：https://github.com/paganini2008/devtools.git


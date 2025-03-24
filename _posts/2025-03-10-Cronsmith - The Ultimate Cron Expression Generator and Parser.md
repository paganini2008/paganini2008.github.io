---
layout: post
title: Cronsmith - The Ultimate Cron Expression Generator & Parser
date: 2025-03-10 20:00:00.000000000 +09:00
---



# Cronsmith - The Ultimate Cron Expression Generator & Parser

### Are cron expressions sometimes difficult to understand? Let's try a different approach to creating scheduled tasks.

**Cronsmith** is a powerful and versatile Java utility library designed for handling complex cron expressions in an intuitive, object-oriented manner. It offers a highly flexible and comprehensive API for generating, parsing, and scheduling cron-based tasks with ease.

Built for seamless integration, **Cronsmith** provides full support for both Spring and Quartz cron expressions, ensuring compatibility with widely used scheduling frameworks. Moreover, it extends traditional cron syntax by introducing advanced patterns — such as multiple numbers combined with 'L' (last) and 'W' (weekday)—which were previously unsupported, offering greater precision and flexibility in scheduling.


## Features

### 1. Object-Oriented <code>CronExpression</code> Builder

**Cronsmith** allows developers to construct complex cron expressions in an intuitive, object-oriented manner. This approach enables easy customization and avoids the need for manual string construction.

#### Example:

```java
@Test
    public void testA() {
        CronExpression cronExpression = new CronBuilder().everySecond(5);
        System.out.println(cronExpression.toString());
        assertEquals("*/5 * * * * ?", cronExpression.toString());
    }

    @Test
    public void testB() {
        CronExpression cronExpression = new CronBuilder().everyMinute(5).second(5).andSecond(10)
                .toSecond(30).andSecond(32).toSecond(59, 2);
        System.out.println(cronExpression.toString());
        assertEquals("5,10-30,32/2 */5 * * * ?", cronExpression.toString());
    }

    @Test
    public void testC() {
        CronExpression cronExpression = new CronBuilder().everyMonth().day(10).andDay(15).andDay(16)
                .andLastDay().everyHour(2).everyMinute(5);
        System.out.println(cronExpression.toString());
        assertEquals("0 */5 */2 10,15,16,L * ?", cronExpression.toString());
    }

    @Test
    public void testD() {
        CronExpression cronExpression = new CronBuilder().everyMonth(3).day(10).andLastWeekday()
                .hour(12).minute(1).toMinute(15, 1);
        System.out.println(cronExpression.toString());
        assertEquals("0 1-15 12 10,LW */3 ?", cronExpression.toString());
    }

    @Test
    public void testE() {
        CronExpression cronExpression =
                new CronBuilder().everyMonth().everyWeek().Mon().toFri().at(15, 10);
        System.out.println(cronExpression.toString());
        assertEquals("0 10 15 ? * MON-FRI", cronExpression.toString());
    }

    @Test
    public void testF() {
        CronExpression cronExpression = new CronBuilder().everyMonth().dayOfWeek(3, 6).everyHour(2);
        System.out.println(cronExpression.toString());
        assertTrue(cronExpression.toString().equals("0 0 */2 ? * SAT#3"));
    }

    @Test
    public void testG() {
        CronExpression cronExpression =
                new CronBuilder().year(2025).Mar().toSept().everyWeek().everyWeekday().at(9, 10);
        System.out.println(cronExpression.toString());
        assertTrue(cronExpression.toString().equals("0 10 9 ? MAR-SEP MON-FRI 2025"));
    }

    @Test
    public void testH() {
        CronExpression cronExpression =
                new CronBuilder().year(2025).toYear(2028).everyMonth(2).lastDay().hour(12);
        System.out.println(cronExpression.toString());
        assertEquals("0 0 12 L */2 ? 2025-2028", cronExpression.toString());
    }

    @Test
    public void testI() {
        CronExpression cronExpression = new CronBuilder().everyYear().June().andJuly().andAug()
                .latestWeekday(15).hour(10).toHour(18, 2);
        System.out.println(cronExpression.toString());
        assertEquals("0 0 10-18/2 15W JUN,JUL,AUG ?", cronExpression.toString());
    }

    @Test
    public void testJ() {
        CronExpression cronExpression = new CronBuilder().everyYear(2).Mar().andApr().andMay()
                .toDec().everyWeek(2).Tues().toFri().hour(10).andHour(12).toHour(22).everyMinute()
                .second(10).andSecond(20).andSecond(30);
        System.out.println(cronExpression.toString());
        assertEquals("10,20,30 * 10,12-22 ? MAR,APR,MAY-DEC TUE-FRI 2025/2",
                cronExpression.toString());
    }

    @Test
    public void testK() {
        CronExpression cronExpression = new CronBuilder().year(2025).toYear(2030).andYear(2035)
                .toEnd(2).everyMonth(2, 2).dayOfWeek(2, DayOfWeek.TUESDAY)
                .and(3, DayOfWeek.WEDNESDAY).andLastFri().hour(2).andHour(3).andHour(4)
                .toHour(17, 2).minute(0).toMinute(12, 3).andMinute(15).toMinute(40, 2).andMinute(46)
                .andMinute(48).andMinute(50).everySecond(5);
        System.out.println(cronExpression.toString());
        assertEquals(
                "*/5 0-12/3,15-40/2,46,48,50 2,3,4-17/2 ? FEB-DEC/2 TUE#2,WED#3,5L 2025-2030,2035/2",
                cronExpression.toString());
    }

    @Test
    public void testL() {
        CronExpression cronExpression = new CronBuilder().everyYear(2025, 4).everyMonth(5, 1)
                .day(10).andDay(15).andDay(20).andLastDay(2).hour(10).toHour(15, 1).at(10, 0)
                .andSecond(15).andSecond(30).andSecond(45);
        System.out.println(cronExpression.toString());
        assertEquals("0,15,30,45 10 10-15 10,15,20,L-2 MAY-DEC ? 2025/4",
                cronExpression.toString());
    }

    @Test
    public void testM() {
        CronExpression cronExpression = new CronBuilder().year().andYear(2026).andYear(2030)
                .toEnd(2).Mar().andJuly().andSept().dayOfWeek(1, DayOfWeek.SATURDAY)
                .and(2, DayOfWeek.THURSDAY).andLast(DayOfWeek.FRIDAY).hour(0).toHour(12, 3)
                .everyMinute(10).second(0).andSecond(15).andSecond(30).andSecond(45).toSecond(59);
        System.out.println(cronExpression.toString());
        assertEquals("0,15,30,45/1 */10 0-12/3 ? MAR,JUL,SEP SAT#1,THU#2,5L 2025,2026,2030/2",
                cronExpression.toString());
    }
```

### 2. Cron Expression String Parsing and Reverse Engineering to <code>CronExpression</code>

**Cronsmith** includes a powerful parser built with ANTLR, allowing you to parse an existing cron expression string and convert it back into a structured `CronExpression` object.

#### Example:

```java
    @Test
    public void testA() {
        String cron = "0 0 12 * * ?";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testB() {
        String cron = "0 15 10 ? * MON-FRI";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testC() {
        String cron = "0 10,20,30 9-17 L * ?";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testD() {
        String cron = "1,3,5,7,9 3-30/3 12-16 ? * TUE#1";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testE() {
        String cron = "0 0 6 ? 1-5 MON,WED,FRI";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testF() {
        String cron = "*/5 1 12,15,18-22 LW * ? 2025";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testG() {
        String cron = "0 1 0-12,15,16-22/2 ? * 1-5 2025-2028";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testH() {
        String cron = "0 1,5,7,13,29,45 12/2 5,10,27W,L-1 APR-NOV ?";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testI() {
        String cron = "5/1 */5 12 ? 1-9 3#1,5#2,6L";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testJ() {
        String cron = "*/5 1,3,5/1 12-16 1,3,20,LW MAR-SEP ? 2026/1";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testK() {
        String cron = "5-30/7 0-12/3,15-45/2 2,3,4-17/2 ? JAN-JUL MON-THU/2 2025-2033";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testL() {
        String cron = "0,15,30,45/1 */10 0-12/3 ? MAR,JUL,SEP SAT#1,THU#2,5L 2025,2026,2030/2";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }

    @Test
    public void testM() {
        String cron = "*/2 0 12 1,5,9,22,L */2 ? 2025/2";
        CronExpression cronExpression = CRON.parse(cron);
        System.out.println(cronExpression);
        assertEquals(cron, cronExpression.toString());
    }
```

Parse Tree: 

``` shell
cron
    second
        secondField
            rangeWithStep
                5
                -
                30
                /
                7
    <missing SPACE>
    minute
        minuteField
            rangeWithStep
                0
                -
                12
                /
                3
        ,
        minuteField
            rangeWithStep
                15
                -
                45
                /
                2
    <missing SPACE>
    hour
        hourField
            2
        ,
        hourField
            3
        ,
        hourField
            rangeWithStep
                4
                -
                17
                /
                2
    <missing SPACE>
    dayOfMonth
        dayOfMonthField
            ?
    <missing SPACE>
    month
        monthField
            monthRange
                monthName
                    JAN
                -
                monthName
                    JUL
    <missing SPACE>
    dayOfWeek
        dayOfWeekField
            weekdayRangeWithStep
                dayOfWeekName
                    MON
                -
                dayOfWeekName
                    THU
                /
                2
    year
        yearField
            yearRange
                2025
                -
                2030
    <EOF>
```

Retrieve next date and times from  <code>CronExpression</code>

```java
public static void main(String[] args) {
   int N = 100; // Retrieve 100 items
   String cron = "10 0 12 LW 1/2 ?";
   CronExpression cronExpression = CRON.parse(cron);
   System.out.println(cronExpression);
   cronExpression.consume(l -> {
       System.out.println(l);
   }, N);
}

// Console:
// 2025-05-30T12:00:10
// 2025-07-31T12:00:10
// 2025-09-30T12:00:10
// 2025-11-28T12:00:10
// 2026-01-30T12:00:10
// 2026-03-31T12:00:10
// 2026-05-29T12:00:10
// 2026-07-31T12:00:10
// 2026-09-30T12:00:10
// 2026-11-30T12:00:10
// 2027-01-29T12:00:10
// 2027-03-31T12:00:10
// 2027-05-31T12:00:10
// 2027-07-30T12:00:10
// 2027-09-30T12:00:10
// 2027-11-30T12:00:10
// 2028-01-31T12:00:10
// 2028-03-31T12:00:10
// 2028-05-31T12:00:10
// ...

```



### 3. Built-in Scheduler

**Cronsmith** provides a built-in scheduler that allows you to execute tasks at specified intervals based on cron expressions. This feature makes it easy to schedule repetitive tasks efficiently.

#### Example: Scheduling a Task Every 5 Seconds

```java
private ScheduledExecutorService scheduledExecutorService;

@Before
public void start() {
    scheduledExecutorService =
                Executors.newScheduledThreadPool(Runtime.getRuntime().availableProcessors() * 2);
}

@Test
public void testSchedulerAndRunTenTimes() {
    int N = 10; // Run 10 times
    final CountDownLatch latch = new CountDownLatch(N);
    final AtomicInteger counter = new AtomicInteger();
    
    CronFuture future = new CronBuilder()
        .everySecond(5)
        .scheduler(scheduledExecutorService)
        .setDebuged(false)
        .runTask(() -> {
            System.out.println("Run task_" + counter.incrementAndGet());
            latch.countDown();
        }, N);
    
    try {
        latch.await();
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
    
    future.cancel(true);
    assertTrue(future.isDone() && counter.get() == N);
}

@After
public void release() {
    scheduledExecutorService.shutdown();
}
```

### 4. Advanced Part

**Can specify day of year**

``` java
public static void main(String[] args) {
   int N = 1000;
   DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
   new CronBuilder().setZoneId(ZoneId.of("UTC")).year(2025).day(208).andDay(330).toLastDay()
                .at(12, 0).consume(ldt -> {
                    System.out.println(ldt.format(dtf));
                }, N);
}
// Console: 
// 2025-07-27 12:00:00
// 2025-11-26 12:00:00
// 2025-11-27 12:00:00
// 2025-11-28 12:00:00
// 2025-11-29 12:00:00
// 2025-11-30 12:00:00
// 2025-12-01 12:00:00
// 2025-12-02 12:00:00
// 2025-12-03 12:00:00
// 2025-12-04 12:00:00
// 2025-12-05 12:00:00
// 2025-12-06 12:00:00
// ...

```

**Can specify week of year**

``` java
public static void main(String[] args) {
    int N = 1000;
    DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    new CronBuilder().setZoneId(ZoneId.of("UTC")).everyYear().week(40).andWeek(45).Mon().toFri()
                .at(12, 0).consume(ldt -> {
                    System.out.println(ldt.format(dtf));
                }, N);
}
// Console: 
// 2025-09-29 12:00:00
// 2025-09-30 12:00:00
// 2025-10-01 12:00:00
// 2025-10-02 12:00:00
// 2025-10-03 12:00:00
// 2025-11-03 12:00:00
// 2025-11-04 12:00:00
// 2025-11-05 12:00:00
// 2025-11-06 12:00:00
// 2025-11-07 12:00:00
// 2026-09-28 12:00:00
// 2026-09-29 12:00:00
// 2026-09-30 12:00:00
// 2026-10-01 12:00:00
// 2026-10-02 12:00:00
// ...
```



## Installation

Support Jdk1.8 or later

To use **Cronsmith** in your Java project, add the following dependency to your `pom.xml` (if using Maven):

```xml
<dependency>
    <groupId>com.github.paganini2008</groupId>
    <artifactId>cronsmith</artifactId>
    <version>1.0.0-beta</version>
</dependency>
```

If using Gradle:

```gradle
dependencies {
    implementation 'com.github.paganini2008:cronsmith:1.0.0'
}
```

## Getting Started

1. Add Cronsmith as a dependency in your project.
2. Use `CronBuilder` to create complex cron expressions.
3. Parse existing cron expressions using `CRON.parse()`.
4. Schedule tasks using the built-in scheduler.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Download

[Click here to download the latest release](https://github.com/yourusername/cronsmith/releases)

---
layout: post
title: Java8 日期类
categories: JavaSE
description: 日期类
keywords: Java8 日期类
---

### Java8 时间日期类操作
Java8的时间类有两个重要的特性
- 线程安全
- 不可变类，返回的都是新的对象        
显然，该特性解决了原来java.util.Date类与SimpleDateFormat线程不安全的问题。同时Java8的时间类提供了诸多内置方法，方便了对时间进行相应的操作。    

![image.png](https://upload-images.jianshu.io/upload_images/14607771-561927eb82ea0f40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)    
上图为Java8时间类的覆盖范围
相关的类有
- LocalDate
- LocalTime
- LocalDateTime
- ZoneId
- ZonedDateTime
- Instant

#### Instant类
Instant类用来表示格林威治时间（UTC）开始的时间点，初始时间为1970-01-01T00:00:00Z。也就是从1970年一月一号开始计时，得到的秒值甚至是是纳秒值。该时间戳可以与日期时间转换。因此可以表示人类世界最完整的时间。该类相比原来java.util.Date类，精确到了纳秒级别。

获取当前的秒值和纳秒值
```
Instant instant = Instant.now();
System.out.println(instant);
System.out.println(instant.getEpochSecond());
System.out.println(instant.getNano());

结果如下
2019-08-28T07:59:54.979Z
1566979194
979000000
```
将指定秒值转为Instant。Instant.ofEpochSecond()方法。
```
Instant instant1 = Instant.ofEpochSecond(1566981233L);
System.out.println(instant1);
```

#### LocalDate、LocalTime、LocalDateTime、ZonedDateTime

Java8使用LocalDate、LocalTime、LocalDateTime、ZonedDateTime分别操作日期、时间、日期和时间。
这四个类的默认使用系统时区
获取当天日期及时间
```
LocalDate today = LocalDate.now();
System.out.println(today);

LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime);

LocalTime localTime = LocalTime.now();
System.out.println(localTime);

ZonedDateTime zonedDateTime = ZonedDateTime.now();
System.out.println(zonedDateTime);

ZoneId zoneId = ZoneId.systemDefault();
System.out.println(zoneId);
```
结果如下
```
2019-08-28
2019-08-28T17:42:01.964
17:42:01.965
2019-08-28T17:42:01.965+08:00[Asia/Shanghai]
Asia/Shanghai
```
指定日期2019-09-30并通过isBefore()判断是否今天在指定日期之前
```
LocalDate future = LocalDate.of(2019, 9, 30);
boolean before = today.isBefore(future);
System.out.println(before);
```
LocalDateTime转String 通过DateTimeFormatter指定转换格式
```
String formatStr = localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss"));
System.out.println(formatStr);
```
String转为LocalDate
```
String str = "2019-01-02";
LocalDate localDate2 = LocalDate.parse(str, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
System.out.println(localDate2);
```

#### LocalDateTime与Instant的互相转换
获取当天的秒值和毫秒值。LocalDateTime转Instant获取时间戳。由于LocalDateTime并没有包含时区，转为Instant需要指明所在时区。北京时间也就是东八区ZoneOffset.of("+8")
```
long milliSecond = LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli();
System.out.println(milliSecond);

long second = LocalDateTime.now().toEpochSecond(ZoneOffset.of("+8"));
System.out.println(second);
```
Instant时间戳转LocalDateTime。使用LocalDateTime.ofInstant方法，需要指定转换为哪个时区的时间
```
LocalDateTime localDateTime2 = LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault()); //使用系统默认时间
System.out.println(localDateTime2);

结果如下
2019-08-28T16:33:53.639
```
#### 参考文章
https://blog.csdn.net/u013066244/article/details/96443952
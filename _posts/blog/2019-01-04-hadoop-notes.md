---
layout: post
title: Hadoop相关笔记
categories: hadoop
description: Hadoop相关笔记
keywords: hadoop
---

### Hive指定分隔符
#### 建表语句如下，后两行指定分隔符
create table t_order( id string, create_time string, amount float, uid string)
row format delimited
fields terminated by ',';
### Beeline连接hive命令
!connect jdbc:hive2://localhost:10000
### Hive创建外部表
create external table t_order( id string, create_time string, amount float, uid string)
row format delimited
fields terminated by ','
location "/xxx";
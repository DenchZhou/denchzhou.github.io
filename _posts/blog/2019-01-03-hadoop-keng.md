---
layout: post
title: Hadoop踩坑笔记
categories: hadoop
description: Hadoop学习过程中的坑
keywords: hadoop
---

## （一）格式化后启动dfs,使用start-dfs.sh命令后有如下报错
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
## 解决方法
在hadoop目录下的sbin目录下
对start-dfs.sh，stop-dfs.sh两个文件在其顶部添加以下参数
```
#!/usr/bin/env bash
HDFS_DATANODE_USER=root
HADOOP_SECURE_DN_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```
对start-yarn.sh，stop-yarn.sh两文件的顶部也添加如下参数
```


#!/usr/bin/env bash
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```
修改后退出重新执行 start-dfs.sh
![image.png](https://upload-images.jianshu.io/upload_images/14607771-108148b9d17cc7f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## （二）错误: 找不到或无法加载主类 org.apache.hadoop.mapreduce.v2.app.MRAppMaster
控制台输入 hadoop classpath 将内容复制到下面的value
编辑yarn-site.xml
```
<property>
<name>yarn.application.classpath</name>
<value></value>
</property>
```
## （三）Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 0 time(s)
```
INFO ] 2018-12-18 16:48:07,397 method:org.apache.hadoop.mapred.ClientServiceDelegate.getProxy(ClientServiceDelegate.java:278)
Application state is completed. FinalApplicationStatus=SUCCEEDED. Redirecting to job history server
[INFO ] 2018-12-18 16:48:09,402 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:11,406 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:13,414 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:15,417 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:17,422 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 4 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:19,426 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 5 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:21,430 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 6 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:23,435 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 7 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:25,439 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 8 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[INFO ] 2018-12-18 16:48:27,444 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:931)
Retrying connect to server: 0.0.0.0/0.0.0.0:10020. Already tried 9 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
[WARN ] 2018-12-18 16:48:28,449 method:org.apache.hadoop.ipc.Client$Connection.handleConnectionFailure(Client.java:913)
Failed to connect to server: 0.0.0.0/0.0.0.0:10020: retries get failed due to exceeded maximum allowed retries number: 10
java.net.ConnectException: Connection refused: no further information
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at org.apache.hadoop.net.SocketIOWithTimeout.connect(SocketIOWithTimeout.java:206)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:531)
	at org.apache.hadoop.net.NetUtils.connect(NetUtils.java:495)
	at org.apache.hadoop.ipc.Client$Connection.setupConnection(Client.java:681)
	at org.apache.hadoop.ipc.Client$Connection.setupIOstreams(Client.java:777)
	at org.apache.hadoop.ipc.Client$Connection.access$3500(Client.java:409)
	at org.apache.hadoop.ipc.Client.getConnection(Client.java:1542)
	at org.apache.hadoop.ipc.Client.call(Client.java:1373)
	at org.apache.hadoop.ipc.Client.call(Client.java:1337)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:227)
	at org.apache.hadoop.ipc.ProtobufRpcEngine$Invoker.invoke(ProtobufRpcEngine.java:116)
	at com.sun.proxy.$Proxy15.getJobReport(Unknown Source)
	at org.apache.hadoop.mapreduce.v2.api.impl.pb.client.MRClientProtocolPBClientImpl.getJobReport(MRClientProtocolPBClientImpl.java:133)
	at sun.reflect.GeneratedMethodAccessor5.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.mapred.ClientServiceDelegate.invoke(ClientServiceDelegate.java:325)
	at org.apache.hadoop.mapred.ClientServiceDelegate.getJobStatus(ClientServiceDelegate.java:429)
	at org.apache.hadoop.mapred.YARNRunner.getJobStatus(YARNRunner.java:617)
	at org.apache.hadoop.mapreduce.Job$1.run(Job.java:323)
	at org.apache.hadoop.mapreduce.Job$1.run(Job.java:320)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1807)
	at org.apache.hadoop.mapreduce.Job.updateStatus(Job.java:320)
	at org.apache.hadoop.mapreduce.Job.isComplete(Job.java:604)
	at org.apache.hadoop.mapreduce.Job.monitorAndPrintJob(Job.java:1400)
	at org.apache.hadoop.mapreduce.Job.waitForCompletion(Job.java:1362)
	at wordcountdemo.JobSubmitter.main(JobSubmitter.java:71)
```
## 解决方法
修改mapred-site.xml   进行如下添加，其中master为个人自己的hostname
```
<property>  
        <name>mapreduce.jobhistory.address</name>  
        <value>master:10020</value>  
</property>
```
在namenode启动
mr-jobhistory-daemon.sh start historyserver 
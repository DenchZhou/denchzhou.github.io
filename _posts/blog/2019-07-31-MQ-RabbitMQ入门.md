---
layout: post
title: RabbitMQ入门
categories: MQ
description: RabbitMQ
keywords: MQ RabbitMQ
---

### 消息队列
在高并发环境下，大量的Insert、Update等操作请求同时到达MySQL，很容易引起请求的堆积导致系统崩溃。消息队列是分布式系统中的重要组件，能够有效的解决应用解耦，异步消息，流量削锋的问题。目前市面上比较常见的主要有ActiveMq、RabbitMq、RocketMq、Kafka等。

### 消息队列的应用场景
异步消息：在用户注册的时候，通常会发送激活邮件或者短信。同步的方式是将用户信息写入数据库，然后发送短信或者激活邮件，全部完成后才返还给客户端。这种方式下，CPU需要等待所有结果完成，系统的并发量、响应时间、吞吐量将存在很大的瓶颈。使用消息队列后，只需要将消息写入队列，就可以立即返回，由其他应用程序异步的读取消息进行相应的处理。

应用解耦：两个应用系统之间存在调用关系的时候，被调用方如果出现问题，很容易导致调用方也无法完成。通过消息队列，如果被调用方出现了问题，并不会影响调用方，消息已经被写入消息队列中。

流量削锋：避免流量过大导致系统挂掉。比如在应用前端加入队列，服务器可以根据自己的处理能力从队列中获得要处理的请求。

### RabbitMQ
RabbitMq是基于AMQP协议（高级消息队列协议）的消息队列，由Erlang开发，支持多种主流操作系统以及多种编程语言。RabbitMQ的基本架构如下图所示。

![rabbitMQ架构图.png](https://upload-images.jianshu.io/upload_images/14607771-81c9773d206dcc7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

RabbitMQ Server：又称为Broker Server, 维护从生产者到消费者这条路线的数据传输。         
Vhost：虚拟主机，一个Broker可能会有多个Vhost，用于不同用户的权限分隔。每个Vhost相当于一个RabbitMQ Server,拥有Queue、Bind、Exchange    
Exchange：生产者将消息提交给Exchange（交换器），由Exchange决定将消息往哪个队列发
Bind：将队列和Exchange通过某种规则绑定起来，如direct、fanout、topic和headers
Queue:消息的载体    

### RabbitMq管理界面
RabbitMq的安装就不说了  启动后可以打开管理界面查看，我的地址是 http://192.168.128.131:15672
### 基本模型（官网的入门案例）
![image.png](https://upload-images.jianshu.io/upload_images/14607771-8b5a3ae92c0bd776.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/950)
##### 简单队列
![简单队列](https://upload-images.jianshu.io/upload_images/14607771-05b7fdc69d6883e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上图中，P表示生产者，C为消费者，队列代表消费者保留的消息缓冲区。
代码如下
```
public class Send {
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception{
        // 定义连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        // 通过工厂获取连接和连接通道
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            String message = "Hello World!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}

public class Recv {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("root");
        factory.setPassword("123456");
        factory.setHost("192.168.128.131");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "'");
        };
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> { });
    }

}
```
##### Work模式
![work模式](https://upload-images.jianshu.io/upload_images/14607771-d6099f114fc70216.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)     
一个生产者，多个消费者，一条消息只能被一个生产者获取
```
public class NewTask {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
            for (int i=0; i<100; i++){
                String msg = "hello " + i;
                channel.basicPublish("", TASK_QUEUE_NAME, null, msg.getBytes("UTF-8"));
                System.out.println(" [x] Sent '" + msg + "'");
                Thread.sleep(500);
            }
        }
    }
}

public class Work {

    private static final String TASK_QUEUE_NAME = "task_queue";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        final Connection connection = factory.newConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(TASK_QUEUE_NAME, true, false, false, null);
        // 同一时刻服务器只会发一条消息给消费者
        channel.basicQos(1);

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");

            System.out.println(" [x] Received '" + message + "'");
            try {
                Thread.sleep(1500);
            } catch (Exception e){
              e.printStackTrace();
            } finally {
                System.out.println(" [x] Done");
                // 手动确认
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            }
        };
        // 第二个参数为autoAck 设为false进行手动确认
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, consumerTag -> { });
    }

}
```
RabbitMq默认是轮询分发任务，在上面的场景中，就是一次性分配，把奇数消息发给一个Work，偶数消息发给另一个Work。即使是其中一个Work处理的很慢。显然这样的方式实际应用中很容易导致某个应用高负载。可以通过设置给消费者分配的消息数量，即上面的 channel.basicQos(1) 和设置手动确认的方式更改为公平的分配。先确认的可以再获取消息。
使用自动确认的时候，如果消费者还未处理结束便系统崩溃了，队列将丢失消费者未处理的消息。因此，建议使用手动确认的方式。
##### Publish/Subscribe

![fanout](https://upload-images.jianshu.io/upload_images/14607771-1c0d4e7961a18a9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
public class EmitLog {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

            String message = "hello world!";

            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + message + "'");
        }
    }
}
public class ReceiveLogs {

    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        // 默认创建一个队列并返回队列名字
        String queueName = channel.queueDeclare().getQueue();
        // 将队列与交换器绑定
        channel.queueBind(queueName, EXCHANGE_NAME, "");

        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + message + "' from " + queueName);
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }

}
```
生产者往交换器发消息，两个队列绑定到了这个交换器上，两个队列都收到了交换器发过来的消息。
##### Routing
![image.png](https://upload-images.jianshu.io/upload_images/14607771-f19d11bee67ee626.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
direct交换器就是根据路由key绑定队列与交换器
```
public class EmitLogDirect {
    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);

            String severity = "black";
            String message = "hello black ";

            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + severity + "':'" + message + "'");
        }
    }
}
public class ReceiveLogDirect {

    private static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, EXCHANGE_NAME, "black");
        System.out.println(" [*] Waiting for messages. To exit press CTRL+C");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" + delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {
        });
    }
}
```
##### Topics
![image.png](https://upload-images.jianshu.io/upload_images/14607771-9b475651b52ddc89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Topic提供了一种规则匹配交换器和队列，使得绑定更为灵活。* 可以匹配一个单词；# 可以匹配0个或者多个单词
```
public class EmitLogTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);

            //String routingKey = "hello.test.log";
            String routingKey = "test.log";
            String message = "hello";

            channel.basicPublish(EXCHANGE_NAME, routingKey, null, message.getBytes("UTF-8"));
            System.out.println(" [x] Sent '" + routingKey + "':'" + message + "'");
        }
    }
}
public class ReceiveLogsTopic {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        String queueName = channel.queueDeclare().getQueue();

        channel.queueBind(queueName, EXCHANGE_NAME, "*.log");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
public class ReceiveLogsTopic2 {

    private static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] argv) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("192.168.128.131");
        factory.setPort(5672);
        factory.setUsername("root");
        factory.setPassword("123456");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        String queueName = channel.queueDeclare().getQueue();

        channel.queueBind(queueName, EXCHANGE_NAME, "#.log");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println(" [x2] Received '" +
                    delivery.getEnvelope().getRoutingKey() + "':'" + message + "'");
        };
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> { });
    }
}
```
上面两个ReceiveLogsTopic分别绑定*.log 和 #.log，分别对hello.test.log和test.log进行测试，*.log只能接收test.log的消息。


### 参考文章
http://www.uml.org.cn/zjjs/201805234.asp
# RocketMQ4.4入门进阶+实战

## 第一章 RocketMQ

### 1.1 MQ介绍

MQ全称Message Queue，消息队列(MQ)是一种应用程序对应用程序的通信方式，应用程序通过读写出入队列的消息(针对应用程序的数据)来通信，而无需专用连接来链接它们，消息传递指的是程序之间通过在消息中发送数据进行通信。对立诶的使用除去了接收和发送应用程序同时执行的要求。

### 1.2 主流MQ对比

目前市面上成熟主流的MQ有Kafka、RocketMQ、RabbitMQ，我们这里对每款MQ做一个简单介绍。

Kafka

```tex
Apache下的一个子项目，使用scala实现的一个高性能分布式Publish/Subscribe消息系统。
1、快速持久化，通过磁盘顺序读写与零拷贝机制，可以在0(1)的系统开销下进行消息持久化；
2、高吞吐，在一台普通的服务器上即可以达到10w/s的吞吐量；
3、高堆积，支持topic下消费者较长时间离线，消息堆积量大；
4、完全的分布式系统，Broker、Producer、Consumer都原生支持分布式、依赖zookeeper自动实现负载均衡；
5、支持Hadoop数据并行加载，对应像Hadoop的一样的日志数据和离线分析系统，但又要求实时处理的限制，这是一个可行的解决方案。
```

RocketMQ

```tex
RocketMQ的前身是Metaq，当Metag3.0发布时，产品名称改为RocketMQ，RocketMQ是一款分布式、队列模型的消息中间件，具有以下特点：
1、能够保证严格的消息顺序
2、提供丰富的消息拉取模式
3、高效的订阅者水平扩展能力
4、实时的消息订阅机制
5、支持事务消息
6、亿级消息堆积能力
```

RabbitMQ

```tcl
使用Erlang编写的一个开源消息队列，本身支持此很多的协议：AMQP、XMPP、SMTP、STOMP，也正是如此，使的它变得非常重量级，更适合于企业级的开发。
```

特性对比

### 1.3 RocktMQ环境要求

RocketMQ对环境有要求，如下：

```tex
64Bit OS, Linux/Unix/Mac is recommended;
64Bit JDK 1.8+;
Maven 3.2.x;
Git;
4g+ free disk for Broker server
```

### 1.4 RocketMQ下载

RocketMQ目前已经捐献给了Apache，官方网址为http://rocketmq.apache.org/目前最新版本已经是4.6.1版本， 我们可以点击Latest releaseV4.6.1进入下载页。

![image-20200313101434315](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200313101434315.png)

![image-20200313101629175](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200313101629175.png)

### 1.5 单节点RocketMQ安装

#### 1.5.1 环境准备

我们先准备一台centos虚拟机，ip：192.168.211.143，在hosts文件中配置地址与IP的映射关系。

| IP              | hostname             | mastername       |
| --------------- | -------------------- | ---------------- |
| 192.168.211.143 | Rocketmq-nameserver1 | Rocketmq-master1 |

修改/etc/hosts文件，加入如下映射关系

```tex
192.168.211.143 rocketmq-nameserver1
192.168.211.143 rocketmq-master1
```

#### 1.5.2 安装配置

我们可以把安装文件上传到虚拟机上，并解压安装

解压文件存放到/usr/local/server/mq目录下

```shell
unzip rocketmq-all-4.6.1-bin-release.zip -d /usr/local/server/mq
```

更新解压后的文件名

```shell
mv rocketmq-all-4.6.1-bin-release rocketmq
```

创建RocketMQ存储文件的目录，执行如下命令

```shell
[root@localhost rocketmq]# mkdir logs
[root@localhost rocketmq]# mkdir store
[root@localhost rocketmq]# cd store
[root@localhost store]# mkdir commitlog
[root@localhost store]# mkdir consumerqueue
[root@localhost store]# mkdir index
```

文件夹说明

```shell
logs：存储RocketMQ日志记录
store：存储RocketMQ数据文件目录
commitlog：存储RocketMQ消息信息
consumerqueue、index：存储消息的索引数据
```

conf目录配置文件说明

```shell
2m-2s-async：2主2从异步
2m-2s-sync ：2主2从同步
2m-noslave ：2主没有从
```

我们这里先配置简单节点，可以修改2m-2s-async的配置实现。

进入2m-2s-async目录，修改第一个配置文件broker-a.properties

```shell
vi broker-a.properties
```

将如下配置覆盖掉broker-a.properties所有配置

```properties
# 所属集群名字
brokerClusterName=rocketmq-cluster

# broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a

# 0 表示Master，> 0表示slave
brokerId=0

# nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver:9876

# 在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4

# 是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

# 是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

# Broker对外服务的监听端口
listenPort=10911

# 删除文件时间点，默认是凌晨4点
deleteWhen=04

# 文件保留时间，默认48小时
fileReservedTime=120

# commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824

# ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000

# destroyMapedFileIntervalForcibly=120000
# redeleteHangedFileInterval=120000
# 检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88

# 存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store

# commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog

# 消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue

# 消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index

# checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint

# abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort

# 限制的消息大小
maxMessageSize=65536

# flushCommitLogLeastPages=4
# flushConsumerQueueLeastPages=2
# flushCommitLogThoroughInterval=10000
# flushConsumerQueueThorougnInterval=60000
# Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=ASYNC_MASTER

# 刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH

# checkTransactionMessageEnable=false
# 发消息线程池数量
# sendMessageThreadPoolNums=128
# 拉消息线程池数量
# pullMessageThreadPoolNums=128
```

进入conf目录，替换所有xml中的${user.name}，保证日志路径正确

```shell
sed -i 's#${user.home}#/usr/local/server/mq/rocketmq#g' *.xml
```

注意：sed -i 在这里起一个批量替换的作用

```shell
sed -i 's#原字符串#新字符#g' 替换的文件
```

RocketMQ对内存要求比较高，最少1G，如果内存太少，会影响RocketMQ的运行效率和执行性能，我们需要修改bin目录小的runbroker.sh和runserver.sh文件

runbroker.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g Xmn4g"
改后：
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g Xmn1g"
```

runserver.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server Xms4g Xmx4g Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
改后：
JAVA_OPT="${JAVA_OPT} -server Xms1g Xmx1g Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

先启动namesrv

```shell
nohup sh mqnamesrv &
```

再启动broker

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-async/broker-a.properties > /dev/null 2>&1 &
```

输入jps查看进程

```shell
[root@localhost rocketmqlogs]# jps
3225 NamesrvStartup
3290 BrokerStartup
3454 Jps
```

#### 1.5.3 RocketMQ控制台安装

我们这里直接将RocketMQ控制代码的源码放到项目中运行，后面课程结束后我们直接运行jar包即可。RocketMQ的控制台由一些RocketMQ爱好者提供。

下载地址：https://github.com/apache/rocketmq-externals/tree/master

### 1.6 RocketMQ架构介绍

![image-20200313142941806](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200313142941806.png)

上面是RocketMQ的部署结构图，RocketMQ 网络部署特点：

```shell
1、Name Server 是一个几乎无状态节点，可集群部署，节点乀间无任何信息同步。
2、Broker部署相对复杂，Broker分为 Master与Slave，一个 Master可以对应多个Slave，但是一个Slave 只能对应一个Master，Master与Slave的对应关系通过指定相同的BrokerName，不同的BrokerId来定义，BrokerId为0表示Master，非0表示Slave，Master也可以部署多个，每个Broker与Name Server 集群中的所有节点建立长连接，定时注册 Topic信息到所有Name Server。
3、Producer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的Master建立长连接，且定时向Master发送心跳。Producer完全无状态，可集群部署。
4、Consumer与Name Server集群中的其中一个节点（随机选择）建立长连接，定期从Name Server取Topic路由信息，并向提供Topic服务的 Master、Slave 建立长连接，且定时向Master、Slave 发送心跳。Consumer既可以从Master订阅消息，也可以从Slave订阅消息，订阅规则由Broker配置决定。
```

## 第二章 RocketMQ快速入门

### 2.1 消息生产和消费介绍

使用RocketMQ可以发送普通消息、顺序消息、事务消息，顺序消息能实现有序消费，事务消息可以解决分布式事务实现数据最终一致性。

RocketMQ有2中常见的消费模式，分别是DefaultMQPushConsumer和DefaultMQPullConsumer模式，这2种模式字面理解一个是推送消息，一个是拉取消息，这里有个误区，其实无论是Push还是Pull，其本质都是拉取消息，只是实现机制不一样。

DefaultMQPushConsumer其实并不是broker主动向consumer推送消息，而是consumer向broker发出请求，保持了一种长连接，broker会每5秒检测一次是否有消息，如果有消息，则将消息推送给consumer。使用DefaultMQPushConsumer实现消费，broker会主动记录消息消费的偏移量。

DefaultMQPullConsumer是消费方主动去broker拉取数据，一般会在本地使用定时任务实现，使用它获得消息状态方便、负载均衡性能可控，但消息的及时性差，而且需要手动记录消息消费的偏移量信息，所以在实际应用场景中推荐使用Push方式。

RocketMQ发送的消息默认会存储到4个队列中去，当然创建几个队列存储数据，可以自己定义。

![image-20200313161014296](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200313161014296.png)

RocketMQ作为MQ消息中间件，ack机制必不可少，在RocketMQ中常见的应答状态如下：

```java
LocalTransactionState:主要针对事务消息的应答状态

public enum LocalTransactionState {
	// 消息提交
	COMMIT_MESSAGE,
	// 消息回滚
	ROLLBACK_MESSAGE,
	// 未知状态，一般用于处理超时等现象
	UNKNOWN;
}
```

```java
ConsumeConcurrentlyStatus:主要针对消息消费的应答状态

public enum ConsumeConcurrentlyStatus {
	// 消息消费成功
	CONSUME_SUCCESS,
	// 消息重试，一般消息消费失败后，RocketMQ为了保障数据的可靠性，具有重试机制
	RECONSUME_LATER;
}
```

重发时间是：(broker.log中有)

```xml
messageDelayLevel=1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
```

### 2.2 RocketMQ普通消息生产者

#### 2.2.1 工程创建

我们实现一个最基本的消息发送，先创建一个springboot工程，工程名叫rocketmq-demo1

pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.leo</groupId>
    <artifactId>study</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>study</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <rocketmq.version>4.6.1</rocketmq.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- rocket mq -->
        <dependency>
            <groupId>org.apache.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>${rocketmq.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

#### 2.2.2 消息发送

消息发送有这么几个步骤：

```java
// 1、创建DefaultMQProducer
// 2、设置Namesrv地址
// 3、开启DefaultMQProducer
// 4、创建消息Message
// 5、发送消息
// 6、关闭DefaultMQProducer
```

我们创建一个Producer类，按照上面步骤实现消息发送，代码如下：

```java
public class Producer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("demo_producer_group");

        // 2、设置Namesrv地址
        producer.setNamesrvAddr("");

        // 3、开启DefaultMQProducer
        producer.start();

        // 4、创建消息Message
        Message message = new Message("Topic_Demo", "Tags", "Keys_1", "Hello RocketMQ".getBytes());

        // 5、发送消息
        SendResult result = producer.send(message);
        System.out.println(result);

        // 6、关闭DefaultMQProducer
        producer.shutdown();
    }
}
```

### 2.3 RocketMQ普通消息消费者

#### 2.3.1消息消费

消费者消费消息有这么几个步骤：

```java
 // 1、创建DefaultMQPushConsumer
 // 2、设置namesrv地址
 // 3、设置消息拉取最大数
 // 4、设置subscribe，这里是要读取订单主题信息
 // 5、创建消息监听MessageListener
 // 6、获取消息信息
 // 7、返回消息读取状态
```

创建Consumer类，按照上面步骤实现消息消费，代码如下：

```java
public class Consumer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 4、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_Demo",
                    "*");

        // 5、创建消息监听MessageListener
        consumer.setMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            // 迭代消息
            for (MessageExt msg : list) {
                // 获取主题
                String topic = msg.getTopic();
                // 获取标签
                String tags = msg.getTags();
                // 获取信息
                try {
                    // 6、获取消息信息
                    String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("Consumer消费信息----topic: " + topic + ", tags: " + tags + ", result: " + result);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    // 消息重试
                    return  ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }

            }

            // 7、返回消息读取状态
            // 消息消费成功
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

### 2.4 RocketMQ顺序消息

消息有序指的是可以按照消息的发送顺序来消费。RocketMQ可以严格的保证消息顺序，但这个顺序，不是全局顺序，只是分区(queue)顺序，要全局顺序只能一个分区。

如何保证顺序

```java
在MQ模型中，顺序需要由3个阶段去保证：
1、消息被发送时保证顺序
2、消息被存储时保持和发送的顺序一致
3、消息被消费时保持和存储的顺序一致
```

发送时保持顺序意味着对于有顺序要求的消息，用户应该在同一线程中采用同步的方式发送。存储保持和顺序一致则要求在同一线程中被发送出来的消息A和B，存储时在空间上A一定在B之前。而消费保持和存储一致则要求消息A、B到达Consumer之后必须按照先A后B的顺序被处理。

创建一个OrderProducer类，按照步骤实现消息发送，代码如下：

```java
public class OrderProducer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("demo_producer_order_group");

        // 2、设置Namesrv地址
        producer.setNamesrvAddr("");

        // 3、开启DefaultMQProducer
        producer.start();

        // 5、发送消息
        // 第一个参数：发送的消息信息
        // 第二个参数：选择指定的消息队列对象(会将所有的消息队列传进来)
        // 第三个参数：指定对应的队列下表

        // 连续发送5条消息
        for (int i = 0; i < 5; i++) {

            // 4、创建消息Message
            // topic：主题，tags: 标签，主要用于消息过滤，keys：消息的唯一值，body：消息体
            Message message = new Message(
                    "Topic_Order_Demo",
                    "Tags", "Keys_" + i,
                    ("Hello RocketMQ_" + i).getBytes(RemotingHelper.DEFAULT_CHARSET)
            );


            SendResult result = producer.send(
                    message,
                    (List<MessageQueue> list, Message msg, Object arg) -> {
                        // 获取队列的下标
                        Integer index = (Integer) arg;
                        return list.get(index);
                    },
                    1
            );
            System.out.println(result);
        }

        // 6、关闭DefaultMQProducer
        producer.shutdown();
    }
}
```

创建一个OrderConsumer类，按照步骤实现消息消费，代码如下：

```java
public class OrderConsumer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_order_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 4、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_Order_Demo",
                    "*");

        // 5、创建消息监听MessageListener
        consumer.setMessageListener((MessageListenerOrderly) (list, consumeOrderlyContext) -> {

            // 迭代消息
            for (MessageExt msg : list) {
                // 获取主题
                String topic = msg.getTopic();
                // 获取标签
                String tags = msg.getTags();
                // 获取信息
                try {
                    // 6、获取消息信息
                    String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("Order Consumer消费信息----topic: " + topic + ", tags: " + tags + ", result: " + result);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    // 消息重试
                    return  ConsumeOrderlyStatus.SUSPEND_CURRENT_QUEUE_A_MOMENT;
                }
            }

            return ConsumeOrderlyStatus.SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

### 2.5 RocketMQ事务消息

在RocketMQ4.3.0版本后，开放了事务消息这一特性，对于分布式事务而言，最常说的还是两阶段提交协议。

#### 2.5.1 RocketMQ事务消息流程

RocketMQ的事务消息，主要是通过消息的异步处理，可以保证本地事务和消息发送同时成功执行或失败，从而保证数据的最终一致性，这里我们先看看一条事务消息从诞生到结束的整个时间线流程：

![image-20200314162145933](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200314162145933.png)

#### 2.5.2 事务消息生产者

我们创建一个事务消息生产者TransactionProducer，事务消息发送消息对象是TransactionMQProducer，为了实现本地事务操作和回查，我们需要创建一个监听器，监听器需要实现TransactionListener接口，实现步骤如下：

```java
// 1、创建TransactionMQProducer
// 2、设置Namesrv地址
// 3、指定消息监听对象，用于执行本地事务和消息回查
// 4、指定线程池
// 5、开启TransactionMQProducer
// 6、创建消息Message
// 7、发送事务消息
// 8、关闭TransactionMQProducer
```

TransactionProducer代码如下：

```java
public class TransactionProducer {

    public static void main(String[] args) throws Exception {

        // 1、创建TransactionMQProducer
        TransactionMQProducer producer = new TransactionMQProducer("demo_producer_transaction_group");

        // 2、设置Namesrv地址
        producer.setNamesrvAddr("");

        // 3、指定消息监听对象，用于执行本地事务和消息回查
        producer.setTransactionListener(new TransactionListenerImpl());

        // 4、指定线程池
        ExecutorService executorService = new ThreadPoolExecutor(
                2,
                5,
                100,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(2000),
                r -> {
                    Thread thread = new Thread(r);
                    thread.setName("client-transaction-msg-check-thread");
                    return thread;
                }
        );
        producer.setExecutorService(executorService);

        // 5、开启TransactionMQProducer
        producer.start();

        // 6、创建消息Message
        // topic：主题，tags: 标签，主要用于消息过滤，keys：消息的唯一值，body：消息体
        Message message = new Message(
                "Topic_Transaction_Demo",
                "Tags",
                "Keys_T",
                "Hello Transaction RocketMQ Message".getBytes(RemotingHelper.DEFAULT_CHARSET)
        );

        // 7、发送事务消息
        // 第一个参数：发送的消息信息
        // 第二个参数：选择指定的消息队列对象(会将所有的消息队列传进来)

        TransactionSendResult result = producer.sendMessageInTransaction(message, "hello-transaction");
        System.out.println(result);

        // 8、关闭TransactionMQProducer
        producer.shutdown();
    }
}
```

监听器代码如下：

```java
public class TransactionListenerImpl implements TransactionListener {

    /**
     * 存储对应事务的状态信息，key：事务ID，value：当前事务执行的状态
     */
    private ConcurrentHashMap<String, Integer> localTrans = new ConcurrentHashMap<>();

    /**
     * 执行本地事务
     * @param message
     * @param o
     * @return
     */
    @Override
    public LocalTransactionState executeLocalTransaction(Message message, Object o) {

        // 事务ID
        String transactionId = message.getTransactionId();

        // 0：执行中，状态未知，1：本地事务执行成功，2：本地事务执行失败
        localTrans.put(transactionId, 0);

        // 业务执行，处理本地事务
        System.out.println("Hello-----Transaction");

        try {

            System.out.println("正在执行本地事务");
            Thread.sleep(2000);
            System.out.println("本地事务执行成功");

            localTrans.put(transactionId, 1);
        } catch (InterruptedException e) {
            e.printStackTrace();
            localTrans.put(transactionId, 2);
            return LocalTransactionState.ROLLBACK_MESSAGE;
        }

        return LocalTransactionState.COMMIT_MESSAGE;
    }

    /**
     * 消息回查
     * @param messageExt
     * @return
     */
    @Override
    public LocalTransactionState checkLocalTransaction(MessageExt messageExt) {

        // 获取事务ID
        String transactionId = messageExt.getTransactionId();

        // 获取对应事务ID的执行状态
        Integer status = localTrans.get(transactionId);

        System.out.println("消息回查--TransactionId：" + transactionId + ", 状态：" + status);

        switch (status) {
            case 0:
                return LocalTransactionState.UNKNOW;
            case 1:
                return LocalTransactionState.COMMIT_MESSAGE;
            case 2:
                return LocalTransactionState.ROLLBACK_MESSAGE;
        }

        return LocalTransactionState.UNKNOW;
    }
}
```

#### 2.5.3 事务消息消费者

事务消息的消费者和普通消费者一样，代码如下：

```java
public class TransactionConsumer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_transaction_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);
        
        // 设置消息顺序
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        // 4、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_Transaction_Demo",
                    "*");

        // 5、创建消息监听MessageListener
        consumer.registerMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            for (MessageExt msg: list) {
                try {
                    String topic = msg.getTopic();
                    String tags = msg.getTags();
                    String keys = msg.getKeys();
                    String body = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("topic: " + topic + ", tags: " + tags + ", keys: " + keys + ", body: " + body);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

#### 2.5.4 RocketMQ实现分布式事务流程

MQ事务消息解决分布式事务问题，但第三方MQ支持事务消息的中间件不多，比如RocketMQ，它支持事务消息的方式也是类似于二阶段提交，但是市面上一些主流的MQ都是不支持事务消息的，比如RabbitMQ和Kafka都不支持。

以阿里的RocketMQ中间件为例，其思路大致为：

第一阶段Prepared消息，会拿到消息的地址。

第二阶段执行本地事务。

第三阶段通过第一阶段拿到的地址去访问消息，并修改状态。

也就是说在业务方法内要向消息队列提交两次请求，一发送消息和一次确认消息。如果确认消息发送失败了，RocketMQ会定期扫描消息集群中的事务消息，这时候发现了Prepared消息，它会向消息发送者确认，所以生产方需要实现一个check接口，RocketMQ会根据发送端设置的策略来决定是回滚还是继续发送确认消息。这样就保证了消息发送与本地事务同时成功或同时失败。

![image-20200315115557057](/Users/huojunlin/Library/Application Support/typora-user-images/image-20200315115557057.png)

### 2.6 消息广播/批量发送

上面发送消息，我们测试的时候，可以发现消息只有一个消费者能收到，如果我们想实现消息广播，让每个消费者都能收到消息也是可以实现的。而且上面发消息的时候，每次都是发送单挑Message对象，能否批量发送呢？答案是可以的。

#### 2.6.1 消息生产者

创建消息生产者BroadcastingProducer，代码如下：

```java
public class BroadcastingProducer {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQProducer
        DefaultMQProducer producer = new DefaultMQProducer("demo_producer_broadcasting_group");

        // 2、设置Namesrv地址
        producer.setNamesrvAddr("");

        // 3、开启DefaultMQProducer
        producer.start();

        // 4、创建消息Message
        // topic：主题，tags: 标签，主要用于消息过滤，keys：消息的唯一值，body：消息体
        List<Message> messages = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            Message message = new Message(
                    "Topic_broadcasting_Demo",
                    "Tags",
                    "Keys_" + i,
                    ("Hello RocketMQ_" + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
            messages.add(message);
        }


        // 5、发送消息
        SendResult result = producer.send(messages);
        System.out.println(result);

        // 6、关闭DefaultMQProducer
        producer.shutdown();
    }
}
```

#### 2.6.2 消息消费者

BroadcastingConsumerA代码如下：

```java
public class BroadcastingConsumerA {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_broadcasting_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、默认是集群模式，改成广播模式
        consumer.setMessageModel(MessageModel.BROADCASTING);

        // 4、、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 5、、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_broadcasting_Demo",
                    "*");

        // 6、创建消息监听MessageListener
        consumer.setMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            // 迭代消息
            for (MessageExt msg : list) {
                // 获取主题
                String topic = msg.getTopic();
                // 获取标签
                String tags = msg.getTags();
                // 获取信息
                try {
                    // 7、获取消息信息
                    String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("Consumer_A消费信息----topic: " + topic + ", tags: " + tags + ", result: " + result);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    // 消息重试
                    return  ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }

            }

            // 8、返回消息读取状态
            // 消息消费成功
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

BroadcastingConsumerB代码如下：

```java
public class BroadcastingConsumerB {

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_broadcasting_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、默认是集群模式，改成广播模式
        consumer.setMessageModel(MessageModel.BROADCASTING);

        // 4、、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 5、、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_broadcasting_Demo",
                "*");

        // 6、创建消息监听MessageListener
        consumer.setMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            // 迭代消息
            for (MessageExt msg : list) {
                // 获取主题
                String topic = msg.getTopic();
                // 获取标签
                String tags = msg.getTags();
                // 获取信息
                try {
                    // 7、获取消息信息
                    String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("Consumer_B消费信息----topic: " + topic + ", tags: " + tags + ", result: " + result);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    // 消息重试
                    return  ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }

            }

            // 8、返回消息读取状态
            // 消息消费成功
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

##第三章 RocketMQ集群

### 3.1 RocketMQ集群模式

#### 3.1.1 单个Master

这是一种风险比较大的方式（无集群），因为一旦Broker重启或者宕机期间，将会导致整个服务不可用，因此是不建议线上环境去使用的。

#### 3.1.2 多个Master

一个集群全部都是Master，没有Slave，它的有点和缺点如下：

优点：配置简单，单个Master宕机或者重启维护对应用没有什么影响，在磁盘配置为RAID10时，即使机器宕机不可恢复的情况下，消息也不会丢失（异步刷盘会丢失少量消息，同步刷盘则是一条都不会丢失），性能最高。

缺点：当单个Broker宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息的实时性会收到影响。

#### 3.1.3 多Master多Slave模式-异步复制

每个Master配置一个Slave，有多对的Master-Slave，HA采用的是异步复制方式，主备有短暂的消息延迟，毫秒级别的（Master收到消息之后立刻向应用返回成功标识，同时向Slave写入消息），优缺点如下：

优点：即使是磁盘损坏了，消息丢失的非常少，且消息实时性不受影响，因为Master宕机之后，消费者仍可以从Slave消费，此过程对应用透明，不需要人工干预，性能同多个Master模式几乎一样。

缺点：Master宕机，磁盘损坏的情况下，会丢失少量的消息。

#### 3.1.4 多Master多Slave模式-同步双写

每个Master配置一个Slave，有多对的Master-Slave，HA采用的是同步双写模式，主备都写成功，才会向应用返回成功。

优点：数据和服务都无单点，Master宕机的情况下，消息无延迟，服务可用性与数据可用性都非常高。

缺点：性能比异步复制模式略低，大约低10%左右，发送单个Master的RT会略高，目前主机宕机后，Slave不同自动切换为主机，后续会支持自动切换功能。

### 3.2 RocketMQ主从搭建

#### 3.2.1 环境准备

我们先准备2台centos虚拟机，ip：192.168.211.142/192.168.211.143，在hosts文件中配置地址与IP的映射关系。

|       IP        |       hostname       |       mastername        |
| :-------------: | :------------------: | :---------------------: |
| 192.168.211.142 | rocketmq-nameserver2 | rocketmq-master1-slave1 |
| 192.168.211.143 | ocketmq-nameserver1  |    rocketmq-master1     |

修改2台机器的/etc/hosts文件，加入如下映射关系：

```shell
192.168.211.143 rocketmq-nameserver1
192.168.211.143 rocketmq-master1
192.168.211.142 rocketmq-nameserver2
192.168.211.142 rocketmq-master1-slave1
```

#### 3.2.2 安装配置(slave)

由于之前已经安装了一台RocketMQ，所以我们只需要安装从节点(192.168.211.142)即可。我们可以把安装文件上传到虚拟机上，并解压安装。

从192.168.211.143目录下将RocketMQ拷贝到192.168.211.142服务器上，如下代码：

```shell
scp rocketmq-all-4.6.1-bin-release.zip 192.168.211.142:/usr/local/server/sofeware
```

解压文件存放到/usr/local/server/mq目录下：

```shell
unzip rocketmq-all-4.6.1-bin-release.zip -d /usr/local/server/mq
```

更改j解压后的文件名：

```shell
mv rocketmq-all-4.6.1-bin-release rocketmq
```

创建RocketMQ存储文件的目录，执行如下命令：

```shell
[root@localhost rocketmq]# mkdir logs
[root@localhost rocketmq]# mkdir store
[root@localhost rocketmq]# cd store
[root@localhost store]# mkdir commitlog
[root@localhost store]# mkdir consumerqueue
[root@localhost store]# mkdir index
```

进入conf目录，替换所有xml中的${user.home}，保证日志路径正确

```shell
sed -i 's#${user.home}#/usr/local/server/mq/rocketmq#g' *.xml
```

注意：sed -i 在这里起一个批量替换的作用

```shell
sed -i 's#原字符串#新字符#g' 替换的文件
```

RocketMQ对内存要求比较高，最少1G，如果内存太少，会影响RocketMQ的运行效率和执行性能，我们需要修改bin目录小的runbroker.sh和runserver.sh文件

runbroker.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g Xmn4g"
改后：
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g Xmn1g"
```

runserver.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server Xms4g Xmx4g Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
改后：
JAVA_OPT="${JAVA_OPT} -server Xms1g Xmx1g Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

#### 3.2.3 主从配置

在192.168.211.143的/usr/local/server/mq/rocketmq/conf/2m-2s-async/目录下，将broker-a-s.properties文件修改成以下：

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示Master，>0表示slave
brokerId=1
#nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver:9876
#在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker对外服务的监听端口
listenPort=10911
#删除文件时间点，默认是凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store
#commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog
#消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue
#消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index
#checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint
#abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumerQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumerQueueThorougnInterval=60000
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=SLAVE
#刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

broker-a-s.properties文件和broker-a.properties文件区别并不大，主要改了brokerId=1和brokerRole=SLAVE

在192.168.211.143的/usr/local/server/mq/rocketmq/conf/2m-2s-async/目录下将配置文件拷贝到192.168.211.142服务器上，代码如下：

```shell
scp broker-a.properties broker-a-s.properties 192.168.211.142:/usr/local/server/mq/rocketmq/conf/2m-2s-async/
```

停止192.168.211.143服务

```shell
sh mqshutdown broker
sh mqshutdown namesrv
```

启动192.168.211.143的namesrv

```shell
nohup sh mqnamesrv &
```

启动192.168.211.142的namesrv

```shell
nohup sh mqnamesrv &
```

启动192.168.211.143的broker

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-async/broker-a.properties > /dev/null 2>&1 &
```

启动192.168.211.143的broker

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-async/broker-a-s.properties > /dev/null 2>&1 &
```

#### 3.2.4 控制台配置

修改rocketmq-console-ng控制台项目的application.properties配置文件，添加上主从地址：

```shell
rocketmq.config.namesrvAddr=192.168.211.143:9876;192.168.211.142:9876
```

运行控制台项目，并打开控制台的集群，可以看到集群信息。

#### 3.2.5 主从模式故障演练

主从模式，即使Master宕机后，消费者仍然可以从Slave消费，但不能接收新的消息，我们通过程序来演示一次该流程。

##### 3.2.5.1 master宕机收消息演示

创建MasterSlaveProducer，实现消息发送，代码如下

```java
public class MasterSlaveProducer {

    /**
     * 指定namesrv地址
     */
    private static String NAMESRV_ADDRESS = "192.168.211.143:9876;192.168.211.142:9876";

    public static void main(String[] args) throws Exception {

        // 创建一个DefaultMQProducer, 指定消息发送组
        DefaultMQProducer producer = new DefaultMQProducer("Test_Quick_Producer_Name");

        // 指定namesrv地址
        producer.setNamesrvAddr(NAMESRV_ADDRESS);

        // 启动producer
        producer.start();

        // 创建消息
        Message message = new Message(
                "Test_Quick_Topic",
                "TagA",
                "KeyA",
                "hello_rocketmq master-slave".getBytes(RemotingHelper.DEFAULT_CHARSET)
        );
        
        // 发送消息
        SendResult result = producer.send(message);
        System.out.println(result);
        
        // 关闭produccer
        producer.shutdown();
    }
}
```

执行消息发送后，可以打开控制台观察集群信息

停掉192.168.211.143的broker节点

```shell
./mqshutdown broker
```

创建MasterSlaveConsumer，实现消息消费，代码如下：

```java
public class MasterSlaveConsumer {

    /**
     * 指定namesrv地址
     */
    private static String NAMESRV_ADDRESS = "192.168.211.143:9876;192.168.211.142:9876";

    public static void main(String[] args) throws Exception {

        // 1、创建DefaultMQPushConsumer
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("demo_consumer_group");

        // 2、设置namesrv地址
        consumer.setNamesrvAddr("");

        // 3、设置消息拉取最大数
        consumer.setConsumeMessageBatchMaxSize(2);

        // 4、设置subscribe，这里是要读取订单主题信息
        // topic：指定消费的主题，subExpression：过滤规则
        consumer.subscribe("Topic_Demo",
                "*");

        // 5、创建消息监听MessageListener
        consumer.setMessageListener((MessageListenerConcurrently) (list, consumeConcurrentlyContext) -> {
            // 迭代消息
            for (MessageExt msg : list) {
                // 获取主题
                String topic = msg.getTopic();
                // 获取标签
                String tags = msg.getTags();
                // 获取信息
                try {
                    // 6、获取消息信息
                    String result = new String(msg.getBody(), RemotingHelper.DEFAULT_CHARSET);
                    System.out.println("Consumer消费信息----topic: " + topic + ", tags: " + tags + ", result: " + result);
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                    // 消息重试
                    return  ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }

            }

            // 7、返回消息读取状态
            // 消息消费成功
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });

        // 开启Consumer
        consumer.start();
    }
}
```

运行程序，发现消费者可以从slave中消费消息

##### 3.2.5.2 master宕机发消息演示

关掉192.169.211.143的broker后，再次尝试发送消息，会报如下错误：

```shell
Exception in thread "main" org.apache.rocketmq.client.exception.MQClientException:No route info of this topic, Test_Quick_Topic
...
```

### 3.3 RocketMQ集群搭建-双主双从

我们这里搭建一个双主双从的集群，采用同步复制异步刷盘方式进行集群，在工作中，我们也推荐这么做，我们先把环境准备一下。

#### 3.3.1 准备工作

准备4台机器，如下：

| ip              | hostname             | mastername             |
| --------------- | -------------------- | ---------------------- |
| 192.168.211.141 | rocketmq-nameserver1 | rocketmq-master1       |
| 192.168.211.142 | rocketmq-nameserver2 | rocketmq-master2       |
| 192.168.211.143 | rocketmq-nameserver3 | rocketmq-master1-slave |
| 192.168.211.144 | ocketmq-nameserver4  | rocketmq-master2-slave |

#### 3.3.2 RocketMQ安装

在每台机器上安装RocketMQ，安装过程和上面的**单节点RocketMQ安装流程**基本类似。

#### 3.3.3 修改映射路径

修改每台机器的/etc/hosts文件，添加如下映射路径

```shell
192.168.211.141 rocketmq-nameserver1
192.168.211.142 rocketmq-nameserver2
192.168.211.143 rocketmq-nameserver3
192.168.211.144 rocketmq-nameserver4
192.168.211.141 rocketmq-master1
192.168.211.142 rocketmq-master2
192.168.211.143 rocketmq-master1-slave
192.168.211.144 rocketmq-master2-slave
```

#### 3.3.4 RocketMQ节点配置

4台机器一起解压rocketmq压缩文件，最后将解压文件存放在/usr/local/server/mq/rockermq/目录下。

分别在4台机器的rocketmq目录下执行如下操作：

```shell
mkdir logs
mkdir store
cd store/
mkdir commitlog
mkdir consumequeue
mkdir index
```

在4台机器的conf目录下执行：

```shell
sed -i 's#${user.home}#/usr/local/server/mq/rocketmq#g' *.xml
```

进入4台机器的bin目录，修改runbroker.sh和runserver.sh

runbroker.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g Xmn4g"
改后：
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g Xmn1g"
```

runserver.sh

```shell
改前：
JAVA_OPT="${JAVA_OPT} -server Xms4g Xmx4g Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
改后：
JAVA_OPT="${JAVA_OPT} -server Xms1g Xmx1g Xmn1g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

进入141集群rocketmq/conf/2m-2s-sync目录下配置对应的配置文件，修改broker-a.properties配置文件，如下：

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示Master，>0表示slave
brokerId=0
#nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876;rocketmq-nameserver3:9876;rocketmq-nameserver4:9876
#在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker对外服务的监听端口
listenPort=10911
#删除文件时间点，默认是凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store
#commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog
#消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue
#消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index
#checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint
#abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumerQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumerQueueThorougnInterval=60000
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

修改broker-b.properties文件，如下：

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示Master，>0表示slave
brokerId=0
#nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876;rocketmq-nameserver3:9876;rocketmq-nameserver4:9876
#在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker对外服务的监听端口
listenPort=10911
#删除文件时间点，默认是凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store
#commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog
#消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue
#消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index
#checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint
#abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumerQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumerQueueThorougnInterval=60000
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=SYNC_MASTER
#刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

修改修改broker-a-s.properties文件，如下：

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a
#0 表示Master，>0表示slave
brokerId=1
#nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876;rocketmq-nameserver3:9876;rocketmq-nameserver4:9876
#在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker对外服务的监听端口
listenPort=10911
#删除文件时间点，默认是凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store
#commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog
#消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue
#消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index
#checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint
#abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumerQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumerQueueThorougnInterval=60000
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=SLAVE
#刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

修改修改broker-b-s.properties文件，如下：

```properties
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b
#0 表示Master，>0表示slave
brokerId=1
#nameServer地址，分号分隔
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876;rocketmq-nameserver3:9876;rocketmq-nameserver4:9876
#在发送消息时，自动创建服务器不存在的Topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许Broker自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许Broker自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker对外服务的监听端口
listenPort=10911
#删除文件时间点，默认是凌晨4点
deleteWhen=04
#文件保留时间，默认48小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumerQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumerQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/server/mq/rocketmq/store
#commitLog存储路径
storePathCommitLog=/usr/local/server/mq/rocketmq/store/commitlog
#消息队列存储路径
storePathConsumerQueue=/usr/local/server/mq/rocketmq/store/consumerqueue
#消息索引存储路径
storePathIndex=/usr/local/server/mq/rocketmq/store/index
#checkpoint文件存储路径
storeCheckpoint=/usr/local/server/mq/rocketmq/store/checkpoint
#abort文件存储路径
abortFile=/usr/local/server/mq/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumerQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumerQueueThorougnInterval=60000
#Broker的角色
# - ASYNC_MASTER 异步复制Master
# - SYNC_MASTER  同步双写Master
# - SLAVE
brokerRole=SLAVE
#刷盘方式
# - ASYNC_FLUSH 异步刷盘
# - SYNC_FLUSH  同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

将这4个文件分别复制到142、143、144服务器对应的目录下（rocketmq/conf/2m-2s-sync）

#### 3.3.5 RocketMQ集群启动测试

bin目录下，

先在每台服务器启动namesrv

```shell
nohup sh mqnamesrv &
```

再启动broker

141服务器执行如下启动命令：

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-sync/broker-a.properties > /dev/null 2>&1 &
```

142服务器执行如下启动命令：

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-sync/broker-b.properties > /dev/null 2>&1 &
```

143服务器执行如下启动命令：

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-sync/broker-a-s.properties > /dev/null 2>&1 &
```

144服务器执行如下启动命令：

```shell
nohup sh mqbroker -c /usr/local/server/mq/rocketmq/conf/2m-2s-sync/broker-b-s.properties > /dev/null 2>&1 &
```

输入jps查看进程

```shell
[root@localhost bin]# jps
43761 NamesrvStartup
43803 BrokerStartup
44093 jps
[root@localhost bin]#
```

启动后，需改控制台的namesrv地址如下：

```properties
rocketmq.config.namesrvAddr=192.168.211.141:9876;192.168.211.142:9876;192.168.211.143:9876;192.168.211.144:9876
```


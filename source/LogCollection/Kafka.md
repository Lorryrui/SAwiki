# Kafka配置与使用

## 一、简要说明

 Apache kafka是消息中间件的一种。以下是Kafka中常见的名词：

 ● producer ：生产者，产生消息。

 ● consumer：消费者，消费消息。

 ● topic：每一类的消息称之为一个主题(Topic)

 ● broker：存放消息的节点

 Kafka在大数据方面有广泛的应用。使用Kafka构建实时的数据流通道，利用Kafka本身的高吞吐量的特点，实现上下游的数据转换。

## 二、配置可用单节点Kafka实例

 这里以 [kafka_2.11-2.0.0](https://archive.apache.org/dist/kafka/2.0.0/kafka_2.11-2.0.0.tgz) 为例，说明在不同的环境下Kafka的配置与使用。

### 2.1 下载包

 在官网下载相应版本的二进制包，同时也要下载Zookeeper。zookeeper主要保存了kafka的topic、consumer等信息，旧版本的kafka消费者是通过连接zookeeper进行消费的。这里下载的是zookeeper-3.4.9。

### 2.2 Zookeeper配置与使用

 二进制Zookeeper包解压之后，bin目录存放的是一些使用脚本，conf目录存放了主要的配置文件。默认config/zoo_sample.cfg文件名修改为zoo.cfg。

```shell
dataDir=/soft/zookeeper-3.4.9/data # zookeeper数据文件存放路径
clientPort=2181 # zookeeper启动端口
```

 bin目录下常用的是zkServer.sh和zkCli.sh脚本。

```shell
./zkServer.sh start # 启动Zookeeper服务端
./zkCli.sh -server ip:port # 启动zookeeper客户端，用于查看zookeeper信息
```

 Zookeeper是常用的配置存放工具。分布式系统中会使用Zookeeper保存配置信息，Java应用程序连接Zookeeper实现程序配置的热加载。类似的工具还有ETCD，详细的介绍可官网。这里连接zookeeper可以查看到当前Kafka实例中Topic的信息：

```shell
[zk: localhost:2181(CONNECTED) 2] ls /config/topics
[hq-useraccess, datacollectlog, hq-vss, __consumer_offsets]
```

### 2.3 Kafka的配置与使用

 和Zookeeper类似，二进制包中bin目录存放的是一些使用脚本，conf目录存放了主要的配置文件。Kafka服务端的配置文件是conf/server.properties。

```shell
broker.id=0 # kafka实例唯一id
# #############################
# 配置公网kafka实例，同时内网消费者使用内网IP访问
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://221.112.112.111:9092
advertised.host.name=bird-web-middleware.novalocal
# ##############################
num.network.threads=3
num.io.threads=4
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/media/kafka_2.11-2.0.0/data # 消息存储的路径
num.partitions=3
num.recovery.threads.per.data.dir=1
log.retention.hours=72 # 消息保存时间，默认168小时
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=192.168.1.4:2181 # zookeeper地址
zookeeper.connection.timeout.ms=6000
```

 kafka后台启动

```shell
./kafka-server-start.sh -daemon ../config/server.properties
```

### 2.4 Kafka管理与监控

 kafka二进制包bin目录下有很多命令行下管理工具，这里介绍的是非官方的三方可视化的管理工具。

#### ● Kafka Manager

 常用的Kafka管理工具，包含Topic的创建、删除、重新分区等操作。在控制台上也可以看到相应topic的消费组及消费情况(offset值)。具体使用方法可以参考：https://github.com/yahoo/kafka-manager 。kafka-manager配置独立的zookeeper地址用于存储不同kafka实例的信息，从而管理不同的kafka集群。

#### ● Kafka Tool

 一款C/S架构的Kafka UI Tool。具体使用方法可以参考：http://www.kafkatool.com/download.html 。使用Kafka Tool可以查看kafka实例的详细信息，并且可以可视化的看到kafka不同topic中的message信息。

 Kafka Tool也可以查看消费者的offset和lag的消息数。但是当kafka集群中数据过多时，这个工具有明显的卡顿。

## 三、配置高可用的Kafka集群

### 3.1 Kafka-Cluster说明

 日志在不同的网络环境下，有的需要走公网传输。搭建的kafka集群需要满足：支持内网和公网访问；Kafka集群作为日志搜集的中转，除了提供实时写入Elasticsearch，同时也可以使用Logstash将数据原样冷备到S3。

### 3.2 下载包

 Kafka版本：https://www.apache.org/dyn/closer.cgi?path=/kafka/2.2.0/kafka_2.11-2.2.0.tgz

 Zookeeper版本：http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz

### 3.3 配置与优化

#### 3.3.1 Zookeeper

> zookeeper-3.4.14/conf/zoo.cfg

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
server.1=kafka-az3-0001:2888:3888
server.2=kafka-az3-0002:2888:3888
server.3=kafka-az3-0003:2888:3888
```

dataDir目录新增myid文件，不同机器id不同用于区分（0-255）

> 启动Zookeeper

```shell
# start/restart/stop/status
/opt/zookeeper-3.4.14/bin/zkServer.sh start
```

> 查看集群状态

```shell
# Leader主节点
/opt/zookeeper-3.4.14/bin/zkServer.sh status
```

#### 3.3.2 Kafka

> /opt/kafka_2.11-2.2.0/config/server.properties

```shell
broker.id=1 # 每个Node id唯一
listeners=PLAINTEXT://0.0.0.0:9092
advertised.listeners=PLAINTEXT://192.168.1.3:9092
advertised.host.name=kafka-az3-0003 # hostname;/etc/hosts需要配置
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
log.dirs=/data/kafka-logs # 消息存储路径
num.partitions=3
num.recovery.threads.per.data.dir=1
log.retention.hours=72
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
zookeeper.connect=192.168.1.1:2181,192.168.1.2:2181,192.168.1.3:2181
zookeeper.connection.timeout.ms=6000
```

> 启动

```shell
./kafka-server-start.sh -daemon ../config/server.properties
```

### 3.4 Kafka的内外网流量分离

 在Kafka实际使用过程中，有时候需要走公网生产消息，但是消费者是在内网。这时候如果流量全部走公网，除了带宽限制也会增加一部分流量成本。这时候需要配置外网流量走外网地址，内网走内网地址。Kafka服务器需要2张网卡或者使用云服务的弹性IP。这里以云服务器的弹性IP为例，修改配置如下：

```shell
listener.security.protocol.map=CLIENT:PLAINTEXT,REPLICATION:PLAINTEXT
advertised.listeners=CLIENT://192.33.10.241:9092,REPLICATION://120.13.81.201:19092
listeners=CLIENT://0.0.0.0:9092,REPLICATION://0.0.0.0:19092
inter.broker.listener.name=REPLICATION
```

## 四、 Kafka常见问答

> 1.kafka节点之间如何复制备份的？

 一个备份数量为n的集群允许n-1个节点失败。在所有备份节点中，有一个节点作为lead节点，这个节点保存了其它备份节点列表，并维持各个备份间的状体同步。

 Leader处理此分区的所有的读写请求，而follower被动的复制数据。（所有的写都发给leader, 然后leader将消息发给follower。）

> 2.kafka消息是否会丢失？为什么？

 通过request.required.acks属性进行配置：

 0代表：不进行消息接收是否成功的确认(默认值)；

 1代表：当Leader副本接收成功后，返回接收成功确认信息；

 -1代表：当Leader和Follower副本都接收成功后，返回接收成功确认信息。

想要更高的吞吐量就设置：异步、ack=0；想要不丢失消息数据就选：同步、ack=-1策略

> 3.kafka的leader选举机制是什么？

 Kafka会在Zookeeper上针对每个Topic维护一个称为ISR。如果某个分区的Leader不可用，Kafka就会从ISR集合中选择一个副本作为新的Leader。

> 4.如果所有的ISR副本都失败了怎么办？

 此时有两种方法可选，一种是等待ISR集合中的副本复活，一种是选择任何一个立即可用的副本，而这个副本不一定是在ISR集合中。这两种方法各有利弊，实际生产中按需选择。

如果要等待ISR副本复活，虽然可以保证一致性，但可能需要很长时间。而如果选择立即可用的副本，则很可能该副本并不一致。

> 5.kafka对硬件的配置有什么要求？

 网络吞吐量决定了Kafka能够处理的最大数据流量。它和磁盘存储是制约Kafka扩展规模的主要因素。

> 6.kafka的消息保证有几种方式？

 同步和异步。

## 五、附录

> Kafka教程

http://orchome.com/kafka/index

> Kafka Manager

https://github.com/yahoo/kafka-manager

> Kafka技术分享目录索引

https://blog.csdn.net/lizhitao/article/details/39499283
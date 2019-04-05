---
title: "Kafka"
layout: page
date: 2019-04-05 09:13
tag: Kafka
---

# Kafka配置与使用

## 一、简要说明

​    Apache kafka是消息中间件的一种。以下是Kafka中常见的名词：

​	 ● producer ：生产者，产生消息。

​	 ● consumer：消费者，消费消息。

​	 ● topic：每一类的消息称之为一个主题(Topic)

​	 ● broker：存放消息的节点

​    Kafka在大数据方面有广泛的应用。使用Kafka构建实时的数据流通道，利用Kafka本身的高吞吐量的特点，实现上下游的数据转换。

## 二、配置可用单节点Kafka实例

​    这里以 [kafka_2.11-2.0.0](https://archive.apache.org/dist/kafka/2.0.0/kafka_2.11-2.0.0.tgz) 为例，说明在不同的环境下Kafka的配置与使用。

### 2.1 下载包

​    在官网下载相应版本的二进制包，同时也要下载Zookeeper。zookeeper主要保存了kafka的topic、consumer等信息，旧版本的kafka消费者是通过连接zookeeper进行消费的。这里下载的是zookeeper-3.4.9。

### 2.2 Zookeeper配置与使用

​    二进制Zookeeper包解压之后，bin目录存放的是一些使用脚本，conf目录存放了主要的配置文件。默认config/zoo_sample.cfg文件名修改为zoo.cfg。

```shell
dataDir=/soft/zookeeper-3.4.9/data # zookeeper数据文件存放路径
clientPort=2181 # zookeeper启动端口
```

​    bin目录下常用的是zkServer.sh和zkCli.sh脚本。

```shell
./zkServer.sh start # 启动Zookeeper服务端
```

```shell
./zkCli.sh -server ip:port # 启动zookeeper客户端，用于查看zookeeper信息
```

​    Zookeeper是常用的配置存放工具。分布式系统中会使用Zookeeper保存配置信息，Java应用程序连接Zookeeper实现程序配置的热加载。类似的工具还有ETCD，详细的介绍可官网。这里连接zookeeper可以查看到当前Kafka实例中Topic的信息：

```shell
[zk: localhost:2181(CONNECTED) 2] ls /config/topics
[hq-useraccess, datacollectlog, hq-vss, __consumer_offsets]
```

###  2.3 Kafka的配置与使用

​    和Zookeeper类似，二进制包中bin目录存放的是一些使用脚本，conf目录存放了主要的配置文件。Kafka服务端的配置文件是conf/server.properties。

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

​    kafka后台启动

```shell
./kafka-server-start.sh -daemon ../config/server.properties
```

### 2.4 Kafka管理与监控

​    kafka二进制包bin目录下有很多命令行下管理工具，这里介绍的是非官方的三方可视化的管理工具。

#### ● Kafka Manager

​    常用的Kafka管理工具，包含Topic的创建、删除、重新分区等操作。在控制台上也可以看到相应topic的消费组及消费情况(offset值)。具体使用方法可以参考：[https://github.com/yahoo/kafka-manager](https://github.com/yahoo/kafka-manager) 。kafka-manager配置独立的zookeeper地址用于存储不同kafka实例的信息，从而管理不同的kafka集群。

![](..\attach\kafkamanager.png)    

#### ● Kafka Tool 

​    一款C/S架构的Kafka UI Tool。具体使用方法可以参考：[http://www.kafkatool.com/download.html](http://www.kafkatool.com/download.html) 。使用Kafka Tool可以查看kafka实例的详细信息，并且可以可视化的看到kafka不同topic中的message信息。

![](..\attach\kafkatooltopic.png)

​    Kafka Tool也可以查看消费者的offset和lag的消息数。但是当kafka集群中数据过多时，这个工具有明显的卡顿。

## 三、配置高可用的Kafka集群

## 四、附录

> Kafka教程

[http://orchome.com/kafka/index](http://orchome.com/kafka/index)

> Kafka Manager

https://github.com/yahoo/kafka-manager

> Kafka技术分享目录索引

[https://blog.csdn.net/lizhitao/article/details/39499283](https://blog.csdn.net/lizhitao/article/details/39499283)
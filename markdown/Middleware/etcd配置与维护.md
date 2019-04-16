---
title: "Middleware"
layout: page
date: 2019-04-15 10:12
tag: etcd
---

# etcd配置与维护

## 一、简介

​    etcd是一款k-v类型的数据库，主要用于分布式系统中的配置管理。目前比较流行的Kubernetes中就是以etcd作为统一配置管理的。etcd主要有v2和v3两种，官网上已经没有v2版本的包下载，如果需要的话可以去github上下载对应的release包。

> https://github.com/etcd-io/etcd

​    etcd提供的二进制下载包中，有etcd和etcdctl两个文件。etcd是程序可执行文件，etcdctl是操作etcd的命令行管理工具。

## 二、单节点配置

​    部署单节点的etcd比较简单，只需要去[https://github.com/etcd-io/etcd/releases](https://github.com/etcd-io/etcd/releases)下载对应的版本即可。这里以下载v3版本的二进制包为例。

```shell
# 启动
./etcd
```

​    启动后会在控制台上打印一些启动信息，如果需要后台启动，使用nohup。

## 三、集群部署

​    集群部署参考官网上 set-up-a-cluster

> https://coreos.com/etcd/docs/latest/demo.html#set-up-a-cluster

​    这里以部署三节点的etcd v3集群为例。

###     3.1 下载

​        下载对应版本etcd二进制包，这里下载当前最新的 v3.3.12

###     3.2 配置集群启动脚本

​        在三个节点的etcd目录里，新建一个start.sh脚本文件：

```shell
TOKEN=etcd-cluster # 加入etcd集群Token
CLUSTER_STATE=new
NAME_1=etcd-node1
NAME_2=etcd-node2
NAME_3=etcd-node3
HOST_1=10.15.0.8
HOST_2=10.15.0.9
HOST_3=10.15.0.10
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380

# 不同的节点这两个配置项不同
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
# #######################

nohup ./etcd --data-dir=data.etcd --name ${THIS_NAME} \
        --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://${THIS_IP}:2380 \
        --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://${THIS_IP}:2379 \
        --initial-cluster ${CLUSTER} \
        --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN} > /dev/null 2>error.log &
```

###     3.3 启动

```shell
./start.sh
```

## 四、etcd使用说明

###     4.1 访问与使用

​          etcd提供http和https两种访问方式，这里以http的etcd集群访问为例。

```shell
# 配置环境变量(/etc/profile)，供etcdctl访问
export ETCDCTL_API=3
export ENDPOINTS=10.15.0.8:2379,10.15.0.9:2379,10.15.0.10:2379
```

```shell
# 查看etcd集群信息
./etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status
# 写（put）
./etcdctl --endpoints=$ENDPOINTS put foo "Hello World!"
# 读（get）
./etcdctl --endpoints=$ENDPOINTS get foo
# 删 (delete)
./etcdctl --endpoints=$ENDPOINTS del key
```

​           更多操作命令，参考 https://coreos.com/etcd/docs/latest/demo.html#set-up-a-cluster

###      4.2 配置HTTP Auth

​              在生产环境中，需要配置用户名/密码或者以HTTPS访问etcd集群。仅支持HTTP形式的etcd集群根据需要配置不同的用户名/密码进行权限控制，详细的配置步骤参考

> https://coreos.com/etcd/docs/latest/demo.html#auth

###       4.3 备份与恢复

​		官方提供的备份方式有两种：snap和backup。旧版本使用的是backup，新版本则是snap。具体的可以参考etcdctl的详细说明

```shell
./etcdctl --help
# snap
        snapshot save           Stores an etcd node backend snapshot to a given file
        snapshot restore        Restores an etcd member snapshot to an etcd directory
        snapshot status         Gets backend snapshot status of a given file
```

​		 非官方的是Github上一些etcd的工具，主要是将k-v保存为json，然后再恢复到新的集群中。

> https://github.com/lavagetto/etcddump

ps. 这个功能有问题，使用该工具请参考：https://github.com/Lorryrui/etcddump
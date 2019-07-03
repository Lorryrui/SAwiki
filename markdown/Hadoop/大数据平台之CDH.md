---
title: "CDH"
layout: page
date: 2019-06-04 09:38
tag: Hadoop
---
# CDH Installation

## 一、简介

​    Cloudera版本（Cloudera’s Distribution Including Apache Hadoop，简称“CDH”），基于Web的用户界面,支持大多数Hadoop组件，包括HDFS、MapReduce、Hive、Pig、 Hbase、Zookeeper、Sqoop,简化了大数据平台的安装、使用难度。

## 二、组件介绍

​    CDH安装后部分组件介绍：

> HDFS

​    Hadoop分布式文件系统，用于存储大规模的数据文件。

> Hbase

​    分布式列式数据库，存储非结构化的数据。

> Hive

​    基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表。

> Oozie

​    Hadoop中的工作流调度系统，主要目的是为了管理不同类型的作业在Hadoop系统中处理。

> Sqoop

​    一款开源工具。用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递，可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

## 三、安装部署

### 3.1 集群说明

​    四台机器（CentOS Linux release 7.4）部署CDH 6.1.1集群，其中一台是管理节点。服务器信息如下： 

```
192.168.208.2  hadoop-01
192.168.208.3  hadoop-02
192.168.208.4  hadoop-03
192.168.89.47  cm-server-M
```

### 3.2 服务器初始化配置

#### 3.2.1 主机名修改

​    hostnamectl set-hostname hadoop-01

#### 3.2.2 /etc/hosts

```
192.168.208.2  hadoop-01
192.168.208.3  hadoop-02
192.168.208.4  hadoop-03
192.168.89.47  cm-server-M
```

#### 3.2.3 新增hadoop用户

​    增加hadoop用户，并添加到/etc/sudoers。

​        echo 'hadoop  ALL=(ALL)    NOPASSWD: ALL' >> /etc/sudoers

​    禁用root用户登录，配置CM节点以hadoop用户SSH免密登录其他服务器。

​        /home/hadoop/.ssh/authorized_keys 增加CM服务器公钥。

#### 3.2.4 配置时钟同步

​    yum -y install ntp；systemctl restart ntpd；systemctl enable ntpd.service

#### 3.2.5 JAVA环境

​    卸载系统中存在的JAVA环境；yum install oracle-j2sdk1.8

#### 3.2.6 操作系统配置调整

```shell
echo "* soft nofile 65535"  >> /etc/security/limits.conf
echo "* hard nofile 65535"  >> /etc/security/limits.conf
echo "fs.file-max = 6553560" >> /etc/sysctl.conf
echo "vm.swappiness=10" >>/etc/sysctl.conf
# 下面两段同时写入/etc/rc.local
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

#### 3.2.7 关闭防火墙、SElinux

#### 3.2.8 重启服务器

​    重启服务器，同时验证以上调整是否生效。

### 3.3  Cloudera Manager安装

​     Cloudera Manager是CDH集群安装管理的可视化工具。通过安装CM工具，可视化的安装、维护CDH提供的相关组件。

#### 3.3.1 配置yum源

> [https://www.cloudera.com/documentation/enterprise/6/6.1/topics/configure_cm_repo.html#cm_repo](https://www.cloudera.com/documentation/enterprise/6/6.1/topics/configure_cm_repo.html#cm_repo)

```shell
wget https://archive.cloudera.com/cm6/6.1.1/redhat7/yum/cloudera-manager.repo -P /etc/yum.repos.d/
rpm --import https://archive.cloudera.com/cm6/6.1.0/redhat7/yum/RPM-GPG-KEY-cloudera
```

#### 3.3.2 安装CM Server

```shell
yum install cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server -y
```

#### 3.3.3 安装数据库

> [https://www.cloudera.com/documentation/enterprise/6/6.1/topics/cm_ig_mysql.html](https://www.cloudera.com/documentation/enterprise/6/6.1/topics/cm_ig_mysql.html)

​    数据库用于存储一些组件的信息，这里使用MySQL 5.6。CM要求数据库字符集必须是utf8。

#### 3.3.4 安装MySQL驱动

> [https://www.cloudera.com/documentation/enterprise/6/6.1/topics/cm_ig_mysql.html#cmig_topic_5_5_3](https://www.cloudera.com/documentation/enterprise/6/6.1/topics/cm_ig_mysql.html#cmig_topic_5_5_3)

​    所有节点上均需要安装MySQL驱动：

```shell
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.46.tar.gz
tar zxvf mysql-connector-java-5.1.46.tar.gz
sudo mkdir -p /usr/share/java/
cd mysql-connector-java-5.1.46
sudo cp mysql-connector-java-5.1.46-bin.jar /usr/share/java/mysql-connector-java.jar
```

#### 3.3.5 创建数据库及用户

​    CM将不同的组件信息存储在不同的数据库中，建立相关的库和用户：

```shell
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'scm';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'amon';
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'rman';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'hue';
CREATE DATABASE metastore DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive';
CREATE DATABASE sentry DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON sentry.* TO 'sentry'@'%' IDENTIFIED BY 'sentry';
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'nav';
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'navms';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie';
flush privileges;
```

#### 3.3.6 Cloudera初始化脚本

​    运行下面的脚本，告诉cloudera 你建立的数据库用户密码等信息：

```shell
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm scm
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql amon amon amon
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql rman rman rman
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql hue hue hue
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql sentry sentry sentry
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql nav nav nav
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql navms navms navms
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql oozie oozie oozie
```

### 四、集群设置

​    systemctl restart cloudera-scm-server 重启CM服务端，等待1-2分钟打开链接 http://<cm_server>:7180进入WEB界面化配置CDH集群。根据需要选择需要添加的组件以及组件所在的节点。

​    ps：注意使用hadoop用户SSH连接其他节点。

### 五、报错说明

#### 5.1 JAVA环境报错

​    使用官方推荐的：yum -y install oracle-j2sdk1.8

#### 5.2 Oozie Web UI报jsp错误

```shell
cd /opt/cloudera/parcels/CDH/lib/oozie/embedded-oozie-server/dependency
unlink jetty-runner-9.3.20.v20170531.jar
unlink javax.servlet.jsp-api-2.3.1.jar
unlink javax.servlet.jsp-2.3.2.jar
```


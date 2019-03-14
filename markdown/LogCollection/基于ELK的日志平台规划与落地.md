---
title: "ELK"
layout: page
date: 2019-03-02 11:00
tag: ELK
---

# 基于ELK的日志平台规划与落地

## 一 简介

​    Elastic Stack是开源([www.elastic.co](www.elastic.co))的日志搜集解决方案，主要是解决分布式系统中日志集中管理。ELK是Elasticsearch、Logstash、Kibana的简称。目前Elastic Stack生态圈的组件很多，适用于各种不同的业务场景，如日志分析、指标分析、网站搜索、安全分析等。

​    搭建基于ELK的日志平台，主要是通过搜集程序日志，了解业务运行状况从而制定相应的调整策略。按照当前规划，制定的ELK整体架构图如下：

![](..\attach\elk stack.jpg)

## 二 Elasticsearch集群搭建

​    2.1 准备

​        Elasticsearch集群节点数量保证是奇数个，这里以3个ES节点为例。服务器初始环境的配置包括：

​           ·  JDK版本： 1.8

​           ·  操作系统文件打开数(/etc/security/limits.conf)：  

​	        echo "* soft nofile 65536"  >> /etc/security/limits.conf  
​		echo "* hard nofile 65536"  >> /etc/security/limits.conf  
​	    · /etc/sysctl.conf

​		fs.file-max = 6553560   

​		vm.max_map_count=262144	   

​	    · 服务器目录规划（目录权限属于elastic用户）

​                  /opt  elasticsearch程序、日志所在路径

​	          /data 新挂载1T盘，做LVM。ES的数据存储路径

​	     · 端口开放

​		   9200为ES启动默认端口；9300为ES集群间通讯端口。开放2个端口。

​	    重启服务器。

​     2.2 集群搭建（Elasticsearch 6.6.2）

​            2.2.1 安装包下载

​		[https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.tar.gz)

​		程序启动只能以非root账号，这里使用elastic

​	    2.2.2 集群配置

​		·  修改配置文件（/opt/elasticsearch-6.6.2/config/elasticsearch.yml）

```python
cluster.name: elk-cluster  
node.name: es-0001  
path.data: /data  
path.logs: /opt/elasticsearch-6.6.2/logs
network.host: 0.0.0.0  # 集群配置 0.0.0.0  
http.port: 9200  # ES服务端口  
discovery.zen.ping.unicast.hosts: ["192.168.1.2", "192.168.1.3", "192.168.1.4"] #参与选主ES
discovery.zen.minimum_master_nodes: 2 # 参与选主实例数/2 + 1    
```

​                  ·  修改JVM内存大小（/opt/elasticsearch-6.6.2/config/jvm.options）

```python
# 默认1G，调整为50%服务器内存
-Xms4g
-Xmx4g
```

​	2.3 Elasticsearch集群验证

```python
curl '192.168.1.2:9200/_cat/health?v'  # 查看节点状态  
curl '192.168.1.2:9200/_cat/nodes?v'   # 查看集群信息		  
```

## 三 Kibana配置与使用 

​    3.1 准备

​	· JDK版本 1.8

​	· 文件打开数配置，参考2.1

​    3.2 安装与配置（Kibana 6.6.2）

​        3.2.1 安装包下载（yum源）

​		[https://www.elastic.co/guide/en/kibana/current/rpm.html](https://www.elastic.co/guide/en/kibana/current/rpm.html)

​	3.2.2 配置连接Elasticsearch（/etc/kibana/kibana.yml）		

```python
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://192.168.1.2:9200","http://192.168.1.3:9200","http://192.168.1.4:9200"]
```

​	 3.2.3 Kibana服务管理

​		/bin/systemctl enable kibana.service
​		/bin/systemctl start kibana.service

​	3.3 Kibana的使用		

## 四 Elastic Stack 接入安全

4. 1Nginx配置HTTP访问加密

​    高版本Elasticsearch的X-pack插件官方已收费，目前ES集群未加密。Kibana由于会在公网使用，所以配置Nginx转发做简单密码认证。

​      4.1.1 Kibana配置调整

​	    server.host: "0.0.0.0" 修改为 server.host: "localhost"

​      4.1.2 生成登录的用户名/密码

```c
sh -c "echo -n 'logcollection:' >> /etc/nginx/.htpasswd"
sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

​      4.1.3 配置Nginx反向代理

```c
# 主要配置
location / {
             auth_basic "Welcome! Please login first";
             auth_basic_user_file /etc/nginx/.htpasswd;
             proxy_pass http://localhost:5601;
        }
```

​       4.1.4 访问Kibana

​	    打开Nginx访问端口，输入账号密码登录

![](..\attach\elklogin.png)




---
title: "ELK Stack"
layout: page
date: 2019-03-02 11:00
tag: ELK
---

# ELK Stack Introduction

## 一 简介

​    Elastic Stack是开源([www.elastic.co](www.elastic.co))的日志搜集解决方案，主要是解决分布式系统中日志集中管理。ELK是Elasticsearch、Logstash、Kibana的简称。目前Elastic Stack生态圈的组件很多，适用于各种不同的业务场景，如日志分析、指标分析、网站搜索、安全分析等。

​    搭建基于ELK的日志平台，主要是通过搜集程序日志，了解业务运行状况从而制定相应的调整策略。按照当前规划，制定的ELK整体架构图如下：![](..\attach\elk stack.jpg)

## 二 Elasticsearch集群搭建

​    2.1 准备

​        Elasticsearch集群节点数量保证是奇数个，这里以3个ES节点为例。服务器初始环境的配置包括：

​           ·  JDK版本： 1.8

​           ·  操作系统文件打开数(/etc/security/limits.conf)：  

​	        echo "* soft nofile 65536"  >> /etc/security/limits.conf  
​		echo "* hard nofile 65536"  >> /etc/security/limits.conf  
​	   · /etc/sysctl.conf

​		fs.file-max = 6553560   

​		vm.max_map_count=262144	   

​	   · 服务器目录规划（目录权限属于elastic用户）

​                  /opt  elasticsearch程序、日志所在路径

​	          /data 新挂载1T盘，做LVM。ES的数据存储路径

​	    · 端口开放

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

## 四 Logstash配置与使用

​    Logstash在ELK Stack中主要负责日志清洗，将源日志格式化输出到Elasticsearch中。Logstash的过滤插件很多，常见的如正则解析、json解析等。清洗之后的结果类似一对对k-v的键值对，在ES中可以方便的查询和做一些聚合的操作。官方插件地址：[https://www.elastic.co/guide/en/logstash/6.6/filter-plugins.html](https://www.elastic.co/guide/en/logstash/6.6/filter-plugins.html)

​    4.1 安装与配置说明

​        安装的方式有很多，对于CentOS 7以上的系统，推荐使用配置yum源的方式安装。低版本可以使用二进制包安装，使用nohup方式启动。

​	4.1.1 安装

​	    ·  java8环境

​            ·  配置yum源（[https://www.elastic.co/guide/en/logstash/current/installing-logstash.html](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html)）

​            ·  yum install logstash

​        4.1.2  配置（/etc/logstash/logstash.yml）

​	    Logstash配置文件中，涉及到数据处理的主要有三个部分：input -> filter -> output 。

​	    ·  input（[https://www.elastic.co/guide/en/logstash/6.6/input-plugins.html](https://www.elastic.co/guide/en/logstash/6.6/input-plugins.html)）

​	        源数据输入部分。支持的数据源格式可以参考官方文档中这部分的插件说明，目前最常见的是接受Beats的数据。Beat是清理级的日志收集组件，带一些简单的过滤功能，如只匹配ERROR行等。Filebeat可以将数据实时发送到Logstash，资源占用较小，对应用服务器压力不大。

​            ·  filter（[https://www.elastic.co/guide/en/logstash/6.6/filter-plugins.html](https://www.elastic.co/guide/en/logstash/6.6/filter-plugins.html)）    

​	        对源数据进行格式化处理。根据不同的日志格式做解析，如正则（grok）、IP地址解析（geoip）等插件。过滤之后，整合成类似k-v的数据。

​	    ·  output（[https://www.elastic.co/guide/en/logstash/6.6/output-plugins.html](https://www.elastic.co/guide/en/logstash/6.6/output-plugins.html)）

​		存储过滤之后的数据。支持的插件也很多，常用的如Kafka、S3、Elasticsearch等。ELK方案中主要是输出到Elasticsearch中。

​	  4.1.3  启动

​	      作为系统服务启动或者nohup /usr/share/logstash/bin/logstash -f logstash.conf >/dev/null 2>&1 &

​     4.2  Logstash使用demo -- 配置Logstash解析类Apache格式日志

​           4.2.1  编写解析规则，如/etc/logstash/apache.conf

```python
input {
    # Logstash读取的数据源，这里是kafka
    kafka {
        bootstrap_servers => "kafka1:9092 kafka2:9092 kafka3:9092"
        group_id => "ELK001"
        topics => ["AI_hxapi_accesslog"]
        consumer_threads => 3
        auto_offset_reset => "earliest"
        enable_auto_commit => "true"
        codec => "json"
    }
}
filter {
    grok {
        # apache标准日志解析规则
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    urldecode {
        # 对url的中文解码
        all_fields => true
    }
}

output {
        # 输出到Elasticsearch
    elasticsearch {
                 hosts => ["es1:9200","es2:9200","es3:9200"]
                 index => "hxapi_access_log-%{+yyyy.MM.dd}"
    }
}
```

​             4.2.2  调试自己的解析规则  

```python
input {
    # 从控制台输入要解析的数据
        stdin {}
}
filter{
        grok {
                match => {"message"=> "^<(?<log_type>[^>]+)><(?<appid>\-)><(?<timestamp>[^>]+)[^>\n]*><(?<thread_id>\d+)><(?<filename_line>\w+\.\w+:\d+)>\s+(?<stockinfo>.+)"}
                }
}

output {
    # 控制台输出解析结果
        stdout {}
        }
```

​	         logstash -f /etc.logstash/test.conf；启动完成后，在控制台上输入要解析的日志内容即可。

![](..\attach\elklogstash.jpg)

## 五  Filebeat配置与使用    

​    5.1 使用说明

​            Filebeat是轻量级的日志抽取工具。与Logstash相比，占用系统的资源比较少，适合全量日志或者简单过滤日志传输。新的架构里，可以用Filebeat抽取日志发送到Logstash，由Logstash进行日志清洗。这样做可以减轻业务程序所在服务器的负担，同时也保证清洗的完整性。

​    5.2  安装与配置说明

​            按照官网给的安装说明，配置yum源。配置参考：

```python
#=========================== Filebeat inputs =============================

filebeat.inputs:
- type: log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /opt/nginx/logs/app1_access.log
  tags: ["app1"] 
- type: log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /opt/nginx/logs/app2_access.log
  tags: ["app2"]
#-------------------------- Kafka output ------------------------------
output.kafka:
  # initial brokers for reading cluster metadata
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]

  # message topic selection + partitioning
  topic: mytopic

  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
  version: 0.10.0.0
#================================ Logging =====================================
# 配置Filebeat程序本身日志保存时长，避免日志量增长导致磁盘空间不足
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7  
  permissions: 0644
         
```

​    5.3  服务启动

​        systemctl start/stop/restart filebeat

## 六 Elastic Stack 接入安全

  6.1 Nginx配置HTTP访问加密

​      高版本Elasticsearch的X-pack插件官方已收费，目前ES集群未开启认证。Kibana由于会在公网使用，所以配置Nginx转发做简单密码认证。

​      6.1.1 Kibana配置调整

​	    server.host: "0.0.0.0" 修改为 server.host: "localhost"

​      6.1.2 生成登录的用户名/密码

```shell
sh -c "echo -n 'logcollection:' >> /etc/nginx/.htpasswd"
sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"
```

​      6.1.3 配置Nginx反向代理

```c
# 主要配置
location / {
             auth_basic "Welcome! Please login.";
             auth_basic_user_file /etc/nginx/.htpasswd;
             proxy_pass http://localhost:5601;
        }
```

​       6.1.4 访问Kibana

​	    打开Nginx访问端口，输入账号密码登录

![](..\attach\elklogin.png)

​       6.2  nginx-auth-ldap模块开启LDAP认证（推荐）

​	  6.2.1 编译安装Nginx Lua环境

> [https://github.com/openresty/lua-nginx-module#installation](https://github.com/openresty/lua-nginx-module#installation)

​              a. LuaJIT环境

​                   [https://github.com/openresty/luajit2/releases]( https://github.com/openresty/luajit2/releases)		    

```c
make && make install
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.1
```

​	      b. ngx_devel_kit (NDK)、ngx_lua 模块

> ​		    [https://github.com/simplresty/ngx_devel_kit/tags](		    https://github.com/simplresty/ngx_devel_kit/tags)
>
> ​		    https://github.com/openresty/lua-nginx-module/tags

​              c. 编译安装

```shell
yum install -y openldap-devel openssl openssl-devel gcc pcre pcre-devel gd gd-devel GeoIP GeoIP-devel provides geoip-devel
```

```shell
./configure --prefix=/opt/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module --with-ld-opt="-Wl,-rpath,/usr/local/lib" --add-module=/usr/local/nginx_module/ngx_devel_kit-0.3.0 --add-module=/usr/local/nginx_module/lua-nginx-module-0.10.14
make && make install
```

​           6.2.2 Nginx配置参考

```python
worker_processes  4;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;

    keepalive_timeout  65;

# LDAP config
    ldap_server mycorp {
        url ldap://ldapserver:389/DC=mycorp,DC=com,DC=cn?sAMAccountName?sub?(objectClass=person);
        binddn "my@mycorp.com.cn";
        binddn_passwd mypasswd;
        group_attribute uniquemember;
        group_attribute_is_dn on;
        require valid_user;
      }

    server {
        listen       15601;
        server_name  mydomain;
        auth_ldap "Welcome! Please login with  LDAP account";
        auth_ldap_servers mycorp;

        location / {
            proxy_pass http://localhost:5601;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```


---
title: "ELK"
layout: page
date: 2019-03-02 11:00
tag: ELK
---

# 基于ELK的日志平台规划与落地

## 一 简介

## 二 Elasticsearch集群搭建

​    2.1 准备

​        Elasticsearch集群节点数量保证是奇数个，这里以3个ES节点为例。服务器初始环境的配置包括：

​            JDK版本： 1.8

​            操作系统文件打开数(/etc/security/limits.conf)：  

​	        echo "* soft nofile 65535"  >> /etc/security/limits.conf  
​		echo "* hard nofile 65535"  >> /etc/security/limits.conf  
​		fs.file-max = 6553560 (/etc/sysctl.conf)  

​     2.2 集群搭建（Elasticsearch 6.6.1）

​            2.2.1 安装包下载

​		[https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.1.tar.gz](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.1.tar.gz)

​		程序启动只能以非root账号，这里使用elastic

​	    2.2.2 集群配置

​    

## 三 Elastic Stack 接入LDAP用户认证

1. 官方文档参考

​    [https://www.elastic.co/guide/en/elastic-stack-overview/current/ldap-realm.html]( https://www.elastic.co/guide/en/elastic-stack-overview/current/ldap-realm.html)

​    [https://www.elastic.co/guide/en/elastic-stack-overview/current/mapping-roles.html](    https://www.elastic.co/guide/en/elastic-stack-overview/current/mapping-roles.html)

2. 配置说明

   使用X-pack插件，该插件收费，需要license。

   




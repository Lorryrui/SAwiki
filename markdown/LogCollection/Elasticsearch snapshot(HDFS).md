---
title: "ELK stack"
layout: page
date: 2020-11-04 10:00
---

# Elasticsearch索引备份到HDFS 

## 创建仓库 -- hdfs RPC端口

PUT _snapshot/hw_hdfs_repository

{

 "type": "hdfs",

 "settings": {

  "uri": "hdfs://192.33.0.253:9820/",

  "path": "/elkHdfsRepo",

  "conf.dfs.client.read.shortcircuit": "false"

 }

}

或者

curl -XPUT "http://192.33.9.141:9200/_snapshot/hdfs_repository" -H 'Content-Type: application/json' -d'

{

 "type": "hdfs",

 "settings": {

  "uri": "hdfs://192.33.0.253:9820/",

  "path": "/elkHdfsRepo",

  "conf.dfs.client.read.shortcircuit": "false"

 }

}'



## 对指定索引进行备份

PUT _snapshot/hw_hdfs_repository/snapshot_k8s

{

 "indices": "k8s-2020.10.10",

"ignore_unavailable": true,

"include_global_state": false,

"partial": true

}

或者

curl -XPUT "http://192.33.9.141:9200/_snapshot/hdfs_repository/snapshot_k8s" -H 'Content-Type: application/json' -d'

{

 "indices": "k8s-2020.10.10",

"ignore_unavailable": true,

"include_global_state": false,

"partial": true

}'



## 恢复指定索引

POST /_snapshot/hw_hdfs_repository/snapshot_k8s/_restore

{

  "indices": "k8s-2020.10.10", 

  "ignore_unavailable": true,

  "include_global_state": true

}

 

 

## 查看备份状态

GET _snapshot/hw_hdfs_repository/snapshot_k8s/_status

或者

curl -XGET "http://192.33.9.141:9200/_snapshot/hdfs_repository/snapshot_k8s/_status"

 

## 查看snap备份列表

GET _snapshot/hdfs_repository/snapshot_k8s?pretty

 

## 删除备份

DELETE _snapshot/hdfs_repository/snapshot_20201103

 

## 查看所有索引的恢复进度

GET /_recovery/

 

> Ps: 每天建一个快照名字（snapshot_20201103），备份需要备份的索引名字。
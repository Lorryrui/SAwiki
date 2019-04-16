---
title: "ELK Stack"
layout: page
date: 2019-03-19 18:12
tag: Elastcisearch backups/restore
---

# Elastcisearch数据备份与恢复

## 一、简介

​    由于磁盘空间有限，需要对Elasticsearch（6.x）里存储的数据进行冷备份。1. 对指定的索引进行备份，保存后可以方便以后查历史数据；2. 处理掉过期数据，节省磁盘空间。

## 二、配置S3存储

###     2.1 Elasticsearch S3插件安装

```shell
./elasticsearch-plugin install repository-s3
# ES集群里面所有节点都要安装repository-s3插件
```

​        集群需要在每个节点上执行这个命令，重启后使用命令验证

```shell
./elasticsearch-plugin list
```

###      2.2 配置Elasticsearch连接S3 

​    备份的过期数据传输到对象存储S3，参考官方文档：

> [https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-s3-repository.html](https://www.elastic.co/guide/en/elasticsearch/plugins/current/repository-s3.html)

​    由于自建的Elasticsearch集群在华为云上，配置使用华为的OBS存储ES备份

​          2.2.1 设置S3连接客户端

```shell
# 创建自己的AK/SK -- Elasticsearch keystore(默认)
bin/elasticsearch-keystore add s3.client.default.access_key
bin/elasticsearch-keystore add s3.client.default.secret_key
# 多bucket情况下，配置不同的对象存储对应的AK/SK
bin/elasticsearch-keystore add s3.client.hwobs.access_key
bin/elasticsearch-keystore add s3.client.hwobs.secret_key
# 查看自己的keystore
bin/elasticsearch-keystore list
```

​          2.2.2  加载配置

```shell
# Kibana->Dev Tools
POST _nodes/reload_secure_settings
```

​          2.2.3  配置华为OBS作为Repository

```shell
# config/jvm.options 增加
-Des.allow_insecure_settings=true
```

```shell
curl -X PUT "localhost:9200/_snapshot/my_s3_repository" -H 'Content-Type: application/json' -d '
{
  "type": "s3",
  "settings": {
    "client": "hwobs",       
    "bucket": "mysql-backup",                     
    "endpoint": "obs.cn-east-2.myhuaweicloud.com",
    "access_key": "AK",                   
    "secret_key": "SK"
  }
}
'
```

​        2.2.4  触发ES进行快照备份（归档）

```shell
# 备份指定index
curl -XPUT 'http://localhost:9200/_snapshot/my_s3_repository/zhongzhuan_0411' -d '{"indices": "zhongzhuan-2019.04.11"}?wait_for_completion=true' -H 'Content-Type: application/json'

# 查看快照情况 Kibana-> Tools
GET _snapshot/my_s3_repository/zhongzhuan_0411
#   "state" : "SUCCESS" 快照执行状态
```

​         2.2.5  恢复指定快照

```shell
# 先删除需要恢复的index，再进行恢复
curl -XPOST 'http://localhost:9200/_snapshot/my_s3_repository/zhongzhuan_0411/_restore' -d '{"indices": "zhongzhuan-2019.04.11"}?wait_for_completion=true' -H 'Content-Type: application/json'

# 查看快照恢复情况 Kibana-> Tools
GET zhongzhuan-2019.04.11/_recovery
```

> 1. ES快照备份过程中，需要等状态变更为"SUCCESS"才可以进行其他快照相关操作
> 2. 快照恢复根据数据量大小，会有一定的等待时间
> 3. Kibana->Management->Index management 中现实index的存储等详细信息，才可以在Kibana查询

## 三、配置本地硬盘存储

###     3.1 Elasticsearch的配置

```yaml
# elasticsearch.yml 
# /esbackups目录权限是ES启动用户
path.repo: ["/esbackups"]
```

###     3.2 创建snapshot存储路径

```shell
# Kibana->Dev Tools
PUT /_snapshot/imservice_backup
{
  "type": "fs",
  "settings": {
    "location": "/esbackups"
  }
}
```

###     3.3 备份指定index的快照

```shell
curl -XPUT 'http://localhost:9200/_snapshot/hxapi_backup/snap_01' -d '{"indices": "hxapi-2019.03.21"}?wait_for_completion=true' -H 'Content-Type: application/json'
```

###     3.4 恢复指定index的快照

```shell
curl -XPOST 'http://localhost:9200/_snapshot/hxapi_backup/snap_02/_restore' -d '{"indices": "hxapi-2019.03.29"}?wait_for_completion=true' -H 'Content-Type: application/json'
```

###     3.5 删除备份中指定index的快照

```shell
curl -XDELETE http://127.0.0.1:9200/_snapshot/hxapi_backup/snap_01
```

## 四、 清理过期数据

```shell
# 删除Elasticsearch中指定index（可用正则）
curl -XDELETE "http://127.0.0.1:9200/imserivce-2019.04.11"
# 删除快照仓库中指定快照
curl -XDELETE http://127.0.0.1:9200/_snapshot/my_s3_repository/zhongzhuan_0411
```


# Logstash Demo

# 实际业务日志解析案例

## Demo 1：Nginx日志解析

1.1 解析流程

 Filebeat --> Kafka --> Logstash --> Elasticsearch

1.2 Nginx日志配置要求

 1.2.1 统一Nginx日志格式

```shell
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
               '"$http_user_agent" "$http_x_forwarded_for" "$request_time" "$http_host"';
```

 1.2.2 字段说明

| 字段名               | 含义                     |
| -------------------- | ------------------------ |
| remote_addr          | 客户端请求IP地址         |
| remote_user          | 用户                     |
| time_local           | 时间戳                   |
| request              | 请求内容                 |
| status               | HTTP状态码               |
| body_bytes_sent      | 发送的数据包大小         |
| http_referer         | 来源页面链接             |
| http_user_agent      | 用户客户端信息           |
| http_x_forwarded_for | 反向代理时用户实际IP地址 |
| request_time         | 请求响应时间             |
| http_host            | 请求域名或者IP           |

 1.2.3 Nginx日志配置规范

 1.2.3.1 日志文件名以 ${program_name}_access.log 命名，存放路径 /opt/nginx/logs

 1.2.3.2 凌晨日志切割，存放在/opt/nginx/logs/archive

 1.2.4 Nginx日志解析规则

```shell
%{COMBINEDAPACHELOG} \"%{IP:x_forwarded_for}\" %{QS:request_time} %{QS:http_host}
```

1.3 Filebeat配置参考

输出到Kafka：https://www.elastic.co/guide/en/beats/filebeat/current/kafka-output.html

```shell
filebeat.inputs:
- type: log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /opt/nginx/logs/dmblapp_access.log
  tags: ["dmblapp"] 
  fields:
    serverip: 192.168.2.5  # 新增当前服务器IP字段
- type: log
  # Change to true to enable this input configuration.
  enabled: true
  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /opt/nginx/logs/lhb_access.log
  tags: ["lhb"]
  fields:
    serverip: 192.168.2.5
output.kafka:
  # initial brokers for reading cluster metadata
  hosts: ["kafka1:9092", "kafka2:9092", "kafka3:9092"]
  # message topic selection + partitioning
  topic: nginx_access_log
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
  version: 0.10.0.0
```

1.4 Kafka配置说明

 Kafka版本建议选择0.11以上。使用KafkaManager管理Kafka集群，Topic创建规则不使用点，如nginx.access调整为nginx_access。副本数和Broker数相同，分区数是Broker整数倍。

1.5 Logstash配置参考

```shell
input {
    kafka {
        bootstrap_servers => "kafka1:9092 kafka2:9092 kafka3:9092"
        group_id => "nginx_access_consumer"  # 多台消费必须统一group_id
        topics => ["nginx_access_log"]
        consumer_threads => 3
        auto_offset_reset => "earliest"
        enable_auto_commit => "true"
        codec => "json"  # json
    }
}


filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG} \"%{IP:x_forwarded_for}\" %{QS:request_time} %{QS:http_host}"}
    }
    date {
        match => ["timestamp","dd/MMM/YYYY:HH:mm:ss +0800"]
    }
    geoip { # IP地址解析
        source => "x_forwarded_for"
        target => "geoip"
        database => "/etc/logstash/GeoLite2-City.mmdb"
        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
    }
    mutate {
        convert => [ "[geoip][coordinates]", "float"]
           }
    useragent { # 客户端信息解析
        target => "useragent"
        source => "agent"
    }
}

output {
    elasticsearch {
    hosts => ["es1:9200","es2:9200","es3:9200"]
    index => "nginx_access-%{+yyyy.MM.dd}"
    }
}
```

1.6 Kibana搜索与展示

 根据index在Kibana上建立nginx_access-*，然后在Discover中可以查看具体字段解析以及搜索内容。
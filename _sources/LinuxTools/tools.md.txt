# Linux常用网络+压测+监控等工具

## 一、 简介

 Linux下有很多用于问题排查、性能调优的工具，涉及到网络、软硬件等方方面面。显然一篇文章是无法详尽工具类型和每个参数的，这里记录下平时较为常用的工具，后续如有新涉及再另做补充。

## 二、 网络相关

> **· tcpdump**

 非HTTP协议下的抓包工具，主要用于分析如TCP/UDP协议的网络分析。Windows下的抓包工具wireshark和tcpdump语法一样，tcpdump主要用于Linux服务器上。

 -- 基本语法

 // 过滤主机

```shell
tcpdump -i eth1 host 192.168.1.1  
tcpdump -i eth1 src host 192.168.1.1  
tcpdump -i eth1 dst host 192.168.1.1  
```

 -i，指定数据包网卡；src/dst host，源IP/目的地址是192.168.1.1；host，来源或者目的地址

 // 过滤端口

```shell
tcpdump -i eth1 port 8080  
tcpdump -i eth1 src port 8080  
tcpdump -i eth1 dst port 8080  
```

 源端口（src）、目的端口（dst）是8080

 // 过滤协议类型

```shell
tcpdump -i eth1 tcp
```

 -- 包数据分析

 保存抓包的数据为pcap格式，使用wireshark客户端进行分析

```shell
tcpdump -i eth1 dst port 9999 -w dump.pcap  
```

> **· nslookup/dig**

 定位域名源站工具。ping命令可以解析域名IP但是不全，nslookup/dig可以查看DNS上域名解析的A记录、Cname等信息。相比较而言，我更新习惯使用dig。windows下dig需要额外安装软件包。

 // 查看DNS上指定域名解析结果

```shell
   dig simiki.org
```

 域名解析的结果可作为一个HTTP WEB网站的入口进行数据流链路梳理，静态网站走CDN一般解析出来的是cname。另外，也可以查看指定DNS服务器上的域名解析结果，用于在申请域名或者域名解析地址变动情况下的问题排查与确认。

```shell
   dig simiki.org @114.114.114.114
```

> **· netstat**

 查看当前系统中端口开放与连接的一些情况。主要用于查看TCP程序端口直接连接信息。

 // 常用参数

```shell
-t : 指明显示TCP端口 
-u : 指明显示UDP端口 
-l :  --listening ，display listening server sockets
-p : 显示进程标识符和程序名称，每一个套接字/端口都属于一个程序。 
-n : 不进行DNS轮询，显示IP
-a ：--all ，display all sockets (default: connected)
```

 // 查看当前所有TCP端口

```shell
netstat -ntlp
[root@test-0002 ~]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1146/sshd          
tcp        0      0 127.0.0.1:32000         0.0.0.0:*               LISTEN      957/java           
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      24094/zabbix_agentd 
tcp6       0      0 :::9778                :::*                    LISTEN      26805/java          
tcp6       0      0 :::22                   :::*                    LISTEN      1146/sshd  
```

 Recv-Q、Send-Q是数据包接受、发送的本地缓冲，如果是非0则出现堵包情况。

 // 查看所有连接，包括TIME_WAIT等其他状态，可作为网络排查与统计的参考

```shell
netstat -anlp|grep :9778
```

> · route,iptables

 单独说明，此处忽略。

## 三、 压测/性能相关

> · iostat/vmstat（输入/输出统计、虚拟内存统计）

1. iostat

   查看所有设备负载情况

主要关注两个cpu的属性：%iowait：CPU等待输入输出完成时间的百分比；%idle：CPU空闲时间百分比。

 %iowait的值过高，存在I/O瓶颈；%idle值高，表示CPU空闲；如果%idle值高但系统响应慢，说明CPU空闲，但是可能内存不足，考虑增加内存；%idle值持续较低，说明CPU占用高，需要调整CPU。

 // 每隔2秒刷新显示，且显示3次

```
iostat 2  3
```

 在做压力测试时，关注的下面这些参数，如tps、read、write等。

```
tps：该设备每秒的传输次数
kB_read/s：每秒从设备（drive expressed）读取的数据量；
kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
kB_read：  读取的总数据量；
kB_wrtn：写入的总数量数据量；
```

这些参数可作为硬件评估，上线前的压测数据。

1. vmstat

主要关注procs和cpu这两块。

 procs：r 运行队列中进程数量，长期较大考虑增加CPU；b 等待IO的进程数量

 cpu：us(user time)，长期过高可能是程序比较占CPU资源，应检查原因

 wa：IO等待时间百分比 wa的值高时，说明IO等待比较严重
​ id：空闲百分比

> ifstat/iftop（网络实时流量监控）

 ifstat只能简单给出网卡一些流量信息，这里主要记录下iftop。

1. 下载与安装（http://www.ex-parrot.com/~pdw/iftop/）

   // 添加EPEL，https://www.vpser.net/manage/centos-rhel-linux-third-party-source-epel.html

   `yum -y install iftop`

2. 界面说明

 中间的<= =>这两个左右箭头，表示的是流量的方向。

 TX：发送流量
​ RX：接收流量
​ TOTAL：总流量
​ Cumm：运行iftop到目前时间的总流量
​ peak：流量峰值
​ rates：分别表示过去 2s 10s 40s 的平均流量

1. 常用参数

-F 显示特定网段的进出流量

```shell
python iftop -F 192.168.1.5/24
```

-i 设定监测的网卡

```shell
python iftop -i eth0
```

-n 使host信息默认直接都显示IP

```shell
iftop -n
```

> · dstat

 是以上几个命令的拓展，增加了监控项，很方便监控并用于基准测试和故障排查。

## 四、故障分析与排查

 这一块主要是一些Linux下调试诊断，分析程序及系统调用的一些工具。目前接触到的如，strace 、perf。这是针对二进制编译的程序，在未开启debug日志下的一些排查手段。针对性比较强，等有实际使用案例再补充。

## 五、结束语

 Linux下涉及的工具很多，除了上面提到的还有一些开源社区提供的工具。在网络、监控、性能分析上都有相关涉及。当然，在业务服务器非常多的时候，这些命令只能起一些辅助的作用，更多的监控项应由专业的监控工具，如Zabbix去收集，设置其报警阈值。
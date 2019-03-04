# 文件同步工具--lsync

## 一、简介

​    rsync是使用比较多的，用于增量同步文件的工具。但是，rsync多配合crontab进行定时同步，无法做到实时对文件目录进行监控。在实际使用过程中，如果使用不当，第一次同步未完成就进行第二次同步，可能会导致文件无法同步。内核2.6以后，使用inotify可以实时监控文件的变动，从而进行触发式的进行rsync同步。这里介绍的是比较好用的lsync。

## 二、安装与使用

​    依赖：Linux 内核版本 2.6+；rsync >= 3.1

   1. 安装

      ```
      yum install lsyncd -y
      ```

2. 配置示例（/etc/lsyncd.conf）  

   2.1 rsync 发送端

   ```
   settings {
   logfile = "/var/log/lsyncd/mylsyncd.log",
   statusFile = "/tmp/lsyncd.stat",
   statusInterval = 5,
   }
   
   sync {
   default.rsync,
   source="/data/output/",
   target="192.168.1.1::mydata",
   rsync = {
           archive  = true,
           compress = true
           }
   }
   ```

    2.2 rsync 接受端（rsync server端配置）

   ​    2.2.1 安装rsync

   ```
      yum install rsync -y
   ```

   ​    2.2.2 配置（/etc/rsyncd.conf） 

   ```python
       log file = /var/log/rsyncd.log
       pid file = /var/run/rsyncd.pid
       lock file = /var/run/rsyncd.lock
       [mydata]
       path = /opt/data/
       hosts allow = 192.168.1.2
       uid = root
       gid = root
   ```

   ​     2.2.3 启动rsync服务端      

   ```python
       rsync --daemon /etc/rsyncd.conf  
   ```

   3. 启动

      ```
      chkconfig lsyncd on
      service lsyncd on
      ```

      

   
---
title: "iptables"
layout: page
date: 2019-07-04 08:42
tag: LinuxTools
---

# iptables使用参考

## 一、简介

​    netfilter/iptables组成Linux平台下的包过滤防火墙 ，完成 封包过滤、封包重定向、网络地址转换NAT 等功能。iptables可以看做是客户端代理，通过它设定规则，netfilter是位于内核空间的真正的防火墙。

> [https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)](https://wiki.archlinux.org/index.php/Iptables_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

​    这里记录常用的部分。

## 二、安装与使用

### 2.1 安装

​    CentOS7需要卸载系统自带的firewall：

```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
```

​    安装iptables：

```shell
yum install iptables-services iptables -y
```

### 2.2 常用部分

#### 2.2.1 INPUT（入站规则）

```shell
# 查看当前规则
iptables -nL
# 所有入站请求TCP对当前IP开放
iptables -A INPUT -p tcp -s 192.168.45.193 -j ACCEPT
# 默认入站deny，需要开通白名单
iptables -P INPUT DROP
# 指定IP访问指定端口
iptables -A INPUT -p tcp --dport  9092  -s 192.168.26.135 -j ACCEPT
# 每次修改都需要保存规则，否则重启失效
service iptables save
```

#### 2.2.2 OUTPUT（出站规则）

```shell
# 默认所有出站容许,如需要设置所有出站deny
iptables -P OUTPUT DROP
# 指定端口出站白名单
iptables -A OUTPUT -p udp --sport 53 -j ACCEPT
# 每次修改都需要保存规则，否则重启失效
service iptables save
```

#### 2.2.3 FORWARD（端口转发）

```shell
# 启用网卡转发
echo 1 > /proc/sys/net/ipv4/ip_forward
# 永久生效（/etc/sysctl.conf）; sysctl -p
net.ipv4.ip_forward = 1
# 端口转发
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 23306 -j DNAT --to 192.168.1.11:3306
iptables -t nat -A POSTROUTING -j MASQUERADE
# 每次修改都需要保存规则，否则重启失效
service iptables save
```

#### 2.2.4 删除规则

```shell
# 列出规则
iptables -L -n --line-number
# 根据规则类型和序号删除相应规则
iptables -D OUTPUT 1 
# 每次修改都需要保存规则，否则重启失效
service iptables save
```


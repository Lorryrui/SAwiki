---
title: "Grafna LDAP"
layout: page
date: 2019-03-01 10:00
tag: Grafna,LDAP
---

# Grafna之LDAP认证接入配置说明

## 一 简介

​    LDAP是轻量目录访问协议，英文全称是Lightweight Directory Access Protocol。使用LDAP作为Grafna的认证方式，管理员不需要关心登录账号密码的分配，只需要管理相应的用户组和权限即可。



## 二 原理说明

​    LDAP管理员为Grafna接入分配一个只读账号(user/password)，给定LDAP接入的域信息，如OU=mygrafna,DC=mygrafna,DC=com,DC=cn。windows下使用ldapadmin客户端连接：

![](..\attach\ldapadmin.jpg)



查看任意一个用户信息，如

 <u>cn: 张三   sn: 张  givenName: 三  name: 张三 sAMAccountName: 10000</u> ... 等相关信息，Grafna接入LDAP之后，会定时同步一份LDAP里面用户名、密码到自身的认证服务里。在配置Grafna时，登录使用的LDAP用户名字段是sAMAccountName，用户信息里姓、名可以配置sn、cn（英文中姓、名排序和中文是反的，可以配置成自己习惯的即可）

 当前使用的域在ldapadmin客户端中也可以查看

![](..\attach\ldap.png)



## 三 接入流程

   1. 向LDAP管理员申请只读账号密码、LDAP服务器与域信息

   2. 使用LDAP客户端(ldapadmin)验证信息的准确性

   3. 在Grafna中配置接入LDAP，如认证不通过，启用debug模式进行调试

      /etc/grafana/grafana.ini

      ```python
      [log]  
      filters = ldap:debug
      ```

      


4. 配置修改后重启Grafna，使用LDAP账号登录
5. 对LDAP权限控制由Grafna默认admin账号分配

## 四 配置说明

​    官方配置参考地址：[http://docs.grafana.org/auth/ldap/](http://docs.grafana.org/auth/ldap/)

   1. Grafna启用LDAP认证（默认/etc/grafana/grafana.ini）  

      ```
      config_file = /etc/grafana/ldap.toml  
      allow_sign_up = true
      ```

      

      

2. 配置需要接入的LDAP服务端信息（默认/etc/grafana/ldap.toml）  

   ```python 
   host = "192.168.12.123"   # LDAP服务器IP 
   port = 389  # 默认端口，未开启SSL  
   bind_dn = "user@mygrafna.com.cn"  # LDAP管理用户名以及相关域信息  
   bind_password = 'password' #LDAP管理用户的密码  
   search_filter = "(sAMAccountName=%s)" # LDAP中存储用户的字段名  
   search_base_dns = ["DC=mygrafna,DC=com,DC=cn"]
   ```

   

   

3. 重启Grafna程序，使用LDAP账号即可登录



## 五 配置参考

```python
[[servers]]
host = "192.168.12.123"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = true
bind_dn = "readonly@mygrafna.com.cn"
bind_password = 'password'
search_filter = "(sAMAccountName=%s)"
search_base_dns = ["DC=mygrafna,DC=com,DC=cn"]
[servers.attributes]
name = "sn"
surname = "givenName"
username = "sAMAccountName"
member_of = "memberOf"
email =  "mail"
[[servers.group_mappings]]
group_dn = "DC=mygrafna,DC=com,DC=cn"
org_role = "Admin"
org_id = 3
[[servers.group_mappings]]
group_dn = "DC=mygrafna,DC=com,DC=cn"
org_role = "Editor"
[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```


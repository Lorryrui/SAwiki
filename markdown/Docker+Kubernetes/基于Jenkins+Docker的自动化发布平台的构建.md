---
title: "Jenkins and Docker"
layout: page
date: 2019-03-07 21:42
tag: Devops
---

# 基于Jenkins+Docker的自动化发布平台的构建

## 一、 Harbor--自建镜像仓库

​    1.1 关于Harbor

​        在使用Docker的时候，通常需要统一的镜像仓库作为镜像存储与分发。Docker Registry虽然为我们提供镜像存储，但是考虑到镜像的权限管理与管理的方便性，在生产环境还是比较推荐使用Harbor作为企业的私有镜像仓库。

> ​       Docker Registry  [https://docs.docker.com/registry/](https://docs.docker.com/registry/)
>
> ​       Harbor  [https://goharbor.io/]( https://goharbor.io/)

​    1.2 Harbor的安装与配置

> [https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md](https://github.com/goharbor/harbor/blob/master/docs/installation_guide.md)

​        1.2.1 下载对应的版本

​            安装Docker：[https://docs.docker.com/install/linux/docker-ce/centos/](https://docs.docker.com/install/linux/docker-ce/centos/)

​	    安装Docker-compose：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

​            Harbor offline版本： [https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.4.tgz](https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.4.tgz)

​        1.2.2 配置（harbor.cfg）

​            详细的配置参数官方文档有说明。我这里主要有 HTTPS和关闭注册账号两项配置，配置内容参考：

```python
		hostname = harbor.mydomain.com.cn
        ui_url_protocol = https
		ssl_cert = /data/cert/server.crt # 域名证书cer
		ssl_cert_key = /data/cert/server.key # 域名证书key
		secretkey_path = /data
		auth_mode = db_auth
        self_registration = off # 关闭注册账号功能
```

​	    LDAP配置参考：[https://kb.vmware.com/s/article/2148949](https://kb.vmware.com/s/article/2148949) 考虑到安全性，慎用。

​        1.2.3 安装与启动

​	    ./prepare  

​            ./install.sh  

​	    docker-compose stop   

​	    docker-compose up -d   

​        1.2.4 访问
​	    [http://harbor.mydomain.com.cn](   http://harbor.mydomain.com.cn)	

​	    默认账号密码： admin/Harbor12345

​    PS：① 建议使用HTTPS，docker pull 默认使用https，否则要修改docker的配置  

​            ②  harbor.cfg中证书以及证书路径不需要修改，否则需要调整docker-compose.yml  

​      1.3 Harbor使用

​	    1.3.1 存储容量

![](..\attach\disk.png)

​		默认是/data所在盘的大小，一般是/目录的大小。建议增加一块存储较大硬盘mount到/data上。

​	    1.3.2 邮箱配置

![](..\attach\harbor_mail.png)

​	     Harbor内置邮件功能的配置，需要配置SMTP邮箱服务相关信息用于发送邮件。

​	   1.3.3 认证模式配置

![](..\attach\harbor_auth.png)

​	    登录认证方式。默认是数据库，可以使用LDAP。允许自注册功能也可以在这里开启和关闭  	  

​		
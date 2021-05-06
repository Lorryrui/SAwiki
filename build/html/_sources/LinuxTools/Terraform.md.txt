# Terraform

## 一、 Terraform简介

>  Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently.

 Terraform是一个Infrastructure as Code的工具，在云服务的大环境下，Terraform提供了一个简洁、方便的方式去管理云资源。除此之外，还提供其他开源组件的操作，如MySQL、HTTP等。简单的说，Terraform是一个集成多服务的编排工具。

### 1.1 安装

 以Linux为例，安装参考

> https://learn.hashicorp.com/terraform/getting-started/install.html

```shell
# terraform --version
Terraform v0.12.9
```

### 1.2 工作目录中文件命名

 在当前工作目录中，文件名命名按照如下规范：

```shell
 terraform.tfvars      -- 配置 provider 要用到的变量
 terraform.tfvars      -- 配置 provider 要用到的变量
 varable.tf            -- 通用变量
 resource.tf           -- 资源定义
 resource.tf           -- 资源定义
 data.tf               -- 包文件定义
 output.tf             -- 输出
```

 实际使用过程中，也可以将所有的内容全部写在.tf结尾的一个文件中，Terraform会按照.tf文件中的编排顺序进行执行。如果是一个比较规范的编排脚本，推荐按照上面的命名规范进行。

### 1.3 执行流程

 Terraform的执行时分为，init -> plan -> apply。init是初始化工作目录，下载包括需要连接操作资源的客户端等；plan检查配置文件并生成执行计划；apply是执行具体操作。

## 二、使用案例

### 2.1 RabbitMQ增加vhost

#### 2.1.1 创建rabbitmq.tf文件，添加如下内容：

```shell
# Configure the RabbitMQ provider
provider "rabbitmq" {
  endpoint = "http://127.0.0.1:15672"
  username = "admin"
  password = "admin"
}

# New vhost
resource "rabbitmq_vhost" "tertaform_test__vhost" {
  name = "tertaform_test__vhost"
}
```

 tf文件中，分为 provider 和 resource两个部分。provider是Terraform工具的对应服务（如RabbitMQ）操作的客户端，目前官方提供的provider部分可以在https://www.terraform.io/docs/providers/index.html列表中查看；resource则是对资源具体的操作，如本例的增加vhost。

#### 2.1.2 init - plan - apply

```shell
# terraform init

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "rabbitmq" (terraform-providers/rabbitmq) 1.1.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.rabbitmq: version = "~> 1.1"

Terraform has been successfully initialized!
# terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # rabbitmq_vhost.tertaform_test__vhost will be created
  + resource "rabbitmq_vhost" "tertaform_test__vhost" {
      + id   = (known after apply)
      + name = "tertaform_test__vhost"
    }

Plan: 1 to add, 0 to change, 0 to destroy.
# terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # rabbitmq_vhost.tertaform_test__vhost will be created
  + resource "rabbitmq_vhost" "tertaform_test__vhost" {
      + id   = (known after apply)
      + name = "tertaform_test__vhost"
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

rabbitmq_vhost.tertaform_test__vhost: Creating...
rabbitmq_vhost.tertaform_test__vhost: Creation complete after 0s [id=tertaform_test__vhost]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

### 2.2 华为云DNS添加一条记录

```shell
# Authentication with AK/SK
provider "huaweicloud" {
  access_key         = "AK"
  secret_key         = "SK"
  region             = "cn-east-2"
  auth_url           = "https://iam.myhwclouds.com:443/v3"
}

resource "huaweicloud_dns_recordset_v2" "rs_codis" {
  zone_id     = "zoneid"
  name        = "hwprivatedns-codis.gw.com.cn."
  description = "An example record set"
  ttl         = 3000
  type        = "A"
  records     = ["192.168.5.2"]
}
```

## 三、总结

 Terraform类似一个统一的工具，通过实现不同服务客户端（provider）从而操作资源。优势是在混合云的情况下可以更方便的跨云管理资源，但是同时由于provider支持的程度不同，有一些操作可能是有限制的甚至是没实现的。
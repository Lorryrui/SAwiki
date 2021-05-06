# Helm基础与使用

## 一、Helm是什么

> *Helm* Charts 帮助您定义、安装和升级最复杂的 Kubernetes 应用程序。

​    Helm是运行在Kubernetes环境中的一个统一打包管理K8S资源的工具，类似于Linux包管理工具yum/apt。使用Helm可以对应用的configmap、service、deployment等一系列的K8S资源的创建和升级，将应用的一系列资源当做一个软件包管理。

​    Helm主要有v2和v3两个版本，v2版本是C/S架构，由客户端组件 helm 和服务端组件 Tiller 组成；v3则直接通过K8S的ApiServer操作K8S资源。这里推荐直接使用v3版本。

## 二、为什么要使用Helm

​    对K8S上的应用管理，一般以yaml文件描述然后部署。这个在生产环境中上线应用非常不方便，也不利用后期的管理和维护。Helm可以很好的描述一个K8S应用的各个资源如deployment、configmap之间的关系，并且可以通过模板化的管理一类应用。

​    简单的说，Helm 可以很好的管理chart，具有版本管理，历史回溯，快速回滚功能，这个是简单的使用编排模板所不能比拟的。    

## 三、如何使用Helm

​    使用Helm之前先了解几个相关的概念：

**helm**

- Helm     是一个命令行下的客户端工具。主要用于     Kubernetes 应用程序 Chart 的创建、打包、发布以及创建和管理本地和远程的     Chart 仓库。

**Chart**

- Helm     的软件包，采用     TAR 格式。类似于 APT 的 DEB 包或者     YUM 的 RPM 包，其包含了一组定义 Kubernetes 资源相关的     YAML 文件。

**Repoistory**

- Helm     的软件仓库，Repository     本质上是一个     Web 服务器，该服务器保存了一系列的 Chart 软件包以供用户下载，并且提供了一个该     Repository 的 Chart 包的清单文件以供查询。Helm     可以同时管理多个不同的     Repository。

**Release**

- 使用     helm install 命令在 Kubernetes 集群中部署的     Chart 称为 Release。可以理解为 Helm 使用     Chart 包部署的一个应用实例。



### 1、创建一个chart

```shell
helm create myapp
```

​    Chart.yaml     chart的一些描述，包括名称、版本号等

​    values.yaml    参数值，提供给模版引用

​    templates       模板文件目录。包括deployment、service、hpa等资源yaml描述

### 2、K8S资源yaml描述文件编写

​    yaml格式格式和之前写的yaml文件类似，这里主要值使用变量进行填充。比如值直接引用：

```shell
app: {{ .Values.appname }}
```

​    循环的编写：

```shell
{{- range .Values.configOverrides }}
  {{ . }}: |
{{ $.Files.Get (printf "%s%s" "configs/" .) | indent 4 }}
{{- end }}
```

​    对于不同的K8S资源模版，存放在不同的文件名中。模版的值内容全部从    values.yaml  或者 Chart.yaml引入。

### 3、测试、打包、部署到K8S

```shell
# 调试

helm install --debug --dry-run ./detailpage/ --generate-name

# 打包

helm package ./detailpage

# 安装

helm install detailpage --namespace webapp ./detailpage-0.1.0.tgz 
```



## 四、例子：使用Helm打包PHP应用

​    打包PHP应用（detailpage）主要有配置文件、程序Docker镜像两个部分。其中，配置文件使用configmap，程序使用deployment，暴露web端口使用service。

detailpage/values.yaml

```yaml
# Default values for yxds.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 2
namespace: webapp

appname: detailpage

image:
  repository: my.com/webapp/detailpage
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "v1"

containerPort: 80

imagePullSecrets:
  - name: dockerkey
nameOverride: ""
fullnameOverride: ""

podAnnotations: {}

# put your configfiles in configs/
configOverrides:
  - config.php

service:
  type: ClusterIP
  port: 8100
  targetPort: 80

autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80
```

detailpage/templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appname }}
  namespace: {{ .Values.namespace }}
spec:
  selector:
    matchLabels:
      app: {{ .Values.appname }}
  replicas: {{ .Values.replicaCount }}
  template:
    metadata:
      labels:
        app: {{ .Values.appname }}  
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.containerPort }}
              protocol: TCP
```

detailpage/templates/configmap.yaml

```yaml
{{- if .Values.configOverrides }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Values.appname }}
  namespace: {{ .Values.namespace }}
data:
{{- range .Values.configOverrides }}
  {{ . }}: |
{{ $.Files.Get (printf "%s%s" "configs/" .) | indent 4 }}
{{- end }}
{{- end }}
```


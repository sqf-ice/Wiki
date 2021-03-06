---
title: "Spark 安装与部署"
layout: page
date: 2099-06-02 00:00
---


# 1. 环境准备



### 1.1. 常规部署

#### 1.1.1. 本地部署
#### 1.1.2. YARN


Spark On Yarn 框架
概念

Spark on Yarn的两种运行模式：cluster和client;
一句话概述两种的区别就是Spark driver到底运行再什么地方
In cluster mode：Driver运行在NodeManage的AM(Application Master)中，它负责向YARN申请资源，并监督作业的运行状况。当用户提交了作业之后，就可以关掉Client，作业会继续在YARN上运行。
In client mode:Driver运行在Client上，通过ApplicationMaster向RM获取资源。本地Driver负责与所有的executor container进行交互
#### 1.1.3. Mesos



### 1.2. 基于docker



#### 1.2.1. 单击版

参考：https://hub.docker.com/r/sequenceiq/spark/
```shell
docker pull sequenceiq/spark:1.6.0
```


#### 1.2.2. 搭建spark集群

##### 1.2.2.1. 设置系统路由转发功能
```shell
修改配置文件 
vim /etc/sysctl.conf
>>>
#新增一条：
net.ipv4.ip_forward=1 
# 0 时表示禁止进行IP转发
# 1 IP转发功能已经打开

#重启网络：
systemctl restart network
#验证配置：
sysctl net.ipv4.ip_forward
```
##### 1.2.2.2. 创建一个网络

为本地群集创建一个网络
创建网络非常简单，可以通过运行以下命令来完成：
桥接网络类似于默认bridge网络
```shell
docker network create spark_network
```
###### 1.2.2.2.1. 启动 spark-master
```shell
docker run --rm -it --name spark-master --hostname spark-master \
    -p 7077:7077 -p 8080:8080 --network spark_network \
    <spark-image-name> /bin/sh
```

###### 1.2.2.2.2. 启动 spark-worker
```shell
docker run --rm -it --name spark-worker1 --hostname spark-worker1 \
    --network spark_network \
    <spark-image-name> /bin/sh
# 输入
/spark/bin/spark-class org.apache.spark.deploy.worker.Worker \
    --webui-port 8080 spark://spark-master:7077
```

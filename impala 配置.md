# Impala 配置

## 1. impala 介绍

### 1.1 impala 概念

impala 是分布式，大规模并行的数据库引擎，由多个不同的线程运行在不同的host中的服务组成

### 1.2 impala 服务

#### The Impala Daemon

impala 核心组件，物理层面表现为 impalad 线程，运行在多个host中，共同组成了impala cluster，其主要功能为一下四点：

- 读写数据文件
- 从impala-shell，Hue，JDBC 和 ODBC 接收查询请求
- 并行执行查询请求，在整个集群当中分发任务
- 将查询结果返回给中央调度员 central coordinator

impala有两种部署方式：

- 与hdfs部署在同一台机器上，并且每一个impalad都部署在hdfs的datanode上
- impalad分布式部署在多台机器上，并且远程读取HDFS，S3，ADLS等等

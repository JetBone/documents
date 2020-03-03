# Impala 配置

## 1. Impala 介绍

### 1.1 Impala 概念

impala 是分布式，大规模并行的数据库引擎，由多个不同的线程运行在不同的host中的服务组成

### 1.2 Impala 服务组件

#### The Impala Daemon

impala 核心组件，物理层面表现为名字叫 impalad 的守护进程，运行在多个host中，共同组成了impala cluster，其主要功能为一下四点：

- 读写数据文件
- 从 impala-shell，Hue，JDBC 和 ODBC 接收查询请求
- 并行执行查询请求，在整个集群当中分发任务
- 将查询结果返回给中央调度员 central coordinator

Impala daemon 有两种部署方式：

- 与 hdfs 部署在同一台机器上，并且每一个 impalad 都部署在 hdfs 的 datanode 上
- Impalad 分布式部署在多台机器上，并且远程读取 HDFS，S3，ADLS 等等

Impala daemon 会与 StateStore 不断的沟通，去确认集群当中的 daemon 是否健康并可以接收新的 work

如有有 impala daemon 被创建，修改或者销毁，群组里的 impala daemon 也会接收到广播信息

#### The Impala Statestore

Impala Statestore 也被称呼为 StateStore，是用来检测 impala 集群当中所有 impala daemon 的健康状况，物理层面表现为名字叫 statestored 的守护进程。整个集群当中只能有一个 StateStore。如果有一个 impala daemon 出现故障，StateStore 会通知其它所有 impala daemon，确保之后的查询请求不会在发送到故障节点。

#### The Impala Catalog Service

Impala Catalog Service 也被称呼为 Catalog Service，是用来将 metadata 的变化发送到集群中的所有节点，物理层面表现为名字叫 catalogd 的守护进程。整个集群当中只能有一个 Catalog Service。建议 Catalog Service 和 StateStore 部署在同一台机器上。

## 2. Impala 部署

Impala 基本组成：

- impalad：每一个 datanode 都需要部署一个
- statestored：一个集群当中只能有一个，一般部署在 namenode 上
- catalogd：一个集群当中只能有一个，一般部署在 namenode 上
- impala-shell：Command-line Interface，可以将通过 shell 输入查询语句，可以部署到任何服务器上，可以通过远程链接到任何 Impala 服务



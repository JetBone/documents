# SPARK 介绍与配置

## 1. Spark 简单介绍

Spark 是一种非常快速，通用的分布式计算系统。它提供了多种语言的高级 API ，比如 Java，Scala，Python 和 R。并且拥有许多非常高级的工具比如 Spark SQL 用于提供针对一些结构化数据的 SQL 操作支持，MLib 用于机器学习，GraphX 用于图形计算，和 Spark Streaming 用于流计算。

### 1.1 Spark 简单概念


## 2. Spark 配置

spark配置后缀都是template，需要把我们需要的文件cp一份然后后缀的template删除掉

## SPARK_HOME/conf/spark-env.sh spark集群环境配置，带不带双引号都无所谓

``` bash
export JAVA_HOME="/usr/java/jdk1.8.0_221-amd64"
export SCALA_HOME=$SCALA_HOME
export HADOOP_HOME="/home/bigdata/env/hadoop"
export HADOOP_CONF_DIR="/home/bigdata/env/hadoop/etc/hadoop"
# spark集群的Master节点的ip地址
export SPARK_MASTER_IP="bdmaster"
# 每个worker节点能够最大分配给exectors的内存大小
export SPARK_WORKER_MEMORY=2g
# 每个worker节点所占有的CPU核数目
export SPARK_WORKER_CORES=2
# 每台机器上开启的worker节点的数目
export SPARK_WORKER_INSTANCES=2
```

## SPARK_HOME/conf/slaves 集群slave节点配置，这里只需要配置slave节点的hostname或者是IP，如果没有集群，则这个文件不需要改动

``` text
bdslave1
bdslave2
```

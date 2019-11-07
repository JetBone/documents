# Spark 集群配置

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

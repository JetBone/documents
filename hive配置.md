# HIVE 安装与配置

## 1. Hive 介绍

### 1.1 什么是 Hive

Hive 是一种存储大型分布式数据集的数据仓库，使用SQL语言进行读写。底层使用HDFS或者HBase进行数据存储

### 1.2 Hive 的组成

介绍在部署 Hive 的时候需要下载的 package：

- hive - 基础包，提供完整的语言支持和运行环境
- hive-metastore - 可选，提供了可以独立部署 Hive MetaStore Service 的脚本
- hive-service2 - 提供脚本支持启动 HiveService2
- hive-hbasehive-hbase - 可选，如果 Hive 底层对接的是 Hbase 则需要下载该包

> 注意：建议 Hive MetaStore Service 独立部署，因为有些其它服务会依赖于这个服务，并且 Hive MetaStore Service 独立部署有利于单独维护 metadata database 配置

## 2. HIVE 部署方式

- Embedded Mode

### 2.1 Embended

## 2. 配置环境变量

### 2.1 按需修改指定的环境变量文件，这里设置为全局变量，修改 /etc/profile

``` bash
# 配置 HIVE HOME
export HIVE_HOME=/your_hive_path
# 配置 PATH
export PATH=$HIVE_HOME/bin:$PATH
```

## 3. Hive 部署

从hive官网根据操作下载stable版本的tar包解压到自己指定的目录，版本和环境按需自行配置，本次文档使用的包：apache-hive-2.3.6-bin.tar.gz

### 3.1 配置 hive-env.sh 文件

将 hive-env.sh.template 复制并重命名 hive-env.sh 然后修改

``` bash
# HADOOP_HOME，如果已经定义了全局HADOOP_HOME则可以直接引用，否则需要写全路径
HADOOP_HOME=$HADOOP_HOME
# 配置文件存放路径
HIVE_CONF_DIR=$HIVE_HOME/conf
# hive需要使用到的jar包路径
HIVE_AUX_JARS_PATH==$HIVE_HOME/lib
```

### 3.2 为 hive 新建存放数据的目录

在 hdfs 里为 hive 新建一个目录用于存放 hive 产生的数据

``` bash
hadoop fs -mkdir /hive/warehouse
```

### 3.3 为 hive 新建存放临时文件的目录

在 hdfs 里为 hive 新建一个用于存放临时文件的路径


``` bash
hadoop fs -mkdir /hive/tmp
```

### 3.4 为 hive 新建存放日志的目录

在 hdfs 里为 hive 新建一个用于存放查询日志的路径

``` bash
hadoop fs -mkdir /hive/querylog
```

### 3.5 配置 hive-site.xml 文件

将 hive-default.xml.template 复制并重命名 hive-site.xml 然后修改部分数据，将上述三个目录配置到 hive 里

``` xml
<!-- hive 临时目录hdfs路径 -->
<property>
  <name>hive.exec.scratchdir</name>
  <value>/hive/tmp</value>
  <description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
</property>
<!-- hive 存放数据的hdfs路径 -->
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>
<!-- hive 查询日志路径 该路径为本地路径 -->
<property>
  <name>hive.querylog.location</name>
  <value>/localLogPath</value>
  <description>Location of Hive run time structured log file</description>
</property>

<!-- hive metastore 配置，需要使用本地ysql数据库 -->
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>
  <description>
    JDBC connect string for a JDBC metastore.
    To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
    For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
  </description>
</property>
<!-- 配置 Mysql 驱动，之后需要将 mysql-connection-java.jar 文件放到hive下的lib目录中，确保hive可以找到 jdbc 驱动com.mysql.jdbc.Driver 和 com.mysql.cj.jdbc.Driver 是一个效果，但是使用前者会收到一个警告，具体信息请直接看警告内容 -->
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.cj.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<!-- Mysql 用户名 -->
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>username</value>
  <description>Username to use against metastore database</description>
</property>
<!-- Mysql 密码 -->
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>password</value>
  <description>password to use against metastore database</description>
</property>
<property>
  <name>datanucleus.schema.autoCreateAll</name>
  <value>true</value>
  <description>Auto creates necessary schema on a startup if one doesn't exist. Set this to false, after creating it once.To enable auto create also set hive.metastore.schema.verification=false. Auto creation is not recommended for production use cases, run schematool command instead.</description>
</property>
<property>
  <name>hive.metastore.schema.verification</name>
  <value>false</value>
  <description>
    Enforce metastore schema version consistency.
    True: Verify that version information stored in is compatible with one from Hive jars.  Also disable automatic
          schema migration attempt. Users are required to manually migrate schema after Hive upgrade which ensures
          proper metastore schema migration. (Default)
    False: Warn if the version information stored in metastore doesn't match with one from in Hive jars.
  </description>
</property>
```

### 3.6 配置 hive 日志

复制 hive-log4j2.properties.template 为 hive-log4j2.properties 按需修改里面的配置即可

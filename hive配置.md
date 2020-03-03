# HIVE 安装与配置

## 1. Hive 简单介绍

### 1.1 Hive 概念

Hive 是一种存储大型分布式数据集的数据仓库，使用SQL语言进行读写。底层使用HDFS或者HBase进行数据存储

### 1.2 Hive 擅长的领域

Hive 设计用来进行简化数据汇总，临时查询和分析大量数据，它提供了SQL语法支持，让用户更容易对数据进行汇总分析查询等等，

### 1.3 Hive 不擅长的领域

Hive 不适合用来在线实时数据处理，一般是用作传统的数据仓库

### 1.4 Hive 的一些概念

#### The MetaStore Database

The MetaStore Database 及元数据数据库，是 Hive 的重要组成部分。一般是一个单独分离出来的数据库，并基于传统的关系型数据库建立，比如 Mysql 或者 PostgreSQL。其内部保存了 Hive 中建立的 数据库，表，列，分片等等元信息。

#### HiveServer2

HiveServer2 提供了远程服务，支持远程客户端提交查询语句，并返回结果给客户端。它替代了之前的 HiveServer1。

HiveServer2 是一个 Hive execution engine 的容器，针对每一个客户端连接，都会单独建立一个上下文环境。HiveServer2 通过 Thrift API-based Hive service 支持远程 JDBC 客户端连接和 ODBC 连接。

HiveServer2 的 CLI 是 Beeline

#### Hive on Spark

传统的 Hive 使用 MapReduce 来进行并行工作。并且可以执行一些较为简单的 SQL 语法，比如 sorting 和 filtering。当然 Hive 也可以使用 Spark 来作为自己的计算引擎。

#### Hive on HBase

HBase 是一种 NoSQL 数据库，支持从 HDFS 里实时 read/write 大型数据集。

#### Hive 的事务支持

Hive 如果要支持事物，则要求底层文件保存为 ORC 文件格式，但是一般建议使用 Parquet 文件格式。

> 有关事物支持具体信息需要进一步查看专业文档，这里只是简单提及一下，不能保证正确性。

- Hive CLI - Hive Command-line Interface。安装hive之后，可以直接输入 hive 命令进入命令行界面，之后可以输入 SQL 语句来执行一系列操作
- HiveServer2 - 由 Hive 0.11 版本引入，代替了原来的HiveServer，其有自己的 CLI 叫做 Beeline
- Beeline - 新的 CLI，代替旧版本的 CLI，由于旧版本的 HiveCLI存在诸多缺陷，目前已经不再被推介使用，并且已经停止了更新，之后新版本可能会删除 HiveCLI，建议之后使用 Beeline

介绍在部署 Hive 的时候需要下载的 package：

- hive - 基础包，提供完整的语言支持和运行环境
- hive-metastore - 可选，提供了可以独立部署 Hive MetaStore Service 的脚本
- hive-service2 - 提供脚本支持启动 HiveServer2
- hive-hbase - 可选，如果 Hive 底层对接的是 Hbase 则需要下载该包

> 注意：建议 Hive MetaStore Service 独立部署，因为有些其它服务会依赖于这个服务，并且 Hive MetaStore Service 独立部署有利于单独维护 metadata database 配置

## 2. Hive 部署方式(Hive MetaStore 的部署方式)

Hive 的部署分为三种部署方式，实际上 Hive 的部署方式也就是 MetaStore 的部署方式。所以一下讲的内容实际上就是 MetaStore 的部署方式

### 2.1 Embedded Mode 内嵌模式

Embedded Mode 是 Hive 默认的 Metastore 部署模式。在这个模式下，Hive 会使用内置的 Derby database 作为 Metadata database，并且 Derby 服务和 metastore 服务都会被内嵌到 HiveServer 的 main 进程当中，其实就是最终启动了一个线程包含全部服务。此部署模式基本不用配置，但是同一时间只允许一个用户登陆，适合用来做个人学习和测试。

### 2.2 Local Mode 本地模式

Local Mode 下，The Metastore Service 仍然内嵌在 HiveServer 的 main 进程当中。与内嵌模式唯一的区别是，Metadata database 使用的是分离出来的数据库(一般使用 Mysql)而不是 Derby，并且该数据库可以在不同的 Host 当中(配置文件中配置 javax.jdo.option.ConnectionURL 的时候指定URL即可)。

### 2.3 Remote Mode 远程模式（推介使用）

Remote Mode 下，Metastore 进程会单独运行在一个 JVM 进程当中。HiveServer2，HCatalog，Impala，或者其它进程都可以通过 Thrift network API (配置文件中配置 hive.metastore.uris 属性)来访问 The Metastore Service。此模式下，The Metastore Service 进程和 HiveServer 的 main 进程可以是同一服务器里的两个进程，或者不同服务器中的两个进程，具体请按需部署即可。

Remote Mode 主要优点是，JDBC 的登陆信息只保存在 The Metastore Service 当中，不需要在每一个 client 当中配置。另外，部署 Impala 的时候虽然本身也可以像 Local Mode 一样，内部配置数据库信息来直接连接 Hive metadata database (Impala 服务依赖 Hive metadata database)，但是建议与 HiveServer2 公用同一个单独部署的 The Metastore Service。HCatalog 服务必须使用此模式。

## 3. Hive 部署

这里使用 Remote Mode 进行部署，所以我们需要三个服务，HiveServer2，Hive Metastore Service，Mysql。Mysql这里不做叙述，所以本篇文章直讲前两个服务的部署步骤

### 3.1 从官方文档下载安装包

官方地址：http://mirror.bit.edu.cn/apache/hive/

- hive 包：apache-hive-2.3.6-bin.tar.gz
- hive-metastore 包：hive-standalone-metastore-3.0.0-bin.tar.gz

> ***hive 包提供了 hive 所有的服务，hive-metastore 只提供了 metastore 服务。也就是说其实我们只需要第一个 hive 包，然后复制多份，其中一个用来部署独立的 metastore 服务，剩下的启动 HiveServer2 服务即可。但是我这里一开始以为两个服务需要分开下载，所以下载了两个包，而且hive包大概200M+，而metastore包只有25M左右，我部署的hive和metastore服务在同一台机器上，既然下载下来了，就先按照这样的方式部署吧。***

### 3.2 设置全局变量

按需修改指定的环境变量文件，这里设置为全局变量，修改 /etc/profile
这里将 metastore 放在 hive 前面

``` bash
# 配置 HIVE METASTORE HOME
export HIVE_METASTORE_HOME=/your_hive_metastore_path
# 配置 HIVE HOME
export HIVE_HOME=/your_hive_path
# 配置 PATH，
export PATH=HIVE_METASTORE_HOME/bin:$HIVE_HOME/bin:$PATH
```

### 3.3 配置 metastore

#### 3.3.1 配置 metastore 日志

进入 metastore_home/conf 目录下修改 metastore-log4j2.properties 文件

``` properties
property.metastore.log.dir=/your_log_dir
```

#### 3.3.2 配置 metastore

进入 metastore_home/conf 目录下修改 metastore-site.xml 文件，并新增如下配置

``` xml
<!-- hive 存放数据的hdfs路径 -->
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>

<!-- hive metastore 配置 -->
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


```

### 3.6 配置 hive 日志

复制 hive-log4j2.properties.template 为 hive-log4j2.properties 按需修改里面的配置即可

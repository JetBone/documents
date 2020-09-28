# NiFi 配置

## 简单介绍 NiFi

NiFi 是一个自动化的数据流(dataflow)管理工具

### NiFi 核心概念

NiFi 的核心设计与 Flow Based Programming（FBP）的主要思想非常接近。下面是 NiFi 的核心概念以及与 FBP 之间的映射关系

| NiFi Term | FBP Term | Description
| - | - | - |
| FlowFile | Information Packet | FlowFile表示系统与系统之间流转的基本对象，其中包换一些 key/value pair 属性，以及0到多个字节的流数据 |
| FlowFile Processor | Black Box | Processors 才是真正的任务执行者，负责对数据做一些组合，转换，清洗等等任务。Processors 可以访问给定的 FlowFile 中的属性和 stream 流数据。并且对这些流数据进行提交工作或者回滚。|
| Connection | Bounded Buffer | Connections 用来连接处理器。他们充当队列，并且允许多个进程以不同的速率执行任务。这些队列可以动态的调整优先级，并且有负载上限。 |
| Flow Controller | Scheduler | Flow Controller 用于维护 Processor 如何连接，以及分配进程进行任务工作。 |
| Process Group | subnet | 进程组是一组特定的流程和连接。可以通过指定的端口接受数据，然后再通过指定端口输出数据。所以进程组何以通过与其它组件组合的方式形成新的进程组。 |

上述介绍的概念，帮助 NiFi 成为了一个功能强大，扩展性很强的数据流系统，他具有如下优点：

- 具有非常优秀的可视化界面，可以非常简单的配置 Processor
- 异步操作，允许非常高的吞吐量和足够的自然缓冲
- 提供高并发模型，开发者不需要担心并发的复杂性

---

## 2. NiFi 配置和部署

NiFi 有两种部署方式，一种是单机末世，一种是集群模式，这里我们之讨论集群模式

NiFi 集群是是交给 zookeeper 管理的，所以首先我们要部署 zookeeper 然后才能部署 NiFi

zookeeper 可以使用自己提前部署好的，但是 NiFi 本身也自带了内置的 zookeeper 服务，这里由于我们已经有了部署好的 zookeeper，所以就不使用 NiFi 内置的 zookeeper 服务了

> 有关 zookeeper 部署请参考其它文章

### 2.1 环境

| IP | services | role |
| - | - | - |
| 192.168.0.171 | zookeeper, NiFi | zookeeper follower, NiFi node |
| 192.168.0.172 | zookeeper, NiFi | zookeeper leader, NiFi node |
| 192.168.0.173 | zookeeper, NiFi | zookeeper follower, NiFi node |

### 2.2 系统要求

必须要拥有 JVM 环境，NiFi是是跑在 JVM 上的服务，如果没有则需要提前安装，这里已经安装好了，正常配置好 JAVA_HOME 和 PATH 即可。

### 2.3 官网下载 NiFi 包

由于我们的系统是 CentOS 7，所以我们直接在官网下载 tar 包到服务器，并且将下载好的包分发给三台服务器，然后解压

### 2.4 配置 NiFi

解压出来的包中，很明显 conf 目录下保存 NiFi 的所有配置，其中我们需要关注的配置文件有两个，其中 zookeeper.properties 是用来配置内置的 zookeeper 服务用的，我们使用的是外部 zookeeper 服务，所以不需要管这个配置文件。

#### 修改 nifi.properties 配置文件

nifi.properties 是 NiFi 的主要配置文件，下面的代码只简单的介绍一些关键配置

``` properties
####################
# State Management #
####################
# 是否使用内置的 zookeeper 服务，默认为 false，我们不用动它
nifi.state.management.embedded.zookeeper.start=false
# 很明显是用来指定内置 zookeeper 服务配置文件地址的，不需要管
nifi.state.management.embedded.zookeeper.properties=./conf/zookeeper.properties

# web properties #
# 配置 nifi 的 web 端地址，默认为空，这里我们填上IP地址
nifi.web.http.host=192.168.0.171
# nifi.web.http.host=192.168.0.172
# nifi.web.http.host=192.168.0.173
# web 端的默认端口，默认为8080，这里改为8181
nifi.web.http.port=8181

# cluster node properties (only configure for cluster nodes) #
# 配置 nifi 是否为集群模式，默认为 false，这里我们改成 true
nifi.cluster.is.node=true
# 当前 nifi 安装的服务器 IP 地址
nifi.cluster.node.address=192.168.0.171
# nifi.cluster.node.address=192.168.0.172
# nifi.cluster.node.address=192.168.0.173
# 当前节点的协议端口，默认为空
nifi.cluster.node.protocol.port=8182
# 指定在选择Flow作为“正确”流之前等待的时间量。如果已投票的节点数等于nifi.cluster.flow.election.max.candidates属性指定的数量，则群集将不会等待这么长时间。默认值为5 mins
nifi.cluster.flow.election.max.wait.time=1 mins
nifi.cluster.flow.election.max.candidates=1

# zookeeper properties, used for cluster management #
# zookeeper 集群配置
nifi.zookeeper.connect.string=192.168.0.171:2181,192.168.0.172:2181,192.168.0.173:2181,
nifi.zookeeper.connect.timeout=5 secs
nifi.zookeeper.session.timeout=5 secs
nifi.zookeeper.root.node=/nifi
```

#### 修改 state-management.xml 配置文件

``` properties
<cluster-provider>
    <id>zk-provider</id>
    <class>org.apache.nifi.controller.state.providers.zookeeper.ZooKeeperStateProvider</class>
    <!--其实只需要修改下面这一行数据-->
    <property name="Connect String">sparkmaster:2181,sparkslave1:2181,sparkslave2:2181</property>
    <property name="Root Node">/nifi</property>
    <property name="Session Timeout">10 seconds</property>
    <property name="Access Control">Open</property>
</cluster-provider>
```

最后将这两个配置文件分别复制在另外两个节点上，并且修改一下自己的ip配置即可

### 2.5 启动 NiFi

``` shell
# 前台启动，Ctrl+C 退出
bin/nifi.sh start
# 后台启动
bin/nifi.sh start
# 查看状态
bin/nifi.sh status
# 关闭
bin/nifi.sh stop
# 要为服务指定自定义名称,请使用可选的第二个参数（该服务的名称）执行该命令。例如,要将NiFi作为具有dataflow名称的服务安装,请使用该命令
bin/nifi.sh install dataflow
```

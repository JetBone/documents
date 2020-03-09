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

### 2.1 Impala 编译依赖

apache-maven-3.6.3

操作记录

#### 1. 解压安装包

``` bash
tar -zxvf ~~~~
```

#### 2. 尝试直接执行 buildall.sh 出现错误说找不到lsb_release

``` bash
yum install redhat-lsb
```

#### 3. 第二步骤明显有问题，根据官方的一些文档，需要进入bin目录下查找一些配置脚本，帮助我们自动安装依赖

可以看到bin目录下有三个脚本符合要求

``` bash
bootstrap_build.sh
bootstrap_development.sh
bootstrap_system.sh
```

第一个脚本虽然官方文档里写是执行，但是可能文档有些旧了，进入里面发现直接使用apt-get命令，再看一下内容是安装 java 和 maven，然后执行 buildall.sh 脚本，所以我们手动安装 java 和maven，并配置好环境变量，java版本是 1.8.x，maven版本是 3.x 

第二个脚本很简单，先执行bootstrap_system.sh 然后执行 impala-config.sh 然后执行 buildall.sh

第三个脚本是真正的安装依赖，所以接下来，单独执行一下 这个脚本

#### 4. 执行 bootstrap_system.sh 脚本

执行脚本后出现如下错误

``` bash
# 这里没截取全，是因为我已经把问题解决了，这里构造了一个假的
`set-no': 不是有效的标识符
```

出现问题的原因如下

``` bash
REAL_APT_GET=$(ubuntu which apt-get)
# 不允许 function name 中有 “-”，这是我没想到
function apt-get {
  for ITER in $(seq 1 20); do
    echo "ATTEMPT: ${ITER}"
    if sudo -E "${REAL_APT_GET}" "$@"
    then
      return 0
    fi
    sleep "${ITER}"
  done
  echo "NO MORE RETRIES"
  return 1
}
```

经过我的调查发现这个函数是不会被调用的，因为本机是centos系统，所以我把函数名中的“-”改成了“_”

执行成功

#### 5. 接下来直接执行 buildall.sh

``` bash
{IMPALA_HOME}/buildall.sh -noclean -skiptests
```

执行查看打出来的日志发现先下载 python 第三方包，看起来使用的是自己内置的 python，第三方包也是下载到了自己指定的位置，但是下载速度太慢了

所以决定修改一下 pypi 的下载源

#### 6. 终止 buildall.sh，根据源代码修改 python 的 pypi 源

根据日志可以看出下载python包的代码在 {impala_home}/infra/python/deps 目录下，其中 requirement.txt 文件用来指定第三方包，download_requirement 脚本用来下载依赖，实际上，download_requirement 脚本内部调用的是 pip_dowland.py 脚本。所以我们打开 pip_download.py 脚本修改 pypi 源

``` python
# 这行是源代码
PYPI_MIRROR = os.environ.get('PYPI_MIRROR', 'https://pypi.python.org')
# 这行是我新加的 aliyun 源
# PYPI_MIRROR = os.environ.get('PYPI_MIRROR', 'https://mirrors.aliyun.com/pypi')
```

修改之后单独运行 pip_download.py 脚本，发现不知道为什么总是无法下载dateutil包，即使这个包手动去找是有的，所以最后使用手动下载并导入到impala保存包的地方(其实和脚本是同一个目录)，再将 pypi 源切回到原始源，这样不用下载，只需要验证包是否完整即可

执行通过

#### 6. 重新执行 buildall.sh

``` bash
{IMPALA_HOME}/buildall.sh -noclean -skiptests
```

出现错误，具体错误信息没截取，但是内容是 cmake 版本过低，要求 3.2 以上版本，但是我的版本只有 2.8

根据上述描述，cmake 应该是某个依赖，我不记得我安装过这个包，所以这应该是impala自己安装的第三方包，那么需要检查一下 bootstrap_system.py 脚本，直接搜索 yum install，结果如下：

``` bash
redhat sudo yum install -y curl gcc gcc-c++ git krb5-devel krb5-server krb5-workstation \
        libevent-devel libffi-devel make ntp ntpdate ntp-perl openssl-devel cyrus-sasl \
        cyrus-sasl-gssapi cyrus-sasl-devel cyrus-sasl-plain \
        python-devel python-setuptools postgresql postgresql-server \
        wget vim-common nscd cmake lzo-devel fuse-devel snappy-devel zlib-devel \
        psmisc lsof openssh-server redhat-lsb java-1.8.0-openjdk-devel \
        java-1.8.0-openjdk-src python-argparse
```

也就是说，impala 自己确实使用 yum 安装了 cmake，但是版本却不是最终 build 需要的版本，我觉得这是很令人匪夷所思的，毕竟这是官方下载的包，提供的 build 脚本前后出现了这种错误很让人困惑。我觉得 Apache Impala 不是个成熟的版本（或者是我环境的问题也有可能），不过我暂时不考虑这个，等到完全走不下的时候再考虑是否换一个安装方式，所以接下来的步骤是 手动安装高版本的 cmake

#### 7. 手动安装 cmake

手动安装很简单，下载源码之后，执行民工三连

``` bash
./configure
make
make install
```

> 这里我调用./configure之后，告诉我使用 gmake 我也没查，直接 使用 gmake和 gmake install，总之也是成功了，没有去管

查看 cmake 版本

``` bash
cmake --version
```

显示为正确版本，及安装成功

#### 8. 重新执行 buildall.sh

``` bash
{IMPALA_HOME}/buildall.sh -skiptests -foramt
```

看日志进入漫长的 toolchain 下载，其中出现了编码错误

#### 9. 出现编码错误，根据日志可知 调用 bootstrap_toolchain.py 脚本的时候出现了编码错误，修改编码

``` python
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```

#### 10. 重新执行 buildall.sh

``` bash
{IMPALA_HOME}/buildall.sh -skiptests -foramt
```

出现错误，python 需要大于 0.9.3 版本的 thrift 但是没找到，在日志中可以发现是通过之前python下载依赖的目录下，有一个 complied_requirement.py 文件制定了版本，但是里面只指定了 thrift_sasl 版本，没有 thrift，经过调查，这是两个东西，所以我尝试手动修改一下这个文件，让它自动下载

``` txt
impyla == 0.15.0
  bitarray == 0.9.0
  sasl == 0.1.3
  six == 1.11.0
  thrift-sasl == 0.1.0
  # 下面这一行是我新增的
  thrift == 0.10.0
```

手动重新跑一下 pip_download.py 脚本后成功下载

#### 11. 重新执行 buildall.sh

``` bash
{IMPALA_HOME}/buildall.sh -skiptests -foramt
```

执行过程当中出现不少解压错误的问题，都是因为包没有下载全，应该是没有对包的完整性进行校验就解压，导致出现问题，所幸日志中有打印出下载地址，所以手动下载然后将包导入到指定目录下 {impala_home}/toolchain，删除掉解压出一半的目录，重新跑即可

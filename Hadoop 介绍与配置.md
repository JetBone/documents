# HADOOP 安装与配置

## 配置 hadoop

### 配置 hdfs-site.xml

``` xml
<configuration>

  <!-- 应该是一些节点自身的信息存放路径 -->
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>/home/bigdata/name/hadoop</value>
    <description>Path on the local filesystem where theNameNode stores the namespace and transactions logs persistently.</description>
  </property>

  <!-- 存放数据的目录 -->
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/bigdata/data/hadoop</value>
    <description>Comma separated list of paths on the localfilesystem of a DataNode where it should store its blocks.</description>
  </property>

  <!-- 文件备份数量 大于等于slave节点数 -->
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>

  <!-- 说明：dfs.permissions配置为false后，可以允许不要检查权限就生成dfs上的文件，方便倒是方便了，但是你需要防止误删除，请将它设置为true，或者直接将该property节点删除，因为默认就是true -->
  <property>
    <name>dfs.permissions</name>
    <value>false</value>
    <description>need not permissions</description>
  </property>

</configuration>
```

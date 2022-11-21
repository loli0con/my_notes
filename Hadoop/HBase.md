# HBase

## 定义
Apache HBase 是以 hdfs 为数据存储的，一种分布式、可扩展的 NoSQL 数据库。

## 数据模型
HBase 数据模型是一个**稀疏**、**分布式**、**多维**、**排序**的映射，其中映射 map 指代非关系型数据库的 key-Value 结构。

### 逻辑结构
存储数据**稀疏**，数据存储**多维**，不同的行具有不同的列。数据存储整体**有序**，按照RowKey的字典序排列，RowKey为Byte数组。

![HBase+20221118151012](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221118151012.png%2B2022-11-18-15-10-16)

### 存储结构
物理存储结构即为数据映射关系，而在概念视图的空单元格，底层实际根本不存储。

![HBase+20221118151228](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221118151228.png%2B2022-11-18-15-12-33)

### 层级结构

#### Name Space
命名空间，类似于关系型数据库的 database 概念，每个命名空间下有多个表。HBase 两
个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default表是用户默认使用的命名空间。

#### Table
类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明列族即可，不需要声明具体的列。因为数据存储时稀疏的，所有往 HBase 写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景。

#### Row
HBase 表中的每行数据都由一个 RowKey 和多个 Column(列) 组成，数据是按照 RowKey 的字典顺序存储的，并且查询数据时只能根据RowKey进行检索，所以RowKey的设计十分重要。

#### Column
HBase 中的每个列都由 **Column Family(列族)**和 **Column Qualifier(列限定符)**进行限定，例如 info:name，info:age。建表时，只需指明列族，而列限定符无需预先定义。

#### Time Stamp
用于标识数据的不同版本(version)，每条数据写入时，系统会自动为其加上该字段，其值为写入 HBase 的时间。

#### Cell
由`{rowkey, column Family:column Qualifier, timestamp}`唯一确定的单元。cell 中的数据全部是字节码形式存贮。


## 基本结构
![HBase+20221118152302](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221118152302.png%2B2022-11-18-15-23-06)

### 架构角色

#### Master
实现类为 HMaster，负责监控集群中所有的 RegionServer 实例。主要作用如下:
1. 管理元数据表格 hbase:meta，接收用户对表格创建修改删除的命令并执行；
2. 监控 region 是否需要进行负载均衡，故障转移和 region 的拆分。

通过启动多个后台线程监控实现上述功能:
1. LoadBalancer 负载均衡器：周期性监控 region 分布在 regionServer 上面是否均衡，由参数 hbase.balancer.period 控制周期时间，默认 5 分钟。
2. CatalogJanitor 元数据管理器：定期检查和清理 hbase:meta 中的数据。
3. MasterProcWAL master 预写日志处理器：把 master 需要执行的任务记录到预写日志 WAL 中，如果 master 宕机，让 backupMaster 读取日志继续执行。

#### Region Server
Region Server 实现类为 HRegionServer，主要作用如下:
1. 负责数据 cell 的处理，例如写入数据 put，查询数据 get 等；
2. 拆分合并 region 的实际执行者，由 master 监控，由 regionServer 执行。

#### Zookeeper
HBase 通过 Zookeeper 实现 master 的高可用，记录 RegionServer 的部署信息，并且存储 meta 表的位置信息。

在 2.3 版本推出 Master Registry 模式，客户端可以直接访问 master。使用此功能，会加大对 master 的压力，减轻对 Zookeeper 的压力。

#### HDFS
HDFS 为 Hbase 提供最终的底层数据存储服务，同时为 HBase 提供高容错的支持。




## 部署

### Zookeeper部署
Zookeeper 集群的正常部署并启动。
```bash
[hadoop@hadoop102 zookeeper-3.5.7]$ bin/zkServer.sh start
[hadoop@hadoop103 zookeeper-3.5.7]$ bin/zkServer.sh start
[hadoop@hadoop104 zookeeper-3.5.7]$ bin/zkServer.sh start
```

### Hadoop部署
Hadoop 集群的正常部署并启动。
```bash
[hadoop@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh
[hadoop@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh
```

### HBase安装
```bash
# 解压 Hbase 到指定目录
[hadoop@hadoop102 software]$ tar -zxvf hbase-2.4.11-bin.tar.gz -C /opt/module/

# 配置环境变量
[hadoop@hadoop102 ~]$ sudo vim /etc/profile.d/jh_env.sh
###### /etc/profile.d/jh_env.sh ######
# 添加->HBASE_HOME
export HBASE_HOME=/opt/module/hbase
export PATH=$PATH:$HBASE_HOME/bin
###### /etc/profile.d/jh_env.sh ######
```

### HBase配置
要修改的配置文件都在`$HBASE_HOME/conf`目录下。

#### hbase-env.sh
```bash
export HBASE_MANAGES_ZK=false
```

#### hbase-site.xml
```xml
<configuration>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>hadoop102,hadoop103,hadoop104</value>
        <description>The directory shared by RegionServers.</description>
    </property>

<!-- 
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/export/zookeeper</value>
        <description>
            记得修改 ZK 的配置文件
            ZK 的信息不能保存到临时文件夹
        </description>
    </property>
 -->

    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://hadoop102:8020/hbase</value>
        <description>The directory shared by RegionServers. </description>
    </property>

    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
</configuration>
```

#### regionservers
```
hadoop102
hadoop103
hadoop104
```

### 解决log4j兼容性问题
解决 HBase 和 Hadoop 的 log4j 兼容性问题，修改(剔除) HBase 的 jar 包，使用 Hadoop 的 jar 包。
```bash
mv $HBASE_HOME/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar $HBASE_HOME/lib/client-facing-thirdparty/slf4j-reload4j-1.7.33.jar.bak
```

### HBase启停
```bash
# 启动
$HBASE_HOME/bin/start-hbase.sh

# 停止
$HBASE_HOME/bin/stop-hbase.sh
```

### HBase查看
启动成功后，可以通过“host:port”的方式来访问 HBase 管理页面，例如: `http://hadoop102:16010`。


### HBase高可用
在 HBase 中 HMaster 负责监控 HRegionServer 的生命周期，均衡 RegionServer 的负载，如果 HMaster 挂掉了，那么整个 HBase 集群将陷入不健康的状态，并且此时的工作状态并不会维持太久。所以 HBase 支持对 HMaster 的高可用配置。

```bash
# 关闭 HBase 集群
$HBASE_HOME/bin/stop-hbase.sh

# 在 conf 目录下创建 backup-masters 文件
touch $HBASE_HOME/conf/backup-masters

# 在 backup-masters 文件中配置高可用 HMaster 节点
echo hadoop103 > $HBASE_HOME/conf/backup-masters

# 将整个 conf 目录 scp 到其他节点
xsync $HBASE_HOME/conf
```


## 详细架构

### Master 架构
![HBase+20221118173250](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221118173250.png%2B2022-11-18-17-32-54)

1. Meta 表格: 全称 **hbase:meta**，只是在 list 命令中被过滤掉了，本质上和 HBase 的其他表格一样。表格中一行数据代表一个 region。
2. RowKey: **(\[table],[region start key],[region id])** 即 表名(存储在HBase中的表)，region起始位置(表被拆分才会有值) 和 regionID。
3. 列: 
   1. **info:regioninfo** 为 region 信息，存储一个 HRegionInfo 对象。
   2. **info:server** 当前 region 所处的 RegionServer 信息，包含端口号。
   3. **info:serverstartcode** 当前 region 被分到 RegionServer 的起始时间。
   4. 如果一个表处于切分的过程中，即 region 切分，还会多出两列 **info:splitA** 和 **info:splitB**，存储值也是 HRegionInfo 对象，拆分结束后，删除这两列。

在客户端对元数据进行操作的时候才会连接 master，如果对数据进行读写，直接连接 zookeeper 读取目录`/hbase/meta-region-server`节点信息(记录了 meta 表格的位置)。此时不需要访问 master，这样可以减轻 master 的压力，相当于 master 专注 meta 表的写操作，客户端可直接读取 meta 表。


### RegionServer 架构
![HBase+20221118174215](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221118174215.png%2B2022-11-18-17-42-18)

1. MemStore: 写缓存，由于 HFile 中的数据要求是有序的，所以数据是先存储在 MemStore 中，排好序后，等到达刷写时机才会刷写到 HFile，每次刷写都会形成一个新的 HFile，写入到对应的文件夹 store 中。
2. WAL: 由于数据要经 MemStore 排序后才能刷写到 HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做 Write-Ahead logfile 的文件中，然后再写入 MemStore 中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。
3. BlockCache: 读缓存，每次查询出的数据会缓存在 BlockCache 中，方便下次查询。



## 写流程
![HBase+20221121112630](https://raw.githubusercontent.com/loli0con/picgo/master/images/HBase%2B20221121112630.png%2B2022-11-21-11-26-36)

写流程顺序和 API 编写顺序一致:
1. 创建 HBase 的重量级连接
   1. 访问 zookeeper，获取 hbase:meta 表位于哪个 Region Server
   2. 访问对应的 Region Server，获取 hbase:meta 表，将其缓存到连接中，作为连接属性 MetaCache，由于 Meta 表格具有一定的数据量，导致了创建连接比较慢(之后使用创建的连接获取 Table，这是一个轻量级的连接)
   3. 只有在第一次创建的时候会检查表格是否存在访问 RegionServer，之后在获取 Table 时不会访问 RegionServer
2. 调用 Table 的 put 方法写入数据，此时还需要解析 RowKey，对照缓存的 MetaCache，查看具体写入的位置有哪个 RegionServer
3. 将数据顺序写入(追加)到 WAL，此处写入是直接落盘的，并设置专门的线程控制 WAL 预写日志的滚动(类似 Flume)
4. 根据写入命令的 RowKey 和 ColumnFamily 查看具体写入到哪个 MemStore，并且在 MemStore 中排序
5. 向客户端发送 ack
6. 等达到 MemStore 的刷写时机后，将数据刷写到对应的 store 中

### MemStore Flush
MemStore 刷写由多个线程控制，条件互相独立，主要的刷写规则是控制刷写文件的大小，在每一个刷写线程中都会进行监控:
1. 当某个 memstroe 的大小达到了 `hbase.hregion.memstore.flush.size(默认值 128M)`，其所在 region 的所有 memstore 都会刷写；当 memstore 的大小达到了 `hbase.hregion.memstore.flush.size(默认值 128M) * hbase.hregion.memstore.block.multiplier(默认值 4)` 时，会刷写同时阻止继续往该 memstore 写数据(由于线程监控是周期性的，所有有可能面对数据洪峰，尽管可能性比较小)。
2. 由 HRegionServer 中的属性 MemStoreFlusher 内部线程 FlushHandler 控制。标准为 LOWER_MARK(低水位线)和 HIGH_MARK(高水位线)，意义在于避免写缓存使用过多的内存造成 OOM。当 region server 中 memstore 的总大小达到低水位线 `java_heapsize * hbase.regionserver.global.memstore.size(默认值 0.4) * hbase.regionserver.global.memstore.size.lower.limit(默认值 0.95)`，region 会按照其所有 memstore 的大小顺序(由大到小)依次进行刷写，直到 region server 中所有 memstore 的总大小减小到上述值以下；当 region server 中 memstore 的总大小达到高水位线 `java_heapsize *hbase.regionserver.global.memstore.size(默认值 0.4)`时，会同时阻止继续往所有的 memstore 写数据。
3. 为了避免数据过长时间处于内存之中，到达自动刷写的时间，也会触发 memstore flush。由 HRegionServer 的属性 PeriodicMemStoreFlusher 控制进行，由于重要性比较低，5min才会执行一次。自动刷新的时间间隔由该属性进行配置 `hbase.regionserver.optionalcacheflushinterval(默认 1 小时)`。
4. 当 WAL 文件的数量超过 hbase.regionserver.max.logs，region 会按照时间顺序依次进行刷写，直到 WAL 文件数量减小到 hbase.regionserver.max.logs 以下(该属性名已经废弃，现无需手动设置，最大值为32)。


## 读流程
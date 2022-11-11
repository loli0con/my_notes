# HDFS

## 定义
HDFS(Hadoop Distributed File System)是一个文件系统，用于存储文件，通过目录树来定位文件;其次，它是分布式的，由很多服务器联合起来实现其功能，集群中的服务器有各自的角色。

HDFS的使用场景: 适合一次写入，多次读出的场景。一个文件经过创建、写入和关闭之后就不需要改变。

## 优点
1. 高容错性
   1. 数据自动保存多个副本。它通过增加副本的形式，提高容错性
   2. 某一个副本丢失以后，它可以自动恢复
2. 适合处理大数据
   1. 数据规模: 能够处理数据规模达到GB、TB、甚至PB级别的数据
   2. 文件规模: 能够处理百万规模以上的文件数量，数量相当之大
3. 可构建在廉价机器上，通过多副本机制，提高可靠性。

## 缺点
1. 不适合低延时数据访问，比如毫秒级的存储数据，是做不到的。
2. 无法高效的对大量小文件进行存储
   1. 存储大量小文件的话，它会占用NameNode大量的内存来存储文件目录和块信息。这样是不可取的，因为NameNode的内存总是有限的
   2. 小文件存储的寻址时间会超过读取时间，它违反了HDFS的设计目标
3. 不支持并发写入、文件随机修改
   1. 一个文件只能有一个写，不允许多个线程同时写
   2. 仅支持数据append(追加)，不支持文件的随机修改

## 组成架构
![HDFS+hdfsarchitecture](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2Bhdfsarchitecture.png%2B2022-11-10-15-34-32)

1. NameNode: 就是Master，它是一个主管、管理者
   1. 管理HDFS的名称空间
   2. 配置副本策略
   3. 管理数据块(Block)映射信息
   4. 处理客户端读写请求
2. DataNode: 就是Slave。NameNode 下达命令，DataNode执行实际的操作
   1. 存储实际的数据块
   2. 执行数据块的读/写操作
3. Client: 就是客户端
   1. 文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行上传
   2. 与NameNode交互，获取文件的位置信息
   3. 与DataNode交互，读取或者写入数据
   4. Client提供一些命令来管理HDFS，比如NameNode格式化
   5. Client可以通过一些命令来访问HDFS，比如对HDFS增删查改操作
4. Secondary NameNode: 并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。
   1. 辅助NameNode，分担其工作量，比如定期合并Fsimage和Edits，并推送给NameNode
   2. 在紧急情况下，可辅助恢复NameNode

## HDFS 文件块
HDFS中的文件在物理上是分块存储(Block)，块的大小可以通过配置参数(dfs.blocksize)来规定，默认大小在Hadoop2.x/3.x版本中是128M，1.x版本中是64M。

1. 寻址时间为传输时间的 1% 时，则为最佳状态
2. HDFS的块设置太小，会增加寻址时间，程序一直在找块的开始位置
3. 如果块设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间。导致程序在处理这块数据时，会非常慢
4. HDFS块的大小设置主要取决于磁盘传输速率


## HDFS Shell
```sh
# bin/hadoop fs
Usage: hadoop fs [generic options]
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] [-l] [-d] <localsrc> ... <dst>]
        [-copyToLocal [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] [-h] [-v] [-t [<storage type>]] [-u] [-x] <path> ...]
        [-cp [-f] [-p | -p[topax]] [-d] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] [-x] <path> ...]
        [-expunge]
        [-find <path> ... <expression> ...]
        [-get [-f] [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getfattr [-R] {-n name | -d} [-e en] <path>]
        [-getmerge [-nl] [-skip-empty-file] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] [-l] [-d] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] [-safely] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setfattr {-n name [-v value] | -x name} <path>]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-truncate [-w] <length> <path> ...]
        [-usage [cmd ...]]

Generic options supported are:
-conf <configuration file>        specify an application configuration file
-D <property=value>               define a value for a given property
-fs <file:///|hdfs://namenode:port> specify default filesystem URL to use, overrides 'fs.defaultFS' property from configurations.
-jt <local|resourcemanager:port>  specify a ResourceManager
-files <file1,...>                specify a comma-separated list of files to be copied to the map reduce cluster
-libjars <jar1,...>               specify a comma-separated list of jar files to be included in the classpath
-archives <archive1,...>          specify a comma-separated list of archives to be unarchived on the compute machines

The general command line syntax is:
command [genericOptions] [commandOptions]
```

## HDFS API

### win开发环境
1. 下载 Hadoop-winutils 工具包(放置非中文路径)
2. 配置 HADOOP_HOME 环境变量
3. 配置 Path 环境变量(%HADOOP_HOME%\bin)

### FileSystem
org.apache.hadoop.fs.FileSystem类提供相关文件系统API，org.apache.hadoop.conf.Configuration类用于配置参数，使用方式如下：
```Java
// 1 获取文件系统
Configuration configuration = new Configuration();
FileSystem fs = FileSystem.get(new URI("hdfs://hadoop102:8020"),configuration,"hadoop");

// 2 执行相关操作
dosomething(fs);

// 3 关闭资源
fs.close();
```

参数优先级排序(由高到低):
1. 客户端代码中设置的值
2. ClassPath 下的用户自定义配置文件
3. 服务器的自定义配置(xxx-site.xml)
4. 服务器的默认配置(xxx-default.xml)



## HDFS写数据流程

### 文件写入
![HDFS+20221111142238](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111142238.png%2B2022-11-11-14-22-40)

1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode 检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block 上传到哪几个 DataNode 服务器上。
4. NameNode 返回 3 个 DataNode 节点，分别为 dn1、dn2、dn3。
5. 客户端通过 FSDataOutputStream 模块请求 dn1 上传数据，dn1 收到请求会继续调用 dn2，然后 dn2 调用 dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3 逐级应答客户端。
7. 客户端开始往 dn1 上传第一个 Block(先从磁盘读取数据放到一个本地内存缓存)，以 Packet 为单位，dn1 收到一个 Packet 就会传给 dn2，dn2 传给 dn3；dn1 每传一个 packet 会放入一个应答队列等待应答。
8. 当一个 Block 传输完成之后，客户端再次请求 NameNode 上传第二个 Block 的服务器。(重复执行 3-7 步)

### 网络拓扑-节点距离计算
在 HDFS 写数据的过程中，NameNode 会选择距离待上传数据最近距离的 DataNode 接收数据。节点距离为两个节点到达最近的共同祖先的距离总和。

假设有数据中心 d1 机架 r1 中的节点 n1，该节点可以表示为/d1/r1/n1，下面给出四种距离描述。
![HDFS+20221111143135](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111143135.png%2B2022-11-11-14-31-37)


### 副本存储节点选择
![HDFS+20221111143306](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111143306.png%2B2022-11-11-14-33-06)

1. 第一个副本在Client所处的节点上。如果客户端在集群外，随机选一个。
2. 第二个副本在另一个机架的随机一个节点
3. 第三个副本在第二个副本所在机架的随机节点


## HDFS读数据流程

### 文件读取
![HDFS+20221111143448](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111143448.png%2B2022-11-11-14-34-50)

1. 客户端通过 DistributedFileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
2. 挑选一台 DataNode(就近原则，然后随机)服务器，请求读取数据。
3. DataNode 开始传输数据给客户端(从磁盘里面读取数据输入流，以 Packet 为单位来做校验)。
4. 客户端以 Packet 为单位接收，先在本地缓存，然后写入目标文件。



## NameNode 和 SecondaryNameNode

### 工作机制
1. NameNode 中的元数据是存储在内存中，同时在磁盘中备份元数据的 FsImage(镜像文件)。
2. 每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到 Edits(编辑日志) 中。
3. 元数据由 FsImage 和 Edits 的合并得到。
4. SecondaryNamenode专门用于 FsImage 和 Edits 的合并。

### 工作流程
![HDFS+20221111144053](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111144053.png%2B2022-11-11-14-40-55)
1. 第一阶段: NameNode 启动
   1. 第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
   2. 客户端对元数据进行增删改的请求。
   3. NameNode 记录操作日志，更新滚动日志。
   4. NameNode 在内存中对元数据进行增删改。
2. 第二阶段 :Secondary NameNode 工作
   1. Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode是否检查结果。
   2. Secondary NameNode 请求执行 CheckPoint。
   3. NameNode 滚动正在写的 Edits 日志。
   4. 将滚动前的编辑日志和镜像文件拷贝到 Secondary NameNode。
   5. Secondary NameNode 加载编辑日志和镜像文件到内存完成合并。
   6. 生成新的镜像文件 fsimage.chkpoint。
   7. 生成新的镜像文件 fsimage.chkpoint。
   8. NameNode 将 fsimage.chkpoint 重新命名成 fsimage。

### Fsimage 和 Edits

#### 概念
1. Fsimage文件:HDFS文件系统元数据的一个永久性的检查点，其中包含HDFS文件系统的所有目录和文件inode的序列化信息。
2. Edits文件:存放HDFS文件系统的所有更新操作的路径，文件系统客户端执行的所有写操作首先 会被记录到Edits文件中。
3. seen_txid文件保存的是一个数字，就是最后一个edits_的数字。
4. 每次NameNode启动的时候都会将Fsimage文件读入内存，加载Edits里面的更新操作，保证内存中的元数据信息是最新的、同步的，可以看成NameNode启动的时候就将Fsimage和Edits文件进行了合并。

NameNode被格式化之后，将在/opt/module/hadoop-3.1.3/data/tmp/dfs/name/current目录中产生如下文件:
* fsimage_0000000000000000000
* fsimage_0000000000000000000.md5
* seen_txid
* VERSION


#### oiv 查看 Fsimage 文件
* 基本语法: `hdfs oiv -p 文件类型 -i 镜像文件 -o 转换后文件输出路径`
* 案例实操: `hdfs oiv -p XML -i fsimage_0000000000000000025 -o /opt/module/hadoop-3.1.3/fsimage.xml`

#### oev 查看 Edits 文件
* 基本语法: `hdfs oev -p 文件类型 -i 编辑日志 -o 转换后文件输出路径`
* 案例实操: `hdfs oev -p XML -i edits_0000000000000000012-0000000000000000013 -o /opt/module/hadoop-3.1.3/edits.xml`

#### CheckPoint 时间设置
通常情况下，SecondaryNameNode 每隔一小时执行一次。
```xml
<!-- hdfs-default.xml -->
<property>
  <name>dfs.namenode.checkpoint.period</name>
  <value>3600s</value>
</property>
```

一分钟检查一次操作次数，当操作次数达到 1 百万时，SecondaryNameNode 执行一次。
```xml
<!-- hdfs-default.xml -->
<property>
   <name>dfs.namenode.checkpoint.txns</name>
   <value>1000000</value>
   <description>操作动作次数</description>
</property>
<property>
   <name>dfs.namenode.checkpoint.check.period</name>
   <value>60s</value>
   <description>1 分钟检查一次操作次数</description>
</property>
```


## DataNode

### 工作机制
![HDFS+20221111145526](https://raw.githubusercontent.com/loli0con/picgo/master/images/HDFS%2B20221111145526.png%2B2022-11-11-14-55-28)

1. 一个数据块在 DataNode 上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
2. DataNode 启动后向 NameNode 注册，通过后，周期性(6 小时)的向 NameNode 上报所有的块信息。
3. 心跳是每 3 秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过 10 分钟没有收到某个 DataNode 的心跳，则认为该节点不可用。
4. 集群运行中可以安全加入和退出一些机器。

### 配置
DN 向 NN 汇报当前解读信息的时间间隔，默认 6 小时。
```xml
<property>
    <name>dfs.blockreport.intervalMsec</name>
    <value>21600000</value>
    <description>Determines block reporting interval in milliseconds.</description>
</property>
```

DN 扫描自己节点块信息列表的时间，默认 6 小时。
```xml
<property>
   <name>dfs.datanode.directoryscan.interval</name>
   <value>21600s</value>
   <description>
   Interval in seconds for Datanode to scan data directories and reconcile the difference between blocks in memory and on the disk.
   Support multiple time unit suffix(case insensitive), as described in dfs.heartbeat.interval.
   </description>
</property>
```

### 数据完整性
DataNode 节点保证数据完整性的方法:
1. 当 DataNode 读取 Block 的时候，它会计算 CheckSum。如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经损坏。常见的校验算法 crc(32)，md5(128)，sha1(160)。
2. DataNode 在其文件创建后周期验证 CheckSum。
3. Client 读取其他 DataNode 上的 Block。


### 掉线时限
1. DataNode进程死亡或者网络故障造成DataNode 无法与NameNode通信
2. NameNode不会立即把该节点判定 为死亡，要经过一段时间，这段时间暂称作超时时长
3. HDFS默认的超时时长为10分钟+30秒
4. 如果定义超时时间为TimeOut，则超时时长的计算公式为`TimeOut = 2 * dfs.namenode.heartbeat.recheck-interval + 10 * dfs.heartbeat.interval`。默认的dfs.namenode.heartbeat.recheck-interval 大小为5分钟，dfs.heartbeat.interval默认为3秒
5. 需要注意的是 hdfs-site.xml 配置文件中的 heartbeat.recheck.recheck-interval 的单位为毫秒，dfs.heartbeat.interval 的单位为秒
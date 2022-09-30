# Broker

## 下载
官方下载地址:http://kafka.apache.org/downloads.html

## 解压
```bash
# 假设使用的Kafka版本为3.0.0
tar -zxvf kafka_2.12-3.0.0.tgz -C /opt/module/
mv kafka_2.12-3.0.0/ kafka
```

后续的pwd为：`/opt/module/kafka`

## 配置

### Kafka服务运行设置demo
config/server.properties
```properties
#broker 的全局唯一编号，不能重复，只能是数字。
broker.id=0

#处理网络请求的线程数量
num.network.threads=3

#用来处理磁盘 IO 的线程数量
num.io.threads=8

#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400

#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400

#请求套接字的缓冲区大小
socket.request.max.bytes=104857600

#kafka 运行日志(数据)存放的路径，路径不需要提前创建，kafka 自动帮你创建，可以配置多个磁盘路径，路径与路径之间可以用","分隔
log.dirs=/opt/module/kafka/datas

#topic 在当前 broker 上的分区个数
num.partitions=1

#用来恢复和清理 data 下数据的线程数量
num.recovery.threads.per.data.dir=1

# 每个topic创建时的副本数，默认时1个副本
offsets.topic.replication.factor=1

#segment 文件保留的最长时间，超时将被删除
log.retention.hours=168

#每个 segment 文件的大小，默认最大 1G
log.segment.bytes=1073741824

# 检查过期数据的时间，默认5分钟检查一次是否数据过期
log.retention.check.interval.ms=300000

#配置连接 Zookeeper 集群地址(在 zk 根目录下创建/kafka，方便管理)
zookeeper.connect=zkhost1:2181,zkhost2:2181,zkhost3:2181/kafka
```

### 宿主机环境配置
下面的命令需要配置到对应的宿主机系统配置文件中，否则仅当前bash生效。
```bash
# 配置环境变量
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin 
```

## 启动
```bash
# 先启动 Zookeeper 集群，然后启动 Kafka。
bin/kafka-server-start.sh -daemon config/server.properties
```

## 关闭
```bash
# 停止 Kafka 集群时，一定要等 Kafka 所有节点进程全部停止后再停止 Zookeeper 集群。因为 Zookeeper 集群当中记录着 Kafka 集群相关信息，Zookeeper 集群一旦先停止，Kafka 集群就没有办法再获取停止进程的信息，只能手动杀死 Kafka 进程了。
bin/kafka-server-stop.sh
```



## 主题命令行操作

### 查看操作主题命令参数
```
bin/kafka-topics.sh
```

|参数|描述|
|---|---|
|--bootstrap-server <String: server toconnect to>|连接的 Kafka Broker 主机名称和端口号。|
|--topic <String: topic>|操作的 topic 名称。|
|--create|创建主题。|
|--delete|删除主题。|
|--alter|修改主题。|
|--list|查看所有主题。|
|--describe|查看主题详细描述。|
|--partitions <Integer: # of partitions>|设置分区数。|
|--replication-factor <Integer: replication factor>|设置分区副本。|
|--config <String: name=value>|更新系统默认的配置。|


### 生产者命令行操作
```
bin/kafka-console-producer.sh
```
|--bootstrap-server <String: server toconnect to>|连接的 Kafka Broker 主机名称和端口号。|
|--topic <String: topic>|操作的 topic 名称。|

### 消费者命令行操作
```
bin/kafka-console-consumer.sh
```
|--bootstrap-server <String: server toconnect to>|连接的 Kafka Broker 主机名称和端口号。|
|--topic <String: topic>|操作的 topic 名称。|
|--from-beginning|从头开始消费。|
|--group <String: consumer group id>|指定消费者组名称。|



## Zookeeper中存储的信息
![Broker+20220928110804](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928110804.png%2B2022-09-28-11-08-06)


## 总体工作流程
![Broker+20220928111802](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928111802.png%2B2022-09-28-11-18-04)

## broker重要配置
|参数名称|参数描述|
|---|---|
|replica.lag.time.max.ms|ISR 中，如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值，默认 30s。|
|auto.leader.rebalance.enable|默认是 true。自动 Leader Partition 平衡。|
|leader.imbalance.per.broker.percentage|默认是 10%。每个 broker 允许的不平衡的 leader 的比率。如果每个 broker 超过了这个值，控制器会触发 leader 的平衡。|
|leader.imbalance.check.interval.seconds|默认值 300 秒。检查 leader 负载是否平衡的间隔时间。|
|log.segment.bytes|Kafka 中 log 日志是分成一块块存储的，此配置是指 log 日志划分成块的大小，默认值 1G。|
|log.index.interval.bytes|默认 4kb，kafka 里面每当写入了 4kb 大小的日志(.log)，然后就往 index 文件里面记录一个索引。|
|log.retention.hours|Kafka 中数据保存的时间，默认 7 天。|
|log.retention.minutes|Kafka 中数据保存的时间，分钟级别，默认关闭。|
|log.retention.ms|Kafka 中数据保存的时间，毫秒级别，默认关闭。|
|log.retention.check.interval.ms|检查数据是否保存超时的间隔，默认是 5 分钟。|
|log.retention.bytes|默认等于-1，表示无穷大。超过设置的所有日志总大小，删除最早的 segment。|
|log.cleanup.policy|默认是 delete，表示所有数据启用删除策略;如果设置值为 compact，表示所有数据启用压缩策略。|
|num.io.threads|默认是 8。负责写磁盘的线程数。整个参数值要占总核数的 50%。|
|num.replica.fetchers|副本拉取线程数，这个参数占总核数的 50% 的 1/3。|
|num.network.threads|默认是 3。数据传输线程数，这个参数占总核数的 50% 的 2/3。|
|log.flush.interval.messages|强制页缓存刷写到磁盘的条数，默认是 long 的大值，9223372036854775807。一般不建议修改，交给系统自己管理。|
|log.flush.interval.ms|每隔多久，刷数据到磁盘，默认是 null。一般不建议修改，交给系统自己管理。|



## 负载均衡操作
在节点服役和退役等场景下，需要执行负载均衡的操作，以达到重新调整分区的目的。
```bash
# 指定要均衡的主题
vim topics-to-move.json
{
"topics": [
        {"topic": "first"}
    ],
    "version": 1
}


# 生成一个负载均衡的计划，其中--broker-list指定参与负载均衡的broker的id。
bin/kafka-reassign-partitions.sh -- bootstrap-server host1:9092 --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2,3" --generate
# 以下为输出内容：
# part1.当前分区指派
# part2.推荐分区重新指派配置
Current partition replica assignment
{"version":1,"partitions":[{"topic":"first","partition":0,"replic as":[0,2,1],"log_dirs":["any","any","any"]},{"topic":"first","par tition":1,"replicas":[2,1,0],"log_dirs":["any","any","any"]},{"to pic":"first","partition":2,"replicas":[1,0,2],"log_dirs":["any"," any","any"]}]}
Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"first","partition":0,"replic as":[2,3,0],"log_dirs":["any","any","any"]},{"topic":"first","par tition":1,"replicas":[3,0,1],"log_dirs":["any","any","any"]},{"to pic":"first","partition":2,"replicas":[0,1,2],"log_dirs":["any"," any","any"]}]}


# 创建副本存储计划（对应上面命令输出的part2内容）
vim increase-replication-factor.json
{"version":1,"partitions":[{"topic":"first","partition":0,"replic as":[2,3,0],"log_dirs":["any","any","any"]},{"topic":"first","par tition":1,"replicas":[3,0,1],"log_dirs":["any","any","any"]},{"to pic":"first","partition":2,"replicas":[0,1,2],"log_dirs":["any"," any","any"]}]}


# 执行副本存储计划
bin/kafka-reassign-partitions.sh -- bootstrap-server host1:9092 --reassignment-json-file increase-replication-factor.json --execute


# 验证副本存储计划
bin/kafka-reassign-partitions.sh --
bootstrap-server host1:9092 --reassignment-json-file increase-replication-factor.json --verify
# 以下为输出内容
Status of partition reassignment:
Reassignment of partition first-0 is complete.
Reassignment of partition first-1 is complete.
Reassignment of partition first-2 is complete.

Clearing broker-level throttles on brokers 0,1,2,3
Clearing topic-level throttles on topic first
```



## Kafka 副本

### 基本介绍
1. Kafka 副本作用: 提高数据可靠性。
2. Kafka 默认副本 1 个，生产环境一般配置为 2 个，保证数据可靠性;太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率。
3. Kafka 中副本分为: Leader 和 Follower。
   1. Kafka 生产者只会把数据发往 Leader
   2. 然后 Follower 找 Leader 进行同步数据。
4. Kafka 分区中的所有副本统称为 AR(Assigned Repllicas)。 AR = ISR + OSR
   1. ISR，表示和 Leader 保持同步的 Follower 集合。如果 Follower 长时间未向 Leader 发送通信请求或同步数据，则该 Follower 将被踢出 ISR。该时间阈值由 replica.lag.time.max.ms 参数设定，默认 30s。Leader 发生故障之后，就会从 ISR 中选举新的 Leader。
   2. OSR，表示 Follower 与 Leader 副本同步时，延迟过多的副本。

### Leader 选举流程
Kafka 集群中有一个 broker 的 Controller 会被选举为 Controller Leader，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 Leader 选举等工作。Controller 的信息同步工作是依赖于 Zookeeper 的。

![Broker+20220928172342](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928172342.png%2B2022-09-28-17-23-45)

### 故障处理

#### Follower故障处理
![Broker+20220928172524](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928172524.png%2B2022-09-28-17-25-26)

#### Leader故障处理
![Broker+20220928172613](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928172613.png%2B2022-09-28-17-26-15)


### 分区副本分配
4 个 broker，创建 16 分区，3 个副本：
```bash
# 创建一个新的 topic，名称为 second。
bin/kafka-topics.sh --bootstrap-server host1:9092 --create --partitions 16 --replication-factor 3 -- topic second

# 查看分区和副本情况
bin/kafka-topics.sh --bootstrap-server host1:9092 --describe --topic second
Topic: second4 Partition: 0    Leader: 0    Replicas: 0,1,2    Isr: 0,1,2
Topic: second4 Partition: 1    Leader: 1    Replicas: 1,2,3    Isr: 1,2,3
Topic: second4 Partition: 2    Leader: 2    Replicas: 2,3,0    Isr: 2,3,0
Topic: second4 Partition: 3    Leader: 3    Replicas: 3,0,1    Isr: 3,0,1

Topic: second4 Partition: 4    Leader: 0    Replicas: 0,2,3    Isr: 0,2,3
Topic: second4 Partition: 5    Leader: 1    Replicas: 1,3,0    Isr: 1,3,0
Topic: second4 Partition: 6    Leader: 2    Replicas: 2,0,1    Isr: 2,0,1
Topic: second4 Partition: 7    Leader: 3    Replicas: 3,1,2    Isr: 3,1,2

Topic: second4 Partition: 8    Leader: 0    Replicas: 0,3,1    Isr: 0,3,1
Topic: second4 Partition: 9    Leader: 1    Replicas: 1,0,2    Isr: 1,0,2
Topic: second4 Partition: 10   Leader: 2    Replicas: 2,1,3    Isr: 2,1,3
Topic: second4 Partition: 11   Leader: 3    Replicas: 3,2,0    Isr: 3,2,0

Topic: second4 Partition: 12   Leader: 0    Replicas: 0,1,2    Isr: 0,1,2 
Topic: second4 Partition: 13   Leader: 1    Replicas: 1,2,3    Isr: 1,2,3
Topic: second4 Partition: 14   Leader: 2    Replicas: 2,3,0    Isr: 2,3,0
Topic: second4 Partition: 15   Leader: 3    Replicas: 3,0,1    Isr: 3,0,1
```
分区副本分配会尽可能均匀，上述例子的情况如下图所示：
![Broker+20220928173731](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928173731.png%2B2022-09-28-17-37-33)


### 增加副本
```bash
# 创建 topic
bin/kafka-topics.sh --bootstrap-server host1:9092 --create --partitions 3 --replication-factor 1 -- topic four

# 手动增加副本存储
vim increase-replication-factor.json
{
    "version":1,
    "partitions":[
        {"topic":"four","partition":0,"replicas":[0,1,2]},
        {"topic":"four","partition":1,"replicas":[0,1,2]},
        {"topic":"four","partition":2,"replicas":[0,1,2]}
    ]
}

# 执行副本存储计划
bin/kafka-reassign-partitions.sh -- bootstrap-server host1:9092 --reassignment-json-file increase-replication-factor.json --execute
```

### 手动调整分区副本
创建一个新的topic，4个分区，两个副本，名称为three。将该topic的所有副本都存储到broker0和broker1两台服务器上。
![Broker+20220928173929](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928173929.png%2B2022-09-28-17-39-30)

```bash
# 创建一个新的 topic，名称为 three。
bin/kafka-topics.sh --bootstrap-server host1:9092 --create --partitions 4 --replication-factor 2 -- topic three

# 创建副本存储计划(所有副本都指定存储在 broker0、broker1 中)。
vim increase-replication-factor.json
{
    "version":1,
    "partitions":[
        {"topic":"three","partition":0,"replicas":[0,1]},
        {"topic":"three","partition":1,"replicas":[0,1]},
        {"topic":"three","partition":2,"replicas":[1,0]},
        {"topic":"three","partition":3,"replicas":[1,0]}
    ]
}

# 执行副本存储计划
bin/kafka-reassign-partitions.sh -- bootstrap-server host1:9092 --reassignment-json-file increase-replication-factor.json --execute
 
# 验证副本存储计划
bin/kafka-reassign-partitions.sh -- bootstrap-server host1:9092 --reassignment-json-file increase-replication-factor.json --verify
```

### Leader Partition 负载平衡
正常情况下，Kafka本身会自动把Leader Partition均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某些broker宕机，会导致Leader Partition过于集中在其他少部分几台broker上，这会导致少数几台broker的读写请求压力过高，其他宕机的broker重启之后都是follower partition，读写请求很低，造成集群负载不均衡。

相关参数
|参数名称|参数描述|
|---|---|
|auto.leader.rebalance.enable|默认是 true。自动 Leader Partition 平衡。生产环境中，leader 重选举的代价比较大，可能会带来性能影响，建议设置为 false 关闭。|
|leader.imbalance.per.broker.percentage|默认是 10%。每个 broker 允许的不平衡的 leader 的比率。如果每个 broker 超过了这个值，控制器会触发 leader 的平衡。|
|leader.imbalance.check.interval.seconds|默认值 300 秒。检查 leader 负载是否平衡的间隔时间。|

#### 不平衡率的计算
假设集群只有一个主题如下图所示:
![Broker+20220928174945](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220928174945.png%2B2022-09-28-17-49-46)

分析如下：
* broker0: 分区2的AR优先副本是broker0节点，但是broker0节点却不是Leader节点，所以不平衡数加1，AR副本总数是4，所以broker0节点不平衡率为1/4>10%，需要再平衡。
* Broker1: 不平衡数为0，不需要再平衡
* broker2: 和broker0不平衡率一样，需要再平衡。
* broker3: 和broker0不平衡率一样，需要再平衡。




## 文件存储

### Topic 数据的存储机制
Topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是Producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制，将每个partition分为多个segment。每个segment包括:“.index”文件、“.log”文件和.timeindex等文件。这些文件位于一个文件夹下，该文件夹的命名规则为:topic名称+分区序号，例如:first-0。

![Broker+20220930101545](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20220930101545.png%2B2022-09-30-10-15-47)

### 查看 index 和 log
```bash
# 查看index
kafka-run-class.sh kafka.tools.DumpLogSegments --files ./00000000000000000000.index
# 输出
Dumping ./00000000000000000000.index
offset: 3 position: 152

# 查看log
kafka-run-class.sh kafka.tools.DumpLogSegments --files ./00000000000000000000.log
# 输出
Dumping datas/first-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 1 count: 2 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1636338440962 size: 75 magic: 2 compresscodec: none crc: 2745337109 isvalid: true
baseOffset: 2 lastOffset: 2 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 75 CreateTime: 1636351749089 size: 77 magic: 2 compresscodec: none crc: 273943004 isvalid: true
baseOffset: 3 lastOffset: 3 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 152 CreateTime: 1636351749119 size: 77 magic: 2 compresscodec: none crc: 106207379 isvalid: true
baseOffset: 4 lastOffset: 8 count: 5 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 229 CreateTime: 1636353061435 size: 141 magic: 2 compresscodec: none crc: 157376877 isvalid: true
baseOffset: 9 lastOffset: 13 count: 5 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 370 CreateTime: 1636353204051 size: 146 magic: 2 compresscodec: none crc: 4058582827 isvalid: true
```

### index 和 log 详解
![Broker+20221001025440](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20221001025440.png%2B2022-10-01-02-54-42)

### 参数配置
日志存储参数配置如下：
|参数|描述|
|---|---|
|log.segment.bytes|Kafka 中 log 日志是分成一块块存储的，此配置是指 log 日志划分成块的大小，默认值 1G。|
|log.index.interval.bytes|默认 4kb，kafka 里面每当写入了 4kb 大小的日志(.log)，然后就往 index 文件里面记录一个索引。稀疏索引。|

### 清理策略
Kafka 中默认的日志保存时间为 7 天，超过该时间，则为过期日志，执行相应的清理策略。可以通过调整如下参数修改保存时间：
* log.retention.hours，最低优先级单位，为小时，默认7天。
* log.retention.minutes，单位为分钟。
* log.retention.ms，最高优先级，单位为毫秒。
* log.retention.check.interval.ms，负责设置检查周期，默认5分钟。

Kafka 中提供的日志清理策略有 delete 和 compact 两种：
* log.cleanup.policy = delete 所有数据启用删除策略
* log.cleanup.policy = compact 所有数据启用压缩策略

#### delete
日志删除，将过期数据删除。
* 基于时间: 默认打开。以 segment 中所有记录中的最大时间戳作为该文件时间戳。
* 基于大小: 默认关闭。超过设置的所有日志总大小，删除最早的 segment。log.retention.bytes，默认等于-1，表示无穷大。

#### compact
日志压缩：对于相同key的不同value值，只保留最后一个版本。
![Broker+20221001030750](https://raw.githubusercontent.com/loli0con/picgo/master/images/Broker%2B20221001030750.png%2B2022-10-01-03-07-51)

压缩后的offset可能是不连续的，比如上图中没有6，当从这些offset消费消息时，将会拿到比这个offset大的offset对应的消息，实际上会拿到offset为7的消息，并从这个位置开始消费。


## 高效原因
1. Kafka 本身是分布式集群，可以采用分区技术，并行度高
2. 读数据采用稀疏索引，可以快速定位要消费的数据
3. 顺序写磁盘: Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端，为顺序写。顺序写之所以快，是因为其省去了大量磁头寻址的时间。
4. 零拷贝 + 页缓存
   1. 零拷贝: Kafka的数据加工处理操作交由Kafka生产者和Kafka消费者处理。Kafka Broker应用层不关心存储的数据，所以就不用走应用层(数据在系统空间/内核，不需要到用户空间)，传输效率高。
   2. Kafka重度依赖底层操作系统提供的PageCache功能。当上层有写操作时，操作系统只是将数据写入 PageCache。当读操作发生时，先从PageCache中查找，如果找不到，再去磁盘中读取。实际上PageCache是把尽可能多的空闲内存都当做了磁盘缓存来使用。

和PageCache相关的两个参数（用于控制数据刷盘）：
|参数|描述|
|---|---|
|log.flush.interval.messages|强制页缓存刷写到磁盘的条数，默认是 long 的最大值，9223372036854775807。一般不建议修改，交给系统自己管理。
|log.flush.interval.ms|每隔多久，刷数据到磁盘，默认是 null。一般不建议修改，交给系统自己管理。|
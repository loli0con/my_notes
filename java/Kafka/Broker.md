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

### Kafka服务运行设置
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

#kafka 运行日志(数据)存放的路径，路径不需要提前创建，kafka 自动帮你创建，可以 配置多个磁盘路径，路径与路径之间可以用"，"分隔
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








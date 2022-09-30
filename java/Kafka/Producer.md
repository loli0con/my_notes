# 生产者

## 发送原理
在消息发送的过程中，涉及到了两个线程，main 线程和 sender 线程。在 main 线程 中创建了一个双端队列 RecordAccumulator。main 线程将消息发送给 RecordAccumulator，sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka Broker。

![Kafka+20220920151530](https://raw.githubusercontent.com/loli0con/picgo/master/images/Kafka%2B20220920151530.png%2B2022-09-20-15-15-33)

NetworkClient 负责将消息发给Kafka Broker，NetworkClient在内部维护了一个 InFlightRequests 队列，对应一个 broker，队头的5个元素可以同时发送。




## 重要参数
|参数名称|作用描述|
|---|---|
|bootstrap.servers|生产者连接集群所需的 broker 地址清单。例如 hadoop102:9092,hadoop103:9092,hadoop104:9092，可以设置 1 个或者多个，中间用逗号隔开。注意这里并非需要所有的 broker 地址，因为生产者从给定的 broker 里查找到其他 broker 信息。|
|key.serializer 和 value.serializer|指定发送消息的 key 和 value 的序列化器的类型。一定要写全类名。|
|buffer.memory|RecordAccumulator 缓冲区总大小，默认 32m。|
|batch.size|缓冲区一批数据最大值，默认 16k。适当增加该值，可以提高吞吐量，但是如果该值设置太大，会导致数据传输延迟增加。|
|linger.ms|如果数据迟迟未达到 batch.size，sender 等待 linger.time 之后就会发送数据。单位 ms，默认值是 0ms，表示没有延迟。生产环境建议该值大小为 5-100ms 之间。|
|acks|0:生产者发送过来的数据，不需要等数据落盘应答。1:生产者发送过来的数据，Leader 收到数据后应答。-1(all):生产者发送过来的数据，Leader 和 isr 队列里面的所有节点收齐数据后应答。默认值是-1，-1 和 all 是等价的。|
|max.in.flight.requests.per.connection|允许最多没有返回 ack 的次数，默认为 5，开启幂等性要保证该值是 1-5 的数字。|
|retries|当消息发送出现错误的时候，系统会重发消息。retries 表示重试次数。默认是 int 最大值，2147483647。如果设置了重试，还想保证消息的有序性，需要设置 MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION=1 否则在重试此失败消息的时候，其他的消息可能发送成功了。|
|retry.backoff.ms|两次重试之间的时间间隔，默认是 100ms。|
|enable.idempotence|是否开启幂等性，默认 true，开启幂等性。|
|compression.type|生产者发送的所有数据的压缩方式。默认是 none，也就是不压缩。支持压缩类型:none、gzip、snappy、lz4 和 zstd。|

## demo

### 导入依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.kafka</groupId> 
        <artifactId>kafka-clients</artifactId>
        <version>3.0.0</version>
     </dependency>
</dependencies>
```

### 发送消息
```JAVA
package com.atguigu.kafka.producer;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

import java.util.Properties;

public class CustomProducer {
    public static void main(String[] args) throws InterruptedException {
        // 1. 创建 kafka 生产者的配置对象
        Properties properties = new Properties();

        // 2. 给 kafka 配置对象添加配置信息:bootstrap.servers
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "hadoop102:9092");
        // key,value 序列化(必须):key.serializer，value.serializer
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer");

        // 3. 创建 kafka 生产者对象
        KafkaProducer<String, String> kafkaProducer = new KafkaProducer<String, String>(properties);

        // 4. 调用 send 方法,发送消息 
        for (int i = 0; i < 5; i++) {

            // 4.1 异步发送，默认
            kafkaProducer.send(new ProducerRecord<>("topic1","value " + i));
            
            // 4.2 异步发送，添加回调
            kafkaProducer.send(new ProducerRecord<>("topic1", "value " + i), new Callback() {
                /** 该方法在Producer收到ack时调用，为异步调用
                 *  该方法有两个参数，分别是元数据信息(RecordMetadata)和异常信息(Exception)
                 *  如果 Exception 为 null，说明消息发送成功，如果 Exception 不为 null，说明消息发送失败。 
                 *  注意: 消息发送失败会自动重试，不需要我们在回调函数中手动重试。
                 */
                @Override
                public void onCompletion(RecordMetadata metadata, Exception exception) {
                    if (exception == null) {
                        // 没有异常,输出信息到控制台
                        System.out.println("主题:" + metadata.topic() + "->" + "分区:" + metadata.partition());
                    } else {
                        // 出现异常打印
                        exception.printStackTrace();
                    } 
                }
            });

            // 4.3 同步发送
            kafkaProducer.send(new ProducerRecord<>("topic1","value" + i)).get();

            // 延迟一会会看到数据发往不同分区
            Thread.sleep(2);
        }

        // 5. 关闭资源
        kafkaProducer.close();
    }
}
```


## 分区

### 好处
1. 便于合理使用存储资源，每个Partition在一个Broker上存储，可以把海量的数据按照分区切割成一块一块数据存储在多台Broker上。合理控制分区的任务，可以实现负载均衡的效果。
2. 提高并行度，生产者可以以分区为单位发送数据；消费者可以以分区为单位进行消费数据。

### 策略

#### 默认分区器DefaultPartitioner
ProducerRecord类，在类中重载了多个构造方法：
1. 指明partition的情况下，直接将指明的值作为partition值
2. 没有指明partition值但有key的情况下，将key的hash值与topic的partition数进行取余得到partition值
3. 既没有partition值又没有key值的情况下，Kafka采用Sticky Partition(黏性分区器)，会随机选择一个分区，并尽可能一直使用该分区，待该分区的batch已满或者已完成，Kafka再随机一个分区进行使用(和上一次的分区不同)

### 自定义分区器

#### 定义
```JAVA
package com.atguigu.kafka.producer;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

import java.util.Map;
/**
 * 1. 实现接口Partitioner
 * 2. 实现 3 个方法:partition,close,configure
 * 3. 编写 partition 方法,返回分区号
 */
public class MyPartitioner implements Partitioner {
    /**
     * 返回信息对应的分区
     * @param topic 主题
     * @param key 消息的 key
     * @param keyBytes 消息的 key 序列化后的字节数组
     * @param value 消息的 value
     * @param valueBytes 消息的 value 序列化后的字节数组
     * @param cluster 集群元数据可以查看分区信息
     * @return
     */
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        // 获取消息
        String msgValue = value.toString();
        // 创建partition
        int partition;
        // 判断消息是否包含atguigu
        if (msgValue.contains("atguigu")){
            partition = 0;
        }else {
            partition = 1;
        }
        // 返回分区号
        return partition;
    }
    
    // 关闭资源
    @Override
    public void close() {
    }

    // 配置方法
    @Override
    public void configure(Map<String, ?> configs) {
    }
}
```

#### 使用
```JAVA
/**
 * 在创建生产者时配置分区器，省略了其他代码
 */
Properties properties = new Properties();
// 添加自定义分区器
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG,"com.atgui gu.kafka.producer.MyPartitioner");
KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
```


## 提高吞吐量
延长批次时间，增大批次数据量：
* linger.ms: 等待时间，修改为5-100ms
* batch.size: 批次大小，默认16k
* compression.type: 压缩snappy
* RecordAccumulator: 缓冲区大小，修改为64m

注意：缺点是消息发送时间变长，降低速度的传输速率


## 数据的可靠性

### 应答级别
ACK应答级别：
* 0: 生产者发送过来的数据，不需要等数据落盘应答。可靠性差，效率高。
* 1: 生产者发送过来的数据，Leader收到数据后应答。可靠性中等，效率中等。
* -1(all): 生产者发送过来的数据，Leader和ISR队列里面的所有节点收齐数据后应答。可靠性高，效率低。

### ISR
Leader维护了一个动态的in-sync replica set(ISR)，意为和Leader保持同步的Follower+Leader集合。

如果Follower长时间未向Leader发送通信请求或同步数据，则该Follower将被踢出ISR。该时间阈值由replica.lag.time.max.ms参数设定，默认30s。这样就不用等长期联系不上或者已经故障的节点。

如果分区副本设置为1个，或者ISR里应答的最小副本数量(min.insync.replicas默认为1)设置为1，和ack=1的效果是一样的，仍然有丢数的风险。（因为只有leader自己了）

### 总结
数据完全可靠条件 = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2


## 数据去重

### 数据传递语义
* 至少一次(At Least Once) = ACK级别设置为-1 + 分区副本大于等于2 + ISR里应答的最小副本数量大于等于2。可以保证数据不丢失，但是不能保证数据不重复。
* 最多一次(At Most Once) = ACK级别设置为0。可以保证数据不重复，但是不能保证数据不丢失。
* 精确一次(Exactly Once):对于一些非常重要的信息，比如和钱相关的数据，要求数据既不能重复也不丢失。

### 幂等性
幂等性就是指Producer不论向Broker发送多少次重复数据，Broker端都只会持久化一条，保证了不重复。

精确一次(Exactly Once) = 幂等性 + 至少一次(ack=-1 + 分区副本数>=2 + ISR最小副本数量>=2)。

重复数据的判断标准：具有`<PID, Partition, SeqNumber>`相同主键的消息提交时，Broker只会持久化一条。
* 其中 PID 是 Producer 每次重启都会分配一个新的
* Partition 表示分区号
* Sequence Number 是单调自增的

所以幂等性只能保证的是在单分区(Partition)单会话(PID)内不重复(Sequence Number)。

幂等性的开启参数是 enable.idempotence，默认为 true。

#### 事务
![Kafka+20220920160408](https://raw.githubusercontent.com/loli0con/picgo/master/images/Kafka%2B20220920160408.png%2B2022-09-20-16-04-10)

Kafka 的事务一共有如下 5 个 API：
1. 初始化事务：void initTransactions();
2. 开启事务：void beginTransaction() throws ProducerFencedException;
3. 在事务内提交已经消费的偏移量(主要用于消费者)：void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets, String consumerGroupId) throws ProducerFencedException;
4. 提交事务：void commitTransaction() throws ProducerFencedException;
5. 放弃事务(类似于回滚事务的操作)：void abortTransaction() throws ProducerFencedException;



## 数据有序
kafka在1.x及以后版本保证数据单分区有序，条件如下：
1. 未开启幂等性：max.in.flight.requests.per.connection需要设置为1。
2. 开启幂等性：max.in.flight.requests.per.connection需要设置小于等于5。

在kafka1.x以后，启用幂等后，kafka broker 会缓存producer发来的最近5个request的元数据，可以保证最近5个request的数据都是有序的。
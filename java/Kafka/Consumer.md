# 消费者

## 消费方式

### 拉
pull(拉)模式: consumer从broker中主动拉取数据。pull模式不足之处是，如果Kafka没有数据，消费者可能会陷入循环中，一直返回空数据。

Kafka采用这种方式。

### 推
push(推)模式: broker主动把数据推给consumer。

Kafka没有采用这种方式，因为由broker决定消息发送速率，很难适应所有消费者的消费速率，Consumer可能会来不及处理消息。

## 消费流程
### 大致流程
![Consumer+20221001032555](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221001032555.png%2B2022-10-01-03-25-57)


### 消费者组
Consumer Group(CG): 消费者组，由多个consumer组成。形成一个消费者组的条件，是所有消费者的groupid相同。
* 消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费。
* 消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
* 如果向消费组中添加更多的消费者，超过主题分区数量，则有一部分消费者就会闲置，不会接收任何消息。

#### 消费者组初始化流程
![Consumer+20221001041556](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221001041556.png%2B2022-10-01-04-15-58)

coordinator: 辅助实现消费者组的初始化和分区的分配。

coordinator节点选择 = groupid的hashcode值 % 50( __consumer_offsets的分区数量)

例如: groupid的hashcode值 = 1，1% 50 = 1，那么__consumer_offsets 主题的1号分区，在哪个broker上，就选择这个节点的coordinator作为这个消费者组的老大。消费者组下的所有的消费者提交offset的时候就往这个分区去提交offset。

### 详细流程
![Consumer+20221001042113](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221001042113.png%2B2022-10-01-04-21-15)
1. Consumer通过sendFetches发送消费请求
2. ConsumerNetworkClient根据Fetch.min.bytes、Fetch.max.wait.ms、Fetch.max.bytes参数的规则从broker中获取数据，并将获取到的数据存放在 completedFetches队列中
3. Consumer通过FetchedRecords从队列中抓取数据，Max.poll.records参数控制一次拉取数据返回消息的最大条数，默认500条


## 重要参数
|参数名称|参数描述|
|---|---|
|bootstrap.servers|向 Kafka 集群建立初始连接用到的 host/port 列表。|
|key.deserializer和value.deserializer|指定接收消息的 key 和 value 的反序列化类型。一定要写全类名。|
|group.id|标记消费者所属的消费者组。|
|enable.auto.commit|默认值为 true，消费者会自动周期性地向服务器提交偏移量。|
|auto.commit.interval.ms|如果设置了 enable.auto.commit 的值为 true， 则该值定义了消费者偏移量向 Kafka 提交的频率，默认 5s。|
|auto.offset.reset|当 Kafka 中没有初始偏移量或当前偏移量在服务器中不存在时的处理逻辑。earliest:自动重置偏移量到最早的偏移量。latest:默认，自动重置偏移量为最新的偏移量。none:如果消费组原来的(previous)偏移量不存在，则向消费者抛异常。anything:向消费者抛异常。|
|offsets.topic.num.partitions|__consumer_offsets 的分区数，默认是 50 个分区。|
|heartbeat.interval.ms|Kafka 消费者和 coordinator 之间的心跳时间，默认 3s。该条目的值必须小于 session.timeout.ms，也不应该高于 session.timeout.ms 的 1/3。|
|session.timeout.ms|Kafka 消费者和 coordinator 之间连接超时时间，默认 45s。超过该值，该消费者被移除，消费者组执行再平衡。|
|max.poll.interval.ms|消费者处理消息的最大时长，默认是 5 分钟。超过该值，该消费者被移除，消费者组执行再平衡。|
|fetch.min.bytes|默认 1 个字节。消费者获取服务器端一批消息最小的字节数。|
|fetch.max.wait.ms|默认 500ms。如果没有从服务器端获取到一批数据的最小字节数。该时间到，仍然会返回数据。|
|fetch.max.bytes|默认Default: 52428800(50m)。消费者获取服务器端一批消息最大的字节数。如果服务器端一批次的数据大于该值(50m)仍然可以拉取回来这批数据，因此，这不是一个绝对最大值。一批次的大小受 message.max.bytes(broker config) or max.message.bytes(topic config)影响。|
|max.poll.records|一次 poll 拉取数据返回消息的最大条数，默认是 500 条。|



## 消费者 API
在消费者 API 代码中必须配置消费者组 id。
```Java
// 1.创建消费者的配置对象
Properties properties = new Properties();

// 2.给消费者配置对象添加参数
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "host1:9092");
// 配置序列化 必须
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
// 配置消费者组(组名任意起名) 必须
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

// 是否自动提交offset
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
// 提交offset的时间周期1000ms，默认5s
properties.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000);

// 3.创建消费者对象
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<String, String>(properties);

// 3.1 注册要消费的主题(可以消费多个主题)
ArrayList<String> topics = new ArrayList<>();
topics.add("first");
kafkaConsumer.subscribe(topics);

// 3.2 消费某个主题的某个分区数据
ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
topicPartitions.add(new TopicPartition("first", 0));
kafkaConsumer.assign(topicPartitions);

// 指定offset消费
Set<TopicPartition> assignment = new HashSet<>();
while (assignment.size() == 0) {
    kafkaConsumer.poll(Duration.ofSeconds(1));
    // 获取消费者分区分配信息(有了分区分配信息才能开始消费)
    assignment = kafkaConsumer.assignment();
}
// 遍历所有分区，并指定 offset 从 1700 的位置开始消费
for (TopicPartition tp: assignment) {
    kafkaConsumer.seek(tp, 1700);
}

// 4.拉取数据打印
while (true) {
    // 设置 1s 中消费一批数据
    ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
    // 打印消费到的数据 
    for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
        System.out.println(consumerRecord);
    }

    // 同步提交offset
    consumer.commitSync();
    // 异步提交offset
    consumer.commitAsync();
}
```


## 分区的分配以及再平衡
一个consumer group中有多个consumer组成，一个topic有多个partition组成，通过配置Kafka的分区分配策略来控制到底哪个consumer消费哪个partition的数据。

Kafka有四种主流的分区分配策略: Range、RoundRobin、Sticky、CooperativeSticky。可以通过配置参数partition.assignment.strategy，修改分区的分配策略。默认策略是Range+CooperativeSticky。Kafka可以同时使用多个分区分配策略。

![Consumer+20221004164227](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221004164227.png%2B2022-10-04-16-42-30)


### Range 以及再平衡
#### 分配
0. Range 是对每个 topic 而言的。
1. 首先对同一个 topic 里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。
   * 假如现在有 7 个分区，3 个消费者，排序后的分区将会是0,1,2,3,4,5,6;消费者排序完之后将会是C0,C1,C2。
2. 通过 partitions数/consumer数 来决定每个消费者应该消费几个分区。如果除不尽，那么前面几个消费者将会多消费 1 个分区。
   * 例如，7/3 = 2 余 1 ，除不尽，那么 消费者 C0 便会多消费 1 个分区。 8/3=2余2，除不尽，那么C0和C1分别多 消费一个。

![Consumer+20221004164951](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221004164951.png%2B2022-10-04-16-49-52)

注意:如果只是针对 1 个 topic 而言，C0消费者多消费 1 个分区影响不是很大。但是如果有 N 多个 topic，那么针对每个 topic，消费者C0都将多消费 1 个分区，topic越多，C0消费的分区会比其他消费者明显多消费 N 个分区。**容易产生数据倾斜!**

#### 再平衡
1. 停止掉 0 号消费者，快速重新发送消息观看结果(45s 以内，越快越好)。
   1. 1 号消费者: 消费到 3、4 号分区数据。
   2. 2 号消费者: 消费到 5、6 号分区数据。
   3. 0 号消费者的任务会整体被分配到 1 号消费者或者 2 号消费者。
   4. 0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。
2. 再次重新发送消息观看结果(45s 以后)。
   1. 1 号消费者: 消费到 0、1、2、3 号分区数据。
   2. 2 号消费者: 消费到 4、5、6 号分区数据。
   3. 消费者 0 已经被踢出消费者组，所以重新按照 range 方式分配。


### RoundRobin 以及再平衡

#### 分配
RoundRobin 针对集群中所有Topic而言。

RoundRobin 轮询分区策略，是把所有的 partition 和所有的 consumer 都列出来，然后按照 hashcode 进行排序，最后通过轮询算法来分配 partition 给到各个消费者。

![Consumer+20221005144449](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005144449.png%2B2022-10-05-14-44-51)

#### 再平衡
1. 停止掉 0 号消费者，快速重新发送消息观看结果(45s 以内，越快越好)。
   1. 1 号消费者:消费到 2、5 号分区数据
   2. 2 号消费者:消费到 4、1 号分区数据
   3. 0 号消费者的任务会按照 RoundRobin 的方式，把数据轮询分成 0 、6 和 3 号分区数据，分别由 1 号消费者或者 2 号消费者消费。
   4. 0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。
2. 再次重新发送消息观看结果(45s 以后)。
   1. 1 号消费者:消费到 0、2、4、6 号分区数据
   2. 2 号消费者:消费到 1、3、5 号分区数据
   3. 消费者 0 已经被踢出消费者组，所以重新按照 RoundRobin 方式分配。


### Sticky 以及再平衡
粘性分区定义: 可以理解为分配的结果带有“粘性的”。即在执行一次新的分配之前，考虑上一次分配的结果，尽量少的调整分配的变动，可以节省大量的开销。

#### 分配
粘性分区是 Kafka 从 0.11.x 版本开始引入这种分配策略，首先会尽量均衡的放置分区到消费者上面，在出现同一消费者组内消费者出现问题的时候，会尽量保持原有分配的分区不变化。

#### 再平衡
1. 停止掉 0 号消费者，快速重新发送消息观看结果(45s 以内，越快越好)。
   1. 1 号消费者:消费到 2、5、3 号分区数据。
   2. 2 号消费者:消费到 4、6 号分区数据。
   3. 0 号消费者的任务会按照粘性规则，尽可能均衡的随机分成 0 和 1 号分区数据，分别由 1 号消费者或者 2 号消费者消费。
   4. 0 号消费者挂掉后，消费者组需要按照超时时间 45s 来判断它是否退出，所以需要等待，时间到了 45s 后，判断它真的退出就会把任务分配给其他 broker 执行。
2. 再次重新发送消息观看结果(45s 以后)。
   1. 1 号消费者:消费到 2、3、5 号分区数据。
   2. 2 号消费者:消费到 0、1、4、6 号分区数据。
   3. 消费者 0 已经被踢出消费者组，所以重新按照粘性方式分配。



## offset位移

### offset 的默认维护位置
![Consumer+20221005145651](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005145651.png%2B2022-10-05-14-56-55)

__consumer_offsets 主题里面采用 key 和 value 的方式存储数据。key 是 **group.id+topic+分区号**，value 就是当前 offset 的值。每隔一段时间，kafka 内部会对这个 topic 进行 compact，也就是每个 **group.id+topic+分区号** 就保留最新数据。

### 自动提交 offset
为了使开发者专注于自身的业务逻辑，Kafka提供了自动提交offset的功能。自动提交offset的相关参数:
* enable.auto.commit: 是否开启自动提交offset功能，默认是true
* auto.commit.interval.ms: 自动提交offset的时间间隔，默认是5s

![Consumer+20221005150023](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005150023.png%2B2022-10-05-15-00-24)


### 手动提交 offset
虽然自动提交offset十分简单便利，但由于其是基于时间提交的，开发人员难以把握offset提交的时机。因此Kafka还提供了手动提交offset的API。

手动提交offset的方法有两种: 分别是commitSync(同步提交)和commitAsync(异步提交)：
* commitSync(同步提交): 必须等待offset提交完毕，再去消费下一批数据。
* commitAsync(异步提交): 发送完提交offset请求后，就开始消费下一批数据了。

两者的相同点是，都会将本次提交的一批数据最高的偏移量提交;不同点是，同步提交阻塞当前线程，一直到提交功，并且会自动失败重试(由不可控因素导致，也会出现提交失败);而异步提交则没有失败重试机制，故有可能提交失败。

![Consumer+20221005150301](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005150301.png%2B2022-10-05-15-03-02)


### 指定 Offset 消费
auto.offset.reset配置决定了当 Kafka 中没有初始偏移量(消费者组第一次消费)或服务器上不再存在当前偏移量 时(例如该数据已被删除)该怎么办：
1. earliest: 自动将偏移量重置为最早的偏移量，--from-beginning。
2. latest(默认值): 自动将偏移量重置为最新偏移量。
3. none: 如果未找到消费者组的先前偏移量，则向消费者抛出异常。

![Consumer+20221005150852](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005150852.png%2B2022-10-05-15-08-53)


### 指定时间消费
在生产环境中，会遇到最近消费的几个小时数据异常，想重新按照时间消费。
```Java
Set<TopicPartition> assignment = new HashSet<>();
while (assignment.size() == 0) {
    kafkaConsumer.poll(Duration.ofSeconds(1));
    // 获取消费者分区分配信息(有了分区分配信息才能开始消费)
    assignment = kafkaConsumer.assignment();
}
HashMap<TopicPartition, Long> timestampToSearch = new HashMap<>();
// 封装集合存储，每个分区对应一天前的数据
for (TopicPartition topicPartition : assignment) {
    timestampToSearch.put(topicPartition, System.currentTimeMillis() - 1 * 24 * 3600 * 1000);
}
// 获取从1天前开始消费的每个分区的offset
Map<TopicPartition, OffsetAndTimestamp> offsets = kafkaConsumer.offsetsForTimes(timestampToSearch);
// 遍历每个分区，对每个分区设置消费时间。
for (TopicPartition topicPartition : assignment) {
    OffsetAndTimestamp offsetAndTimestamp = offsets.get(topicPartition);
    // 根据时间指定开始消费的位置
    if (offsetAndTimestamp != null) {
        kafkaConsumer.seek(topicPartition, offsetAndTimestamp.offset());
    }
}
```


### 重复消费和漏消费
* 重复消费: 已经消费了数据，但是 offset 没提交。
* 漏消费: 先提交 offset 后消费，有可能会造成数据的漏消费。

![Consumer+20221005171349](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005171349.png%2B2022-10-05-17-13-53)

### 消费者事务
如果想完成Consumer端的精准一次性消费，那么需要Kafka消费端将消费过程和提交offset过程做原子绑定。此时需要将Kafka的offset保存到支持事务的自定义介质(比如 MySQL)。

![Consumer+20221005171558](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005171558.png%2B2022-10-05-17-16-00)



### 数据积压
如果是Kafka消费能力不足，则可以考虑增加Topic的分区数，并且同时提升消费组的消费者数量，消费者数 = 分区数。(两者缺一不可)

![Consumer+20221005171722](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005171722.png%2B2022-10-05-17-17-23)

如果是下游的数据处理不及时，则需要提高每批次拉取的数量。批次拉取数据过少(拉取数据/处理时间 < 生产速度)， 使处理的数据小于生产的数据，也会造成数据积压。

![Consumer+20221005171806](https://raw.githubusercontent.com/loli0con/picgo/master/images/Consumer%2B20221005171806.png%2B2022-10-05-17-18-07)
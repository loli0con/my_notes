# MapReduce

## 定义
MapReduce 是一个分布式运算程序的**编程框架**，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。

MapReduce 核心功能是**将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序**，并发运行在一个 Hadoop 集群上。


## 优点
1. MapReduce 易于编程：它简单的实现一些接口，就可以完成一个分布式程序，跟写一个简单的串行程序是一模一样的简单。
2. 良好的扩展性：当计算资源不足时，可以通过简单的增加机器来扩展。
3. 高容错性：当其中一台运行机器挂了，上面的计算任务会被转移到另外一个节点上运行，这个过程不需要人工参与，由 Hadoop 内部完成。
4. 适合 PB 级以上海量数据的离线处理：可以实现上千台服务器集群并发工作。

## 缺点
1. 不擅长实时计算：无法在毫秒或者秒级内返回结果。
2. 不擅长流式计算：MapReduce 的输入数据集是静态的，而流式计算的输入数据是动态的。
3. 不擅长 DAG(有向无环图) 计算：在多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出的情况下，每个 MapReduce 作业(应用程序)的输出结果都会写入到磁盘， 会造成大量的磁盘 IO，导致性能非常的低下。


## 核心思想
![MapReduce+20221111153940](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221111153940.png%2B2022-11-11-15-39-42)

1. 分布式的运算程序往往需要分成至少 2 个阶段。
2. 第一个阶段的 MapTask 并发实例，完全并行运行，互不相干。
3. 第二个阶段的 ReduceTask 并发实例互不相干，但是他们的数据依赖于上一个阶段的所有 MapTask 并发实例的输出。
4. MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业
务逻辑非常复杂，那就只能多个 MapReduce 程序，串行运行。

## MR进程
一个完整的 MapReduce 程序在分布式运行时有三类实例进程:
1. MrAppMaster: 负责整个程序的过程调度及状态协调。
2. MapTask: 负责 Map 阶段的整个数据处理流程。
3. ReduceTask: 负责 Reduce 阶段的整个数据处理流程。


## 编程规范
用户编写的程序分成三个部分: Mapper、Reducer 和 Driver。
1. Mapper阶段
   1. 用户自定义的Mapper要继承自己的父类
   2. Mapper的输入数据是KV对的形式(KV的类型可自定义)
   3. Mapper中的业务逻辑写在map()方法中
   4. Mapper的输出数据是KV对的形式(KV的类型可自定义)
   5. map()方法(MapTask进程)对每一个<K,V>调用一次
2. Reducer阶段
   1. 用户自定义的Reducer要继承自己的父类
   2. Reducer的输入数据类型对应Mapper的输出数据类型，也是KV
   3. Reducer的业务逻辑写在reduce()方法中
   4. ReduceTask进程对每一组相同k的<k,v>组调用一次reduce()方法
3. Driver阶段：相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是封装了MapReduce程序相关运行参数的job对象。
   1. 获取配置信息，获取job对象实例
   2. 指定本程序的jar包所在的本地路径
   3. 关联Mapper/Reducer业务类
   4. 指定Mapper输出数据的kv类型
   5. 指定最终输出的数据的kv类型
   6. 指定job的输入原始文件所在目录
   7. 指定job的输出结果所在目录
   8. 提交作业


## Hadoop序列化
Java 的序列化是一个重量级序列化框架(Serializable)，一个对象被序列化后，会附带很多额外的信息(各种校验信息，Header，继承体系等)，不便于在网络中高效传输。所以，Hadoop 自己开发了一套序列化机制(Writable)。

### 常用数据序列化类型
|Java 类型|Hadoop Writable 类型|
|---|---|
|Boolean|BooleanWritable|
|Byte|ByteWritable|
|Int|IntWritable|
|Float|FloatWritable|
|Long|LongWritable|
|Double|DoubleWritable|
|String|Text|
|Map|MapWritable|
|Array|ArrayWritable|
|Null|NullWritable|

### 自定义 bean 对象实现序列化接口(Writable)
在 Hadoop 框架内部传递一个 bean 对象，那么该对象就需要实现序列化接口。具体实现 bean 对象序列化步骤如下 7 步: 
1. 必须**实现 Writable 接口**
2. 反序列化时，需要反射调用空参构造函数，所以**必须有空参构造**
3. **重写序列化方法**
4. **重写反序列化方法**
5. 注意**反序列化的顺序和序列化的顺序完全一致**
6. 要想把结果显示在文件中，需要**重写 toString()**，可用"\t"分开，方便后续用
7. 如果需要将自定义的 bean 放在 key 中传输，则还需要**实现 Comparable 接口**，因为 MapReduce 框中的 Shuffle 过程要求对 key 必须能排序。

#### demo
```Java
import org.apache.hadoop.io.WritableComparable;
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

@Getter
@Setter
public class FlowBean implements WritableComparable<FlowBean> {
    private long upFlow; //上行流量
    private long downFlow; //下行流量
    private long sumFlow; //总流量

    //提供无参构造
    public FlowBean() { }

    //实现序列化和反序列化方法,注意顺序一定要一致
    @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(this.upFlow);
        out.writeLong(this.downFlow);
        out.writeLong(this.sumFlow);
    }
    @Override
    public void readFields(DataInput in) throws IOException {
        this.upFlow = in.readLong();
        this.downFlow = in.readLong();
        this.sumFlow = in.readLong();
    }

    //重写 ToString,最后要输出 FlowBean
    @Override
    public String toString() {
        return upFlow + "\t" + downFlow + "\t" + sumFlow;
    }

    @Override
    public int compareTo(FlowBean o) {
        //按照总流量比较,倒序排列
        if(this.sumFlow > o.sumFlow){
            return -1;
        }else if(this.sumFlow < o.sumFlow){
            return 1;
        }else {
            return 0;
        }
    }
}
```



## 框架原理
![MapReduce+20221116150638](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116150638.png%2B2022-11-16-15-06-39)

### MapTask
![MapReduce+20221116150821](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116150821.png%2B2022-11-16-15-08-24)

1. Read 阶段: MapTask 通过 InputFormat 获得的 RecordReader，从输入 InputSplit(记录了切片的元数据信息,YARN上的MrAppMaster就可以根据切片规划文件计算开启MapTask个数) 中解析出一个个 key/value。
2. Map 阶段: 该节点主要是将解析出的 key/value 交给用户编写 map()函数处理，并 产生一系列新的 key/value。
3. Collect 收集阶段: 在用户编写 map()函数中，当数据处理完成后，一般会调用 OutputCollector.collect()输出结果。在该函数内部，它会将生成的 key/value 分区(调用 Partitioner分区器)，并写入一个环形内存缓冲区中。
4. Spill 阶段: 即“溢写”，当环形缓冲区满后，MapReduce 会将数据写到本地磁盘上， 生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。
   1. 利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号 Partition 进行排序，然后按照 key 进行排序。这样，经过排序后，数据以分区为单位聚集在一起，且同一分区内所有数据按照 key 有序。
   2. 按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文件 output/spillN.out(N 表示当前溢写次数)中。如果用户设置了 Combiner，则写入文件之前，对每个分区中的数据进行一次聚集操作。
   3. 将分区数据的元信息写到内存索引数据结构 SpillRecord 中，其中每个分区的元信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大小超过 1MB，则将内存索引写到文件 output/spillN.out.index 中。
5. Merge 阶段: 当所有数据处理完成后，MapTask 会将所有临时文件合并成一个大文件(即最终只会生成一个数据文件)，并保存到文件 output/file.out 中，同时生成相应的索引文件 output/file.out.index。
   * 在进行文件合并过程中，MapTask 以分区为单位进行合并。对于某个分区，它将采用多轮递归合并的方式。每轮合并 mapreduce.task.io.sort.factor(默认 10)个文件，并将产生的文件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。
   * 让每个 MapTask 最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量小文件产生的随机读取带来的开销。

### ReduceTask
![MapReduce+20221116152007](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116152007.png%2B2022-11-16-15-20-10)

1. Copy 阶段: ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
2. Sort 阶段:在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。
3. Reduce 阶段: reduce()函数将计算结果写到 HDFS 上。

### Shuffle
在 MapTask 和 ReduceTask 之间，Shuffle 过程从第 7(包含) 步开始到第 16(不包含) 步结束：
1. MapTask 收集 map()方法输出的 kv 对，放到内存缓冲区中
2. 从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件
3. 多个溢出文件会被合并成大的溢出文件
4. 在溢出过程及合并的过程中，都要调用 Partitioner 进行分区和针对 key 进行排序
5. ReduceTask 根据自己的分区号，去各个 MapTask 机器上取相应的结果分区数据
6. ReduceTask 会抓取到同一个分区的来自不同 MapTask 的结果文件，ReduceTask 会将这些文件再进行合并(归并排序)
7. 合并成大文件后，Shuffle 的过程也就结束了，后面进入 ReduceTask 的逻辑运算过
程(从文件中取出一个一个的键值对 Group，调用用户自定义的 reduce()方法)

![MapReduce+20221116175359](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116175359.png%2B2022-11-16-17-54-02)

#### 注意
1. Shuffle 中的缓冲区大小会影响到 MapReduce 程序的执行效率，原则上说，缓冲区越大，磁盘 io 的次数越少，执行速度就越快。
2. 缓冲区的大小可以通过参数调整，参数: mapreduce.task.io.sort.mb 默认 100M。






## 应用

### Join应用

#### Reduce Join
1. Map 端的主要工作: 为来自不同表或文件的 key/value 对，打标签以区别不同来源的记录。然后用连接字段作为 key，其余部分和新加的标志作为 value，最后进行输出。
2. Reduce 端的主要工作: 在 Reduce 端以连接字段作为 key 的分组已经完成，我们只需要在每一个分组当中将那些来源于不同文件的记录(在 Map 阶段已经打标志)分开，最后进行合并就 ok 了。

这种方式中，合并的操作是在 Reduce 阶段完成，Reduce 端的处理压力太大，Map
节点的运算负载则很低，资源利用率不高，且在 Reduce 阶段极易产生数据倾斜。

#### Map Join

##### 使用场景
Map Join 适用于一张表十分小、一张表很大的场景。

##### 优点
在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数
据的压力，尽可能的减少数据倾斜。

##### 具体实现
采用 DistributedCache:
1. 在 Mapper 的 setup 阶段，将文件读取到缓存集合中。
2. 在 Driver 驱动类中加载缓存。

```Java
//缓存普通文件到 Task 运行节点。
job.addCacheFile(new URI("file:///e:/cache/pd.txt"));
//如果是集群运行,需要设置 HDFS 路径
job.addCacheFile(new URI("hdfs://hadoop102:8020/cache/pd.txt"));
```


### 数据清洗(ETL)
ETL，是英文 Extract-Transform-Load 的缩写，用来描述将数据从来源端经过抽取 (Extract)、转换(Transform)、加载(Load)至目的端的过程。

在运行核心业务 MapReduce 程序之前，往往要先对数据进行清洗，清理掉不符合用户要求的数据。清理的过程往往只需要运行 Mapper 程序，不需要运行 Reduce 程序。
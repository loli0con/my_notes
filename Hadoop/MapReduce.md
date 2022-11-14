# MapReduce

## 定义
MapReduce 是一个分布式运算程序的编程框架，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。

MapReduce 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的分布式运算程序，并发运行在一个 Hadoop 集群上。


## 优点
1. MapReduce 易于编程
2. 良好的扩展性
3. 高容错性
4. 适合 PB 级以上海量数据的离线处理

## 缺点
1. 不擅长实时计算
2. 不擅长流式计算
3. 不擅长 DAG(有向无环图) 计算


## 核心思想
![MapReduce+20221111153940](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221111153940.png%2B2022-11-11-15-39-42)

1. 分布式的运算程序往往需要分成至少 2 个阶段。
2. 第一个阶段的 MapTask 并发实例，完全并行运行，互不相干。
3. 第二个阶段的 ReduceTask 并发实例互不相干，但是他们的数据依赖于上一个阶段的所有 MapTask 并发实例的输出。
4. MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业
务逻辑非常复杂，那就只能多个 MapReduce 程序，串行运行。

## 进程
一个完整的 MapReduce 程序在分布式运行时有三类实例进程:
1. MrAppMaster: 负责整个程序的过程调度及状态协调。
2. MapTask: 负责 Map 阶段的整个数据处理流程。
3. ReduceTask: 负责 Reduce 阶段的整个数据处理流程。


## 编程规范
用户编写的程序分成三个部分:Mapper、Reducer 和 Driver。

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
1. 必须实现 Writable 接口
2. 反序列化时，需要反射调用空参构造函数，所以必须有空参构造
3. 重写序列化方法
4. 重写反序列化方法
5. 注意反序列化的顺序和序列化的顺序完全一致
6. 要想把结果显示在文件中，需要重写 toString()，可用"\t"分开，方便后续用
7. 如果需要将自定义的 bean 放在 key 中传输，则还需要实现 Comparable 接口，因为MapReduce 框中的 Shuffle 过程要求对 key 必须能排序。

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
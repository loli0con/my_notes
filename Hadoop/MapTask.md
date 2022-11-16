# MapTask

![MapReduce+20221116150638](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116150638.png%2B2022-11-16-15-06-39)

## InputFormat 数据输入

### 切片 与 MapTask 并行度
数据切片是 MapReduce 程序计算输入数据的单位，一个切片会对应启动一个 MapTask，而MapTask 的并行度决定 Map 阶段的任务处理并发度，进而影响到整个 Job 的处理速度。

### 数据块和数据切片的区别
* 数据块: Block 是 HDFS 物理上把数据分成一块一块。数据块是 HDFS 存储数据单位。
* 数据切片: 数据切片是在逻辑上对MR程序的输入进行分片，并不会在磁盘上将其切分成片进行存储。

### 切片方案对比
![MapTask+20221111165237](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapTask%2B20221111165237.png%2B2022-11-11-16-52-38)

### Job提交过程
![MapTask+20221111170328](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapTask%2B20221111170328.png%2B2022-11-11-17-03-30)

### FileInputFormat
在运行 MapReduce 程序时，输入的文件格式包括：基于行的日志文件、二进制格式文件、数据库表等，针对不同的数据类型，MapReduce 使用 FileInputFormat(抽象类) 进行读取，常见的实现类包括：
1. TextInputFormat
2. KeyValueTextInputFormat
3. NLineInputFormat
4. CombineTextInputFormat

也可以直接实现 InputFormat 接口。

#### 切片机制
FileInputFormat切片机制：
1. 简单地按照文件的内容长度进行切片
2. 切片大小，默认等于Block大小
3. 切片时不考虑数据集整体，而是逐个针对每一个文件单独切片

#### 切片参数

##### 切片大小计算公式
`Math.max(minSize, Math.min(maxSize, blockSize));`:
1. mapreduce.input.fileinputformat.split.minsize=1
2. mapreduce.input.fileinputformat.split.maxsize=Long.MAXValue
3. 默认情况下，切片大小=blocksize

##### 切片大小设置
1. maxsize(切片最大值): 参数如果调得比blockSize小，则会让切片变小，而且就等于配置的这个参数的值。
2. minsize(切片最小值):参数调的比blockSize大，则可以让切片变得比blockSize还大。

##### 切片信息API
```Java
// 根据文件类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSplit();
// 获取切片的文件名称
String name = inputSplit.getPath().getName();
```

#### 切片过程
`fileInputFormat.getSplits(job)`过程详解:
1. 程序先找到你数据存储的目录
2. 开始遍历处理(规划切片)目录下的每一个文件，切片时不考虑数据集整体，而是逐个针对每一个文件单独切片
3. 遍历第一个文件ss.txt
   1. 获取文件大小fs.sizeOf(ss.txt)
   2. 计算切片大小:`computeSplitSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M`
   3. 默认情况下，切片大小=blocksize
   4. 开始切，形成第1个切片:ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M
   5. 每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片
   6. 将切片信息写到一个切片规划文件中
   7. 整个切片的核心过程在getSplit()方法中完成
   8. InputSplit只记录了切片的元数据信息，比如起始位置、长度以及所在的节点列表等
4. 提交切片规划文件到YARN上，YARN上的MrAppMaster就可以根据切片规划文件计算开启MapTask个数

#### TextInputFormat
TextInputFormat 是默认的 FileInputFormat 实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量，LongWritable 类型。值是这行的内容，不包括任何行终止符(换行符和回车符)，Text 类型。

#### CombineTextInputFormat
框架默认的 TextInputFormat 切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个 MapTask，这样如果有大量小文件，就会产生大量的 MapTask，处理效率极其低下。CombineTextInputFormat 用于小文件过多的场景，它可以将多个小文件从逻辑上规划到一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。

##### 虚拟存储切片最大值设置
```Java
CombineTextInputFormat.setMaxInputSplitSize(job, 4194304);// 4m
```
注意: 虚拟存储切片最大值设置最好根据实际的小文件大小情况来设置具体的值。

![MapTask+20221116173811](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapTask%2B20221116173811.png%2B2022-11-16-17-38-13)

##### 切片过程
生成切片过程包括: 虚拟存储过程和切片过程二部分，
1. 虚拟存储过程: 将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍，那么以最大值切割一块;当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时将文件均分成 2 个虚拟存储块(防止出现太小切片)。
2. 切片过程:
   1. 判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独形成一个切片。
   2. 如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。


## 分区

### 默认Partitioner分区
```Java
public class HashPartitioner<K, V> extends Partitioner<K, V> {
    public int getPartition(K key, V value, int numReduceTasks) {
        // 默认分区是根据key的hashCode对ReduceTasks个数取模得到的，用户没法控制哪个 key存储到哪个分区。
        return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    } 
}
```

### 自定义Partitioner
1. 自定义类继承Partitioner，重写getPartition()方法
2. 在Job驱动中，设置自定义Partitioner: `job.setPartitionerClass(CustomPartitioner.class);`
3. 自定义Partition后，要根据自定义Partitioner的逻辑设置相应数量的ReduceTask: `job.setNumReduceTasks(5);`

```Java
public class CustomPartitioner extends Partitioner<Text, FlowBean> {
    @Override
    public int getPartition(Text key, FlowBean value, int numPartitions) {
        // 控制分区代码逻辑
        return partition;
    }
}
```

1. 如果ReduceTask的数量 > getPartition的结果数，则会多产生几个空的输出文件part-r-000xx;
2. 如果1 < ReduceTask的数量 < getPartition的结果数，则有一部分分区数据无处安放，会Exception;
3. 如果ReduceTask的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个 ReduceTask，最终也就只会产生一个结果文件 part-r-00000;
4. 分区号必须从零开始，逐一累加。



## WritableComparable 排序
MapTask和ReduceTask均会对数据按照key进行排序。该操作属于 Hadoop 的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。默认排序是按照字典顺序排序，且实现该排序的方法是快速排序。
1. 对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使 用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。
2. 对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大文件;如果内存中文件大小或者 数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。


## Combiner合并

### 定义
1. Combiner是MR程序中Mapper和Reducer之外的一种组件。
2. Combiner组件的父类就是Reducer。
3. Combiner和Reducer的区别在于运行的位置。
4. Combiner是在每一个MapTask所在的节点运行，Reducer是接收全局所有Mapper的输出结果;
5. Combiner的意义就是对每一个MapTask的输出进行局部汇总，以减小网络传输量。
Combiner能够应用的前提是不能影响最终的业务逻辑，而且，Combiner的输出kv应该跟Reducer的输入kv类型要对应起来。

### 自定义Combiner
自定义一个 Combiner 继承 Reducer，重写 Reduce 方法：
```Java
public class WordCountCombiner extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable outV = new IntWritable();
   
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable value : values) {
            sum += value.get();
        }
        outV.set(sum);
        context.write(key,outV);
    } 
}
```

### 指定Combiner
```Java
// 指定需要使用 combiner，以及用哪个类作为 combiner 的逻辑
job.setCombinerClass(WordCountCombiner.class);
job.setCombinerClass(WordCountReducer.class);
```
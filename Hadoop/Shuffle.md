# Shuffle

![MapReduce+20221116150638](https://raw.githubusercontent.com/loli0con/picgo/master/images/MapReduce%2B20221116150638.png%2B2022-11-16-15-06-39)

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

### 分区数目
1. 如果ReduceTask的数量 > getPartition的结果数，则会多产生几个空的输出文件part-r-000xx;
2. 如果1 < ReduceTask的数量 < getPartition的结果数，则有一部分分区数据无处安放，会Exception;
3. 如果ReduceTask的数量=1，则不管MapTask端输出多少个分区文件，最终结果都交给这一个 ReduceTask，最终也就只会产生一个结果文件 part-r-00000;
4. 分区号必须从零开始，逐一累加。

## WritableComparable 排序
MapTask和ReduceTask均会对数据按照key进行排序。该操作属于 Hadoop 的默认行为。任何应用程序中的数据均会被排序，而不管逻辑上是否需要。默认排序是按照字典顺序排序，且实现该排序的方法是快速排序。
1. 对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使 用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。
2. 对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。

### 排序分类
1. 部分排序: MapReduce根据输入记录的键对数据集排序，保证输出的每个文件内部有序。
2. 全排序: 最终输出结果只有一个文件，且文件内部有序。实现方式是只设置一个ReduceTask。但该方法在处理大型文件时效率极低，因为一台机器处理所有文件，完全丧失了MapReduce所提供的并行架构。
3. 辅助排序: (GroupingComparator分组) 在Reduce端对key进行分组。应用于:在接收的key为bean对象时，想让一个或几个字段相同(全部字段比较不相同)的key进入到同一个reduce方法时，可以采用分组排序。
4. 二次排序: 在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。

### 自定义排
bean 对象做为 key 传输，需要实现 WritableComparable 接口重写 compareTo 方法，就可以实现排序。

具体见[MapReduce](MapReduce.md)中的[demo](MapReduce.md#demo)的 compareTo 方法。

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
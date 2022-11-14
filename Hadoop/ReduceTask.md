# ReduceTask


## OutputFormat 数据输出

### OutputFormat 接口实现类
OutputFormat是MapReduce输出的基类，所有实现MapReduce输出都实现了 OutputFormat。默认输出格式TextOutputFormat。

![ReduceTask+20221114145156](https://raw.githubusercontent.com/loli0con/picgo/master/images/ReduceTask%2B20221114145156.png%2B2022-11-14-14-51-58)


## ReduceTask 并行度

### 设置 ReduceTask 并行度
ReduceTask 的并行度同样影响整个 Job 的执行并发度和执行效率，但与 MapTask 的并发数由切片数决定不同，ReduceTask 数量的决定是可以直接手动设置:
```Java
// 默认值是 1，手动设置为 4
job.setNumReduceTasks(4);
```

### 注意事项
1. ReduceTask=0，表示没有Reduce阶段，输出文件个数和Map个数一致。
2. ReduceTask默认值就是1，所以输出文件个数为一个。
3. 如果数据分布不均匀，就有可能在Reduce阶段产生数据倾斜。
4. ReduceTask数量并不是任意设置，还要考虑业务逻辑需求，有些情况下，需要计算全局汇总结果，就只能有1个ReduceTask。
5. 具体多少个ReduceTask，需要根据集群性能而定。
6. 如果分区数不是1，但是ReduceTask为1，则不执行分区过程。因为在MapTask的源码中，执行分区的前提是先判断ReduceNum个数是否大于1。不大于1肯定不执行。
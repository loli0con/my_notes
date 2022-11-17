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

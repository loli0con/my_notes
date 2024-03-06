# Performance Schema
在高负载下调优数据库性能是一个迭代循环的过程。每次进行更改以调优数据库的性能时，都需要了解更改是否有什么影响。**Performance Schema**是一个*存储数据库性能相关数据*的**数据库**。

## 介绍

### 与Performance Schema的工作机制相关的两个概念

#### instrument
第一个概念是程序插桩(instrument)。程序插桩在MySQL代码中插入探测代码，以获取我们想了解的信息。例如，如果想收集关于元数据锁的使用情况，需要启用wait/lock/meta-data/sql/mdl这个插桩。

> 程序插桩，最早是由J.C. Huang 教授提出的，它是在保证被测程序原有逻辑完整性的基础上在程序中插入一些探针（又称为“探测仪”，本质上就是进行信息采集的代码段，可以是赋值语句或采集覆盖信息的函数调用），通过探针的执行并抛出程序运行的特征数据，通过对这些数据的分析，可以获得程序的控制流和数据流信息，进而得到逻辑覆盖等动态信息，从而实现测试目的的方法。

#### consumer
第二个概念是消费者表(consumer)，指的是存储关于程序插桩代码信息的表。如果我们为查询模块添加插桩，相应的消费者表将记录诸如执行总数、未使用索引的次数、花费的时间等信息。

### Performance Schema的一般功能
![performance_schema+20240222153055](https://raw.githubusercontent.com/loli0con/picgo/master/images/performance_schema+20240222153055.png+2024-02-22-15-30-55)

当应用程序用户连接到MySQL并执行被测量的插桩指令时，performance_schema将每个检查的调用封装到两个宏中，然后将结果记录在相应的消费者表中。这里的要点是，启用插桩会调用额外的代码，这意味着插桩会消耗CPU资源。

### 插桩元件
在*performance_schema*中，*setup_instruments*表包含所有支持的插桩的列表。所有插桩的名 称都由用斜杠分隔的部件组成。下面的例子展示了插桩的命名规则:
* statement/sql/select
* wait/synch/mutex/innodb/autoinc_mutex

插桩名称的最左边部分表示插桩的类型。因此，statement表示插桩类型是statement，wait表示插桩类型是wait，以此类推。名称字段中的其余部分从左至右依次表示从通用到特定的子系统。在前面的示例中，select是sql子系统的一部分，属于statement类型。或者autoinc_mutex属于innodb，它是更通用的插桩类mutex的一部分，而mutex又是更通用的插桩类型wait的sync插桩的一部分。

大多数插桩名称是自描述型的。与示例中一样，statement/sql/select是一个SELECT查询，wait/synch/mutex/innodb/autoinc_mutex是InnoDB在自增列上设置的一个互斥体。*setup_instruments*表中还有一个*DOCUMENTATION*列，其中包含更多详细信息。

不幸的是，对于许多插桩而言，*DOCUMENTATION*列可能为空，因此需要根据插桩的名称、你的直觉以及对MySQL源代码的认识来理解特定插桩检查的内容。

### 消费者表
消费者表是插桩发送信息的目的地，测量结果存储在Performance Schema数据库的多个表中。

#### 事件数据
存放事件的表名包含如下结尾:
- *_current：当前服务器上进行中的事件。
- *_history： 每个线程最近完成的10个事件。
- *_history_long 从全局来看，每个线程最近完成的10000个事件。

\*_history和\*_history_long表的大小是可配置的。

#### 当前和历史数据
当前和历史数据:
* events_waits：底层服务器等待，例如获取互斥对象。
* events_statements：SQL查询语句。
* events_stages：配置文件信息，例如创建临时表或发送数据。
* events_transactions：事务。

#### 汇总表和摘要
汇总表保存有关该表所建议的内容的聚合信息。例如，memory_summary_by_thread_by_event_name表保存了用户连接或任何后台线程的每个MySQL线程的聚合内存使用情况。

摘要是一种通过删除查询中的变量来聚合查询的方法。例如以下查询:
```sql
SELECT user,birthdate FROM users WHERE user_id=19
SELECT user,birthdate FROM users WHERE user_id=23
SELECT user,birthdate FROM users WHERE user_id=27
```

该查询的摘要是:
```sql
SELECT user,birthdate FROM users WHERE user_id=?
```
这允许Performance Schema跟踪摘要的延迟等指标，而无须单独保留查询的每个变体。

#### 实例表(Instance)
实例是指对象实例，用于MySQL安装程序。例如，file_instances表包含文件名和访问这些文件的线程数。

#### 设置表(Setup)
设置表用于performance_schema的运行时设置。

#### 其他表
还有一些表的名称没有遵循严格的模式。例如，metadata_locks表保存关于元数据锁的数据。

### 资源消耗
Performance Schema收集的数据保存在内存中。可以通过设置消费者表的最大大小来限制其使用的内存量。performance_schema中的一些表支持自动伸缩，这意味着它们在启动时分配最小数量的内存，并根据需要调整其大小。然而，一旦分配了内存，即使禁用了特定的插桩并截断了表，也不会再释放该内存。

如前所述，每个插桩指令的调用都会再添加两个宏调用，以将数据存储在performance_schema中。这意味着插桩越多，CPU的使用率就越高。对CPU使用率的实际影响取决于特定的插桩。例如，与statement相关的插桩在查询过程中只能被调用一次，而wait类插桩的被调用频率要高得多。例如，要扫描一个有一百万行的InnoDB表，引擎需要 设置和释放一百万行锁。如果对锁使用wait类插桩，CPU使用率可能会显著增加。但是，同一查询本来就需要一次调用来确定是否是statement/sql/select。因此，如果启用statement类插桩，你不会注意到CPU负载的任何增加。内存或元数据锁类型的插桩也是如此。

### 局限性
* 它必须得到MySQL组件的支持：例如，假设使用内存插桩来计算哪个MySQL组件或线程使用了大部分内存。你发现使用最多内存的组件是一个不支持内存插桩的存储引擎。在这种情况下，你将无法找到内存的去向。
* 它只在特定的插桩和用户启用后才收集数据：例如，如果在禁用所有插桩的情况下启动服务器，然后决定检测内存使用情况，则无法知道全局缓冲区(如InnoDB缓冲池)分配的确切数量，因为在启用内存插桩之前已分配了该缓冲区。
* 它很难释放内存：可以在启动时限制消费者表的大小，也可以让其自动调整大小。在后一种情况下，它们不会在启动时分配内存，而是仅在收集启用的数据时分配内存。但是，即使以后禁用了特定的插桩或消费者表，也不会释放内存，除非重新启动服务器。

### sys Schema
标准MySQL发行版包括一个和*performance_schema*数据配套使用的*sys* schema，它全部基于*performance_schema*上的视图和存储例程组成。它的设计目的是让*performance_schema*体验更加流畅，它本身并不存储任何数据。如果在*sys* schema中找不到你想看的数据，可尝试
在*performance_schema*的基表中查找。

### 线程

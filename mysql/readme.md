# MySQL

## 变种
* Drizzle：可用性增强，但很多地方不兼容
* MariaDB：Mysql之父在mysql被收购时创立，是Mysql的超集，完全兼容
* Percona：性能增强，《高性能Mysql》的作者创立
* Postgre：稳定性极强，抗压；数据类型丰富
* SQLite：小巧，嵌入式

## 目录
* MySQL软件配置与维护
  * [MySQL8程序](MySQL8程序.md)
  * [配置文件](配置文件.md)
  * [主要目录](主要目录.md)
  * [默认数据库](默认数据库.md)
* MySQL架构
  * [逻辑架构](逻辑架构.md)
  * [SQL执行流程](SQL执行流程.md)
    * [执行阶段耗时分析profiling](profiling.md)
  * [数据库缓冲池](数据库缓冲池.md)
* InnoDB的索引结构与存储结构
  * InnoDB的索引结构
    * [索引的概述](索引的概述.md)
      * [什么是索引](索引的概述.md#什么是索引)
      * [为什么使用索引](索引的概述.md#为什么使用索引)
      * [索引的优缺点](索引的概述.md#索引的优缺点)
    * [索引的详解](索引的详解.md)
        * [索引的推演](索引的详解.md#索引的推演)
        * [聚簇索引和二级索引](索引的详解.md#聚簇索引和二级索引)
          * [聚簇索引](索引的详解.md#聚簇索引)
          * [二级索引](索引的详解.md#二级索引辅助索引非聚簇索引)
            * [回表](索引的详解.md#回表)
          * [聚簇索引和二级索引的区别](索引的详解.md#区别)
        * [联合索引](索引的详解.md#联合索引)
        * [索引的注意事项](索引的详解.md#索引的注意事项)
          1. [根页面位置万年不动](索引的详解.md#1-根页面位置万年不动)
          2. [内节点非叶子节点中目录项记录的唯一性](索引的详解.md#2-内节点非叶子节点中目录项记录的唯一性)
          3. [一个页面最少存储2条记录](索引的详解.md#3-一个页面最少存储2条记录)
        * [MyISAM中的索引方案](索引的详解.md#MyISAM中的索引方案)
          * [MyISAM索引的原理](索引的详解.md#MyISAM索引的原理)
          * [MyISAM与InnoDB对比](索引的详解.md#MyISAM与InnoDB对比)
  * [InnoDB的存储结构](InnoDB的存储结构.md)
    * 页结构
      * [页结构概述](InnoDB的存储结构.md#页结构概述)
        * [页的大小](InnoDB的存储结构.md#页的大小)
        * [页的上层结构](InnoDB的存储结构.md#页的上层结构)
      * [页的内部结构](InnoDB的存储结构.md#页的内部结构)
        1. [文件头&文件尾](InnoDB的存储结构.md#第一部分文件头file-header-文件尾file-tailer)
        2. [空闲空间&用户记录&最大最小记录](InnoDB的存储结构.md#第二部分空闲空间free-space-用户记录user-records-最大最小记录)
        3. [页目录&页面头部](InnoDB的存储结构.md#第三部分页目录page-directory-页面头部page-header)
    * [行格式](InnoDB的存储结构.md#innodb行格式)
      * [COMPACT行格式](InnoDB的存储结构.md#COMPACT行格式)
        * [变长字段长度列表](InnoDB的存储结构.md#变长字段长度列表)
        * [NULL值列表](InnoDB的存储结构.md#null值列表)
        * [记录头信息](InnoDB的存储结构.md#记录头信息5字节)
        * [记录的真实数据](InnoDB的存储结构.md#记录的真实数据)
      * [行溢出](InnoDB的存储结构.md#行溢出)
      * [其它行格式](InnoDB的存储结构.md#其他行格式)
        * [Dynamic和Compressed](InnoDB的存储结构.md#dynamic和compressed)
        * [Redundant](InnoDB的存储结构.md#redundant)
    * [区、段与碎片区](InnoDB的存储结构.md#区段与碎片区)
      * [为什么要有区？](InnoDB的存储结构.md#为什么要有区)
      * [为什么要有段？](InnoDB的存储结构.md#为什么要有段)
      * [为什么要有碎片区？](InnoDB的存储结构.md#为什么要有碎片区)
      * [区的分类](InnoDB的存储结构.md#区的分类)
    * [表空间](InnoDB的存储结构.md#表空间)
      * [独立表空间](InnoDB的存储结构.md#独立表空间)
      * [系统表空间](InnoDB的存储结构.md#系统表空间)
    * [数据页加载的三种方式](InnoDB的存储结构.md#数据页加载的三种方式)
* 索引的使用与数据库调优
  * [索引的使用](索引的使用.md)
    * [索引的分类](索引的使用.md#索引的分类)
    * [索引的基本用法](索引的使用.md#基本用法)
      * [创建索引](索引的使用.md#创建索引)
      * [删除索引](索引的使用.md#删除索引)
      * [隐藏索引](索引的使用.md#隐藏索引)
    * [索引的设计原则](索引的使用.md#索引的设计原则)
      * [适合创建索引的情况](索引的使用.md#适合创建索引的情况)
      * [不适合创建索引的情况](索引的使用.md#不适合创建索引的情况)
    * [索引失效案例](索引的使用.md#索引失效案例)
  * [性能分析工具的使用](性能分析工具的使用.md)
    * [数据库服务器的优化步骤](性能分析工具的使用.md#数据库服务器的优化步骤)
    * [查看系统性能参数](性能分析工具的使用.md#查看系统性能参数)
    * [统计SQL的查询成本](性能分析工具的使用.md#统计sql的查询成本last_query_cost)
    * [定位执行慢的SQL](性能分析工具的使用.md#定位执行慢的sql慢查询日志)
    * [分析查询语句](性能分析工具的使用.md#分析查询语句explain)
    * [分析优化器执行计划](性能分析工具的使用.md#分析优化器执行计划trace)
    * [MySQL监控分析视图](性能分析工具的使用.md#mysql监控分析视图-sys-schema)
  * [逻辑查询优化](逻辑查询优化.md)
    * [关联查询优化](逻辑查询优化.md#关联查询优化)
      * [外连接](逻辑查询优化.md#外连接)
      * [内连接](逻辑查询优化.md#内连接)
      * [JOIN的底层原理](逻辑查询优化.md#JOIN的底层原理)
        * [驱动表和被驱动表](逻辑查询优化.md#驱动表和被驱动表)
        * [Simple Nested-Loop Join（简单嵌套循环连接）](逻辑查询优化.md#simple-nested-loop-join简单嵌套循环连接)
        * [Index Nested-Loop Join（索引嵌套循环连接）](逻辑查询优化.md#index-nested-loop-join索引嵌套循环连接)
        * [Block Nested-Loop Join（块嵌套循环连接）](逻辑查询优化.md#block-nested-loop-join块嵌套循环连接)
        * [Hash Join](逻辑查询优化.md#hash-join)
    * [子查询优化](逻辑查询优化.md#子查询优化)
    * [排序优化](逻辑查询优化.md#排序优化)
      * [双路排序](逻辑查询优化.md#双路排序-慢)
      * [单路排序](逻辑查询优化.md#单路排序-快)
    * [GROUP BY优化](逻辑查询优化.md#group-by优化)
    * [分页查询优化](逻辑查询优化.md#分页查询优化)
    * [覆盖索引](逻辑查询优化.md#覆盖索引)
    * [前缀索引](逻辑查询优化.md#前缀索引)
    * [索引下推](逻辑查询优化.md#索引下推)
    * [唯一索引和普通索引](逻辑查询优化.md#唯一索引和普通索引)
    * [其它查询优化策略](逻辑查询优化.md#其它查询优化策略)
      * [EXISTS和IN](逻辑查询优化.md#EXISTS和IN)
      * [COUNT(*)与COUNT(具体字段)](逻辑查询优化.md#count与count具体字段)
      * [关于SELECT(*)](逻辑查询优化.md#关于select)
      * [`LIMIT 1`对优化的影响](逻辑查询优化.md#limit-1-对优化的影响)
      * [多使用COMMIT](逻辑查询优化.md#多使用COMMIT)
  * [数据库调优策略](数据库调优策略.md)
    * [调优策略概述](数据库调优策略.md#调优策略概述)
      * [调优的目标](数据库调优策略.md#调优的目标)
      * [如何定位调优问题](数据库调优策略.md#如何定位调优问题)
      * [调优的维度和步骤](数据库调优策略.md#调优的维度和步骤)
        1. [选择适合的DBMS](数据库调优策略.md#第1步选择适合的dbms)
        2. [优化表设计](数据库调优策略.md#第2步优化表设计)
        3. [优化逻辑查询](数据库调优策略.md#第3步优化逻辑查询)
        4. [优化物理查询](数据库调优策略.md#第4步优化物理查询)
        5. [使用Redis作为缓存](数据库调优策略.md#第5步使用Redis作为缓存)
        6. [库级优化](数据库调优策略.md#第6步库级优化)
          * [读写分离](数据库调优策略.md#读写分离)
          * [数据分片](数据库调优策略.md#数据分片)
    * [优化MySQL服务器](数据库调优策略.md#优化MySQL服务器)
      * [优化服务器硬件](数据库调优策略.md#优化服务器硬件)
      * [优化MySQL的参数](数据库调优策略.md#优化MySQL的参数)
    * [优化数据库结构](数据库调优策略.md#优化数据库结构)
      * [冷热数据分离](数据库调优策略.md#表拆分冷热数据分离)
      * [增加中间表](数据库调优策略.md#增加中间表)
      * [增加冗余字段](数据库调优策略.md#)
      * [优化数据类型](数据库调优策略.md#优化数据类型)
      * [优化插入记录的速度](数据库调优策略.md#优化插入记录的速度)
        * [MyISAM引擎的表](数据库调优策略.md#MyISAM引擎的表)
          * [禁用索引](数据库调优策略.md#禁用索引)
          * [禁用唯一性检查](数据库调优策略.md#禁用唯一性检查)
          * [使用批量插入](数据库调优策略.md#使用批量插入)
          * [使用`LOAD DATA INFILE`批量导入](数据库调优策略.md#使用load-data-infile批量导入)
        * [InnoDB引擎的表](数据库调优策略.md#InnoDB引擎的表)
          * [禁用外键检查](数据库调优策略.md#禁用外键检查)
          * [禁止自动提交](数据库调优策略.md#禁止自动提交)
      * [使用非空约束](数据库调优策略.md#使用非空约束)
      * [分析表、检查表与优化表](数据库调优策略.md#分析表检查表与优化表)
        * [分析表](数据库调优策略.md#分析表)
        * [检查表](数据库调优策略.md#检查表)
        * [优化表](数据库调优策略.md#优化表)
    * [大表优化](数据库调优策略.md#大表优化)
      * [限定查询的范围](数据库调优策略.md#限定查询的范围)
      * [读写分离](数据库调优策略.md#主写从读)
      * [垂直拆分](数据库调优策略.md#垂直拆分)
      * [水平拆分](数据库调优策略.md#水平拆分)
    * [其它调优策略](数据库调优策略.md#其它调优策略)
      * [服务器语句超时处理](数据库调优策略.md#服务器语句超时处理)
      * [创建全局通用表空间](数据库调优策略.md#创建全局通用表空间)
      * [隐藏索引对调优的帮助](数据库调优策略.md#隐藏索引对调优的帮助)
* [数据库范式](数据库范式.md)
* 事务
  * [事务基础知识](事务基础知识.md)
    * [数据库事务概述](事务基础知识.md#数据库事务概述)
      * [存储引擎支持情况](事务基础知识.md#存储引擎支持情况)
      * [基本概念](事务基础知识.md#基本概念)
      * [事务的ACID特性](事务基础知识.md#事务的ACID特性)
        * [原子性](事务基础知识.md#原子性atomicity)
        * [一致性](事务基础知识.md#一致性consistency)
        * [隔离型](事务基础知识.md#隔离型isolation)
        * [持久性](事务基础知识.md#持久性durability)
      * [事务的状态](事务基础知识.md#事务的状态)
    * [使用数据库事务](事务基础知识.md#使用数据库事务)
      * [显式事务](事务基础知识.md#显式事务)
      * [隐式事务](事务基础知识.md#隐式事务)
        * [隐式提交数据的情况](事务基础知识.md#隐式提交数据的情况)
          * [数据定义语言](事务基础知识.md#数据定义语言data-definition-language缩写为ddl)
          * [隐式使用或修改mysql数据库中的表](事务基础知识.md#隐式使用或修改mysql数据库中的表)
          * [事务控制或关于锁定的语句](事务基础知识.md#事务控制或关于锁定的语句)
          * [加载数据的语句](事务基础知识.md#加载数据的语句)
          * [关于MySQL复制的一些语句](事务基础知识.md#关于MySQL复制的一些语句)
          * [其它的一些语句](事务基础知识.md#其它的一些语句)
    * [事务隔离级别](事务基础知识.md#事务隔离级别)
      * [数据并发问题](事务基础知识.md#数据并发问题)
        * [脏写](事务基础知识.md#脏写dirty-write)
        * [脏读](事务基础知识.md#脏读dirty-read)
        * [不可重复读](事务基础知识.md#不可重复读non-repeatable-read)
        * [幻读](事务基础知识.md#幻读phantom)
      * [SQL中的四种隔离级别](事务基础知识.md#sql中的四种隔离级别)
      * [MySQL支持的四种隔离级别](事务基础知识.md#MySQL支持的四种隔离级别)
      * [设置事务的隔离级别](事务基础知识.md#设置事务的隔离级别)
    * [事务的常见分类](事务基础知识.md#事务的常见分类)
      * [扁平事务](事务基础知识.md#扁平事务)
      * [带有保存点的扁平事务](事务基础知识.md#带有保存点的扁平事务)
      * [链事务](事务基础知识.md#链事务)
      * [嵌套事务](事务基础知识.md#嵌套事务)
      * [分布式事务](事务基础知识.md#分布式事务)
  * [MySQL事务日志](MySQL事务日志.md)
    * [redo日志](MySQL事务日志.md#redo日志)
      * [为什么需要redo日志](MySQL事务日志.md#为什么需要redo日志)
      * [redo日志的好处与特点](MySQL事务日志.md#redo日志的好处特点)
        * [好处](MySQL事务日志.md#好处)
        * [特点](MySQL事务日志.md#特点)
      * [redo的组成](MySQL事务日志.md#)
        * [Redo Log Buffer（重做日志缓冲）](MySQL事务日志.md#redo-log-buffer重做日志缓冲)
        * [Redo Log File（重做日志文件）](MySQL事务日志.md#redo-log-file重做日志文件)
      * [redo日志的刷盘策略](MySQL事务日志.md#redo日志的刷盘策略)
      * [不同刷盘策略演示](MySQL事务日志.md#不同刷盘策略演示)
        * [innodb_flush_log_at_trx_commit=1](MySQL事务日志.md#innodb_flush_log_at_trx_commit1)
        * [innodb_flush_log_at_trx_commit=2](MySQL事务日志.md#innodb_flush_log_at_trx_commit2)
        * [innodb_flush_log_at_trx_commit=0](MySQL事务日志.md#innodb_flush_log_at_trx_commit0)
      * [写入Redo Log Buffer过程](MySQL事务日志.md#写入redo-log-buffer过程)
        * [Mini-Transaction](MySQL事务日志.md#mini-transaction)
        * [redo日志写入Log Buffer](MySQL事务日志.md#redo日志写入log-buffer)
      * [Redo Log Block的结构图](MySQL事务日志.md#redo-log-block的结构图)
      * [Redo Log File详解](MySQL事务日志.md#redo-log-file详解)
        * [相关参数设置](MySQL事务日志.md#相关参数设置)
        * [日志文件组](MySQL事务日志.md#日志文件组)
        * [checkpoint](MySQL事务日志.md#checkpoint)
      * [小结](MySQL事务日志.md#小结)
    * [undo日志](MySQL事务日志.md#undo日志)
      * [如何理解undo日志](MySQL事务日志.md#如何理解undo日志)
      * [undo日志的作用](MySQL事务日志.md#undo日志的作用)
        * [回滚数据](MySQL事务日志.md#回滚数据)
        * [MVCC](MySQL事务日志.md#MVCC)
      * [undo的存储结构](MySQL事务日志.md#undo的存储结构)
        * [回滚段与undo页](MySQL事务日志.md#回滚段与undo页)
          * [undo页的重用](MySQL事务日志.md#undo页的重用)
        * [回滚段与事务](MySQL事务日志.md#回滚段与事务)
        * [回滚段中的数据分类](MySQL事务日志.md#回滚段中的数据分类)
      * [undo的类型](MySQL事务日志.md#undo的类型)
      * [undo log的生命周期](MySQL事务日志.md#undo-log的生命周期)
        * [简要生成过程](MySQL事务日志.md#简要生成过程)
        * [详细生成过程](MySQL事务日志.md#详细生成过程)
        * [undo log是如何回滚的](MySQL事务日志.md#undo-log是如何回滚的)
        * [undo log的删除](MySQL事务日志.md#undo-log的删除)
    * [总结](MySQL事务日志.md#总结)
* [锁](锁.md)
  * [概述](锁.md#概述)
  * [并发访问相同记录的情况](锁.md#并发访问相同记录的情况)
    * [读-读情况](锁.md#读-读情况)
    * [写-写情况](锁.md#写-写情况)
    * [读-写或写-读情况](锁.md#读-写或写-读情况)
  * [并发问题的解决方案](锁.md#并发问题的解决方案)
    1. [读操作利用多版本并发控制（MVCC），写操作进行加锁](锁.md#方案一读操作利用多版本并发控制mvcc写操作进行加锁)
    2. [读、写操作都采用加锁的方式](锁.md#方案二读写操作都采用加锁的方式)
    * [方案对比](锁.md#方案对比)
  * [锁的不同角度分类](锁.md#锁的不同角度分类)
    * [从数据操作的类型划分：读锁、写锁](锁.md#从数据操作的类型划分读锁写锁)
      * [锁定读](锁.md#锁定读)
        * [对读取的记录加S锁](锁.md#对读取的记录加S锁)
        * [对读取的记录加X锁](锁.md#对读取的记录加X锁)
      * [写操作](锁.md#写操作)
    * [从数据操作的粒度划分：表级锁、页级锁、行锁](锁.md#从数据操作的粒度划分表级锁页级锁行锁)
      * [表锁（Table Lock）](锁.md#表锁table-lock)
        * [表级别的S锁、X锁](锁.md#表级别的s锁x锁)
        * [意向锁 (Intention Lock)](锁.md#意向锁-intention-lock)
          * [意向锁要解决的问题](锁.md#意向锁要解决的问题)
          * [意向锁的并发性](锁.md#意向锁的并发性)
          * [意向锁总结](锁.md#意向锁总结)
        * [自增锁（AUTO-INC锁）](锁.md#自增锁auto-inc-lock)
        * [元数据锁（MDL锁）](锁.md#元数据锁mdl锁)
      * [InnoDB中的行锁](锁.md#InnoDB中的行锁)
        * [记录锁（Record Locks）](锁.md#记录锁record-locks)
        * [间隙锁（Gap Locks）](锁.md#间隙锁gap-locks)
        * [临键锁（Next-Key Locks）](锁.md#临键锁next-key-locks)
        * [插入意向锁（Insert Intention Locks）](锁.md#插入意向锁insert-intention-locks)
      * [页锁](锁.md#页锁)
    * [从对待锁的态度划分：乐观锁、悲观锁](锁.md#从对待锁的态度划分乐观锁悲观锁)
      * [悲观锁（Pessimistic Locking）](锁.md#悲观锁pessimistic-locking)
        * [秒杀案例-悲观锁](锁.md#秒杀案例-悲观锁)
      * [乐观锁（Optimistic Locking）](锁.md#乐观锁optimistic-locking)
        * [版本号机制](锁.md#版本号机制)
        * [时间戳机制](锁.md#时间戳机制)
        * [秒杀案例-乐观锁](锁.md#秒杀案例-乐观锁)
      * [悲观锁和乐观锁的适用场景](锁.md#悲观锁和乐观锁的适用场景)
    * [按加锁的方式划分：显式锁、隐式锁](锁.md#按加锁的方式划分显式锁隐式锁)
      * [隐式锁](锁.md#隐式锁)
      * [显式锁](锁.md#显式锁)
    * [全局锁](锁.md#全局锁)
  * [死锁](锁.md#死锁)
    * [什么是死锁](锁.md#什么是死锁)
      * [死锁-举例1](锁.md#死锁-举例1)
      * [死锁-举例2](锁.md#死锁-举例2)
    * [产生死锁的必要条件](锁.md#产生死锁的必要条件)
    * [如何处理死锁](锁.md#如何处理死锁)
      1. [等待，直到超时（innodb_lock_wait_timeout=50s）](锁.md#方式一等待直到超时innodb_lock_wait_timeout50s)
      2. [使用死锁检测进行死锁处理](锁.md#方式二使用死锁检测进行死锁处理)
        * [死锁检测的成本分析](锁.md#死锁检测的成本分析)
    * [如何避免死锁](锁.md#如何避免死锁)
 * [锁的内存结构](锁.md#锁的内存结构)
 * [锁监控](锁.md#锁监控)
* [多版本并发控制](多版本并发控制.md)
  * [什么是MVCC](多版本并发控制.md#什么是MVCC)
  * [快照读与当前读](多版本并发控制.md#快照读与当前读)
    * [快照读](多版本并发控制.md#快照读)
    * [当前读](多版本并发控制.md#当前读)
  * [先导概念回顾](多版本并发控制.md#先导概念回顾)
    * [隔离级别](多版本并发控制.md#隔离级别)
    * [隐藏字段和Undo Log版本链](多版本并发控制.md#隐藏字段和undo-log版本链)
  * [MVCC的实现原理](多版本并发控制.md#MVCC的实现原理)
    * [什么是ReadView](多版本并发控制.md#什么是ReadView)
    * [ReadView的设计思路](多版本并发控制.md#ReadView的设计思路)
    * [ReadView的规则](多版本并发控制.md#ReadView的规则)
  * [MVCC的整体流程](多版本并发控制.md#MVCC的整体流程)
  * [MVCC的案例详解](多版本并发控制.md#MVCC的案例详解)
    * [READ COMMITTED隔离级别下](多版本并发控制.md#read-committed隔离级别下)
    * [REPEATABLE READ隔离级别下](多版本并发控制.md#repeatable-read隔离级别下)
  * [MVCC如何解决幻读](多版本并发控制.md#MVCC如何解决幻读)
  * [MVCC总结](多版本并发控制.md#MVCC总结)
* [其他数据库日志](其他数据库日志.md)
  * [日志类型](其他数据库日志.md#日志类型)
  * [日志的弊端](其他数据库日志.md#日志的弊端)
  * [慢查询日志（slow query log）](其他数据库日志.md#慢查询日志slow-query-log)
  * [通用查询日志（general query log）](其他数据库日志.md#通用查询日志general-query-log)
  * [错误日志（error log）](其他数据库日志.md#错误日志error-log)
  * [二进制日志（bin log）](其他数据库日志.md#二进制日志bin-log)
    * [使用日志恢复数据](其他数据库日志.md#使用日志恢复数据)
    * [写入机制](其他数据库日志.md#写入机制)
    * [binlog与redolog对比](其他数据库日志.md#binlog与redolog对比)
    * [两阶段提交](其他数据库日志.md#两阶段提交)
  * [中继日志（relay log）](其他数据库日志.md#中继日志relay-log)
* [主从复制](主从复制.md)
  * [概述](主从复制.md#概述)
    * [如何提升数据库并发能力](主从复制.md#如何提升数据库并发能力)
    * [主从复制的作用](主从复制.md#主从复制的作用)
      * [读写分离](主从复制.md#读写分离)
      * [数据备份](主从复制.md#数据备份)
      * [高可用性](主从复制.md#高可用性)
  * [原理](主从复制.md#主从复制的原理)
    * [主从复制的三个线程](主从复制.md#主从复制的三个线程)
    * [主从复制的三个步骤](主从复制.md#主从复制的三个步骤)
    * [主从复制的问题](主从复制.md#主从复制的问题)
  * [基本原则](主从复制.md#基本原则)
  * [一主一从架构搭建](主从复制.md#一主一从架构搭建)
    * [准备工作](主从复制.md#准备工作)
    * [配置文件](主从复制.md#主从机配置文件修改)
      * [主机配置文件](主从复制.md#主机配置文件)
        * [binlog格式设置](主从复制.md#binlog格式设置)
          * [STATEMENT模式](主从复制.md#statement模式基于sql语句的复制statement-based-replicationsbr)
          * [ROW模式](主从复制.md#row模式基于行的复制row-based-replicationrbr)
          * [MIXED模式](主从复制.md#mixed模式混合模式复制mixed-based-replicationmbr)
      * [从机配置文件](主从复制.md#从机配置文件)
    * [主机：建立账户并授权](主从复制.md#主机建立账户并授权)
    * [从机：配置需要复制的主机](主从复制.md#从机配置需要复制的主机)
      1. [设置复制目标主机](主从复制.md#步骤1设置复制目标主机)
      2. [启动同步](主从复制.md#步骤2启动同步)
    * [停止主从同步](主从复制.md#停止主从同步)
      * [重新配置主从](主从复制.md#重新配置主从)
    * [双主双从架构](主从复制.md#双主双从架构)
  * [同步数据一致性问题](主从复制.md#同步数据一致性问题)
    * [主从同步的要求](主从复制.md#主从同步的要求)
    * [理解主从延迟问题](主从复制.md#理解主从延迟问题)
    * [主从延迟问题原因](主从复制.md#主从延迟问题原因)
    * [如何减少主从延迟](主从复制.md#如何减少主从延迟)
    * [如何解决一致性问题](主从复制.md#如何解决一致性问题)
      1. [异步复制](主从复制.md#方法1异步复制)
      2. [半同步复制](主从复制.md#方法2半同步复制)
      3. [组复制](主从复制.md#方法3组复制)
  * [读写分离实现](主从复制.md#读写分离实现)
* [数据备份与恢复](数据备份与恢复.md)
  * [物理备份与逻辑备份](数据备份与恢复.md#物理备份与逻辑备份)
  * [逻辑备份与恢复](数据备份与恢复.md#逻辑备份与恢复)
    * [mysqldump实现逻辑备份](数据备份与恢复.md#mysqldump实现逻辑备份)
      * [备份一个数据库](数据备份与恢复.md#备份一个数据库)
        * [基本语法](数据备份与恢复.md#基本语法)
        * [备份文件剖析](数据备份与恢复.md#备份文件剖析)
      * [备份全部数据库](数据备份与恢复.md#备份全部数据库)
      * [备份部分数据库](数据备份与恢复.md#备份部分数据库)
      * [备份部分表](数据备份与恢复.md#备份部分表)
      * [备份单表的部分数据](数据备份与恢复.md#备份单表的部分数据)
      * [排除某些表的备份](数据备份与恢复.md#排除某些表的备份)
      * [只备份结构或只备份数据](数据备份与恢复.md#只备份结构或只备份数据)
        * [只备份结构](数据备份与恢复.md#只备份结构)
        * [只备份数据](数据备份与恢复.md#只备份数据)
      * [备份中包含存储过程、函数、事件](数据备份与恢复.md#备份中包含存储过程函数事件)
      * [mysqldump常用选项](数据备份与恢复.md#mysqldump常用选项)
    * [mysql命令恢复数据](数据备份与恢复.md#mysql命令恢复数据)
      * [单库备份中恢复单库](数据备份与恢复.md#单库备份中恢复单库)
      * [全量备份恢复](数据备份与恢复.md#全量备份恢复)
      * [从全量备份中恢复单库](数据备份与恢复.md#从全量备份中恢复单库)
      * [从单库备份中恢复单表](数据备份与恢复.md#从单库备份中恢复单表)
  * [物理备份与恢复](数据备份与恢复.md#物理备份与恢复)
    * [物理备份:直接复制整个数据库](数据备份与恢复.md#物理备份直接复制整个数据库)
    * [物理恢复:直接复制到数据库目录](数据备份与恢复.md#物理恢复直接复制到数据库目录)
  * [表的导出与导入](数据备份与恢复.md#表的导出与导入)
    * [表的导出](数据备份与恢复.md#表的导出)
      * [使用SELECT...INTO OUTFILE导出文本文件](数据备份与恢复.md#使用selectinto-outfile导出文本文件)
      * [使用mysqldump命令导出文本文件](数据备份与恢复.md#使用mysqldump命令导出文本文件)
      * [使用mysql命令导出文本文件](数据备份与恢复.md#使用mysql命令导出文本文件)
    * [表的导入](数据备份与恢复.md#表的导入)
      * [使用LOAD DATA INFILE方式导入文本文件](数据备份与恢复.md#使用load-data-infile方式导入文本文件)
      * [使用mysqlimport方式导入文本文件](数据备份与恢复.md#使用mysqlimport方式导入文本文件)
  * [数据库迁移](数据备份与恢复.md#数据库迁移)
    * [迁移方案](数据备份与恢复.md#迁移方案)
      * [物理迁移](数据备份与恢复.md#物理迁移)
      * [逻辑迁移](数据备份与恢复.md#逻辑迁移)
    * [迁移注意点](数据备份与恢复.md#迁移注意点)
      * [相同版本的数据库之间迁移注意点](数据备份与恢复.md#相同版本的数据库之间迁移注意点)
      * [不同版本的数据库之间迁移注意点](数据备份与恢复.md#不同版本的数据库之间迁移注意点)
      * [不同数据库之间迁移注意点](数据备份与恢复.md#不同数据库之间迁移注意点)
      * [迁移小结](数据备份与恢复.md#迁移小结)
  * [删库了不敢跑，能干点啥？](数据备份与恢复.md#删库了不敢跑能干点啥)
    * [delete:误删行](数据备份与恢复.md#delete误删行)
      * [处理措施1:数据恢复](数据备份与恢复.md#处理措施1数据恢复)
      * [处理措施2:预防](数据备份与恢复.md#处理措施2预防)
    * [truncate/drop:误删库/表](数据备份与恢复.md#truncatedrop误删库表)
      * [背景](数据备份与恢复.md#背景)
      * [方案](数据备份与恢复.md#方案)
      * [预防](数据备份与恢复.md#预防)
        * [权限分离](数据备份与恢复.md#权限分离)
        * [制定规范操作](数据备份与恢复.md#制定规范操作)
        * [设置延迟复制备库](数据备份与恢复.md#设置延迟复制备库)
    * [rm:误删MySQL实例](数据备份与恢复.md#rm误删mysql实例)
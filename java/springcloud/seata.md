# seata
[Seata](http://seata.io/zh-cn/)是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

## 分布式事务问题
一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题。

![seata+20211101110439](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101110439.png%2B2021-11-01-11-04-40)
单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源，业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由本地事务来保证，但是全局的数据一致性问题没法保证。

## 基本概念
* Transaction ID XID：全局唯一的事务ID
* Transaction Coordinator (TC)：事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚
* Transaction Manager (TM)：控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议
* Resource Manager (RM)：控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚

## 处理过程
![seata+20211101121214](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101121214.png%2B2021-11-01-12-12-17)

1. TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的 XID； 
2. XID 在微服务调用链路的上下文中传播； 
3. RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖； 
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议； 
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。 


## 安装Seata-Server
1. 下载https://github.com/seata/seata/releases
2. 修改file.conf：自定义事务组名称+事务日志存储模式为db+数据库连接信息。（也可以把配置写在nacos里面，下面会讲解）
   1. service模块：vgroup_mapping.my_test_tx_group = "seata-auto-scan-news-tx"
   2. store模块：mode = "db"，url = "jdbc:mysql://127.0.0.1:3306/seata"，user = "root"，password = "你自己密码"
3. 数据库配置
   1. 新建库“seata”（根据上文的配置的数据库名）
   2. 执行db_store.sql
4. 修改registry.conf：配置注册中心
   1. registry模块：type = "nacos"
   2. nacos模块：serverAddr = "localhost:8848"
   3. 从nacos获取配置（也可以使用本地配置）
      ```conf
      config {
        # 读取tc服务端的配置文件的方式，这里是从nacos配置中心读取，这样如果tc是集群，可以共享配置
        type = "nacos"
        # 配置nacos地址等信息
        nacos {
            serverAddr = "localhost:8848"
            namespace = ""
            group = "SEATA_GROUP"
            username = ""
            password = ""
            dataId = "seataServer.properties"
        }
      }
      ```
5. 启动nacos再启动seata-server

## demo

### 配置
在nacos中配置“seataServer.properties”：

```properties
# 数据存储方式，db代表数据库
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://192.168.200.130:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=root
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
# 事务、日志等配置
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000

# 客户端与服务端传输方式
transport.serialization=seata
transport.compressor=none
# 关闭metrics功能，提高性能
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```

### pom
```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <!--版本较低，因此排除-->
        <exclusion>
            <artifactId>seata-spring-boot-starter</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<!--seata starter 采用1.4.2版本，要根据你使用的seata版本进行选择，保持一致-->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>1.4.2</version>
</dependency>
```

### yml
```yml
seata:
  registry: # TC服务注册中心的配置，微服务根据这些信息去注册中心获取tc服务地址
    # 参考tc服务自己的registry.conf中的配置
    type: nacos
    nacos: # tc
      server-addr: localhost:8848
      namespace: ""
      group: DEFAULT_GROUP
      application: seata-tc-server # tc服务在nacos中的服务名称
  tx-service-group: seata-auto-scan-news-tx # 事务组，根据这个获取tc服务的cluster名称
  service:
    vgroup-mapping:
      seata-auto-scan-news-tx: default
#   data-source-proxy-mode: XA
#   data-source-proxy-mode: AT
```

### 业务类
```java
@GlobalTransactional
```

## 事务模式

### AT
![seata+20211101185420](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101185420.png%2B2021-11-01-18-54-21)

阶段一RM的工作：
- 注册分支事务
- 记录undo-log（数据快照）
- 执行业务sql并提交
- 报告事务状态

阶段二提交时RM的工作：
- 删除undo-log即可

阶段二回滚时RM的工作：
- 根据undo-log恢复数据到更新前

#### 写隔离
对同一个记录，是用本地锁+全局锁避免脏写：
1. 拿到本地锁即对数据进行修改
2. 阶段一，拿到全局锁，即对修改进行提交，提交后释放本地锁
3. 阶段二，全局事务提交后，才会释放全局锁
4. 阶段二，全局事务回滚时，需要再次获得本地锁

![seata+20211101194748](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101194748.png%2B2021-11-01-19-47-50)

![seata+20211101195159](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101195159.png%2B2021-11-01-19-52-01)

#### 读隔离
在数据库本地事务隔离级别读已提交（Read Committed）或以上的基础上，Seata（AT 模式）的默认全局隔离级别是读未提交（Read Uncommitted）。

如果应用在特定场景下，必需要求全局的读已提交，目前 Seata 的方式是通过 SELECT FOR UPDATE 语句的代理。

出于总体性能上的考虑，Seata 目前的方案并没有对所有 SELECT 语句都进行代理，仅针对 FOR UPDATE 的 SELECT 语句。

### XA
![seata+20211101195458](https://raw.githubusercontent.com/loli0con/picgo/master/images/seata%2B20211101195458.png%2B2021-11-01-19-54-59)

RM一阶段的工作：
1. 注册分支事务到TC
​2. **执行分支业务sql但不提交**
​3. 报告执行状态到TC

TC二阶段的工作：
- TC检测各分支事务执行状态
  1. 如果都成功，通知所有RM提交事务
  2. 如果有失败，通知所有RM回滚事务

RM二阶段的工作：
- 接收TC指令，提交或回滚事务

### TCC
一个分布式的全局事务，整体是**两阶段提交**的模型。全局事务是由若干分支事务组成的，分支事务要满足**两阶段提交**的模型要求，即需要每个分支事务都具备自己的：
* 一阶段 prepare 行为
* 二阶段 commit 或 rollback 行为


TCC模式，不依赖于底层数据资源的事务支持，它把自定义的分支事件（即自定义prepare、commit和rollback行为）纳入全局事务的管理中。

### Saga
Saga模式是SEATA提供的长事务解决方案，在Saga模式中，业务流程中每个参与者都提交本地事务，当出现某一个参与者失败则补偿前面已经成功的参与者，一阶段正向服务和二阶段补偿服务都由业务开发实现。

适用场景：
* 业务流程长、业务流程多
* 参与者包含其它公司或遗留系统服务，无法提供 TCC 模式要求的三个接口

优势：
* 一阶段提交本地事务，无锁，高性能
* 事件驱动架构，参与者可异步执行，高吞吐
* 补偿服务易于实现

缺点：
* 不保证隔离性（应对方案见后面文档）

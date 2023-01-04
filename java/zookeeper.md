# ZooKeeper

## 概述
Zookeeper 是一个开源的分布式的，为分布式框架提供**协调服务**的 Apache 项目。

### 机制
Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它**负责存储和管理大家都关心的数据**，然后**接受观察者的注册**，一旦这些数据的状态发生变化，Zookeeper就将**负责通知已经在Zookeeper上注册的那些观察者**做出相应的反应。

Zookeeper = 文件系统(数据结构) + 通知机制(设计模式)

### 特点
1. Zookeeper：一个领导者(Leader)，多个跟随者(Follower)组成的集群。
2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。所以Zookeeper适合安装奇数台服务器。
3. 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4. 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。
5. 数据更新原子性，一次数据更新要么成功，要么失败。
6. 实时性，在一定时间范围内，Client能读到最新数据。

### 数据结构
ZooKeeper数据模型的结构与**Unix文件系统**很类似，整体上可以看作是一棵树，每个节点称做一个ZNode。每一个 ZNode 默认能够存储 **1MB** 的数据，每个 ZNode 都可以**通过其路径唯一标识**。

![zookeeper+20210902003333](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902003333.png%2B2021-09-02-00-33-34)

节点可以分为四大类：
* PERSISTENT 持久化节点：客户端与Zookeeper断开连接后，该节点依旧存在。
* PERSISTENT_SEQUENTIAL 持久化顺序节点：`-s`，客户端与Zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
* EPHEMERAL 临时节点：`-e`，客户端与Zookeeper断开连接后，该节点被删除。
* EPHEMERAL_SEQUENTIAL 临时顺序节点：`-es`，客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

### 应用场景
提供的服务包括：
1. 统一命名服务：在分布式环境下，经常需要对应用或服务进行统一命名，便于识别
2. 统一配置管理：分布式环境下，配置文件同步非常常见
   1. 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
   2. 对配置文件修改后，希望能够快速同步到各个节点上
   3. 配置管理可交由ZooKeeper实现
      1. 可将配置信息写入ZooKeeper上的一个Znode
      2. 各个客户端服务器监听这个Znode
      3. 一旦Znode中的数据被修改，ZooKeeper将通知各个客户端服务器
3. 统一集群管理：分布式环境中，实时掌握每个节点的状态是必要的，这样可以根据节点实时状态做出一些调整。ZooKeeper可以实现实时监控节点状态变化。
   1. 可将节点信息写入ZooKeeper上的一个ZNode
   2. 监听这个ZNode可获取它的实时状态变化
4. 服务器节点动态上下线，客户端能实时洞察到服务器上下线的变化
   1. 服务端启动时去注册信息(创建都是临时节点)
   2. 客户端通过Zookeeper获取到当前在线服务器列表，并且注册监听
   3. 服务器节点下线
   4. Zookeeper触发服务器节点上下线事件通知给客户端
   5. 客户端重新再去Zookeeper获取服务器列表，并注册监听
5. 软负载均衡：在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求
6. 分布式锁
7. 注册中心



## Zookeeper程序

### 运行流程
1. 下载https://zookeeper.apache.org/
2. 修改配置（`mv zoo_sample.cfg zoo.cfg`）
3. 启动Zookeeper服务器
4. 启动Zookeeper客户端

### 配置参数
Zookeeper中的配置文件zoo.cfg中参数含义解读如下：
1. tickTime = 2000：通信心跳时间，Zookeeper服务器与客户端心跳时间，单位毫秒
2. initLimit = 10：LF初始通信时限，Leader和Follower初始连接时能容忍的最多心跳数(tickTime的数量)
3. syncLimit = 5：LF同步通信时限，Leader和Follower之间通信时间如果超过syncLimit*tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。
4. dataDir：保存Zookeeper中的数据。默认的tmp目录，容易被Linux系统定期删除，所以一般不用默认的tmp目录。
5. clientPort = 2181：客户端连接端口，通常不做修改

### 常用命令
我们可以通过zookeeper的客户端工具或者zookeeper的Api连接服务端：
![zookeeper+20210902004910](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902004910.png%2B2021-09-02-00-49-10)

#### 服务端
```sh
# 处于zookeeper的bin目录下

# 启动 ZooKeeper 服务
./zkServer.sh start

# 查看 ZooKeeper 服务状态
./zkServer.sh status

# 停止 ZooKeeper 服务
./zkServer.sh stop 

# 重启 ZooKeeper 服务
./zkServer.sh restart 
```

#### 客户端
|命令基本语法|功能描述|
|---|---|
|help|显示所有操作命令|
|ls *路径*|使用ls命令来查看当前znode的子节点(可监听)<br/>-w 监听子节点变化<br/>-s 附加次级信息|
|create *路径*|普通创建<br/>-s 含有序列<br />-e 临时(重启或者超时消失)|
|get *路径*|获得节点的值(可监听)<br/>-w 监听节点内容变化<br />-s 附加次级信息|
|set *路径* *值*|设置节点的具体值|
|stat *路径*|查看节点状态|
|delete *路径*|删除节点|
|deleteall *路径*|递归删除节点|

附加次级信息的内容：
- cZxid：节点被创建的事务ID
- ctime：创建时间(从1970年开始的毫秒数)
- mZxid：最后一次被更新的事务ID
- mtime：修改时间(从1970年开始的毫秒数)
- pZxid：子节点列表最后一次被更新的事务ID
- cversion：子节点的版本号，znode子节点修改次数
- dataVersion：数据版本号
- aclVrsion：权限版本号
- ephemeralOwner：代表临时节点拥有者的session id，持久节点则为0
- dataLength：节点存储的数据的长度 
- numChildren：当前节点的子节点个数


事务ID：
1. 每次修改ZooKeeper状态都会产生一个ZooKeeper事务ID。
2. 每次修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。
3. 事务ID是ZooKeeper中所有修改总的次序。

##### 连接和退出
```sh
# 连接zookeeper服务端, 可以不指定服务端地址，默认连接localhost:2181
./zkCli.sh
# 连接指定服务端地址
./zkCli.sh –server localhost:2181   
# 断开连接
quit
```

##### 查看目录/节点的孩子
```sh
# 显示指定目录下节点 / 代表根目录
ls /

# 查看 /zookeeper 节点下的信息
ls /zookeeper
```

##### 查看节点详情
```sh
# 查看 /zookeeper 节点的详细数据
ls -s /zookeeper
```

##### 创建节点
```sh
# 在根节点下创建app1节点, 并设置值为a1
create /app1 a1

# 在根节点下创建app2节点, 且不设置值
create /app2 "" # 3.5版本后不需要 ""

# 创建子节点
create /app2/app2-2 ""

# 创建临时节点 /app1，如果断开连接，该节点就没了
create -e /app1 a1 

# 创建顺序节点, 会在app1后面加一堆的数字，方便进行排序
create -s /app1 a1
create -s /app1 a1
create -s /app1 a1
```
顺序结点：
1. 创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护
2. 在分布式系统中，顺序号可以被用于为所有的事件进行全局排序，这样客户端可以通过顺序号推断事件的顺序

##### 获取节点值
```sh
# 获取/app1节点的值
get /app1
```

##### 设置节点值
```sh
# 相当于是修改/app1节点的值为itheima2
set /app1 p1
```

##### 删除节点
```sh
# 删除单个节点，如果下面有子节点就删除不了
delete /app1

# 删除带有子节点的节点，有些版本是rmr
deleteall /app2
```



## 监听器
说明：
* ZooKeeper 允许客户端在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper 服务端会将事件通知到感兴趣的客户端上去，该机制是 ZooKeeper 实现分布式协调服务的重要特性。
* ZooKeeper 中引入了Watcher机制来实现了发布/订阅功能能，能够让多个订阅者同时监听某一个对象，当一个对象自身状态变化时，会通知所有订阅者。
* ZooKeeper提供了三种Watcher：
  * NodeCache：只是监听某一个特定的节点
  * PathChildrenCache：监控一个ZNode的子节点
  * TreeCache：可以监控整个树上的所有节点，类似于PathChildrenCache和NodeCache的组合
![zookeeper+20210902011101](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902011101.png%2B2021-09-02-01-11-02)




## JavaAPI - Curator
Curator 是Apache ZooKeeper 的Java客户端库，可以更容易的使用ZooKeeper，包含几个包：
- Curator client：用来替代ZooKeeper提供的类，它封装了底层的管理并提供了一些有用的工具。
- Curator framework：提供了高级的API来简化ZooKeeper的使用，增加了很多基于ZooKeeper的特性，帮助管理ZooKeeper的连接以及重试操作。
- Curator Recipes：封装了一些高级特性，如：Cache事件监听、选举、分布式锁、分布式计数器、分布式Barrier等
- Curator Test：提供了基于ZooKeeper的单元测试工具。

### demo
#### 依赖
```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-client</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>
```

#### 测试类
```java
import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;
import org.junit.Test;

/**
 * TODO
 *
 * @Author LK
 * @Date 2021/3/30
 */
public class CuratorTest {

    private CuratorFramework client;
    /**
     * 测试连接
     * namespace的作用是指定根目录为/itheima，主要是为了简化路径写法，后面操作节点是在/itheima这个根目录下操作；
     */
    @Test
    public void testConnection(){
        // 1.创建重试策略对象，指定重试策略，参数1-重试间隔时间（单位：毫秒），参数2-重试次数
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(3000, 2);

        // 2.创建连接客户端对象，
        client = CuratorFrameworkFactory.builder()
                .connectString("127.0.0.1:2181")
                .retryPolicy(retryPolicy) 
                .namespace("itheima") 
                .build();

        // 3.启动，建立连接
        client.start();
    }

    /**
     * 基本创建
     */
    @Test
    public void testCreate1() throws Exception{
        // 创建节点时不设置数据，默认将客户端所在服务器的ip地址作为数据。
        String path = client.create().forPath("/app1");
        System.out.println(path);
    }

    /**
     * 创建节点，有数据
     */
    @Test
    public void testCreate2() throws Exception{
        String path = client.create().forPath("/app2", "hehe".getBytes());
        System.out.println(path);
    }

    /**
     * 创建节点，指定节点类型
     */
    @Test
    public void testCreate3() throws Exception{
        // PERSISTENT 持久化节点
        // PERSISTENT_SEQUENTIAL 持久化顺序节点
        // EPHEMERAL 临时节点
        // EPHEMERAL_SEQUENTIAL 临时数据节点
        String path = client.create().withMode(CreateMode.EPHEMERAL).forPath("/app3");
        System.out.println(path);
    }
    
    /**
     * 创建多级节点
     */
    @Test
    public void testCreate4() throws Exception{
        // creatingParentsIfNeeded 如果父节点不存在则创建父节点
        String path = client.create().creatingParentsIfNeeded().forPath("/app4/app4-1");
        System.out.println(path);
    }	


    /**
    * 查询节点数据
    查询节点数据：get /itheima/app2
    */
    @Test
    public void testGet1() throws Exception{
        // 相当于get /itheima/app2
        byte[] bytes = client.getData().forPath("/app2");
        System.out.println("结果 = " + bytes.toString());
    }

    /**
    * 查询目录
    查询目录节点：ls /itheima/app4
    */
    @Test
    public void testGet2() throws Exception{
        // 相当于ls /itheima/app4，返回结果是所有子节点的名称
        List<String> children = client.getChildren().forPath("/app4");

        for (String child : children) {
            System.out.println(child);
        }
    }

    /**
    * 查询节点详情
    查询节点状态信息：ls -s
    */
    @Test
    public void testGet3() throws Exception{

        Stat status = new Stat();
        // 将 ls -s /app1 的结果数据放到status中
        client.getData().storingStatIn(status).forPath("/app1");

        // 打印节点创建时间
        System.out.println(status.getCtime());

        // 打印子节点个数
        System.out.println(status.getNumChildren());
    }
    

    /**
    * 设置数据：set /app1 goodboy
    */
    @Test
    public void testSetData() throws Exception{
        client.setData().forPath("/app1", "goodBoy".getBytes());
    }

    /**
    * 根据版本设置数据
    */
    @Test
    public void testSetData2() throws Exception{

        Stat status = new Stat();
        // 将 ls -s /app1 的结果数据放到status中
        client.getData().storingStatIn(status).forPath("/app1");

        // 获取数据当前的版本号
        int version = status.getVersion();

        client.setData().withVersion(version).forPath("/app1", "goodBoy".getBytes());
    }


    /**
    * 删除节点 delete deleteall
    * 1、删除单个节点
    * 2、删除节点及其子节点
    */
    @Test
    public void testDelete() throws Exception {
        client.delete().forPath("/app1");
    }

    @Test
    public void testDeleteAll() throws Exception {
        client.delete().deletingChildrenIfNeeded().forPath("/app2");
    }


    /**
    * 监控某个节点
    */
    @Test
    public void NodeCache() throws Exception{
        
        // 1. 创建监听器对象
        final NodeCache nodeCache = new NodeCache(client, "/app1");

        // 2. 绑定监听器
        nodeCache.getListenable().addListener(new NodeCacheListener() {

            // 当/app1节点发生改变，就会回调nodeChanged
            public void nodeChanged() throws Exception {
                System.out.println("节点改变了...");

                // 获取改变后的数据
                ChildData currentData = nodeCache.getCurrentData();

                byte[] data = currentData.getData();
                
                // 打印数据
                System.out.println(new String(data));
            }
        });
        // 3. 开启监听，如果参数为true开启监听时，如果设置为true，那么NodeCache在第一次启动的时候就会立刻在Zookeeper上读
        // 取对应节点的数据内容，并保存在Cache中。
        nodeCache.start(true);

        // 不让线程结束
        while(true){

        }
    }

    /**
    * PathChildrenCache：监控某一个节点的子节点的变化
    */
    @Test
    public void pathChildrenCache() throws Exception {
        
        // 1. 创建监听器对象，参数3-是否加载缓存数据
        PathChildrenCache pathChildrenCache = new PathChildrenCache(client, "/app2", true);
        // 2. 绑定监听器
        pathChildrenCache.getListenable().addListener(new PathChildrenCacheListener() {
            
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event)
                throws Exception {
                // event={type=CHILD_ADDED, data=ChildData{path='/app2/p2', stat=..,data=null}}
                // event={type=CHILD_UPDATED, data=ChildData{path='/app2/p2', stat=..,data=null}}
                // event={type=CHILD_REMOVED, data=ChildData{path='/app2/p2', stat=..,data=null}}
                System.out.println("子节点变化了，" + event);
                // 获取type类型
                PathChildrenCacheEvent.Type type = event.getType();
                if (type.equals(PathChildrenCacheEvent.Type.CHILD_UPDATED)){
                    byte[] data = event.getData().getData();
                    System.out.println("获取变化后数据：" + new String(data));
                }
            }
        });
        // 3. 开启监听
        pathChildrenCache.start();

        while (true){
            
        }
    }

    /**
    * TreeCache : 可以监控整个树上的所有节点，相当于NodeCache+PathChildrenCache组合
    */
    @Test
    public void treeCache() throws Exception {
        // 1. 创建监听器对象
        TreeCache treeCache = new TreeCache(client, "/app3");
        // 2. 绑定监听器
        treeCache.getListenable().addListener(new TreeCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, TreeCacheEvent event) throws Exception {
                System.out.println("子节点变化了，" + event);
                
                // 获取type类型
                TreeCacheEvent.Type type = event.getType();
                // 监听修改节点的事件，获取修改后的值（不管自己还是儿子发生变化都监控到）
                if (type.equals(TreeCacheEvent.Type.NODE_UPDATED)){
                    byte[] data = event.getData().getData();
                    System.out.println("获取变化后数据：" + new String(data));
                }
            }
        });
        // 3. 开启监听
        treeCache.start();

        while (true){
            
        }
    }


    @After
    public void close() {
        if (client != null) {
            client.close();
        }
    }
}
```


## 集群部署

### 部署流程
1. 集群规划：在 hadoop102、hadoop103 和 hadoop104 三个节点上都部署 Zookeeper。
2. 解压安装
3. 配置服务器编号：在数据目录（zoo.cfg指定的dataDir目录）下创建一个myid文件，在文件中添加与 server 对应的编号。hadoop102服务器对应的编号为2，hadoop103对应的编号为3，hadoop104服务器对应的编号为4。
4. 配置zoo.cfg文件：修改数据存储路径配置（zoo.cfg指定的dataDir目录）。增加如下配置
```
#######################cluster##########################
server.2=hadoop102:2888:3888
server.3=hadoop103:2888:3888
server.4=hadoop104:2888:3888
```

#### 配置参数解读
```
server.A=B:C:D
```
A 是一个数字，表示这个是第几号服务器;  
集群模式下配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值，Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是哪个 server。

B 是这个服务器的地址;

C 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口;

D 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的
Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

### 选举机制
假设有五台服务器。

![zookeeper+20210902091037](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902091037.png%2B2021-09-02-09-10-38)

![zookeeper+20210902091150](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902091150.png%2B2021-09-02-09-11-51)

#### 第一次启动
1. 服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上(3票)，选举无法完成，服务器1状态保持为LOOKING
2. 服务器2启动，再发起一次选举。服务器1和2分别投自己一票并交换选票信息:此时服务器1发现服务器2的myid比自己目前投票推举的(服务器1)
大，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1，2状态保持LOOKING 
3. 服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果:服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING
4. 服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果:服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING;
5. 服务器5启动，同4一样当小弟。

![zookeeper+20210902090136](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902090136.png%2B2021-09-02-09-01-38)

第一次启动选举规则:  
投票过半数时，服务器 id 大的胜出

#### 非第一次启动
1. 当ZooKeeper集群中的一台服务器出现以下两种情况之一时，就会开始进入Leader选举:
   1. 服务器初始化启动。
   2. 服务器运行期间无法和Leader保持连接。
2. 而当一台机器进入Leader选举流程时，当前集群也可能会处于以下两种状态:
   1. 集群中本来就已经存在一个Leader。
   对于第一种已经存在Leader的情况，机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器来说，仅仅需要和Leader机器建立连 接，并进行状态同步即可。
   2. 集群中确实不存在Leader。
   假设ZooKeeper由5台服务器组成，SID分别为1、2、3、4、5，ZXID分别为8、8、8、7、7，并且此时SID为3的服务器是Leader。某一时刻，3和5服务器出现故障，因此开始进行Leader选举。

![zookeeper+20210902090705](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902090705.png%2B2021-09-02-09-07-07)

|                            | (EPOCH，ZXID，SID) | (EPOCH，ZXID，SID) | (EPOCH，ZXID，SID) |
| -------------------------- | ------------------ | ------------------ | ------------------ |
| SID为1、2、4的机器投票情况 | (1,8,1)            | (1,8,2)            | (1,7,4)            |

第2～n次启动选举规则:
1. EPOCH大的直接胜出
2. EPOCH相同，事务id大的胜出
3. 事务id相同，服务器id大的胜出

### 客户端向服务端写数据流程

#### 写流程之写入请求直接发送给Leader节点
![zookeeper+20210902090859](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902090859.png%2B2021-09-02-09-09-02)

#### 写流程之写入请求发送给follower节点
![zookeeper+20210902090931](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902090931.png%2B2021-09-02-09-09-33)


## Zookeeper应用——分布式锁
分布式锁：一种更加高级的锁机制，能处理种跨机器的进程之间的数据同步问题，保证了分布式系统中多个进程能够有序的访问该临界资源。

### 原理
![zookeeper+20210902101939](https://raw.githubusercontent.com/loli0con/picgo/master/images/zookeeper%2B20210902101939.png%2B2021-09-02-10-19-42)

#### 核心思想
当客户端要获取锁，则创建节点，使用完锁，则删除该节点。

#### 过程描述
1. 客户端准备获取锁时，在lock节点下创建临时顺序节点。
2. 然后获取lock下面的所有子节点，如果发现自己创建的子节点序号最小，那么就认为该客户端获取到了锁。使用完锁后，将该节点删除。
3. 如果发现自己创建的节点并非lock所有子节点中最小的，说明自己还没有获取到锁，此时客户端需要找到比自己小的那个节点，同时对其注册事件监听器，监听删除事件。
4. 如果发现比自己小的那个节点被删除，则客户端的Watcher会收到相应通知，此时再次判断自己创建的节点是否是lock子节点中序号最小的，如果是则获取到了锁，如果不是则重复以上步骤继续获取到比自己小的一个节点并注册监听。

### 常用锁
- InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）
- InterProcessMutex：分布式可重入排它锁
- InterProcessReadWriteLock：分布式读写锁（读读共享，读写互斥，写写互斥）
- InterProcessMultiLock：对多个对象加锁
- InterProcessSemaphoreV2：共享信号量
# 缓存
MyBatis 内置了一个强大的事务性查询缓存机制，默认情况下只启用了本地的会话缓存，它仅仅对一个会话中的数据进行缓存。 




## 一级缓存
一级缓存是SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问。




## 二级缓存
二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存。二级缓存基于namespace级别的缓存，一个名称空间，对应一个二级缓存。

二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被
缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取。

### 开启
二级缓存开启的条件:
1. 在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
2. 在映射文件中设置标签`<cache />`
3. 二级缓存必须在SqlSession关闭或提交之后有效
4. 查询的数据所转换的实体类类型必须实现序列化的接口

### 配置
在mapper配置文件中添加的cache标签可以设置一些属性:
* eviction属性:缓存回收策略。默认的是 LRU。
  * LRU(Least Recently Used) – 最近最少使用的:移除最长时间不被使用的对象。
  * FIFO(First in First out) – 先进先出:按对象进入缓存的顺序来移除它们。
  * SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
  * WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。
  * flushInterval属性:刷新间隔，单位毫秒。默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新
* size属性:引用数目，正整数。代表缓存最多可以存储多少个对象，太大容易导致内存溢出
* readOnly属性:只读，true/false。默认是 false。
  * true:只读缓存;会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了 很重要的性能优势。
  * false:读写缓存;会返回缓存对象的拷贝(通过序列化)。这会慢一些，但是安全，因此默认是 false。 

#### demo
```xml
<!-- 这个更高级的配置创建了一个 FIFO 缓存，每隔 60 秒刷新，最多可以存储结果对象或列表的 512 个引用，而且返回的对象被认为是只读的，因此对它们进行修改可能会在不同线程中的调用者产生冲突。 -->
<cache
 eviction="FIFO"
 flushInterval="60000"
 size="512"
 readOnly="true"/>
```

### 命名空间
缓存的配置和缓存实例会被绑定到 SQL 映射文件的命名空间中，同一命名空间中的所有语句和缓存将通过命名空间绑定在一起。每条语句可以自定义与缓存交互的方式，或将它们完全排除于缓存之外，这可以通过在每条语句上使用两个简单属性来达成。默认情况下，语句会这样来配置：
```xml
<select ... flushCache="false" useCache="true"/>
<insert ... flushCache="true"/>
<update ... flushCache="true"/>
<delete ... flushCache="true"/>
```
如果想改变默认的行为，只需要设置 flushCache 和 useCache 属性。

#### cache-ref
想要在多个命名空间中共享相同的缓存配置和实例，可以使用 cache-ref 元素来引用另一个缓存。
```xml
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```

### 自定义
除了上述自定义二级缓存的方式，你也可以通过实现你自己的缓存，或为其他第三方缓存方案创建适配器，来完全覆盖二级缓存的行为。

```xml
<cache type="com.domain.something.MyCustomCache"/>
```

#### 接口
自定义的缓存实现类必须实现 org.apache.ibatis.cache.Cache 接口，且提供一个接受 String 参数作为 id 的构造器。
```Java
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

#### 配置
为了对缓存进行配置，只需要简单地在缓存实现中添加公有的 JavaBean 属性，然后通过 cache 元素传递属性值，例如，下面的例子将在你的缓存实现上调用一个名为 setCacheFile(String file) 的方法：
```xml
<cache type="com.domain.something.MyCustomCache">
  <!-- 可以使用所有简单类型作为 JavaBean 属性的类型，MyBatis 会进行转换 -->
  <!-- setCacheFile(String file) -->
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>

  <!-- 也可以使用占位符（如 ${cache.file2}），以便替换成在配置文件属性中定义的值。 -->
  <!-- setCacheFile2(String file2) -->
  <property name="cacheFile2" value="${cache.file2}"/>
</cache>
```

#### 初始化
MyBatis 已经支持在所有属性设置完毕之后，调用一个初始化方法。 如果想要使用这个特性，请在自定义缓存类里实现 org.apache.ibatis.builder.InitializingObject 接口。
```java
public interface InitializingObject {
  void initialize() throws Exception;
}
```



## 缓存查询的顺序
1. 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。
2. 如果二级缓存没有命中，再查询一级缓存
3. 如果一级缓存也没有命中，则查询数据库
4.  SqlSession关闭之后，一级缓存中的数据会写入二级缓存



## 缓存原理图
![cache+20220822173703](https://raw.githubusercontent.com/loli0con/picgo/master/images/cache%2B20220822173703.png%2B2022-08-22-17-37-04)



## EhCache
Ehcache是一种广泛使用的java分布式缓存，用于通用缓存。

### pom
```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
   <groupId>org.mybatis.caches</groupId>
   <artifactId>mybatis-ehcache</artifactId>
   <version>1.1.0</version>
</dependency>
```

### 配置
如果在加载时未找到/ehcache.xml资源或出现问题，则将使用默认配置。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
        updateCheck="false">
   <!--
      diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
      user.home – 用户主目录
      user.dir – 用户当前工作目录
      java.io.tmpdir – 默认临时文件路径
    -->
   <diskStore path="./tmpdir/Tmp_EhCache"/>
   
   <!--
      defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
    -->
   <defaultCache
           eternal="false"
           maxElementsInMemory="10000"
           overflowToDisk="false"
           diskPersistent="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="259200"
           memoryStoreEvictionPolicy="LRU"/>

   <cache
           name="cloud_user"
           eternal="false"
           maxElementsInMemory="5000"
           overflowToDisk="false"
           diskPersistent="false"
           timeToIdleSeconds="1800"
           timeToLiveSeconds="1800"
           memoryStoreEvictionPolicy="LRU"/>
   <!--
     name：缓存名称。
     maxElementsInMemory：在内存中缓存的element的最大数目。
     maxElementsOnDisk：在磁盘上缓存的element的最大数目，若是0表示无穷大。
     eternal：设定缓存的elements是否永远不过期。如果为 true，则缓存的数据始终有效（），如果为false那么还要根据timeToIdleSeconds、timeToLiveSeconds判断。
     overflowToDisk：设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上。
     timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
     timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0，也就是对象存活时间无穷大。
     diskPersistent：在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
     diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
     diskExpiryThreadIntervalSeconds：磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作。
     clearOnFlush：内存数量最大时是否清除。
     memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。可以设置为FIFO（先进先出）或是LFU（较少使用）。
        FIFO，first in first out，这个是大家最熟的，先进先出。
        LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
        LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
  -->
</ehcache>
```

### 实例
```xml
<mapper namespace = “com.domain.FooMapper” > 
   <cache type = “org.mybatis.caches.ehcache.EhcacheCache” /> 
</mapper>
```
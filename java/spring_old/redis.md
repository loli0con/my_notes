# Spring + redis

## 相关类
|类名|介绍|
|---|---|
|RedisStandaloneConfiguration|Redis的独立配置类，只需指定主机名和端口号即可使用|
|JedisConnectionFactory|Jedis的连接工厂类，用于获取Jedis的连接对象，使用的时候注入上面的配置类|
|RedisTemplate|Redis模板对象，用于简化Java访问Redis的操作，无需我们创建和关闭连接，需要注入上面的连接工厂对象|
|StringRedisSerializer|String的序列化策略，用于将值序列化成字符串类型|
|GenericJackson2JsonRedisSerializer|JSON的序列化策略，用于对象与JSON字符串之间的转换|
|JdkSerializationRedisSerializer|默认的序列化策略：JDK的序列化策略，以字节的方式进行序列化和反序列化，用于无需类型转换的场景|


## RedisTemplate

### 方法
|需要操作的类型|应该使用的方法|方法返回的对象|
|---|---|---|
|string|opsForValue()|ValueOperations|
|hash|opsForHash()|HashOperations|
|list|opsForList()|ListOperations|
|set|opsForSet()|SetOperations|
|zset|opsForZSet()|ZSetOperations|


### 方法返回的对象

#### 通用方法
|方法名|说明|
|---|---|
|Boolean delete(K key)|删除指定的键|
|Boolean hasKey(K key)|判断指定的键是否存在|
|Boolean expire(K key, long timeout, TimeUnit unit)|设置指定的键过期的时间<br />TimeUnit用于指定时间的单位：毫秒，秒，小时等|
|Long getExpire(K key)|获取键过期的时间|

#### ValueOperations\<K, V>类的方法
|方法名|说明|
|---|---|
|void set(K key, V value)|添加键和值|
|void set(K key, V value, long timeout, TimeUnit unit)|添加键和值，并且指定过期时间|
|Boolean setIfAbsent(K key, V value)|如果键不存在才添加|
|V get(Object key)|获取指定的键|


## 示例
```java
public void test01() {
    //测试1：简单类型的字符串
    redisTemplate.opsForValue().set("name","Jack");
    System.out.println("获取数据："+redisTemplate.opsForValue().get("name"));

    // 测试2: 测试value是java对象,并设置过期时间
    User user = new User(100,"Rose");
    redisTemplate.opsForValue().set("user",user, Duration.ofMinutes(1));
    System.out.println("获取数据："+redisTemplate.opsForValue().get("user"));
}
```

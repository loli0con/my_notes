# Properties
Java集合库提供了一个Properties来表示和处理配置文件。Properties内部本质上是一个Hashtable，注意仅使用getProperty()和setProperty()方法，不要调用继承而来的get()和put()等方法。

## 读取配置文件
用Properties读取配置文件，一共有三步：
1. 创建Properties实例；
2. 调用load()读取文件；
3. 调用getProperty()获取配置。

如果有多个.properties文件，可以反复调用load()读取，后读取的key-value会覆盖已读取的key-value。

需要注意，load(InputStream)默认总是以ASCII编码读取字节流，可能会导致读到乱码。

## 写入配置文件
如果通过setProperty()修改了Properties实例，可以使用store()方法把配置写入文件，以便下次启动时获得最新配置。
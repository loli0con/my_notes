# 日期和时间

### 表示  
python中的日期和时间，可以用以下三种最基本的形式来表示：
- 时间&日期对象(object，标准库支持)
    - datetime
    - date
    - time
    - time tuple
- 时间戳(int，timestamp)
- 字符串(string，遵循ISO 8061标准)

### 时间&日期对象
> 日期和时间对象可以根据它们是否包含时区信息而分为“[感知型](https://docs.python.org/zh-cn/3/library/datetime.html#aware-and-naive-objects)”和“[简单型](https://docs.python.org/zh-cn/3/library/datetime.html#aware-and-naive-objects)”两类。

感知型的对象包含了“**时间调整信息**”，例如时区信息、夏令时等。

举一个例子：库克向你发了一条邮件，邀请你观看苹果的新品发布会，时间是~~2020年9月15日上午10点~~。
聪明的你很快就会有疑问，这个时间到底应该怎么算，到底是美国的时间，还是中国的时间。
而且美国本土横跨西五区至西八区，共四个时区，每个时区对应一个标准时间。
如果邮件里仅仅只有简简单单的时间信息，没有时区信息，恐怕即使是美国本土居民也会和你一样蒙圈。
当然苹果公司肯定不会犯这样的错误，所以你收到的信息必定是<u>美国西部时间2020年9月15日上午10点</u>。
即**北京时间是9月16日凌晨1点**，由于这个对象中存在了时区信息，所以全世界的果粉都能知道发布会的正确时间。

##### datetime模块
- 时区[timezone](https://docs.python.org/zh-cn/3/library/datetime.html#datetime.timezone)
- 日期时间[datetime](https://docs.python.org/zh-cn/3/library/datetime.html#datetime.timezone)
- 日期[date](https://docs.python.org/zh-cn/3/library/datetime.html#datetime.date)
- 时间[time](https://docs.python.org/zh-cn/3/library/datetime.html#datetime.time)
- 时间间隔[timedelta](https://docs.python.org/zh-cn/3/library/datetime.html#datetime.timedelta)
##### time模块
- 时间元组[struct_time](https://docs.python.org/zh-cn/3/library/time.html#time.struct_time)
- Unix时间[time](https://docs.python.org/zh-cn/3/library/time.html#time.time)

# 写不下去了，拖更！
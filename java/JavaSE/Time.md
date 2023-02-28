# 时间
有关时间的一切。


## 时间和日期
时间有两种概念，一种是不带日期的时间，例如，12:30:59。另一种是带日期的时间，例如，2020-1-1 20:21:59，只有这种带日期的时间能唯一确定某个时刻，不带日期的时间是无法确定一个唯一时刻的。

## 本地时间
当我们说当前时刻是2019年11月20日早上8:15的时候，我们说的实际上是本地时间。在国内就是北京时间。不同的时区，在同一时刻，本地时间是不同的。全球一共分为24个时区，伦敦所在的时区称为标准时区，其他时区按东／西偏移的小时区分，北京所在的时区是东八区。

## 时区
因为光靠本地时间还无法唯一确定一个准确的时刻，所以我们还需要给本地时间加上一个时区。时区有好几种表示方式。

一种是以GMT或者UTC加时区偏移表示，例如：GMT+08:00或者UTC+08:00表示东八区。

GMT和UTC可以认为基本是等价的，只是UTC使用更精确的原子钟计时，每隔几年会有一个闰秒，我们在开发程序的时候可以忽略两者的误差，因为计算机的时钟在联网的时候会自动与时间服务器同步时间。

另一种是缩写，例如，CST表示China Standard Time，也就是中国标准时间。但是CST也可以表示美国中部时间Central Standard Time USA，因此，缩写容易产生混淆，我们尽量不要使用缩写。

最后一种是以洲／城市表示，例如，Asia/Shanghai，表示上海所在地的时区。特别注意城市名称不是任意的城市，而是由国际标准组织规定的城市。

因为时区的存在，东八区的2019年11月20日早上8:15，和西五区的2019年11月19日晚上19:15，他们的时刻是相同的。时刻相同的意思就是，分别在两个时区的两个人，如果在这一刻通电话，他们各自报出自己手表上的时间，虽然本地时间是不同的，但是这两个时间表示的时刻是相同的。

## 夏令时
所谓夏令时，就是夏天开始的时候，把时间往后拨1小时，夏天结束的时候，再把时间往前拨1小时。

因为涉及到夏令时，相同的时区，如果表示的方式不同，转换出的时间是不同的。

实行夏令时的不同地区，进入和退出夏令时的时间很可能是不同的。同一个地区，根据历史上是否实行过夏令时，标准时间在不同年份换算成当地时间也是不同的。因此，计算夏令时，没有统一的公式，必须按照一组给定的规则来算，并且，该规则要定期更新。计算夏令时请使用标准库提供的相关类，不要试图自己计算夏令时。

## 本地化
在计算机中，通常使用Locale表示一个国家或地区的日期、时间、数字、货币等格式。Locale由语言_国家的字母缩写构成，例如，zh_CN表示中文+中国，en_US表示英文+美国。语言使用小写，国家使用大写。

对于日期来说，不同的Locale，例如，中国和美国的表示方式如下：
* zh_CN：2016-11-30
* en_US：11/30/2016

计算机用Locale在日期、时间、货币和字符串之间进行转换。一个电商网站会根据用户所在的Locale对用户显示如下：
||中国用户|美国用户|
|---|---|---|
|购买价格|12000.00|12,000.00|
|购买日期|2016-11-30|11/30/2016|


## Epoch Time
“同一个时刻”在计算机中存储的本质上只是一个整数，我们称它为Epoch Time。Epoch Time是计算从1970年1月1日零点（格林威治时区／GMT+00:00）到现在所经历的秒数。

例如：1574208900表示从1970年1月1日零点GMT时区到该时刻一共经历了1574208900秒，换算成伦敦、北京和纽约时间分别是：
* 伦敦时间2019-11-20 0:15:00
* 北京时间2019-11-20 8:15:00
* 纽约时间2019-11-19 19:15:00

因此，在计算机中，只需要存储一个整数1574208900表示某一时刻。当需要显示为某一地区的当地时间时，我们就把它格式化为一个字符串。

在Java程序中，时间戳通常是用long表示的毫秒数。要获取当前时间戳，可以使用System.currentTimeMillis()，这是Java程序获取时间戳最常用的方法。

## 标准库
Java标准库有两套处理日期和时间的API：
* 一套定义在java.util这个包里面，主要包括Date、Calendar和TimeZone这几个类
  * Date: 日期时间的表示与比较，不携带时区信息
  * Calendar: 除了日期时间的表示和比较，还可以精确设定日期时间，同时携带时区信息
  * TimeZone: 时区的表示（即时区信息）
  * Locale: 地区的表示（用于展示符合地区文化的文本）
  * SimpleDateFormat: 字符串和时间的相互转化的工具
* 一套新的API是在Java 8引入的，定义在java.time这个包里面，主要包括LocalDateTime、ZonedDateTime、ZoneId等
  * LocalDateTime: 不带时区信息的日期时间
  * ZonedDateTime: 带时区信息的日期时间
  * ZoneId: 时区的表示（即时区信息）
  * Instant: epoch time的表示
  * DateTimeFormatter: 字符串和时间的相互转化的工具


### 旧API 转 新API
如果要把旧式的Date或Calendar转换为新API对象，可以通过toInstant()方法转换为Instant对象，再继续转换为ZonedDateTime：
```Java
// Date(不带时区信息) -> Instant:
Instant ins1 = new Date().toInstant();

// Calendar(带时区信息) -> Instant -> ZonedDateTime(带时区信息):
Calendar calendar = Calendar.getInstance();
Instant ins2 = calendar.toInstant();
// TimeZone -> ZoneId, 注入时区信息
ZonedDateTime zdt = ins2.atZone(calendar.getTimeZone().toZoneId());
```

旧的TimeZone提供了一个toZoneId()方法，可以把自己变成新的ZoneId。

### 新API 转 旧API
如果要把新的ZonedDateTime转换为旧的API对象，只能借助long型时间戳做一个“中转”：
```Java
// ZonedDateTime -> long:
ZonedDateTime zdt = ZonedDateTime.now();
long ts = zdt.toEpochSecond() * 1000;

// long -> Date:
Date date = new Date(ts);

// long -> Calendar:
Calendar calendar = Calendar.getInstance();
calendar.clear();
// ZoneId -> TimeZone
calendar.setTimeZone(TimeZone.getTimeZone(zdt.getZone().getId()));
/*
ZoneId zoneId = zdt.getZone();
String id = zondId.getId();
TimeZone timeZone = TimeZone.getTimeZone(id);
*/
calendar.setTimeInMillis(zdt.toEpochSecond() * 1000); 
```

新的ZoneId转换为旧的TimeZone，需要借助ZoneId.getId()返回的String完成。


## 数据库
在数据库中，也存在几种日期和时间类型：
* DATETIME：表示日期和时间；
* DATE：仅表示日期；
* TIME：仅表示时间；
* TIMESTAMP：和DATETIME类似，但是数据库会在创建或者更新记录的时候同时修改TIMESTAMP。

在使用Java程序操作数据库时，需要把数据库类型与Java类型映射起来。下表是数据库类型与Java新旧API的映射关系：
|数据库|对应Java类（旧）|对应Java类（新）|
|---|---|---|
|DATETIME|java.util.Date|LocalDateTime|
|DATE|java.sql.Date|LocalDate|
|TIME|java.sql.Time|LocalTime|
|TIMESTAMP|java.sql.Timestamp|LocalDateTime|

java.sql.Date和java.sql.Time都继承自java.util.Date，java.sql.Date会自动忽略所有时间相关信息，java.sql.Time会自动忽略所有日期相关信息。

实际上，在数据库中，需要存储的最常用的是时刻（`Instant`），因为有了时刻信息，就可以根据用户自己选择的时区，显示出正确的本地时间。所以，最好的方法是直接用长整数long表示，在数据库中存储为BIGINT类型。
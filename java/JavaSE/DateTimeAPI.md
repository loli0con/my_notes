# 时间日期API

## todo
UTC和GMT
时区
夏令时

## 时间线

### 时间点
在Java中，Instant表示时间线上的某个点。被称为“新纪元”的**时间线原点**被设置为穿过伦敦格林尼治皇家天文台的本初子午线所处时区的1970年1月1日的午夜。这与UNIX/POSIX时间中使用的惯例相同。从该原点开始，时间按照每天86400秒向前或向回度量，精确到纳秒。

Instant.MIN的值往回可追溯10亿年，Instant.MAX的值是公元1000000000年的12月31日。

静态方法调用Instant.now()会给出当前的时刻。你可以按照常用的方式，用equals和compareTo方法来比较两个Instant对象，因此你可以将Instant对象用作时间戳。

### 时间量
Duration是两个时刻之间的时间量。为了得到两个时刻之间的**时间差**，可以使用静态方法Duration.between。例如，下面的代码展示了如何度量算法的运行时间：
```Java
Instant start = Instant.now(); // 开始时间
runAlgorithm();  // 运行算法
Instant end = Instant.now(); // 结束时间
Duration timeElapsed = Duration.between(start, end); // 时间差
long millis = timeElapsed.toMillis(); // 换算成毫秒
```

Instant和Duration类都是不可修改的类，所以诸如multipliedBy和minus这样的方法都会返回一个新的实例。

### 相关API
java.time.Instant：
* static Instant now()：从最佳的可用系统时钟中获取当前的时刻。
* Instant plus(TemporalAmount amountToAdd)
* Instant minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration和Period实现了TemporalAmount接口。
* Instant (plus|minus)(Nanos|Millis|Seconds)(long number)：产生一个时刻，该时刻与当前时刻距离给定数量的纳秒、微秒或秒。

java.time.Duration：
* static Duration of(Nanos|Millis|Seconds|Minutes|Hours|Days)(long number)：产生一个给定数量的指定时间单位的时间间隔。
* static Duration between(Temporal startlnclusive, Temporal endExclusive)：产生一个在给定时间点之间的Duration对象。Instant类实现了Temporal接口，LocalDate/LocalDateTime/LocalTime和ZonedDateTime也实现了该接口。
* long toNanos()
* long toMillis()
* long toSeconds()
* long toMinutes()
* long toHours()
* long toSeconds()
* long toSeconds()
* long toDays()：获取当前时长按照方法名中的时间单位度量的数量。
* int toDaysPart()
* long to(Nanos|Millis|Seconds|Minutes|Hours)Part()：当前时长中给定时间单位的部分。例如，在100秒的时间间隔中，分钟的部分是1，秒的部分是40。
* Instant plus(TemporalAmount amountToAdd)
* Instant minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration和Period类实现了TemporalAmount接口。
* Duration multipliedBy(long multiplicand)
* Duration dividedBy(long divisor)
* Duration negated()：产生一个时长，该时长是通过当前时刻乘以或除以给定的量或-1得到的。
* boolean isZero()
* boolean isNegative()：如果当前Duration对象是0或负数，则返回true。
* Duration (plus|minus)(Nanos|Millis|Seconds|Minutes|Hours|Days)(long number)：产生一个时长，该时长是通过当前时刻加上或减去给定的数量的指定时间单位而得到的。

## 本地日期
在Java API中有两种人类时间，**本地日期/时间**和**时区时间**。**本地日期/时间**包含日期和当天的时间，但是它与时区信息没有任何关联，它并不对应精确的时刻。

LocalDate是带有年、月、日的日期。为了构建LocalDate对象，可以使用now或of静态方法：
```Java
LocalDate today = LocalDate.now(); // Today's date
LocalDate alonzosBirthday = LocalDate.of(1903, 6, 14);
alonzosBirthday = LocalDate.of(1903, Month.JUNE, 14); // Uses the Month enumeration
```

两个Instant之间的时长是Duration，而用于本地日期的等价物是Period，它表示的是流逝的年、月或日的数量。可以调用birthday.plus(Period.ofYears(1))来获取下一年的生日。当然，也可以直接调用birthday.plusYears(1)。但是birthday.plus(Duration.ofDays(365))在闰年是不会产生正确结果的。

util方法会产生两个本地日期之间的时长。但是该方法返回的Period结果并不是很有用，因为每个月的天数不尽相同。为了确定到底有多少天，可以使用：`indepndenceDay.until(Christmas, ChronoUnit.DAYS)`。

LocalDate API中的有些方法可能会创建出并不存在的日期。例如，在1月31日上加上1个月不应该产生2月31日。这些方法并不会抛出异常，而是会返回该月有效的最后一天。例如，`LocalDate.of(2016, 1, 31).plusMonths(1)`和`LocalDate.of(2016, 3, 31).minusMonths(1)`都将产生2016年2月29日。

getDayOfWeek会产生星期日期，即DayOfWeek枚举的某个值。DayOfWeek.MONDAY的枚举值为1，而DayOfWeek.SUNDAY的枚举值为7。DayOfWeek枚举具有便捷方法plus和minus，以7为模计算星期日期。

datesUntil方法可以产生LocalDate对象流：
```Java
LocalDate start = LocalDate.of(2000, 1, 1);
LocalDate endExclusive = LocalDate.now();
Stream<LocalDate> allDays = start.datesUntil(endExclusive);
Stream<LocalDate> firstDaysInMonth = start.datesUntil(endExclusive, Period.ofMonths(1));
```

除了LocalDate之外，还有MonthDay、YearMonth和Year类可以描述部分日期。

### 相关API
java.time.LocalDate：
* static LocalDate now()：获取当前的LocalDate。
* static LocalDate of(int year, int month, int dayOfMonth)
* static LocalDate of(int year, Month month, int dayOfMonth)：用给定的年、月（1到12之间的整数或者Month枚举的值）和日（1到31之间）产生一个本地日期。
* LocalDate (plus|minus)(Days|Weeks|Months|Years)(long number)：产生一个LocaLDate，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的。
* LocalDate plus(TemporalAmount amountToAdd)
* LocalDate minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration和Period类实现了TemporalAmount接口。
* LocalDate WithDayOfMonth(int dayOfMonth)
* LocalDate withDayOfYear(int dayOfYear)
* LocalDate withMonth(int month)
* LocalDate withYear(int year)：返回一个新的LocalDate，将月份日期、年日期、月或年修改为给定值。
* int getDayOfMonth()：获取月份日期（1到31之间）。
* int getDayOfYear()：获取年日期（1到366之间）。
* DayOfWeek getDayOfWeek()：获取星期日期，返回某个DayOfWeek枚举值。
* Month getMonth()
* int getMonthValue()：获取用Month枚举值表示的月份，或者用1到12之间的数字表示的月份。
* int getYear()：获取年份，在-999999999到999999999之间。
* Period until(ChronoLocalDate endDateExclusive)：获取直到给定终止日期的period。LocalDate和date类针对非公历实现了ChronoLocalDate接口。
* boolean isBefore(ChronoLocalDate other)
* boolean isAfter(ChronoLocalDate other)：如果该日期在给定日期之前或之后，则返回true。
* boolean isLeapYear()：如果当前是闰年，则返回true。即，该年份能够被4整除，但是不能被100整除，或者能够被400整除。该算法应该可以应用于所有已经过去的年份，尽管在历史上它并不准确（闰年是在公元前46年发明出来的，而涉及整除100和400的规则是在1582年的公历改革中引入的。这场改革经历了300年才被广泛接受）。
* Stream&lt;LocalDate> datesUntil(LocalDate endExclusive)
* Stream&lt;LocalDate> datesUntil(LocalDate endExclusive, Period step)：产生一个日期流，从当前的LocalDate对象直至参数endExclusive指定的日期，其中步长尺寸为1，或者是给定的period。

java.time.Period：
* static Period of(int years, int months, int days)
* Period of(Days|Weeks|Months|Years)(int number)：用给定数量的时间单位产生一个Period对象。
* int get(Days |Months |Years)()：获取当前Period对象的日、月或年。
* Period (plus|minus)(Days|Months|Years)(long number)：产生一个LocaLDate，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的。
* Period plus(TemporalAmount amountToAdd)
* Period minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻距离给定的时间量。Duration和Period类实现了TemporalAmount接口。
* Period with(Days|Months|Years)(int number)：返回一个新的Period，将日、月、年修改为给定值。

## 日期调整器
TemporalAdjusters类提供了大量用于常见调整的静态方法。你可以将调整方法的结果传递给with方法。例如，某个月的第一个星期二可以像下面这样计算：
```Java
LocalDate firstTuesday = LocalDate.of(year, month, 1).with(TemporalAdjusters.nextOrSame(DayOfWeek.TUESDAY));
```

with方法会返回一个新的LocalDate对象，而不会修改原来的对象。

还可以通过实现TemporalAdjuster接口来创建自己的调整器。下面是用于计算下一个工作日的调整器。
```Java
TemporalAdjuster NEXT_WORKDAY = w -> // w的参数类型为Temporal
    {
        var result = (LocalDate) w;  // 转型为LocalDate
        do
        {
            result = result.plusDays(1);
        }
        while (result.getDayOfWeek().getvalue() >= 6);
        return result;
    };

LocalDate backToWork = today.with(NEXT_WORKDAY);
```

注意，lambda表达式的参数类型为Temporal，它必须被强制转型为LocalDate。你可以用ofDateAdjuster 方法来避免这种强制转型，该方法期望得到的参数是类型为UnaryOperator&lt;LocalDate>的lambda表达式。
```Java
TemporalAdjuster NEXTWORKDAY = TemporalAdjusters.ofDateAdjuster(w ->
    {
        LocalDate result = w;
        do
        {
            result = result.plusDays(1);
        }
        while (result.getDayOfWeek().getvalue() >= 6);
        return result;
    });
```

### 相关API
java.time.LocalDate：
* LocalDate with(TemporalAdjuster adjuster)：返回该日期通过给定的调整器调整后的结果。

java.time.temporal.TemporalAdjusters：
* static TemporalAdjuster next(DayOfweek dayOfWeek)
* static TemporalAdjuster nextOrSame(DayOfweek dayOfWeek)
* static TemporalAdjuster previous(DayOfweek dayOfWeek)
* static TemporalAdjuster previousOrSame(DayOfweek dayOfWeek)：返回一个调整器，用于将日期调整为给定的星期日期。
* static TemporalAdjuster dayOfWeekInMonth(int n, DayOfweek dayOfWeek)
* static TemporalAdjuster lastInMonth(DayOfweek dayOfWeek)：返回一个调整器，用于将日期调整为月份中第n个或最后一个给定的星期日期。
* static TemporalAdjuster firstDayOfMonth()
* static TemporalAdjuster firstDayOfNextMonth()
* static TemporalAdjuster firstDayOfYear()
* static TemporalAdjuster firstDayOfNextYear()
* static TemporalAdjuster lastDayOfMonth()
* static TemporalAdjuster lastDayOfYear()：返回一个调整器，用于将日期调整为月份或年份中给定的日期。

## 本地时间
LocalTime表示当日时刻，例如15:30:00。可以用now或of方法创建其实例：
```Java
LocalTime rightNow = LocalTime.now();
LocalTime bedtime = LocalTime.of(22, 30); // or LocalTime.of(22, 30, 0)
```

plus和minus操作是按照一天24小时循环操作的。例如：
```Java
LocalTime wakeup = bedtime.plusHours(8); // wakeup is 6:30:00
```

还有一个表示日期和时间的LocalDateTime类。这个类适合存储固定时区的时间点。

### 相关API
java.time.LocalTime：
* static LocalTime now()：获取当前的LocalTime。
* static LocalTime of(int hour, int minute)
* static LocalTime of(int hour, int minute, int second)
* static LocalTime of(int hour, int minute, int second, int nanoOfSecond)：产生一个LocalTime，它具有给定的小时（0到23之间）、分钟、秒（0到59之间）和纳秒（0到999999999之间）。
* LocalTime (plus|minus)(Hours|Minutes|Seconds|Nanos)(long number)：产生一个LocalTime，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的。
* LocalTime plus(TemporalAmount amountToAdd)
* LocalTime minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻距离给定的时间量。
* LocalTime with(Hour|Minute|Second|Nano)(int value)：返回一个新的LocalTime，将小时、分钟、秒或纳秒修改为给定值。
* int getHour()：获取小时（0到23之间）。
* int getMinute()
* int getSecond()：获取分钟或秒（0到59之间）。
* int getNano()：获取纳秒（0到999999999之间）。
* int toSecondOfDay()
* long toNanoOfDay()：产生自午夜到当前LocalTime的秒或纳秒数。
* boolean isBefore(LocalTime other)
* boolean isAfter(LocalTime other)：如果当期日期在给定日期之前或之后，则返回true。

## 时区时间
互联网编码分配管理机构（Internet Assigned Numbers Authority，IANA）保存着一个数据库，里面存储着世界上所有已知的时区（www.iana.org/time-zones），它每年会更新数次，其中许多更新是为了处理夏令时的变更规则。Java使用了IANA数据库。

每个时区都有一个ID，例如America/New_York和Europe/Berlin。要想找出所有可用的时区，可以调用ZoneId.getAvailableZoneIds。

给定一个时区ID,静态方法ZoneId.of(id)可以产生一个ZoneId对象。可以通过调用local.atZone(zoneId)用这个ZoneId对象将LocalDateTime对象转换为ZonedDateTime对象，或者可以通过调用静态方法ZonedDateTime.of(year, month, day, hour, minute, second, nano, zoneId)来构造一个ZonedDateTime对象。例如：
```Java
ZonedDateTime apollo11launch = ZonedDateTime.of(1969, 7, 16, 9, 32, 0, 0, ZoneId.of("America/New_York")); // 阿波罗11号发射时间
System.out.println(apollo11launch); // 1969-07-16T09:32-04:00[America/New_York]
```

这是一个具体的时刻，调用apollo11launch.toInstant可以获得对应的Instant对象。反过来，如果你有一个时刻对象，调用instant.atZone(ZoneId.of("UTC"))可以获得格林尼治皇家天文台的ZonedDateTime对象，或者使用其他的ZoneId获得地球上其他地方的ZoneId。
```Java
Instant instant = apollo11launch.toInstant();
ZonedDateTime zonedDateTime = instant.atZone(ZoneId.of("UTC"));
System.out.println(zonedDateTime); // 1969-07-16T13:32Z[UTC]

// 虽然两者能够表示相同的时刻，但是它们所代表的时间和时区却是不同的
System.out.println(zonedDateTime == apollo11launch); // false
```

当夏令时开始时，时钟要向前拨快一小时。当你构建的时间对象正好落入了这跳过去的一个小时内时，会发生什么？例如，在2013年，中欧地区在3月31日2:00切换到夏令时，如果你试图构建的时间是不存在的3月31日2:30,那么你实际上得到的是3:30。

```Java
ZonedDateTime skipped = ZonedDateTime.of(
    LocalDate.of(2013, 3, 31),
    LocalTime.of(2, 30),
    ZoneId.of("Europe/Berlin"));
    // Constructs March 31 3:30
System.out.println(skipped); // 2013-03-31T03:30+02:00[Europe/Berlin]
```

> 2013年柏林夏令时于3月31日凌晨2点开始，届时时钟向前拨快一小时至3点。夏令时在10月27日凌晨2点结束，时钟向后拨回一小时至1点。
>
> 关键时间节点序列如下：
> * ……
> * 2013-03-31T01:58+01:00[Europe/Berlin]
> * 2013-03-31T01:59+01:00[Europe/Berlin]
> * 2013-03-31T03:00+02:00[Europe/Berlin]
> * 2013-03-31T03:01+02:00[Europe/Berlin]
> * 2013-03-31T03:02+02:00[Europe/Berlin]
> * ……
> * 2013-10-27T01:59+02:00[Europe/Berlin]
> * 2013-10-27T02:00+02:00[Europe/Berlin]
> * 2013-10-27T02:01+02:00[Europe/Berlin]
> * ……
> * 2013-10-27T02:59+02:00[Europe/Berlin]
> * 2013-10-27T02:00+01:00[Europe/Berlin]
> * 2013-10-27T02:01+01:00[Europe/Berlin]
> * ……

反过来，当夏令时结束时，时钟要向回拨慢一小时，这样同一个本地时间就会出现两次。当你构建位于这个时间段内的时间对象时，就会得到这两个时刻中较早的一个：
```Java
ZonedDateTime ambiguous = ZonedDateTime.of(
    LocalDate.of(2013, 10, 27), // End of daylight savings time
    LocalTime.of(2, 30),
    ZoneId.of("Europe/Berlin"));
System.out.println(ambiguous); // 2013-10-27T02:30+02:00[Europe/Berlin]

ZonedDateTime anHourLater = ambiguous.plusHours(1);
System.out.println(anHourLater); // 2013-10-27T02:30+01:00[Europe/Berlin]
```
一个小时后的时间会具有相同的小时和分钟，但是时区的偏移量会发生变化。

你还需要在调整跨越夏令时边界的日期时特别注意。例如，如果你将会议设置在下个星期，不要直接加上一个7天的Duration，而是应该使用Period类。
```Java
ZonedDateTime nextMeeting1 = meeting.plus(Duration.ofDays(7)); // Caution! Won't work with daylight savings time

ZonedDateTime nextMeeting2 = meeting.plus(Period.ofDays(7)); // OK
```

### 相关API
java.time.ZonedDateTine：
* static ZonedDateTime now()：获取当前的ZonedDateTime。
* static ZonedDateTime of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanoOfSecond, ZoneId zone)
* static ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone)
* static ZonedDateTime of(LocalDateTime localDateTime, ZoneId zone)
* static ZonedDateTime ofInstant(nstant instant, ZoneId zone)用给定的参数和时区产生一个ZonedDateTime。
* ZonedDateTime (plus|minus)(Days|Weeks|Months|Years|Hours|Minutes|Seconds|Nanos)(long number)
：产生一个ZonedDateTime，该对象是通过在当前对象上加上或减去给定数量的时间单位获得的。
* ZonedDateTime plus(TemporalAmount amountToAdd)
* ZonedDateTime minus(TemporalAmount amountToSubtract)：产生一个时刻，该时刻与当前时刻相差给定的时间量。
* ZonedDateTime With(DayOfMonth|DayOfYear|Month|Year|Hour|Minute|Second|Nano)(int value)：返回一个新的ZonedDateTime，用给定的值替换给定的时间单位。
* ZonedDateTime withZoneSameInstant(ZoneId zone)
* ZonedDateTime withZoneSameLocal(ZoneId zone)：返回一个新的ZonedDateTime，位于给定的时区，它与当前对象要么表示相同的时刻，要么表示相同的本地时间。
* int getDayOfMonth()：获取月份日期（1到31之间）。
* int getDayOfYear()：获取年份日期（1到366之间）。
* DayOfWeek getDayOfWeek()：获取星期日期，返回DayOfWeek枚举的值。
* Month getMonth()
* int getMonthValue()：获取用Month枚举值表示的月份，或者用1到12之间的数字表示的月份。
* int getYear()：获取年份，在-999999999到999999999之间。
* int getHour()：获取小时（0到23之间）。
* int getMinute()
* int getSecond()：获取分钟到秒（0到59之间）。
* int getNano()：获取纳秒（0到999999999之间）。
* public ZoneOffset getOffset()：获取与UTC的时间差距。差距可在-12:00~+14:00变化。有些时区还有小数时间差。时间差会随着夏令时变化。
* LocalDate toLocalDate()
* LocalTime toLocalTime()
* LocalDateTime toLocalDateTime()
* Instant toInstant()：生成当地日期、时间，或日期／时间，或相应的瞬间。
* boolean isBefore(ChronoZonedDateTime other)
* boolean isAfter(ChronoZonedDateTime other)：如果这个时区日期／时间在给定的时区日期／时间之前或之后，则返回true。

## 格式化和解析
DateTimeFormatter类提供了三种用于打印日期/时间值的格式器:
* 预定义的格式器
* locale相关的格式器
* 带有定制模式的格式器

### 预定义的格式器
|格式器|描述|示例|
|---|---|---|
|BASIC ISO DATE|年、月、日、时区偏移量，中间没有分隔符|19690716-0500|
|ISO_LOCAL_DATE,<br>ISO_LOCAL_TIME,<br>ISO_LOCAL_DATE_TIME|分隔符为-、:、T|1969-07-16,<br>09:32:00,<br>1969-07-16T09:32:00|
|ISO_OFFSET_DATE,<br>ISO_OFFSET_TIME,<br>ISO_OFFSET_DATE_TIME|类似ISO_LOCAL_XXX，但是有时区偏移量|1969-07-16-05:00,<br>09:32:00-05:00,<br>1969-07-16T09:32:00-05:00|
|ISO_ZONED_DATE_TIME|有时区偏移量和时区ID|1969-07-16T09:32:00-05:00\[America/New_York]|
|ISO_INSTANT|在UTC中，用Z时区ID来表示|1969-07-16T14:32:00Z|
|ISO_DATE,<br>ISO_TIME,<br>ISO_DATE_TIME|类似ISO_OFFSET_DATE、ISO_OFFSET_TIME和ISO_ZONED_DATE_TIME，但是时区信息是可选的|1969-07-16-05:00,<br>09:32:00-05:00,<br>1969-07-16T09:32:00-05:00\[America/New_York]|
|ISO_ORDINAL_DATE|LocalDate的年和年日期|1969-197|
|ISO_WEEK_DATE|LocalDate的年、星期和星期日期|1969-W29-3|
|RFC_1123_DATE_TIME|用于邮件时间戳的标准，编纂于RFC822，并在RFC1123中将年份更新到4位|Wed, 16 Jul 1969 09:32:00 -0500|

要使用标准的格式器，可以直接调用其format方法：
```Java
String formatted = DateTimeFonuatter.ISO_OFFSET_DATE_TIME.format(apollo11launch);
// 1969-07-16T09:32:00-04:00
```

### locale相关的格式器
对于日期和时间而言，有4种与locale相关的格式化风格，即SHORT、MEDIUM、LONG和FULL。

|风格|日期|时间|
|---|---|---|
|SHORT|7/16/69|9:32 AM|
|MEDIUM|Jul 16, 1969|9:32:00 AM|
|LONG|July 16, 1969|9:32:00 AM EDT|
|FULL|Wednesday, July 16, 1969|9:32:00 AM EDT|

静态方法ofLocalizedDate、ofLocalizedTime和ofLocalizedDateTime可以创建这种格式器。这些方法使用了默认的locale。为了切换到不同的locale，可以直接使用withLocale方法。
```Java
DateTimeFormatter formatter = DateTimeFonnatter.ofLocalizedDateTime(FormatStyle.LONG);
String formatted = fomatter.format(apollo11launch); // July 16, 1969 9:32:00 AM EDT
fomatted = formatter.withLocale(Locale.FRENCH).format(apollo11launch); // 16 juillet 1969 09:32:00 EDT
```

DayOfWeek和Month枚举都有getDisplayName方法，可以按照不同的locale和格式给出星期日期和月份的名字。
```Java
for (DayOfWeek w : DayOfWeek.values())
{
    System.out.print(w.getDisplayName(Textstyle.SHORT, Locale.ENGLISH) + "");
}
// Prints Mon Tue Wed Thu Fri Sat Sun
```

### 带有定制模式的格式器
可以通过指定模式来定制自己的日期格式。例如：
```Java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("E yyyy-MM-dd HH:mm");
String formatted = fomatter.format(apollo11launch); // Wed 1969-07-16 09:32
```

"E yyyy-MM-dd HH:mm"中的每个字母都表示一个不同的时间域，而字母重复的次数对应于所选择的特定格式。下表展示了最有用的模式元素。

|时间域或目的|示例|
|---|---|
|ERA|G: AD, GGGG: Anno Domini, GGGGG: A|
|YEAR_OF_ERA|yy: 69, yyyy: 1969|
|MONTH_OF_YEAR|M: 7, MM: 07, MMM: JuL, MMMM: July, MMMMM: J|
|DAY_OF_MONTH|d: 6, dd: 06|
|DAY_OF_WEEK|e: 3, E: Wed, EEEE: Wednesday, EEEEE: W|
|HOUR_OF_DAY|H: 9, HH: 09|
|CLOCK_HOUR_OF_AM_PM|K: 9, KK: 09|
|AMPM_OF_DAY|a: AM|
|MINUTE_OF_HOUR|mm: 02|
|SECOND_OF_MINUTE|ss: 00|
|NANO_OF_SECOND|nnnnnn: 000000|
|时区ID|VV: America/New_York|
|时区名|z: EDT, zzzz: Eastern Daylight Time, v: ET, vvvv: Eastern time|
|时区偏移量|x: -04, xx: -0400, xxx: -04:00, XXX: 与xxx相同，但是用*Z*表示*0时区*|
|本地化的时区偏移量|O: GMT-4, OOOO: GMT-04:00|
|修改后的儒略日|g: 58243|

为了解析字符串中的日期/时间值，可以使用众多的諍态parse方法之一。例如：
```Java
LocalDate churchsBirthday = LocalDate.parse("1903-06-14"); 
ZonedDateTime apollo11launch = ZonedDateTime.parse("1969-07-16 03:32:00-0400", DateTimeFormatter.ofPattem("yyyy-MM-dd HH:mm:ssxx"));
```
第一个调用使用了标准的ISO_LOCAL_DATE格式器，而第二个调用使用的是一个定制的格式器。

### 相关API
java.time.format.DateTimeFormatter：
* String format(TemporalAccessor temporal)：格式化给定值。Instant、LocalDate、LocalTime、LocalDateTime和ZonedDateTime，以及许多其他类，都实现了TemporalAccessor接口。
* static DateTimeFormatter ofLocalizedDate(FormatStyle dateStyle)
* static DateTimeFormatter ofLocalizedTime(FormatStyle timeStyle)
* static DateTimeFormatter ofLocalizedDateTime(FormatStyle dateTimeStyle)
* static DateTimeFormatter ofLocalizedDateTime(FormatStyle dateStyle, FormatStyle timeStyle)：产生一个用于给定风格的格式器。FormatStyle枚举的值包括SHORT、MEDIUM、LONG和FULL。
* DateTimeFormatter withLocale(Locale locale)：用给定的locale产生一个等价于当前格式器的格式器。
* static DateTimeFormatter ofPattern(String pattern)
* static DateTimeFormatter ofPattern(String pattern, Locale locale)：用给定的模式和locale产生一个格式器。

java.time.LocalDate：
* static LocalDate parse(CharSequence text)
* static LocalDate parse(CharSequence text, DateTimeFormatter formatter)：用默认的格式器或给定的格式器产生一个LocalDate。

java.time.ZonedDateTime：
* static ZonedDateTime parse(CharSequence text)
* static ZonedDateTime parse(CharSequence text, DateTimeFormatter formatter)：用默认的格式器或给定的格式器产生一个ZonedDateTime。

## 与遗留代码的互操作
java.time类与遗留类之间的转换表如下：

|java.time类|遗留类|转换到遗留类|转换自遗留类|
|---|---|---|---|
|Instant|java.util.Date|Date.frow(instant)|date.toInstant()|
|ZonedDateTime|java.util.GregorianCalendar|GregorianCalendar.from(zonedDateTime)|cal.toZonedDateTime()|
|Instant|java.sql.Timestamp|TimeStamp.from(instant)|timestamp.toInstant()|
|LocalDateTime|java.sql.Timestamp|Timestamp.valueOf(localDateTime)|timeStamp.toLocalDateTime()|
|LocalDate|java.sql.Date|Date.valueOf(localDate)|date.toLocalDate()|
|LocalTime|java.sql.Time|Time.valueOf(LocalTime)|time.toLocalTime()|
|DateTimeFormatter|java.text.DateFormat|formatter.toFormat()|无|
|ZoneId|java.util.TimeZone|timeZone.toZoneId()|Timezone.getTimeZone(id)|
|Instant|java.nio.file.attribute.FileTime|fileTime.toInstant()|FileTime.from(instant)|

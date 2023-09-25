# 日志



## 什么是日志
日志：在计算机领域，日志文件（logfile）是一个记录了发生在运行中的操作系统或其他软件中的事件的文件，或者记录了在网络聊天软件的用户之间发送的消息

日志记录（Logging）：是指保存日志的行为。最简单的做法是将日志写入单个存放日志的文件。



## 日志的级别
* FATAL — 表示需要立即被处理的系统级错误。当该错误发生时，表示服务已经出现了某种程度的不可用，系统管理员需要立即介入。这属于最严重的日志级别，因此该日志级别必须慎用，如果这种级别的日志经常出现，则该日志也失去了意义。通常情况下，一个进程的生命周期中应该只记录一次FATAL级别的日志，即该进程遇到无法恢复的错误而退出时。当然，如果某个系统的子系统遇到了不可恢复的错误，那该子系统的调用方也可以记入FATAL级别日志，以便通过日志报警提醒系统管理员修复；
* ERROR — 该级别的错误也需要马上被处理，但是紧急程度要低于FATAL级别。当ERROR错误发生时，已经影响了用户的正常访问。从该意义上来说，实际上ERROR错误和FATAL错误对用户的影响是相当的。FATAL相当于服务已经挂了，而ERROR相当于好死不如赖活着，然而活着却无法提供正常的服务，只能不断地打印ERROR日志。特别需要注意的是，ERROR和FATAL都属于服务器自己的异常，是需要马上得到人工介入并处理的。而对于用户自己操作不当，如请求参数错误等等，是绝对不应该记为ERROR日志的；
* WARN — 该日志表示系统可能出现问题，也可能没有，这种情况如网络的波动等。对于那些目前还不是错误，然而不及时处理也会变为错误的情况，也可以记为WARN日志，例如一个存储系统的磁盘使用量超过阀值，或者系统中某个用户的存储配额快用完等等。对于WARN级别的日志，虽然不需要系统管理员马上处理，也是需要及时查看并处理的。因此此种级别的日志也不应太多，能不打WARN级别的日志，就尽量不要打；
* INFO — 该种日志记录系统的正常运行状态，例如某个子系统的初始化，某个请求的成功执行等等。通过查看INFO级别的日志，可以很快地对系统中出现的 WARN,ERROR,FATAL错误进行定位。INFO日志不宜过多，通常情况下，INFO级别的日志应该不大于TRACE日志的10%；
* DEBUG or TRACE — 这两种日志具体的规范应该由项目组自己定义，该级别日志的主要作用是对系统每一步的运行状态进行精确的记录。通过该种日志，可以查看某一个操作每一步的执 行过程，可以准确定位是何种操作，何种参数，何种顺序导致了某种错误的发生。可以保证在不重现错误的情况下，也可以通过DEBUG（或TRACE）级别的日志对问题进行诊断。需要注意的是，DEBUG日志也需要规范日志格式，应该保证除了记录日志的开发人员自己外，其他的如运维，测试人员等也可以通过 DEBUG（或TRACE）日志来定位问题；

日志级别优先级：ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF



## 日志的作用
日志记录了系统行为的时间、地点、状态等相关信息，能够帮助我们了解并监控系统状态，在发生错误或者接近某种危险状态时能够及时提醒我们处理，同时在系统产生问题时，能够帮助我们快速的定位、诊断并解决问题。



## 常用的日志框架
在Java程序中常用日志框架可以分为两类：
- 门面型日志框架
  1. Commons Logging：Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging。
  2. SLF4J(Simple Logging Facade for Java，缩写Slf4j)：是一套简易Java日志门面，本身并无日志的实现。
- 记录型日志框架
  1. Log4j：Apache Log4j是一个基于Java的日志记录工具。它是由Ceki Gülcü首创的，现在则是Apache软件基金会的一个项目。
  2. JUL(Java Util Logging)：JDK中的日志记录工具，也常称为JDKLog、jdk-logging，自Java1.4以来的官方日志实现。
  3. Logback：一个具体的日志实现框架，和Slf4j是同一个作者，但其性能更好。
  4. Log4j2：一个具体的日志实现框架，是Log4j1的下一个版本，与Log4j1发生了很大的变化，Log4j2不兼容Log4j1。

### 框架的发展历史
[省略](https://juejin.cn/post/6905026199722917902#heading-4)

### 门面框架/门面模式(设计模式)
日志门面框架是一套提供了日志相关功能的接口而无具体实现的框架，其调用具体的实现框架来进行日志记录。比较常用的组合使用方式是Slf4j与Logback组合使用，Commons Logging与Log4j组合使用。

### 实现机制

#### Commons logging实现机制
Commons logging是通过动态查找机制，在程序运行时，使用自己的ClassLoader寻找和载入本地具体的实现。详细策略可以查看commons-logging-*.jar包中的org.apache.commons.logging.impl.LogFactoryImpl.java文件。由于OSGi不同的插件使用独立的ClassLoader，OSGI的这种机制保证了插件互相独立, 其机制限制了commons logging在OSGi中的正常使用。

#### Slf4j实现机制
Slf4j在编译期间，静态绑定本地的LOG库，因此可以在OSGi中正常使用。它是通过查找类路径下org.slf4j.impl.StaticLoggerBinder，然后绑定工作都在这类里面进行。



## 日志规约
应用中不可直接使用日志系统（log4j、logback）中的 API ，而应依赖使用日志框架 SLF4J 中的 API 。使用门面模式的日志框架，有利于维护和各个类的日志处理方式的统一。

推荐使用SLF4J。



## SLF4J

### 设计思想
* Slf4j的设计思想比较简洁，使用了Facade设计模式，Slf4j本身只提供了一个slf4j-api-version.jar包，这个jar中主要是日志的抽象接口，jar中本身并没有对抽象出来的接口做实现。
* 对于不同的日志实现方案(例如Logback，Log4j...)，封装出不同的桥接组件(例如logback-classic-version.jar，slf4j-log4j12-version.jar)，这样使用过程中可以灵活的选取自己项目里的日志实现。

### 组件集成
![log+20220119162505](https://raw.githubusercontent.com/loli0con/picgo/master/images/log%2B20220119162505.png%2B2022-01-19-16-25-06)

如图所示，应用调了sl4j-api，即日志门面接口。日志门面接口本身通常并没有实际的日志输出能力，它底层还是需要去调用具体的日志框架API的，也就是实际上它需要跟具体的日志框架结合使用。由于具体日志框架比较多，而且互相也大都不兼容，日志门面接口要想实现与任意日志框架结合可能需要对应的桥接器，上图红框中的组件即是对应的各种桥接器！

|桥接器.jar（✳️代表版本）|说明|
|---|---|
|logback-classic-✳️.jar|Slf4j的原生实现，Logback直接实现了Slf4j的接口，因此使用Slf4j与Logback的结合使用也意味更小的内存与计算开销|
|slf4j-log4j12-✳️.jar|Log4j1.2版本的桥接器，你需要将Log4j.jar加入Classpath。|
|slf4j-jdk14-✳️.jar|java.util.logging的桥接器，Jdk原生日志框架。|
|slf4j-jcl-✳️.jar|Jakarta Commons Logging 的桥接器. 这个桥接器将Slf4j所有日志委派给Jcl。|
|slf4j-simple-✳️.jar|一个简单实现的桥接器，输出所有事件到System.err；只有Info以及高于该级别的消息被打印。|
|slf4j-nop-✳️.jar|NOP桥接器，默默丢弃一切日志。|

具体的介入方式：
![log+20220119165300](https://raw.githubusercontent.com/loli0con/picgo/master/images/log%2B20220119165300.png%2B2022-01-19-16-53-01)

### 重定向
在开发中我么想使用slf4j+logback，但是对于一些遗留项目中例如Spring（commons-logging）、Hibernate（jboss-logging）…等等，如何去做到日志统一呢？

![log+20230919170219](https://raw.githubusercontent.com/loli0con/picgo/master/images/log%2B20230919170219.png%2B2023-09-19-17-02-20)

让系统中所有的日志都统一到slf4j的具体步骤：
1. 将系统中其他日志框架先排除出去
2. 用中间包来替换原有的日志框架
3. 导入slf4j其他的实现

SpringBoot同样使用了上述的步骤来实现日志功能。
![log+20230919170510](https://raw.githubusercontent.com/loli0con/picgo/master/images/log%2B20230919170510.png%2B2023-09-19-17-05-10)
SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志。当引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。

### 源码分析

#### 核心类/接口
|类与接口|用途|
|---|---|
|org.slf4j.LoggerFactory(class)|给调用方提供的创建Logger的工厂类，在编译时绑定具体的日志实现组件|
|org.slf4j.Logger(interface)|给调用方提供的日志记录抽象方法，例如debug(String msg),info(String msg)等方法|
|org.slf4j.ILoggerFactory(interface)|获取的Logger的工厂接口，具体的日志组件实现此接口|
|org.slf4j.helpers.NOPLogger(class)|对org.slf4j.Logger接口的一个没有任何操作的实现，也是Slf4j的默认日志实现|
|org.slf4j.impl.StaticLoggerBinder(class)|与具体的日志实现组件实现的桥接类，具体的日志实现组件需要定义org.slf4j.impl包，并在org.slf4j.impl包下提供此类，注意在slf4j-api-✳️.jar中不存在org.slf4j.impl.StaticLoggerBinder，在源码包slf4j-api-✳️-source.jar中才存在此类|

#### demo
##### pom配置
```pom
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.30</version>
</dependency>
```

##### 程序入口类
```java
package fun.pigyun;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class App {
    private final static Logger logger = LoggerFactory.getLogger(App.class);

    public static void main(String[] args) {
        logger.info("Hello World");
    }
}
```

##### 源码追踪
[跟着教程跑一遍](https://juejin.cn/post/6905026199722917902#heading-19)，这个debug的过程相对简单，CET-4词汇量即可看懂源码。



## SpringBoot实践

### 修改默认配置
修改SpringBoot的配置文件（application.yml等）：
```yml
#修改日志的级别，默认root是info
#logging.level.root=trace

# 不指定路径在当前项目下生成springboot.log日志
#logging.file=springboot.log
# 可以指定完整的路径；
logging.file=d://springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用spring.log 作为默认文件
logging.path=/spring/log

# 在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

### 指定日志文件
给类路径下放上每个日志框架自己的配置文件，SpringBoot就不使用他默认配置了。

|Logging System|Customization|
|---|---|
|Logback|logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy|
|Log4j2|log4j2-spring.xml or log4j2.xml|
|JDK (Java Util Logging)|logging.properties|

### springProfile
logback.xml文件会直接就被日志框架识别，而对于logback-spring.xml日志框架就不直接加载日志的配置项，因为需要先解析springProfile标签的内容（pringBoot的高级Profile功能），此时由SpringBoot解析日志配置。

```XML
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
  <layout class="ch.qos.logback.classic.PatternLayout">
    <!-- configuration to be enabled when the "dev" profile is active -->
    <springProfile name="dev">
      <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ---- [%thread] ---- %-5level %logger{50} - %msg%n</pattern>
    </springProfile>
    <springProfile name="!dev">
      <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
    </springProfile>
  </layout>
</appender>
```

### 切换日志框架

#### log4j
```XML
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
      <exclusion>
        <artifactId>log4j-to-slf4j</artifactId>
        <groupId>org.apache.logging.log4j</groupId>
      </exclusion>
      <exclusion>
        <artifactId>logback-classic</artifactId>
        <groupId>ch.qos.logback</groupId>
      </exclusion>
    </exclusions>
  </dependency>
  <dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
  </dependency>
</dependencies>
```

#### log4j2
```XML
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
      <exclusion>
        <artifactId>spring-boot-starter-logging</artifactId>
        <groupId>org.springframework.boot</groupId>
      </exclusion>
    </exclusions>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
  </dependency>

</dependencies>
```



## Logback详解
spring-boot-starter包含spring-boot-starter-logging，该依赖内容就是SpringBoot默认的日志框架Logback+SLF4J。

### 默认配置
默认情况下Spring Boot将日志输出到控制台，不会写到日志文件。如果要编写除控制台输出之外的日志文件，则需在application.properties中设置logging.file或logging.path属性。

```properties
# 注：二者不能同时使用，如若同时使用，则只有logging.file生效
logging.file=文件名
logging.path=日志文件路径
 
logging.level.包名=指定包下的日志级别
logging.pattern.console=日志打印规则
```
* logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log
* logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容.如：logging.path=/var/log

### logback-spring.xml详解
Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml），命名为logback-spring.xml的日志配置文件，将xml放至src/main/resource下面。

也可以使用自定义的名称，比如logback-config.xml，只需要在application.properties文件中使用logging.config=classpath:logback-config.xml指定即可。

Logback基于三个主要类：Logger，Appender和Layout。这三种类型的组件协同工作，使开发人员能够根据消息类型和级别记录消息，并在运行时控制这些消息的格式以及报告的位置。


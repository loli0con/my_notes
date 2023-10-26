# Logback

## 简介
Logback 继承自 log4j。

Logback 的架构非常的通用，适用不同的使用场景。Logback 被分成三个不同的模块：logback-core，logback-classic，logback-access。

logback-core 是其它两个模块的基础。logback-classic 模块可以看作是 log4j 的一个优化版本，它天然的支持 SLF4J，所以你可以随意的从其它日志框架（例如：log4j 或者 java.util.logging）切回到 logack。

logback-access 可以与 Servlet 容器进行整合，例如：Tomcat、Jetty。它提供了 http 访问日志的功能。

## 模块依赖
logback-classic 模块需要在 classpath 添加 slf4j-api.jar、logback-core.jar 以及 logback-classic.jar。

## 使用步骤
通过如下的三个步骤可以启用 logback 来记录日志：
1. 配置logback环境，可以通过简单或者复杂的方式来做。
2. 如果你想在每个类中打印日志，那么你需要将当前类的全称或者当前类当作参数，调用`org.slf4j.LoggerFactory.getLogger()`方法获取**实例logger**。
3. 使用**实例logger**来调用不同的方法来打印日志。例：debug()、info()、warn()、error()。通过这些方法将会在配置好的appender中输出日志。

## 默认策略
logback 默认的配置策略：当没有默认的配置时（即没有logback.xml配置文件），logback 将会在 root logger 中新增一个 ConsoleAppender。

`Appender` 类被看作为输出的目的地。Appenders 包括 console，files，Syslog，TCP Sockets，JMS 等等其它的日志输出目的地。用户可以根据自己的情况轻松的创建自己的 Appender。

## StatusPrinter
Logback 通过一个内部的状态系统来报告它本身的状态信息。发生在 logback 生命周期中的事件可以通过 StatusManager 来获取。
```Java
// 打印内部的状态
LoggerContext lc = (LoggerContext)LoggerFactory.getILoggerFactory();
StatusPrinter.print(lc);

/*
12:23:49,258 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
12:23:49,258 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
12:23:49,258 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.xml]
12:23:49,262 |-INFO in ch.qos.logback.classic.BasicConfigurator@2e5d6d97 - Setting up default configuration.
*/
```
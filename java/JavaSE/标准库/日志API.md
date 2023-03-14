# 日志API
标准库的日志API只是定义了记录消息的最小API，开发者可将这些日志消息路由到各种主流的日志框架（如SLF4J、Log4J等），否则默认使用Java传统的java.util.logging日志API。

## 日志级别
该日志级别是一个非常有用的东西：在开发阶段调试程序时，可能需要大量输出调试信息；在发布软件时，又希望关掉这些调试信息。此时就可通过日志来实现，只要将系统日志级别调高，所有低于该级别的日志信息就都会被自动关闭，如果将日志级别设为OFF，那么所有日志信息都会被关闭。

|最新日志级别|传统日志级别|说明|
|---|---|---|
|ALL|ALL|最低级别，系统将会输出所有日志信息。因此将会生成非常多、非常冗余的日志信息|
|TRACE|FINER|输出系统的各种跟踪信息，也会生成很多、很冗余的日志信息|
|DEBUG|FINE|输出系统的各种调试信息，会生成较多的日志信息|
|INFO|INFO|输出系统内需要提示用户的提示信息，生成中等冗余的日志信息|
|WARNING|WARNING|只输出系统内警告用户的警告信息，生成较少的日志信息|
|ERROR|SEVERE|只输出系统发生错误的的错误信息，生成很少的日志信息|
|OFF|OFF|关闭日志输出|

## 用法
这套日志API的用法非常简单，只要两步即可：
1. 调用System类的getLogger(String name)方法获取System.Logger对象。
2. 调用System.Logger对象的log()方法输出日志。该方法的第一个参数用于指定日志级别。

```Java
// 获取System.Logger对象
System.Logger IMyLogger = System.getLogger("my_logger");

// 由于此处并未使用第三方日志框架，因此系统默认使用java.util.logging日志作为实现
Logger myLogger = Logger.getLogger("my_logger");
// 设置系统日志级别，高于或等于INFO级别的日志信息都会被输出
myLogger.setLevel(Level.INFO);
// 设置使用output.xml保存日志记录
myLogger.addHandler(new FileHandler("output.xml"));

IMyLogger.log(System.Logger.Level.DEBUG,"debug信息");
IMyLogger.log(System.Logger.Level.INFO,"info信息");
IMyLogger.log(System.Logger.Level.ERROR,"error信息");
```

## 国际化
日志API也支持国际化——System类除使用简单的getLogger(String name)方法获取System.Logger对象之外，还可使用getLogger(String name, ResourceBundle bundle)方法来获取该对象，该方法需要传入一个国际化语言资源包，这样该Logger对象即可根据key来输出国际化的日志信息。

在获取System.Logger时加载了ResourceBundle资源包，接下来调用System.Logger的log()方法输出日志信息时，第二个参数应该使用国际化消息key，这样即可输出国际化的日志信息。
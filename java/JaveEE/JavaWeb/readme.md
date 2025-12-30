# JavaWeb

JavaEE是Java Platform Enterprise Edition的缩写，即Java企业平台。JavaEE不是凭空冒出来的，它实际上是完全基于JavaSE，只是多了一大堆服务器相关的库以及API接口。所有的JavaEE程序，仍然是运行在标准的JavaSE的虚拟机上的。

由于Oracle将JavaEE移交给Eclipse开源组织时，不允许他们继续使用Java商标，所以JavaEE再次改名为Jakarta EE。因为这个拼写比较复杂而且难记，所以后面还是用JavaEE这个缩写。

JavaEE并不是一个软件产品，它更多的是一种软件架构和设计思想。我们可以把JavaEE看作是在JavaSE的基础上，开发的一系列基于服务器的组件、API标准和通用架构。

JavaEE最核心的组件就是基于Servlet标准的Web服务器，开发者编写的应用程序是基于Servlet API并运行在Web服务器内部的：
```
┌─────────────┐
│┌───────────┐│
││ User App  ││
│├───────────┤│
││Servlet API││
│└───────────┘│
│ Web Server  │
├─────────────┤
│   JavaSE    │
└─────────────┘
```

## 运行环境
JavaWeb服务器：
* [Tomcat](tomcat.md)

## 开发组件
JavaWeb三大组件：
* [Servlet 程序](servlet.md)
  * [Jsp 动态网页](jsp.md)
* [Filter 过滤器](filter.md)
* [Listener 监听器](listener.md)
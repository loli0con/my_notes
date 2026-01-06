# Listener
监听器是专门用于对[三大域对象](servlet.md#三大域对象)*身上发生的事件*或*状态改变*进行监听和相应处理的对象。

JavaWeb中定义八个监听器接口作为监听器的规范：
* 应用域
  * ServletContextListener：监听ServletContext对象的创建与销毁。
  * ServletContextAttributeListener：监听ServletContext中属性的添加、移除和修改。
* 会话域
  * HttpSessionListener：监听HttpSession对象的创建与销毁。
  * HttpSessionAttributeListener：监听HttpSession中属性的添加、移除和修改。
  * HttpSessionBindingListener：监听<u>当前监听器对象</u>在Session域中的增加与移除。
  * HttpSessionActivationListener：监听<u>某个对象</u>在Session中的序列化与反序列化。
* 请求域
  * ServletRequestListener：监听ServletRequest对象的创建与销毁。
  * ServletRequestAttributeListener：监听ServletRequest中属性的添加、移除和修改。

## 应用域监听器

### ServletContextListener
|方法名|作用|
|---|---|
|contextInitialized(ServletContextEvent sce)|ServletContext创建时调用|
|contextDestroyed(ServletContextEvent sce)|ServletContext销毁时调用|

方法参数中的ServletContextEvent对象代表从ServletContext对象身上捕获到的事件，通过这个事件对象的getServletContext()方法可以获取到ServletContext对象。

### ServletContextAttributeListener
|方法名|作用|
|---|---|
|attributeAdded(ServletContextAttributeEvent scab)|向ServletContext中添加属性时调用|
|ttributeRemoved(ServletContextAttributeEvent scab)|从ServletContext中移除属性时调用|
|attributeReplaced(ServletContextAttributeEvent scab)|当ServletContext中的属性被修改时调用|

ServletContextAttributeEvent对象代表属性变化事件，它包含的方法如下：
|方法名|作用|
|---|---|
|getName()|获取修改或添加的属性名|
|getValue()|获取被修改或添加的属性值|
|getServletContext()|获取ServletContext对象|

## 会话域监听器

### HttpSessionListener
|方法名|作用|
|---|---|
|sessionCreated(HttpSessionEvent hse)|HttpSession对象创建时调用|
|sessionDestroyed(HttpSessionEvent hse)|HttpSession对象销毁时调用|

HttpSessionEvent对象代表从HttpSession对象身上捕获到的事件，通过这个事件对象的getSession()方法可以获取到触发事件的HttpSession对象。

### HttpSessionAttributeListener
|方法名|作用|
|---|---|
|attributeAdded(HttpSessionBindingEvent se)|向HttpSession中添加属性时调用|
|attributeRemoved(HttpSessionBindingEvent se)|从HttpSession中移除属性时调用|
|attributeReplaced(HttpSessionBindingEvent se)|当HttpSession中的属性被修改时调用|

HttpSessionBindingEvent对象代表属性变化事件，它包含的方法如下：
|方法名|作用|
|---|---|
|getName()|获取修改或添加的属性名|
|getValue()|获取被修改或添加的属性值|
|getSession()|获取触发事件的HttpSession对象|

### HttpSessionBindingListener
|方法名|作用|
|---|---|
|valueBound(HttpSessionBindingEvent event)|该类的实例被放到Session域中时调用|
|valueUnbound(HttpSessionBindingEvent event)|该类的实例从Session中移除时调用|

HttpSessionBindingEvent对象代表属性变化事件，它包含的方法如下：
|方法名|作用|
|---|---|
|getName()|获取当前事件涉及的属性名|
|getValue()|获取当前事件涉及的属性值|
|getSession()|获取触发事件的HttpSession对象|

### HttpSessionActivationListener
|方法名|作用|
|---|---|
|sessionWillPassivate(HttpSessionEvent se)|该类实例和Session一起钝化到硬盘时调用|
|sessionDidActivate(HttpSessionEvent se)|该类实例和Session一起活化到内存时调用|

HttpSessionEvent对象代表事件对象，通过getSession()方法获取事件涉及的HttpSession对象。

## 请求域监听器

### ServletRequestListener
|方法名|作用|
|---|---|
|requestInitialized(ServletRequestEvent sre)|ServletRequest对象创建时调用|
|requestDestroyed(ServletRequestEvent sre)|ServletRequest对象销毁时调用|

ServletRequestEvent对象代表从HttpServletRequest对象身上捕获到的事件，通过这个事件对象的getServletRequest()方法可以获取到触发事件的HttpServletRequest对象。另外还有一个getServletContext()方法可以获取到当前Web应用的ServletContext对象。

### ServletRequestAttributeListener
|方法名|作用|
|---|---|
|attributeAdded(ServletRequestAttributeEvent srae)|向ServletRequest中添加属性时调用|
|attributeRemoved(ServletRequestAttributeEvent srae)|从ServletRequest中移除属性时调用|
|attributeReplaced(ServletRequestAttributeEvent srae)|当ServletRequest中的属性被修改时调用|

ServletRequestAttributeEvent对象代表属性变化事件，它包含的方法如下：
|方法名|作用|
|---|---|
|getName()|获取修改或添加的属性名|
|getValue()|获取被修改或添加的属性值|
|getServletRequest()|获取触发事件的ServletRequest对象|

## Listener的注册方式

### 通过注解注册Listener
在监听器类上添加@WebListener注解：
```Java
@WebListener
public class MyListener implements ...
```

### 通过xml注册Listener
在webapp/WEB-INF/web.xml中添加如下内容：
```xml
<listener>
    <listener-class>org.example.MyListener</listener-class>
</listener>
```
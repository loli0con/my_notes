# listener
服务器对三大域对象（request，session，servletContext）的创建与销毁，里面存储数据的增、删、改进行监听定义了监听器接口，我们可以通过实现监听器接口实现对域变化时进行业务控制。

## 分类
![listener+20210722104517](https://raw.githubusercontent.com/loli0con/picgo/master/images/listener%2B20210722104517.png%2B2021-07-22-10-45-19)

### ServletContextListener
```java
/**
 * 目标：开发第一个监听器，监听上下文域对象的创建与销毁
 */
@WebListener  // 注解方式配置
public class MyServletContextListener implements ServletContextListener {

    //监听上下文域对象的创建的时候调用
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        //servletContextEvent.getServletContext() 可以获取到创建的上下文域对象

        // do something ..., such as ⬇️
        String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        System.out.println("上下文域对象创建的时间："+ time);
    }

    //监听上下文域对象的销毁的前调用
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        // do something ..., such as ⬇️
        String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date());
        System.out.println("上下文域对象销毁的时间："+ time);
    }
}
```
#### web.xml方式配置
```xml
<listener>
    <listener-class>
        MyServletContextListener类全名
    </listener-class>
</listener>
```

### ServletContextAttributeListener
当对servletContext域中的数据键增加setAttribute()、修改setAttribute()、删除removeAttribute()时候会触发ServletContextAttributeListener 监听器的运行

#### 接口方法
![listener+20210722105235](https://raw.githubusercontent.com/loli0con/picgo/master/images/listener%2B20210722105235.png%2B2021-07-22-10-52-35)

#### ServletContextAttributeEvent 对象中的方法
![listener+20210722105324](https://raw.githubusercontent.com/loli0con/picgo/master/images/listener%2B20210722105324.png%2B2021-07-22-10-53-25)

#### 示例
```java
@WebListener
public class MyServletContextAttributeListener implements ServletContextAttributeListener {

    //写入数据的监听
    @Override
    public void attributeAdded(ServletContextAttributeEvent sc) {

        // do something such as ⬇️
        //新增属性名获取
        String name = sc.getName();
        //新增属性值获取
        String value = (String) sc.getValue();
        System.out.println("新增属性名:"+name+",属性值:"+value);
    }

    //修改数据的监听
    @Override
    public void attributeReplaced(ServletContextAttributeEvent sc) {

        // do something such as ⬇️
        //修改属性名获取
        String name = sc.getName();
        //修改前属性值获取
        String oldValue = (String) sc.getValue();
        //修改后的属性值获取
        String newValue = (String) sc.getServletContext().getAttribute(name);

        System.out.println("修改属性名:"+name+",修改前属性值:"+oldValue+",修改后属性值:"+newValue);

    }

    //删除数据的监听
    @Override
    public void attributeRemoved(ServletContextAttributeEvent sc) {

        // do something such as ⬇️
        //删除属性名获取
        String name = sc.getName();
        //删除前属性值获取
        String oldValue = (String) sc.getValue();
        System.out.println("删除属性名:"+name+",修改前属性值:"+oldValue);
    }
}
```

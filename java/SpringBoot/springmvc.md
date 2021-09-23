# 整合SpringMVC

## 静态资源
### 导入静态资源
![springmvc+20210825170430](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825170430.png%2B2021-08-25-17-04-36)

默认的静态资源访问路径有四个：
- classpath:/META-INF/resources/
- classpath:/resources/
- classpath:/static/
- classpath:/public/

静态资源放在这些目录中任何一个都可以，但习惯会把静态资源放在`classpath:/static/`目录下。

#### 静态资源访问前缀
![springmvc+20210923160426](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210923160426.png%2B2021-09-23-16-04-27)
默认无前缀，即在*当前项目 + static-path-pattern + 静态资源名 = 静态资源文件夹*下找对应的静态资源。

### 导入Webjars
Webjars本质就是以jar包的方式引入静态资源。

![springmvc+20210825180414](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825180414.png%2B2021-08-25-18-04-16)

可以去网站[webjasrs.org](https://www.webjars.org)里，搜寻一些常用的WebJars。

#### Demo
以使用jQuery为例。

引入依赖：
```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

查看webjars目录结构:
![springmvc+20210825180944](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825180944.png%2B2021-08-25-18-09-44)

访问静态资源：
![springmvc+20210825181003](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825181003.png%2B2021-08-25-18-10-03)


### 自定义
#### 静态资源路径（不推荐）
我们也可以自己通过配置文件来指定一下，哪些文件夹是需要我们放静态资源文件的，在application.properties中配置：
```properties
spring.resources.static-locations=classpath:/coding1/,classpath:/coding2/
```
一旦自己定义了静态文件夹的路径，原来的自动配置就都会失效了！

### 核心代码
![springmvc+20210923172249](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210923172249.png%2B2021-09-23-17-22-53)

## REST
浏览器默认不支持PUT和DELETE方法，需要通过其他方式——**隐藏HTTP方法过滤器**来实现。
```html
<input name="_method" type="hidden" value="PUT" />
<input name="_method" type="hidden" value="DALETE" />
```

![springmvc+20210923191909](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210923191909.png%2B2021-09-23-19-19-13)

![springmvc+20210923193120](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210923193120.png%2B2021-09-23-19-31-23)

![springmvc+20210923194346](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210923194346.png%2B2021-09-23-19-43-50)

也可以自己实现该方法：当HiddenHttpMethodFilter类不存在容器里面时，才会自动配置该方法，所以自己可以编写一个方法，给容器里面放入HiddenHttpMethodFilter类，让自己编写的方法执行而默认的配置方法不执行。



## 拓展MVC

下面是官方文档+翻译：
```
// Spring MVC 自动配置
Spring MVC Auto-configuration

// Spring Boot为Spring MVC提供了自动配置，它可以很好地与大多数应用程序一起工作。
Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

// 自动配置在Spring默认设置的基础上添加了以下功能：
The auto-configuration adds the following features on top of Spring’s defaults:

// 包含视图解析器
Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.

// 支持静态资源文件夹的路径，以及webjars
Support for serving static resources, including support for WebJars 

// 自动注册了Converter、GenericConverter和转换器
// Converter：这就是我们网页提交数据到后台自动封装成为对象的东西，比如把"1"字符串自动转换为int类型
// Formatter：格式化器，比如页面给我们了一个2019-8-10，它会给我们自动格式化为Date对象
Automatic registration of Converter, GenericConverter, and Formatter beans.

// 支持HttpMessageConverters：
// SpringMVC用来转换Http请求和响应的的，比如我们要把一个User对象转换为JSON字符串，可以去看官网文档解释；
Support for HttpMessageConverters (covered later in this document).

// 定义错误代码生成规则的
Automatic registration of MessageCodesResolver (covered later in this document).

// 首页定制
Static index.html support.

// 图标定制
Custom Favicon support (covered later in this document).

// 初始化数据绑定器：帮我们把请求数据绑定到JavaBean中！
Automatic use of a ConfigurableWebBindingInitializer bean (covered later in this document).

/*
如果您希望保留Spring Boot MVC功能，并且希望添加其他MVC配置（拦截器、格式化程序、视图控制器和其他功能），则可以添加自己的@configuration类，类型为webmvcconfiguer，但不添加@EnableWebMvc。如果希望提供RequestMappingHandlerMapping、RequestMappingHandlerAdapter或ExceptionHandlerExceptionResolver的自定义实例，则可以声明WebMVCregistrationAdapter实例来提供此类组件。
*/
If you want to keep Spring Boot MVC features and you want to add additional MVC configuration 
(interceptors, formatters, view controllers, and other features), you can add your own 
@Configuration class of type WebMvcConfigurer but without @EnableWebMvc. If you wish to provide 
custom instances of RequestMappingHandlerMapping, RequestMappingHandlerAdapter, or 
ExceptionHandlerExceptionResolver, you can declare a WebMvcRegistrationsAdapter instance to provide such components.

// 如果您想完全控制Spring MVC，可以添加自己的@Configuration，并用@EnableWebMvc进行注释。
If you want to take complete control of Spring MVC, you can add your own @Configuration annotated with @EnableWebMvc.
```

![springmvc+20210825195314](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825195314.png%2B2021-08-25-19-53-15)

除了实现接口，如果需要一些定制化的功能，可以重写这个组件，然后将它交给Springboot，让Springboot帮我们自动装配。

## 扩展案例

### 首页处理
#### 欢迎页
![springmvc+20210825194109](https://raw.githubusercontent.com/loli0con/picgo/master/images/springmvc%2B20210825194109.png%2B2021-08-25-19-41-11)
静态资源文件夹下的index.html文件，会作为欢迎页展现。

#### 网站图标
与其他静态资源一样，Spring Boot在配置的静态内容位置中查找 favicon.ico。如果存在这样的文件，它将自动用作应用程序的favicon。

注意需要关闭默认图标：
```properties
#关闭默认图标
spring.mvc.favicon.enabled=false
```

### 拦截器
```java
@Configuration
public class MvcConfiguration implements WebMvcConfigurer {

    /**
     * 重写接口中的addInterceptors方法，添加自定义拦截器
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 添加拦截器，通过addPathPatterns来添加拦截路径
        // **代表拦截多层，*代表只拦截一层
        registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/**");
    }
}

/**
 * 自定义拦截器
 */
class LoginInterceptor implements HandlerInterceptor {

    @Override
    // 前置处理，返回true代表放行，false代表拦截
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("前置处理");
        return true;
    }

    @Override
    // 后置处理
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("后置处理");
    }

    @Override
    // 渲染后处理
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("渲染后处理");
    }
}
```


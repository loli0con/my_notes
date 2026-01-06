# Tomcat

## Tomcat目录结构
Tomcat的安装目录`$CATALINA_HOME`:
* bin：该目录下存放的是二进制可执行文件
  * startup.sh/startup.bat：启动Tomcat的脚本
  * shudown.sh/shudown.bat：停止Tomcat的脚本
* conf：配置目录
  * server.xml：配置整个服务器信息。例如修改端口号。
  * tomcat-users.xml：存储Tomcat用户的文件，保存Tomcat的用户名和密码，以及用户的角色信息。
  * web.xml：部署描述符文件，这个文件中注册了很多MIME类型，即文档类型。这些MIME类型是客户端与服务器之间说明文档类型的，如用户请求一个html网页，那么服务器还会告诉客户端浏览器响应的文档是text/html类型的，这就是一个MIME类型。客户端浏览器通过这个MIME类型就知道如何处理它了。当然是在浏览器中显示这个html文件了。但如果服务器响应的是一个exe文件，那么浏览器就不可能显示它，而是应该弹出下载窗口才对。MIME就是用来说明文档的内容是什么类型的！
  * context.xml：对所有应用的统一配置，通常我们不会去配置它。
* lib：Tomcat的类库
* logs：[日志](http://xstarcd.github.io/wiki/Java/tomcat_log.html)目录
  * 服务器日志：catalina.${date}.out
    * 是Tomcat的标准输出(stdout)和标准出错(stderr)
    * 这是在Tomcat的启动脚本里指定的，如果没有修改的话stdout和stderr会重定向到这里。所以我们在应用里使用System.out打印的东西都会到这里来。
    * 另外，如果我们在应用里使用其他的日志框架，配置了向Console输出的，则也会在这里出现。
    * 比如以logback为例，如果配置ch.qos.logback.core.ConsoleAppender则会输出到catalina.out里。
  * Web应用日志：localhost.${date}.log
    * 程序异常没有被捕获的时候抛出的地方
    * Tomcat下内部代码丢出的日志（jsp页面内部错误的异常，org.apache.jasper.runtime.HttpJspBase.service类丢出的，日志信息就在该文件！）
    * 应用初始化(listener, filter, servlet)未处理的异常最后被Tomcat捕获而输出的日志，而这些未处理异常最终会导致应用无法启动。
  * HTTP访问日志：localhost_access_log.${date}.log
  * 管理日志：manager.${date}.log
  * 虚拟主机日志：host-manager.${date}.log
* temp：临时数据目录，存放Tomcat的临时文件，这个目录下的东西可以在停止Tomcat后删除。
* webapps：存放web项目的目录，其中每个文件夹都是一个项目。如果这个目录下已经存在了目录，那么都是Tomcat自带的项目。其中ROOT是一个特殊的项目，在地址栏中访问：`http://127.0.0.1:8080`，没有给出项目目录时，对应的就是ROOT项目。而“examples”项目（同名文件夹）通过`http://127.0.0.1:8080/examples`进行访问。
* work：工作目录，运行时生成的文件，最终运行的文件都在这里。
  * 当客户端用户访问一个JSP文件时，Tomcat会通过JSP生成Java文件，然后再编译Java文件生成class文件，生成的java和class文件都会存放到这个目录下。
  * 存放session钝化的文件

## 终端日志中文乱码问题
修改conf/logging.properties，将以.encoding为结尾后缀的配置项修从UTF-8改为GBK。

## WEB项目的标准发布结构
一个标准的可以用于发布的WEB项目标准结构如下。

![1681453620343](https://raw.githubusercontent.com/loli0con/picgo/master/1681453620343.png)

* app：Web应用根目录
  * **WEB-INF**：必要目录，必须叫WEB-INF，受保护的资源目录，浏览器无法通过url直接访问该目录。
    * classes：源代码和配置文件在编译后会存放于该目录下，web项目中如果没有源码，则该目录不会出现。
    * lib：项目依赖的jar在编译后会存放于该目录下，web项目要是没有依赖任何jar，则该目录不会出现。
    * web.xml：web项目的基本配置文件。较新的版本中可以不需要该文件。 
  * static：非必要目录（可通过url直接访问），约定俗成的名字，一般在此处放静态资源（css、js、img等）。
  * index.html：非必要文件（可通过url直接访问），index.html/index.htm/index.jsp为默认的欢迎页。

## WEB项目部署的方式
1. 直接将编译好的项目放在webapps目录下
2. 将编译好的项目打成war包放在webapps目录下，Tomcat启动后会自动解压war包
3. 可以将项目放在非webapps的其他目录下，在Tomcat中通过配置文件指向app的实际磁盘路径

### webapps真实目录发布
适合生产环境
#### 步骤
|发布位置|访问方式|
|---|---|
|$CATALINA_HOME/webapps/资源目录/资源文件|http://localhost:8080/资源目录/资源文件|
|$CATALINA_HOME/webapps/ROOT/资源目录|http://localhost:8080/资源文件|

### 虚拟目录发布
#### 步骤
1. 编辑$CATALINA_HOME/conf/server.xml文件
2. 在host标签里面增加Context标签节点
3. `<Context path="虚拟资源目录" docBase="真实发布资源目录"/>`
4. 通过 http://localhost:8080/虚拟资源目录/资源文件 访问

#### 优缺点
* 优点：解决webapps发布资源堆积问题
* 缺点：需要重启服务器

### 独立xml文件虚拟目录发布
适合开发环境
#### 步骤
1. 在$CATALINA_HOME/conf/Catalina/localhost下创建.xml配置文件
2. 配置文件的名字相当于"虚拟资源目录"
3. 配置文件的内容为`<Context docBase="真实发布资源目录"/>`

#### 优点
不需要重启服务器
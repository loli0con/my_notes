# Tomcat

## 目录结构
Tomcat的安装目录`$CATALINA_HOME`:
* bin - 命令目录
  * catalina.sh
  * startup.sh 启动
  * shudown.sh 停止
* conf - 配置目录
  * server.xml
    * 修改端口 <Connector port="端口号" ...>
    * 虚拟目录映射
  * web.xml
    * 欢迎页面的配置
* logs - [日志](http://xstarcd.github.io/wiki/Java/tomcat_log.html)目录
  * 服务器⽇志 - catalina.${date}.out
    * 是tomcat的标准输出(stdout)和标准出错(stderr)
    * 这是在tomcat的启动脚本里指定的，如果没有修改的话stdout和stderr会重定向到这里。所以我们在应用里使用System.out打印的东西都会到这里来。
    * 另外，如果我们在应用里使用其他的日志框架，配置了向Console输出的，则也会在这里出现。
    * 比如以logback为例，如果配置ch.qos.logback.core.ConsoleAppender则会输出到catalina.out里。
  * Web 应⽤⽇志 - localhost.${date}.log
    * 程序异常没有被捕获的时候抛出的地方
    * Tomcat下内部代码丢出的日志（jsp页面内部错误的异常，org.apache.jasper.runtime.HttpJspBase.service类丢出的，日志信息就在该文件！）
    * 应用初始化(listener, filter, servlet)未处理的异常最后被tomcat捕获而输出的日志，而这些未处理异常最终会导致应用无法启动。
  * HTTP 访问⽇志 - localhost_access_log.${date}.log
  * 管理日志 - manager.${date}.log
  * 虚拟主机日志 - host-manager.${date}.log
* lib - 依赖库目录
* temp - 临时数据目录
* work - 工作目录
  * 存放jsp翻译为Servlet的源码
  * 存放session钝化的文件
* webapps - 发布资源根目录/web工程部署目录

## 发布资源
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
3. \<Context path="虚拟资源目录" docBase="真实发布资源目录"/>
4. 通过 http://localhost:8080/虚拟资源目录/资源文件 访问

#### 优缺点
* 优点：解决webapps发布资源堆积问题
* 缺点：需要重启服务器

### 独立xml文件虚拟目录发布
适合开发环境
#### 步骤
1. 在$CATALINA_HOME/conf/Catalina/localhost下创建.xml配置文件
2. 配置文件的名字相当于"虚拟资源目录"
3. 配置文件的内容为 \<Context docBase="真实发布资源目录"/>

#### 优点
不需要重启服务器

## 资源结构
* webapp工程发布目录
  * 资源文件：jsp、html、css、图片等
  * WEB-INF：web应用目录（浏览器无法直接访问该目录）
    * classes文件目录
    * lib依赖目录
    * web.xml部署配置文件

## 组件
![tomcat+20210722094529](https://raw.githubusercontent.com/loli0con/picgo/master/images/tomcat%2B20210722094529.png%2B2021-07-22-09-45-31)
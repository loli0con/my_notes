# Tomcat

## 目录结构
Tomcat的安装目录`$CATALINA_HOME`:
* bin命令目录
  * catalina.sh
  * startup.sh 启动
  * shudown.sh 停止
* conf配置目录
  * server.xml
    * 修改端口
    * 虚拟目录映射
  * web.xml
    * 欢迎页面的配置
* logs日志目录
* webapps发布资源根目录

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
  * WEB-INF：web应用目录
    * classes文件目录
    * lib依赖目录
    * web.xml部署配置文件

## 组建
![tomcat+20210722094529](https://raw.githubusercontent.com/loli0con/picgo/master/images/tomcat%2B20210722094529.png%2B2021-07-22-09-45-31)
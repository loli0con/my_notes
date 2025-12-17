# Profile
<b>构建配置文件(Profile)</b>是一系列的配置项的值，可以用来设置或者覆盖Maven构建默认值。使用**构建配置文件**，你可以为不同的环境，比如说生产环境（Production）和开发（Development）环境，定制构建方式。

**Profile**在pom.xml文件中使用activeProfiles或者profiles元素指定，并且可以通过各种方式触发。**Profile**在构建时修改POM，并且用来给参数设定不同的目标环境（比如说，开发（Development）、测试（Testing）和生产环境（Production）中数据库服务器的地址）。

## Profile的类型
构建配置文件大体上有三种类型：
|类型|在哪定义|
|---|---|
|项目级（Per Project）|定义在项目的POM文件pom.xml中|
|用户级（Per User）|定义在Maven的设置xml文件中（%USER_HOME%/.m2/settings.xml）|
|全局（Global）|定义在Maven全局的设置xml文件中（%M2_HOME%/conf/settings.xml）|

## Profile激活
**Profile**可以让我们定义一系列的配置信息，然后指定其激活条件。这样我们就可以定义多个**Profile**，然后每个**Profile**对应不同的激活条件和配置信息，从而达到不同环境使用不同配置信息的效果。

**Profile**可以通过多种方式激活：
* 使用命令控制台输入显式激活
* 通过maven设置。
* 基于环境变量（用户或者系统变量）。
* 操作系统设置（比如说，Windows系列）。
* 文件的存在或者缺失。

### 通过命令行参数
构建配置文件采用的是`<profiles>`节点，其中`<id>`区分了不同的`<profiles>`执行不同的任务。通过命令行参数输入指定的id：`mvn 生命周期阶段 -Pmyid`，这个参数通过`-P`来传输。

### 通过Maven设置
配置 setting.xml 文件，增加`<activeProfiles>`属性：
```XML
<settings xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
   ...
   <activeProfiles>
      <activeProfile>myid</activeProfile>
   </activeProfiles>
</settings>
```

### 通过环境变量
在 pom.xml 里面的`<id>`为myid的`<profile>`节点，加入`<activation>`节点：
```XML
<profiles>
    <profile>
        <id>myid</id>
        <activation>
            <property>
                <name>env</name>
                <value>myenv</value>
            </property>
        </activation>
        ...
    </profile>
    ...
</profiles>   
```
执行命令：
1. 使用`-D`传递环境变量，执行命令`mvn 生命周期阶段 -Denv=myenv`，其中env对应刚才设置的`<name>`值，myenv对应`<value>`。
2. 用户/系统的环境变量

### 通过操作系统
activation元素包含下面的操作系统信息。当系统为 windows XP 时，myid Profile 将会被触发：
```XML
<profile>
   <id>test</id>
   <activation>
      <os>
         <name>Windows XP</name>
         <family>Windows</family>
         <arch>x86</arch>
         <version>5.1.2600</version>
      </os>
   </activation>
</profile>
```

### 通过文件的存在或者缺失
使用 activation 元素包含下面的信息：
```XML
<activation>
    <file>
        <!-- 如果指定的文件存在，则激活profile。 -->
        <exists>
            /usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/
        </exists>
        <!-- 如果指定的文件不存在，则激活profile。 -->
        <missing>
            /usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/
        </missing>
    </file>
</activation>
```
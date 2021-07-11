# Maven
Maven是一个软件**项目管理**及自动**构建**工具。

项目开发过程中会涉及到很多东西：项目描述、版本、构建、输出物、文档、项目关系、移植性、开发者列表、许可证、缺陷管理等，Maven能够我们管理这些。

经过大量的学习和总结，Maven提供了一套成熟的规则，对项目开发和构建时的方方面面进行了规范化。只要遵循这些规则，程序员就能节省大量的时间和精力。

## 项目

### 项目结构
Maven提倡使用一个标准的目录结构：
![maven+20210709151523](https://raw.githubusercontent.com/loli0con/picgo/master/images/maven%2B20210709151523.png%2B2021-07-09-15-15-25)

实际举例（${basedir}为“my-maven-project”）：
```
my-maven-project
├── pom.xml(项目的核心配置文件)
├── src
│   ├── main
│   │   ├── java(java源代码目录)
│   │   └── resources(资源文件目录)
│   └── test
│       ├── java(测试代码目录)
│       └── resources(测试资源文件目录)
└── target(输出目录，所有的输出物都存放在这个目录下)
```

### POM
Maven将项目开发和管理过程抽象成一个项目对象模型（POM），正对应着上文里的pom.xml文件。也就是说，一个pom.xml定义了一个Maven项目。POM包含了项目的基本信息，它用于描述项目如何构建、声明项目依赖等等。

在Maven的安装目录中，有一个预设的、顶级的pom，类似Java里的Object。所有的Maven项目都会继承这个pom.xml，所以通常把这个pom叫超级pom。

### pom.mxl
项目描述文件pom.xml，它的内容大致如下：
```xml
<project xmlns = "http://maven.apache.org/POM/4.0.0"
    xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
 
    <!-- 模型版本 -->
    <modelVersion>4.0.0</modelVersion>


    <!-- 公司或者组织的唯一标志，并且配置时生成的路径也是由此生成， 如com.companyname.project-group，maven会将该项目打成的jar包放本地路径：/com/companyname/project-group -->
    <groupId>com.companyname.project-group</groupId>
 
    <!-- 项目的唯一ID，一个groupId下面可能多个项目，就是靠artifactId来区分的 -->
    <artifactId>project</artifactId>
 
    <!-- 工程的版本号，用来区分不同的版本 -->
    <version>1.0</version>

    <!-- 项目依赖，一个项目通常会依赖其他的项目，也就是说项目之间可能会存在依赖关系 -->
    <dependencies>
        <!-- 其中一个依赖 -->
        <dependency>
            <!-- junit单元测试 -->
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
         </dependency>
    </dependencies>

</project>
```

可以通过阅读[POM标签依赖大全](https://www.runoob.com/maven/maven-pom.html)获取更多知识，但通常我们会按需获取。

### 创建项目
想要创建一个Maven项目，最原始的方式便是按照上述目录结构手动创建，不借助任何的工具。

其次还可以通过使用命令的方式来创建：
* 使用命令向导一步步创建项目
   1. 在硬盘上创建一个空的目录，用来存放Maven项目
   2. 打开命令行窗口，切换到该目录
   3. 输入“mvn archetype:generate”，按 Enter 键。
   4. 会一步步提示输入 groupId、artifactId、version、package 等信息
* 在命令中输入所有必要信息直接创建项目
   1. 在硬盘上创建一个空的目录，用来存放 Maven 项目
   2. 打开命令行窗口，切换到该目录
   3. 输入“mvn archetype:generate -DgroupId=mygroup -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false”

另外也能通过idea等IDE开发工具来创建Maven项目，本质上是同理的。

主要注意的地方：
1. 第一次Maven使用可能需要初始化
2. 有些开发工具会内置Maven（当然你知道我在说idea）
3. 创建一个简单的项目要记得“archetype”为“quickstart”（快速开始）


## 生命周期
### 构建
构建就是「把工程/项目经过编译～得到的编译结果～部署到服务器上」的一整个过程，或者说就是程序员写完代码到项目上线运行的一整个过程。

主要有下面几个环节：
1. 清理clean：将以前编译得到的旧文件class字节码文件删除
2. 编译compile：将java源程序编译成class字节码文件
3. 测试test：自动测试，自动调用junit程序
4. 报告report：测试程序执行的结果
5. 打包package：动态Web工程打War包，java工程打jar包
6. 安装install：Maven特定的概念——将打包得到的文件复制到“仓库”中
7. 部署deploy：将工程生成的包复制到容器下，使其运行

### 引入
在 Maven 之前，项目构建的生命周期概念就已经存在了。软件开发人员每天都要对项目进行清理、编译、测试、打包以及安装部署。

通过学习、分析、反思和总结以前工作中对项目的构建过程，Maven 抽象出了一个适合于所有项目的构建生命周期，并将它们统一规范。

需要注意的是，**Maven 中项目的构建生命周期只是 Maven 根据实际情况抽象提炼出来的一个统一*标准和规范***，是不能做具体事情的。也就是说，Maven 没有提供一个编译器能在编译阶段编译源代码。 Maven 只是规定了生命周期的各个阶段和步骤，**具体事情，由集成到 Maven 中的插件完成**。比如前面创建项目，就是由 maven-archetype-quickstart 插件完成的。

### 三个生命周期
Maven 拥有三套独立的生命周期，它们分别是 clean、default 和 site。clean 生命周期的目的是清理项目；default 生命周期的目的是构建项目；site 生命周期的目的是建立项目站点。

每个生命周期又包含了多个**阶段**，这些阶段在执行的时候是**有固定顺序**的，后面的阶段一定要等前面的阶段执行完成后才能被执行

#### clean 生命周期
clean 生命周期的目的是清理项目，它包括以下三个阶段：
1. pre-clean：执行清理前需要完成的工作。
2. clean：清理上一次构建过程中生成的文件，比如编译后的 class 文件等。
3. post-clean：执行清理后需要完成的工作。

#### default 生命周期
default 生命周期定义了构建项目时所需要的执行步骤，它是所有生命周期中最核心部分，包含的阶段如下表所述，比较常用的阶段用粗体标记。
|名称|说明|
|-----|-----|
|validate（校验）|验证项目结构是否正常，必要的配置文件是否存在|
|initialize（初始化）|做构建前的初始化操作，比如初始化参数、创建必要的目录等|
|generate-sources（生成源代码）|产生在编译过程中需要的源代码|
|process-sources（处理源代码）|处理源代码，比如过滤值|
|generate-resources（生成资源文件）|生成将会包含在项目包中的资源文件|
|process-resources（处理资源文件）|	复制和处理资源到目标目录，为打包阶段最好准备|
|**compile（编译）**|编译项目中的源代码|
|process-classes（处理类文件）|处理编译生成的文件，比如说对Java class文件做字节码改善优化|
|generate-test-sources（生成测试源代码）|产生编译过程中测试相关的代码|
|process-test-sources（处理测试源代码）|处理测试源代码，比如说，过滤任意值|
|generate-test-resources（生成测试资源文件）|为测试创建资源文件|
|process-test-resources（处理测试资源文件）|复制和处理测试资源到目标目录|
|test-compile（编译测试源码）|编译测试源代码到测试目标目录|
|process-test-classes（处理测试类文件）|处理测试源码编译生成的文件|
|**test（测试）**|使用合适的单元测试框架运行测试（Juint是其中之一）|
|prepare-package（准备打包）|处理打包前需要初始化的准备工作|
|**package（打包）**|将编译后的 class 和资源打包成可分发格式的文件，比如JAR、WAR或者EAR文件|
|pre-integration-test（集成测试前）|做好集成测试前的准备工作，比如集成环境的参数设置|
|integration-test（集成测试）|处理和部署项目到可以运行集成测试环境中|
|post-integration-test（集成测试后）|完成集成测试后的收尾工作，比如清理集成环境的值|
|verify（验证）|运行任意的检查来验证项目包有效且达到质量标准|
|**install（安装）**|将打包的组件以构件的形式，安装到本地依赖仓库中，以便共享给本地的其他项目，用作其他本地项目的依赖|
|**deploy**|将最终的项目包复制到远程仓库中与其他开发者和项目共享|

这些阶段的详细介绍内容可以参考链接：  
http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html

#### site 生命周期
site 生命周期的目的是建立和发布项目站点。Maven 可以基于 pom 所描述的信息自动生成项目的站点，同时还可以根据需要生成相关的报告文档集成在站点中，方便团队交流和发布项目信息。site 生命周期包括如下阶段：
1. pre-site：执行生成站点前的准备工作。
2. site：生成站点文档。
3. post-site：执行生成站点后需要收尾的工作。
4. site-deploy：将生成的站点发布到服务器上。


## 常用命令
执行maven命令必须进入到pom.xml的目录中进行执行，通过这些命令让 Maven 执行生命周期的特定阶段。常用命令如下：
1. mvn clean：清理，这个命令可以用来清理已经编译好的文件
2. mvn compile：编译主程序，将 Java 代码编译成 Class 文件
3. mvn test：执行单元测试测试，打印测试结果
4. mvn package：打包，根据用户的配置将项目打成 jar 包或者 war 包
5. mvn install：安装，手动向本地仓库安装一个 jar 包供其他项目共享使用


## 仓库
在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称之为**构件**。

在 Maven 的术语中，仓库是一个位置（place）。  
这个用于存放构建的位置就叫仓库，它帮我们管理这些构件。

Maven 仓库有三种类型：
* 本地（local）
* 远程（remote）
  * 中央（central）
  * 公共仓库
  * 私服

### 本地仓库
Maven 的本地仓库，在安装 Maven 后并不会创建，它是在第一次执行 maven 命令的时候才被创建。默认位置在`～/.m2/repository`，即`用户目录`下`.m2`文件夹下的`respository`文件夹。

一个构件只有存在本地仓库后才能被 Maven 项目使用。将构件保存到本地仓库最常见的有两种方式，一种是以依赖的形成，从远程仓库下载到本地仓库；另一种是将本地项目编译打包后，安装到本地仓库。

### 中央仓库
Maven 中央仓库是由 Maven 社区提供的仓库，其中包含了大量常用的库。
中央仓库包含了绝大多数流行的开源Java构件，以及源码、作者信息、SCM、信息、许可证信息等。一般来说，简单的Java项目依赖的构件都可以在这里下载到。

### 远程仓库
如果 Maven 在中央仓库中也找不到依赖的文件，它会停止构建过程并输出错误信息到控制台。为避免这种情况，Maven 提供了远程仓库的概念，它是开发人员自己定制仓库，包含了所需要的代码库或者其他工程中用到的 jar 文件。

举例说明，使用下面的 pom.xml，Maven 将从远程仓库中下载该 pom.xml 中声明的所依赖的（在中央仓库中获取不到的）文件。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.companyname.projectgroup</groupId>
   <artifactId>project</artifactId>
   <version>1.0</version>
   
   <!-- 远程仓库配置 -->
   <repositories>
      <repository>
        <id>companyname.lib1</id>
        <url>http://download.companyname.org/maven2/lib1</url>
      </repository>
      <repository>
        <id>companyname.lib2</id>
        <url>http://download.companyname.org/maven2/lib2</url>
        <releases>
          <enabled>true</enabled>
        </releases>
        <snapshots>
          <enabled>false</enabled>
        </snapshots>
        <layout>default</layout>
      </repository>
   </repositories>
</project>
```
在 pom 中可以配置多个仓库，每个仓库的 id 都要是唯一的。而且需要注意的是，在 Maven 的超级 pom 中，已经默认配置了一个中央仓库，它的 id 为 central。

### 依赖搜索顺序
当我们执行 Maven 构建命令时，Maven 开始按照以下顺序查找依赖的库：
1. 在本地仓库中搜索，如果找不到，执行步骤 2，如果找到了则执行其他操作。
2. 在中央仓库中搜索，如果找不到，并且有一个或多个远程仓库已经设置，则执行步骤 4，如果找到了则下载到本地仓库中以备将来引用。
3. 如果远程仓库没有被设置，Maven 将简单的停滞处理并抛出错误（无法找到依赖的文件）。
4. 在一个或多个远程仓库中搜索依赖的文件，如果找到则下载到本地仓库以备将来引用，否则 Maven 将停止处理并抛出错误（无法找到依赖的文件）。

### 镜像仓库
如果仓库 A 能提供仓库 B 存储的所有服务，那么就把 A 叫作 B 的镜像。  
由于地理位置的因素，该镜像往往能够提供比中央仓库更快的服务。所以，为了提高 Maven 效率，可以通过配置文件用镜像代替。

修改 maven 根目录下的 conf 文件夹中的 settings.xml 文件，在 mirrors 节点上，添加内容如下：
```xml
<mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <!-- 仓库地址 -->
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <!-- 仓库类型 -->
      <mirrorOf>central</mirrorOf>        
    </mirror>
</mirrors>
```
![maven+20210709195834](https://raw.githubusercontent.com/loli0con/picgo/master/images/maven%2B20210709195834.png%2B2021-07-09-19-58-35)

### 仓库搜索服务
在实际开发过程中，用户可能只知道需要使用的构件项目名称，但是在 Maven 依赖配置中，一定要指定详细的坐标信息。这时候，就可以使用仓库的搜索服务，根据构件的关键字来查找 Maven 的详细坐标。

一个常用的 Maven 仓库搜索服务：[MVNRepository](https://mvnrepository.com/)。
它提供基于关键字搜索、依赖声明代码片段、构件下载、构件所包含信息等功能。


## 坐标
Maven通过坐标管理每个构件。  
在 Maven 仓库中，通过坐标精确地找到所需的构件。

一个完整的坐标信息，由 groupId、artifactId、version、packaging、classifier 组成。组成坐标的 5 个要素中，groupId、artifactId 和 version 是必需的，packaging 是可选的，默认是 jar，而 classifier 是不能直接定义的。

Maven 项目的构件文件名与坐标也是有对应关系的，一般规则是 groupId/artifactId/version/artifactId-version\[-classifier].packaging。

如下是一个简单的坐标定义：
```xml
<groupId>org.SpringFramework</groupId>
<artifactId>spring-core</artifactId>
<version>4.2.7.RELEASE</version>
<packaging>jar</packaging>
```

### groupId
定义当前 Maven 项目从属的实际项目。groupId 的表述形式同 Java 包名的表述方式类似，通常与域名反向一一对应。

Maven 项目和实际项目不一定是一一对应的。比如 SpringFramework，它对应的 Maven 项目就有很多，如 spring-core、spring-context、spring-security 等。造成这样的原因是模块的概念，所以一个实际项目经常会被划分成很多模块。

### artifactId
定义实际项目中的一个 Maven 项目（实际项目中的一个模块）。

推荐命名的方式为：实际项目名称-模块名称。

比如，org.springframework 是实际项目名称，而现在用的是其中的核心模块（core的中文翻译），它的 artifactId 为 spring-core。

### version
定义 Maven 当前所处的版本。如上的描述，用的是 4.2.7.RELEASE 版本。需要注意的是，Maven 中对版本号的定义是有一套规范的。具体规范请参考《[版本管理](http://c.biancheng.net/view/4828.html)》的介绍。

### packaging
定义 Maven 项目的打包方式。

打包方式通常与所生成的构件文件的扩展名对应，比如，.jar、.ear、.war、.pom 等。另外，打包方式是与工程构建的生命周期对应的。比如，jar 打包与 war 打包使用的命令是不相同的。最后需要注意的是，可以不指定 packaging，这时候 Maven 会自动默认成 jar。

### classifier
定义构件输出的附属构件。

附属构件同主构件是一一对应的，比如上面的 spring-core-4.2.7.RELEASE.jar 是 spring-core Maven spring-core 项目的主构。

Maven spring-core 项目除了可以生成上面的主构件外，也可以生成 spring-core-4.2.7.RELEASE-javadoc.java 和 spring-core-4.2.7.RELEASE-sources.jar 这样的附属构件。这时候，javadoc 和 sources 就是这两个附属构件的 classifier。这样就为主构件的每个附属构件也定义了一个唯一的坐标。

最后需要特别注意的是，不能直接定义一个 Maven 项目的 classifier，因为附属构件不是由 Maven 项目构建的时候直接默认生成的，而是由附加的其他插件生成的。


## 插件
在Maven生命周期中都包含着一系列的阶段。这些阶段就相当于Maven提供的统一的接口，然后这些阶段的实现由 Maven 的插件来完成。

例如输入`mvn clean`命令，clean对应clean生命周期中的clean阶段，该阶段的具体操作是由 maven-clean-plugin 插件来实现的。

Maven 实际上是一个依赖插件执行的框架，每个任务实际上是由插件完成。

但是从插件本身来说，一个插件可以实现生命周期多个阶段的任务，为方便指定执行插件的某个功能，将插件的每个功能叫目标。这样就可以实现*在哪个阶段，执行哪个插件，达到哪个目标*。

插件通常提供了一个目标的集合,这些目标可能被绑定到多个阶段或者无绑定，不绑定到任何构建阶段的目标可以在构建生命周期之外通过直接调用执行。

在给不同的生命周期阶段绑定不同的插件目标后，这些目标的执行自然是按阶段的顺序逐个执行。如果在一个阶段上绑定了多个目标则按插件声明的顺序执行。

### 插件类型
Maven 提供了下面两种类型的插件：
* Build plugins：在构建时执行，并在 pom.xml 的 元素中配置。
* Reporting plugins：在网站生成过程中执行，并在 pom.xml 的 元素中配置。

### 常用插件
下面是一些常用插件的列表：
* clean：构建之后清理目标文件。删除目标目录。
* compiler：编译 Java 源文件。
* surefile：运行 JUnit 单元测试。创建测试报告。
* jar：从当前工程中构建 JAR 文件。
* war：从当前工程中构建 WAR 文件。
* javadoc：为工程生成 Javadoc。
* antrun：从构建过程的任意一个阶段中运行一个 ant 任务的集合。

### 绑定
Maven的生命周期和插件需要实现相互绑定，让 Maven 构建工程时自动调用插件完成指定的任务。实现生命周期的阶段同插件目标的绑定，一共有两种方式：内置绑定和自定义绑定。

#### 内置绑定
Maven 在安装好后，自动为生命周期的主要阶段绑定很多插件的目标。当用户通过命令或图形界面执行生命周期的某个阶段时，对应的插件目标就会自动执行，从而完成任务。

#### 自定义绑定
除了 Maven 内置的绑定外，也可以指定在某个阶段绑定某个插件的某个目标，这样就使得 Maven 在构建项目时能执行更多的任务。这样的任务，Maven 没有内置绑定到生命周期的阶段上,所以这就需要用户自己配置了。这样的配置可以加在 pom.xml 中。

##### 例子
使用maven-antrun-plugin插件把数据输出到控制台上。
```xml
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.companyname.projectgroup</groupId>
    <artifactId>project</artifactId>
    <version>1.0</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.1</version>
                <executions>
                    <execution>
                        <id>id.clean</id>
                        <phase>clean</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <!-- 插件运行参数，局部参数 -->
                        <configuration>
                            <tasks>
                                <echo>clean phase</echo>
                            </tasks>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
打开命令终端跳转到 pom.xml 所在的目录，并执行下面的 mvn 命令：
```shell
mvn clean
```
Maven 将开始处理并显示 clean 生命周期的 clean 阶段：
```log
[INFO] Scanning for projects...
[INFO] ------------------------------------------------------------------
[INFO] Building Unnamed - com.companyname.projectgroup:project:jar:1.0
[INFO]    task-segment: [post-clean]
[INFO] ------------------------------------------------------------------
[INFO] [clean:clean {execution: default-clean}]
[INFO] [antrun:run {execution: id.clean}]
[INFO] Executing tasks
     [echo] clean phase
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------
[INFO] BUILD SUCCESSFUL
[INFO] ------------------------------------------------------------------
[INFO] Total time: < 1 second
[INFO] Finished at: Sat Jul 07 13:38:59 IST 2012
[INFO] Final Memory: 4M/44M
[INFO] ------------------------------------------------------------------
```
上面的例子展示了以下关键概念：
* 插件是在 pom.xml 中使用 plugins 元素定义的。
* 每个插件可以有多个目标。
* 可以定义阶段，插件会使用它的 phase 元素开始处理。例子中使用了 clean 阶段。
* 可以通过绑定到插件的目标的方式来配置要执行的任务。例子中绑定了 echo 任务到 maven-antrun-plugin 的 run 目标。
* Maven 将处理剩下的事情，它将下载本地仓库中获取不到的插件，并开始处理。

### 参数配置
通过命令行和 pom 配置两种方式给这些目标设置参数值。
#### 命令行配置参数
在 Maven 命令中，使用 -D后面接参数名称＝参数值的方式配置目标参数。

比如，maven-surefire-plugin 插件中提供了一个 maven.test.skip 参数，当它的值为 true 时，就不会执行 test 案例。具体语法是：
```shell
mvn install -Dmaven.test.skip=true
```

#### pom 配置参数
上面的示例maven-antrun-plugin示例中已经展示过了在pom中配置局部参数的方法，下面的示例展示的是配置全局参数的方法。
```xml
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
    http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.companyname.projectgroup</groupId>
    <artifactId>project</artifactId>
    <version>1.0</version>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>1.1</version>
                <!-- 插件运行参数，全局参数 -->
                <configuration>
                    <tasks>
                        <echo>clean phase</echo>
                    </tasks>
                </configuration>
                <executions>
                    <execution>
                        <id>id.clean</id>
                        <phase>clean</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```


### 获取插件信息
#### 在线查找插件
插件基本上都来源于两处，一个是 Apache（官方）；另一个是 Codehaus（三方）。

#### 使用 maven-help-plugin 查看插件
语法如下，可以选择查询整个插件或某个目标，并指定是否要查看详情。
```shell
mvn help:describe -dplugin=插件 -Dgoal=目标 -Ddetail
```

### 调用插件
可以直接在命令行执行插件，例如前面的查看插件信息的命令，就是在调用 help 插件的 describe 目标来完成查看任务的。

使用命令行执行 Maven 插件的语法如下：
```shell
maven <插件名称|前缀>:<目标> [-D参数名=参数值 ...]
```

### 解析插件
在输入 mvn compiler:compile 命令用到的是 compiler 插件。这里有没有感觉到疑惑，使用插件的话一般要指定插件的坐标信息（groupId、artifactId、version）才能唯一指定一个插件，为什么这里只是输入了 compiler 就可以指定使用的是 maven-compiler-plugin 插件呢？

其实这是 Maven 为了方便用户提供的一种简单方式，可以使用插件的前缀来指定插件。接下来详细介绍一下 Maven 的运行机制，来从本质上把握 mvn 命令的语法。

#### 插件仓库
同依赖构件一样，插件构件也是基于坐标存储在 Maven 仓库中的。在需要时，Maven 先从本地仓库中查找插件，如果没有，就从远程仓库查找。找到插件后，下载到本地仓库使用。

#### 插件默认的 groupId
在使用插件或者在 pom 中配置插件的时候，如果使用的是 Maven 官方插件的话，是可以不指定 groupId 的，因为这些插件的 groupId 都是一样的，都是 org.apache.maven.plugins。
如果没有配置指定 groupId 的话，Maven 默认认为是 org.apache.maven.plugins。

#### 解析插件的版本
如果没有指定插件的版本，Maven 对版本处理的方式是：如果插件不属于核心插件范畴，Maven 会去检测所有仓库中的版本，最终会选择最新版本，而且这个最新版本不排除是快照版本。

#### 解析插件的前缀
插件前缀与 groupId:artifactId 是一一对应的（也就是知道插件前缀，就可以找到groupId和artifactId）。这种对应关系保存在仓库的元数据中，这样的元数据为 groupId/maven-metadata.xml。

目前绝大部分插件都是放在 http://repo1.maven.org 和 http://repository.codehaus.org 中的，它们的 groupId 对应的是 org.apache.maven.plugins 和 org.codehaus.mojo。

Maven 在解析插件仓库元数据的时候，会默认使用 org.apache.maven.plugins 和 org.codehaus.mojo 两个 groupId，也就是说，Maven 会自动检测 http://repo1.maven.org/maven2/org/apache/maven/plugins/maven-metadata.xml和 http://repository.codehaus.org/org/codehaus/mojo/maven-metadata.xml 中的元数据。

插件仓库数据的内容是大概是这样的：
```xml
<!-- org.apache.maven.plugins 中 groupId 部分的内容 -->
<metadata>
    <plugins>
        <plugin>
            <name>Maven Clean Plugin</name>
            <prefix>clean</prefix>
            <artifactId>maven-clean-plugin</artifactId>
        </plugin>
        <plugin>
            <name>Maven Compiler Plugin</name>
            <prefix>compiler</prefix>
            <artifactId>maven-compiler-plugin</artifactId>
        </plugin>
        <plugin>
            <name>Maven Dependency Plugin</name>
            <prefix>dependency</prefix>
            <artifactId>maven-dependency-plugin</artifactId>
        </plugin>
        ...
    </plugins>
</metadata>
```
从上面的内容中可以发现，maven-clean-plugin 的前缀是 clean，maven-compiler-plugin 的前缀是 compiler，maven-dependency-plugin 的前缀是 dependency。

当 Maven 解析到 compiler:compile 命令后，它首先基于默认的 groupId 归并所有插件仓库的元数据 org/apache/maven/plugins/maven-metadata.xml。接着检查归并后的元数据，找到对应的 artifactId 为 maven-compiler-plugin。

接下来再结合当前元数据的 groupId 为 org.apache.maven.plugins。最后找到仓库中最新的 version，从而就可以得到一个插件的完整坐标信息。

如果在第一个 metadata.xml 中没有找到目标插件，就用同样的流程找其他的 metadata.xml，包括用户自己定义的 metadata.xml。如果所有的地方都没有找到对应的前缀，这就以报错的形式结束了。

## 依赖
当我们开始做一个新项目的时候，通过直接指定坐标，让Maven把对应的构件从仓库中找出来，集成到新项目中。这些被引入的构件，就是新项目的依赖。

### 依赖的配置
依赖是配置在 pom.xml 中的，如下是关于依赖配置的大概内容：
```xml
<project>
    ...
    <dependencies>
        <dependency>
            <groupId>...</groupId>
            <artifactId>
                ...
            </artifactId>
            <version>...</version>
            <type>...</type>
            <scope>...</scope>
            <optional>...</optional>
            <exclusions>
                <exclusion>...</exclusion>
            </exclusions>
        </dependency>
        ...
    </dependencies>
    ...
</project>
```

依赖配置中除了构件的坐标信息、groupId、artifactId 和 version 之外，还有其他的元素，这些元素的作用：
* groupId、artifactId 和 version：依赖的基本坐标。对于任何依赖，基本坐标是最基本、最重要的，因为 Maven 是根据坐标找依赖的。
* type：依赖的类型，同项目中的 packaging 对应。大部分情况不需要声明，默认是 jar。
* scope：依赖的范围
* optional：标记依赖是否可选
* exclusions：排除传递性依赖

### 依赖的范围
Java 中有个环境变量叫 classpath。JVM 运行代码的时候，需要基于 classpath 查找需要的类文件，才能加载到内存执行。

Maven 在编译项目主代码的时候，使用的是一套 classpath，主代码编译时需要的依赖就添加到这个 classpath 中去；Maven 在编译和执行测试代码的时候，又会使用一套 classpath，这个动作需要的依赖就添加到这个 classpath 中去；Maven 项目具体运行的时候，又有一个独立的 classpath，同样运行时需要的依赖，肯定也要加到这个 classpath 中。这些 classpath，就是依赖的范围。

依赖的范围，就是用来控制这三种 classpath 的关系（编译 classpath、测试 classpath 和运行 classpath），接下来分别介绍依赖的范围的名称和意义。

#### compile
编译依赖范围。如果在配置的时候没有指定，就默认使用这个范围。使用该范围的依赖，对编译、测试、运行三种 classpath 都有效。

#### test
测试依赖范围。使用该范围的依赖只对测试 classpath 有效，在编译主代码或运行项目的时候，这种依赖是无效的。

#### provided
已提供依赖范围。使用此范围的依赖，只在编译和测试 classpath 的时候有效，运行项目的时候是无效的。比如 Web 应用中的 servlet-api，编译和测试的时候就需要该依赖，运行的时候，因为容器中自带了 servlet-api，就没必要使用了。如果使用了，反而有可能出现版本不一致的冲突。

#### runtime
运行时依赖范围。使用该范围的依赖，只对测试和运行的 classpath 有效，但在编译主代码时是无效的。比如 JDBC 驱动实现类，就需要在运行测试和运行主代码时候使用，编译的时候，只需 JDBC 接口就行。

#### system
系统依赖范围。该范围与 classpath 的关系，同 provided 一样。但是，使用 system 访问时，必须通过 systemPath 元素指定依赖文件的路径。因为该依赖不是通过 Maven 仓库解析的，建议谨慎使用。

如下代码是一个使用 system 范围的案例。
```xml
<dependency>
    <groupId>xxx</groupId>
    <artifactId>xxx</artifactId>
    <version>xx</version>
    <scope>system</scope>
    <systemPath>e:/xxxx/xxx/xx.jar</systemPath>
</dependency>
```

#### import
导入依赖范围。该依赖范围不会对三种 classpath 产生实际的影响。它的作用是将其他模块定义好的 dependencyManagement 导入当前 Maven 项目 pom 的 dependencyManagement 中。比如有个 SpringPOM Maven 工程，它的 pom 中的 dependencyManagement 配置如下：
```xml
<project>
    ...
    <groupId>cn.com.mvn.pom</groupId>
    <artifactId>SpringPOM</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    ...
    <dependencyManagement>
        <dependencies>
            <!-- spring -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${project.build.spring.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${project.build.spring.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${project.build.spring.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    ...
</project>
```

接下来创建一个新的 Maven 工程 Second，要将 First 工程中 pom 中定义的 dependency-Management 原样合并过来，除了复制、继承之外，还可以编写如下代码，将它们导入进去。
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>cn.com.mvn.pom</groupId>
            <artifactId>SpringPOM</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 依赖的传递
Maven 会解析项目中的每个直接依赖，然后将那些必要的间接依赖以传递依赖的形式引入项目中。
传递依赖在将间接依赖引入项目的过程中也有它自己的规则和范围。

现在有三个项目（A、B 和 C 项目），假设 A 依赖 B，B 依赖 C，这样把 A 对 B 的依赖叫第一直接依赖，B 对 C 的依赖叫第二直接依赖，而 A 对 C 的依赖叫传递依赖（通过 B 传递的）。

中间 A 到 B 第一直接依赖的范围和 B 到 C 第二直接依赖的范围，就共同决定了 A 到 C 的传递依赖范围。它们的影响效果，就如表 1 所示。

![maven+20210710182305](https://raw.githubusercontent.com/loli0con/picgo/master/images/maven%2B20210710182305.png%2B2021-07-10-18-23-06)

### 依赖的调解
当多个直接依赖都带来了同一个间接依赖，而且是不同版本的间接依赖时，就会引起重复依赖的问题，因此需要进行调解。

#### 依赖调解原则
Maven 依赖调解原则有两个：一个是路径优先原则；另一个是声明优先原则。当路径优先原则搞不定的时候，再使用声明优先原则。

##### 路径优先原则
比如有个项目 A，它有两个依赖：A→B→C→T（1.0），A→D→T（2.0）。会发现，A 最终对 T（1.0）和 T（2.0）都有间接依赖。这时候 Maven 会自动判断它的路径，发现 T（2.0）的路径长度为 2，T（1.0）的路径长度为 3，以最短路径为原则，将 T（2.0）引入当前项目 A。

##### 声明优先原则
如果有个项目 A，它有两个依赖：A→B→T（1.0），A→C→T（2.0）。这时候两条路径都是一样的长度 2，Maven 会判断哪个依赖在 pom.xml 中先声明，选择引入先声明的依赖。

### 依赖的排除
在依赖的配置里面添加 exclusions→exclusion 元素，指定要排除依赖的 groupId 和 artifactId，这样可以排除依赖。
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>${project.build.hibernate.version}</version>
    <exclusions>
        <exclusion>
            <groupId>xxx</groupId>
            <artifactId>xxx</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 依赖的优化
在软件开发过程中，需要通过重构等方式不断优化代码，使其变得更简洁、灵活、高效。同样，程序员也应该对 Maven 项目的依赖了然于胸，并对其进行优化。因此程序员必须对项目中的依赖有全面的了解，这样才能更有效地达到目的。

可以使用如下的命令查看依赖：
* mvn dependency:list，列出所有的依赖列表。
* mvn dependency:tree，以树形结构方式，列出依赖和层次关系。
* mvn dependency:analyze，分析主代码、测试代码编译的依赖。

## 属性
在 Maven 中一共有六类属性。

### 内置属性
在 Maven 中主要有两个常用的内置属性，它们分别是＄{basedir}和＄{version}属性。＄{basedir}表示项目的根目录，也就是包含 pom.xml 文件的目录；＄{version}表示项目的版本。

### POM 属性
用户可以通过 POM 属性，引用 POM 文件中对应元素的值，比如＄{project.artifactId}就对应 project→artifactId 元素的值。常用的 POM 属性包括以下方面：
* ＄{project.build.sourceDirectory}：项目的主源码目录，默认是 src/main/java。
* ＄{project.build.testSourceDirectory}：项目的测试源码目录，默认是 src/test/java。
* ＄{project.build.directory}：项目构建输出目录，默认是 target。
* ＄{project.outputDirectory}：项目主代码编译输出目录，默认是 target/classes。
* ＄{project.testOutputDirectory}：项目测试代码编译输出目录，默认是 target/testclasses。
* ＄{project.groupId}：项目的 groupId。
* ＄{project.artifactId}：项目的 artifactId。
* ＄{project.version}：项目的版本。
* ＄{project.build.finalName}：项目输出的文件名称，默认为“＄{project.artifactId}-＄{project.version}”。

这些属性都在 pom 中有对应的元素。它们中一些属性的默认值都在超级 pom 中有定义，详细情况可以参考超级 pom.xml。

### 自定义属性
用户可以在 pom 的 properties 中定义自己的 Maven 属性，然后在后面重复使用。

### Settings 属性
Settings 属性同 POM 属性是一样的，可以用以“settings.”开头的属性引用 settings.xml 文件中 XML 元素的值。如使用“＄{settings.localRepository}”指向用户本地仓库的地址。

### Java 系统属性
所有的 Java 系统属性都可以通过 Maven 属性引用，比如“＄{user.home}”指向的就是用户目录。用户可以通过使用“mvn help:system”命令查看所有的 Java 系统属性。

### 环境变量属性
所有的环境变量都可以用以“evn.”开头的 Maven 属性引用。比如，“＄{evn.JAVA_HOME}”就指向引用了 JAVA_HOME 环境变量的值。同查看 Java 系统属性一样，用户可以使用命令“mvn help:system”查看到所有的环境变量。

### 在资源中使用属性
在 Maven 中，对资源文件进行处理的工作是由 maven resource plugin 插件完成的，它默认的工作是将项目主资源文件复制到主代码编译输出目录中，将测试资源文件复制到测试代码编译输出目录中。

通过修改的 pom 配置，让 maven resource plugin 插件能解析资源文件中的 Maven 属性，也就是开启资源过滤功能。为了开启资源过滤功能，需要在 pom 中添加一个 filtering 配置，代码如下：
```xml
<resources>
    <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
<testResources>
    <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
        <filtering>true</filtering>
    </testResource>
</testResources>
```
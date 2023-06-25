# Dockerfile
![DockerFile+20230217153457](https://raw.githubusercontent.com/loli0con/picgo/master/images/DockerFile%2B20230217153457.png%2B2023-02-17-15-34-58)

## 基本定义
Dockerfile是用来构建docker镜像的构建文件，是由一系列命令和参数构成的脚本

## 语法要求
1. 每条保留字指令都**必须为大写字母**且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

## 构建流程
1. 编写Dockerfile文件：Dockerfile是软件的原材料 
2. docker build命令构建镜像：Docker镜像是软件的交付品 
   1. docker从基础镜像运行一个容器
   2. 执行一条指令并对容器作出修改
   3. 执行类似docker commit的操作提交一个新的镜像层
   4. docker再基于刚提交的镜像运行一个新容器
   5. 执行dockerfile中的下一条指令直到所有指令都执行完成
3. docker run依照镜像运行容器实例：Docker容器则可以认为是软件镜像的运行态




## 保留关键字
|BUILD构建阶段|BOTH|RUN运行阶段|
|:---:|:---:|:---:|
|FROM|WORKDIR|CMD|
|MAINTAINER|USER|ENV|
|COPY||EXPOSE|
|ADD||VOLUME|
|RUN||ENTRYPOINT|
|ONBUILD|||
|.dockerignore|||

1. FROM：基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from
2. MAINTAINER：镜像维护者的信息（名字和邮箱地址）
3. RUN：容器构建时需要运行的命令
   1. 格式1: `RUN 命令行命令`
   2. 格式2: `RUN [可执行文件,参数1,参数2,...]`
4. EXPOSE：指定当前容器对外暴露除的端口
5. WORKDIR：指定在创建容器后，终端默认登录进来的工作目录，一个落脚点
6. ENV：用来在构建镜像过程中设置环境变量信息，格式`ENV 变量名 变量值`，这个环境变量可以在后续的其他指令（如RUN）中使用
7. ADD：将宿主机目录下的文件拷贝进镜像，且ADD命令会自动处理URL和解压tar压缩包
8. COPY：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件或者目录复制到新的一层的镜像内的<目标路径>位置
   1. 格式1: `COPY src dest`
   2. 格式2: `COPY ["src","dest"]`
   3. src：源文件或者源目录
   4. dest：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建
9.  VOLUME：容器数据卷，用于数据保存和持久化工作
10. CMD：指定一个容器启动时要运行的命令。DockerFile中可以有多个CMD指令，但只有最后一个生效，而且CMD会被docker run之后的运行命令的参数替换。
    1. shell格式：`CMD 命令`
    2. exec格式：`CMD [可执行文件,参数1,参数2,...]`
    3. 参数列表格式：`CMD [参数1,参数2,...]`，在指定了ENTRYPOINT指令后，用CMD指定具体的参数
11. ENTRYPOINT：指定一个容器启动时要运行的命令。ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数。后面的命令字符串追加到前面的命令字符串后。格式为：`ENTRYPOINT [可执行文件,参数1,参数2,...]`。ENTRYPOINT可以和CMD一起用，当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是将CMD的内容作为参数传递给ENTRYPOINT指令，他两个组合会变成：`ENTRYPOINT CMD`。可以使用ENTRYPOINT指令指定运行的命令字符串作为固定参数，使用CMD指令指定运行命令字符串作为可变参数，在运行容器的时候传入参数改变可变参数或者直接用默认的可变参数。
12. ONBUILD：当构建一个被继承的DockerFile时运行命令，父镜像在被子继承后父镜像的onbuild被触发，见[示例](#onbuild示例)

### ONBUILD示例

#### 父
![DockerFile+20230217155347](https://raw.githubusercontent.com/loli0con/picgo/master/images/DockerFile%2B20230217155347.png%2B2023-02-17-15-53-47)

#### 子
![DockerFile+20230217155424](https://raw.githubusercontent.com/loli0con/picgo/master/images/DockerFile%2B20230217155424.png%2B2023-02-17-15-54-24)


## Demo

### 目标
Centos7镜像加上 vim + ifconfig + jdk8

### 过程
```sh
$ pwd
/myfile

$ ls
Dockerfile jdk-8u171-linux-x64.tar.gz

$ vim Dockerfile
===⬇️写入内容⬇️===
FROM centos
MAINTAINER pig<pig@168.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

#安装vim编辑器 
RUN yum -y install vim
#安装ifconfig命令查看网络IP
RUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jre
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH $JAVA_HOME/bin:$PATH

EXPOSE 80

CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
===⬆️写入内容⬆️===

$ docker build -t cj8:1.5 .

$ docker run -it cj8:1.5
```

## build命令
```
Usage:  docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --cgroup-parent string    Optional parent cgroup for the container
      --compress                Compress the build context using gzip
      --cpu-period int          Limit the CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int           Limit the CPU CFS (Completely Fair Scheduler) quota
  -c, --cpu-shares int          CPU shares (relative weight)
      --cpuset-cpus string      CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string      MEMs in which to allow execution (0-3, 0,1)
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is 'PATH/Dockerfile')
      --force-rm                Always remove intermediate containers
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
  -m, --memory bytes            Memory limit
      --memory-swap bytes       Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --network string          Set the networking mode for the RUN instructions during build (default "default")
      --no-cache                Do not use cache when building the image
      --pull                    Always attempt to pull a newer version of the image
  -q, --quiet                   Suppress the build output and print image ID on success
      --rm                      Remove intermediate containers after a successful build (default true)
      --security-opt strings    Security options
      --shm-size bytes          Size of /dev/shm
      --squash                  Squash newly built layers into a single new layer
  -t, --tag list                Name and optionally a tag in the 'name:tag' format
      --target string           Set the target build stage to build.
      --ulimit ulimit           Ulimit options (default [])
```
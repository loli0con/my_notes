# 容器编排
Compose 是 Docker 公司推出的一个工具软件，可以管理多个 Docker 容器组成一个应用。你需要定义一个 YAML 格式的配置文件docker-compose.yml，写好多个容器之间的调用关系。然后，只要一个命令，就能同时启动/关闭这些容器。

Compose允许用户通过一个单独的docker-compose.yml模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。用户可以很容易地用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。Docker-Compose解决了容器与容器之间如何管理编排的问题。 

[Compose file version 3 reference](https://docs.docker.com/compose/compose-file/compose-file-v3)

## 工程
由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

## 服务
一个个应用容器实例，比如订单微服务、库存微服务、mysql容器、nginx容器或者redis容器。

## 编排
1. 编写Dockerfile定义各个微服务应用并构建出对应的镜像文件
2. 使用 docker-compose.yml 定义一个完整业务单元，安排好整体应用中的各个容器服务
3. 最后，执行docker-compose up命令来启动并运行整个应用程序，完成一键部署上线

## 常用命令
1. `docker-compose -h`: 查看帮助
2. `docker-compose up`: 启动所有 docker-compose服务
3. `docker-compose up -d`: 启动所有 docker-compose服务并后台运行
4. `docker-compose down`: 停止并删除容器、网络、卷、镜像
5. `docker-compose exec yml里面的服务id`: 进入容器实例内部
6. `docker-compose exec yml里面的服务id /bin/bash`
7. `docker-compose ps`: 展示当前docker-compose编排过的运行的所有容器
8. `docker-compose top`: 展示当前docker-compose编排过的容器进程
9. `docker-compose logs yml里面的服务id`: 查看容器输出日志
10. `docker-compose config`: 检查配置
11. `docker-compose config -q`: 检查配置，有问题才有输出
12. `docker-compose restart`: 重启服务
13. `docker-compose start`: 启动服务
14. `docker-compose stop`: 停止服务

这些命令在执行时都会默认读取`./docker-compose.yml`路径下的文件。
# ReadMe

## 总览
![ReadMe+20230216145413](https://raw.githubusercontent.com/loli0con/picgo/master/images/ReadMe%2B20230216145413.png%2B2023-02-16-14-54-13)

## 基本命令
* 查看docker概要信息：docker info
* 查看docker总体帮助文档：docker --help
* 查看docker命令帮助文档：docker 具体命令 --help
* 查看镜像/容器/数据卷所占的空间：docker system df
* docker服务管理（systemctl）
  * 启动docker：systemctl start docker
  * 停止docker：systemctl stop docker
  * 重启docker：systemctl restart docker
  * 查看docker状态：systemctl status docker
  * 开机启动：systemctl enable docker
  * 开机不启动：systemctl disable docker
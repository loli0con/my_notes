# Portainer
Portainer 是一款轻量级的应用，它提供了图形化界面，用于方便地管理Docker环境，包括单机环境和集群环境。

## 使用安装
1. 安装：`docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer`
2. 注册：
   1. 访问xxx.xxx.xxx.xxx:9000
   2. 用户名，直接用默认admin
   3. 密码，12345678
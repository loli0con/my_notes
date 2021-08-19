# 配置centos虚拟机
## 需求
Hadoop课程，linux课程，都需要用到虚拟机。
考虑过vps，在Digitalocean上面部署集群也不是不行。
最后还是用vmware算了，操作性强一点。

> 因为已经有一个aliyun实例了，好久没用过vmware，复习一下

## 实施
简要地记录一下过程
### vmware
[下载、安装并且激活](https://www.52pojie.cn/thread-1027984-1-1.html)

#### 网络
这里用的是NAT模式，方便yum下载东西。
![virtual_machine_arrangement+WechatIMG46441](https://raw.githubusercontent.com/loli0con/picgo/master/images/virtual_machine_arrangement%2BWechatIMG46441.png%2B2021-03-08-17-21-23)
> 默认的，没有改过，够用就行

### centos
#### 下载
[下载地址](https://mirrors.scau.edu.cn/centos/7/isos/x86_64/)
![virtual_machine_arrangement+截屏2021-03-08 17.02.34](https://raw.githubusercontent.com/loli0con/picgo/master/images/virtual_machine_arrangement%2B%E6%88%AA%E5%B1%8F2021-03-08%2017.02.34.png%2B2021-03-08-17-05-13)
请根据自己的网络选择合适的下载地址（官网、阿里云、163、中科大等）
> 老师让用minimal，实际上我觉得用everything省事一点。当然就是占用空间大一点，反应慢一点……？感觉不明显。

#### 配置
幸运的是，装完vm，vm自动地发现了电脑里面已经有centos实例了。看来是之前的没清理干净2333。  
更幸运的是，里面刚好两个实例，一个是已经该是相当完整的（带gui），另一个是最迷你的，两者对应了我上述两个版本。

##### 完整版
忘记当时用的是啥密码了，随便猜一个，进去了。好像桌面软件有点bug？不影响，反正当服务器使用的。
![virtual_machine_arrangement+WechatIMG46442](https://raw.githubusercontent.com/loli0con/picgo/master/images/virtual_machine_arrangement%2BWechatIMG46442.png%2B2021-03-08-17-26-49)
确实很省事，能用了。
> 原来不是bug，是桌面壁纸，我还以为显示驱动有问题。。。

##### 迷你版
之前听某海信大佬说坑很多，我想了想应该是网络问题？啊，还真是。。。
```shell
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens33  # 查看网卡配置信息，我这里是ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=56dada61-a86e-4eb3-a7aa-bc9c76ffa1e1
DEVICE=ens33
ONBOOT=no
[root@localhost ~]# # BOOTPROTO=dhcp，这里确认配置了dhcp动态获取ip地址
[root@localhost ~]# ip link set ens33 up # 启用网卡
[root@localhost ~]# dhclient ens33 # 动态获取一个ip地址
```
配置完，可以上网了

重点来了，上面是一次性的配置，重启之后就莫得了。  
重点是把文件`/etc/sysconfig/network-scripts/ifcfg-ens33`里面的**ONBOOT=no**修改为**ONBOOT=yes**  
这样重启系统之后，系统就能自动通过dhcp协议获取到ip地址了。

#### 优化
修改一下yum源比较好，根据自己的网络选择合适的源。后续课程里面，我觉得应该用得着。特别是minimal，还要按需填坑的。  
[教程不唯一，仅供参考](https://mirrors.scau.edu.cn/mirrors-help/centos.html)
### shell
老师让用xshell，我这里推荐[MobaXterm](https://www.52pojie.cn/thread-1121116-1-1.html)。顺带，windows terminal也不错（需要美化）。
![virtual_machine_arrangement+WechatIMG46440](https://raw.githubusercontent.com/loli0con/picgo/master/images/virtual_machine_arrangement%2BWechatIMG46440.png%2B2021-03-08-17-15-22)
给wt点个赞！

## 后记
TODO
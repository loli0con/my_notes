# mac - 😪
MacBook的做梦法则

## 参考
1. 必看，必看，必看：https://sspai.com/post/61379
2. 选看：man pmset

## 名词解释
* 浅睡/小睡 = 休眠
* 深睡/大睡 = 睡眠

## 我的配置
### -c 外部供电
```sh
# 120 分钟后进入休眠，延长清醒时间
# --- 如果要彻底防止休眠，则使用Caffeinated之类的软件 ---
sudo pmset -c sleep 240

# 显示器休眠时间：120 分钟
# sudo pmset -c displaysleep 120
# 显示器关闭的时间在系统配置界面中进行设置

# 硬盘休眠时间：60 分钟，这里的休眠指的是降速
sudo pmset -c disksleep 60

# 内存供电，内存镜像不写入硬盘
sudo pmset -c hibernatemode 0
# 关闭 standby 模式
sudo pmset -c standby 0
# 关闭 autopoweroff
sudo pmset -c autopoweroff 0

# 休眠时持续联网，其实可以不设置，默认值就是“1”
sudo pmset -c tcpkeepalive 1
```

### -b 电池供电
我的配置原则是，用电池供电，就尽快休眠/睡眠吧，减少对电池的损耗。  
当然有时候可能是在搬运电脑从工位到会议室之类的，一些短途的移动，还是让它别这么快睡熟了（保持休眠，但不睡眠）。

```sh
# 20 分钟后进入休眠
sudo pmset -b sleep 20
# 向硬盘写入镜像，同时内存供电
sudo pmset -b hibernatemode 3

# 显示器休眠时间：15 分钟
# sudo pmset -b displaysleep 15
# 显示器关闭的时间在系统配置界面中进行设置

# 硬盘休眠时间：10 分钟
sudo pmset -b disksleep 15

# 高电量下 standby: 1.5小时
sudo pmset -b standbydelayhigh 5400
# 低电量下 standby: 1小时
sudo pmset -b standbydelaylow 3600
# standby 电量阈值：90%
sudo pmset -b highstandbythreshold 90

# 休眠时断网
sudo pmset -b tcpkeepalive 0
# 开盖唤醒
sudo pmset -b lidwake 1
# 关闭被同一 iCloud 下的设备唤醒
sudo pmset -b acwake 0
```
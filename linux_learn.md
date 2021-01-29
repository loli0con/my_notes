# linux
曾经我有一个梦想，后来我不敢去想，现在我有突然有点想。  
没错就是它,——[brother bird's private dishes of linux](https://tiramisutes.github.io/images/PDF/vbird-linux-basic-4e.pdf)。

## 第三章
####在终端接登录linux
* 登录之后显示在屏幕上形如<u>[dmtsai@study ~] $</u>的字符串，叫做命令行提示符。可以修改。
默认root的提示字符为<u>#</u>，而一般身份使用者的提示字符为<u>$</u>。
* 登录成功后显示的内容，例如系统名称、内核版本等信息，来源于/etc/issue这个文件。
* `exit`命令登出。

####下达指令
* 不管几个空格，shell都会视为1个空格。
* 指令太长，可以使用反斜线<u>\\</u>来跳脱[Enter]符号，使得指令连续到下一行。
* linux是可以支持多国语系的，但是终端机接口<u>terminal</u>可能只支持英文（极端情况只支持ASCII编），所以需要修改语系
```shell
$ LANG=en_US.utf8  # 仅控制输出讯息
$ export LC_ALL=en_US.utf8  # 控制全部

$ lacale  # 查看目前的语系
```

####基础指令的操作
三个简单的指令：
* date 显示或者设置日期与时间
* cal 显示日历
* bc 简单计算器

下达指令之后的两种情况：
1. 直接显示结果然后回到命令提示字符等待下一个指令的输入
2. 进入到该指令的环境，直到结束该指令才回到命令提示符的环境

####几个重要的热键
* **\[tab\]** 按键：命令补齐 / 文件补齐 / 选项或参数补齐
* **\[ctrl\]-c** 按键：中断正在运行的的指令
* **\[ctrl\]-d** 按键：

####帮助信息
* help: 指令的基本说明
* man: manual，即操作说明
    * 指令/文件 的 基本意义号码(重点记忆1、5、8)
    
        |代号|代表内容|
        |:---|:---|
        |1|使用者在shell环境中可以操作的指令或可可执行文件|
        |2|系统核心可调用的函数与工具等|
        |3|一些常用的函数(function)与函数库(library)，大部分为C的函数库(libc)|
        |4|设备文件的说明，通常在/dev下的文件|
        |5|配置文件或者是某些文件的格式|
        |6|游戏(games)|
        |7|惯例与协定等，例如Linux文件系统、网络协定、ASCII code等等的说明|
        |8|系统管理员可用的管理指令|
        |9|跟kernel有关的文件|
    * man的数据源默认为`/usr/share/man`，man page搜寻路径配置文件`/etc/man.conf`，不同发行版该文件名字可能不同
    * man -f 相当于 whatis  
    man k 相当于 apropos  
    这两个命令依赖whatis数据库 
* info: 一个基于菜单的超文本帮助系统，比man要再细致些
* 说明文档: `/usr/share/doc`

####关机
* 将数据同步写入硬盘中的指令: sync
* 惯用的关机指令: shutdown
* 重新开机，关机: reboot, halt, poweroff

## 4
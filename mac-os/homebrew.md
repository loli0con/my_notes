# HomeBrew

## 参考资料
* https://juejin.cn/post/6844903549051076622
* https://wsgzao.github.io/post/homebrew/
* https://swiftcafe.io/post/home-brew
* https://sspai.com/post/56009
* https://www.jianshu.com/p/d7013f24f342
* https://osx-guide.readthedocs.io/zh_CN/latest/devenv/homebrew.html
* https://www.zhihu.com/question/443828638

## 简介
HomeBrew中文翻译为**家酿酒**，是macOS上包管理的事实标准。

### 名词解释
在开始使用HomeBrew之前，我们有必要了解一下怎么酿酒。
| 英文       | 翻译          | 实际含义                                                                                                |
| ---------- | ------------- | ------------------------------------------------------------------------------------------------------- |
| formula(e) | 配方          | 安装包的描述文件，formulae为复数。**酿酒配方**                                                          |
| cellar     | 酒窖          | 包安装好后所在的目录。**存放酒的地方**                                                                  |
| keg        | 小桶          | 具体某个包所在的目录，keg 是 cellar 的子目录。**小酒桶**                                                |
| bottle     | 瓶子;一瓶的量 | 预先编译好的包，不需要现场下载编译源码，速度会快很多;官方库中的包大多都是通过 bottle 方式安装。**酒瓶** |
| tap        | 水龙头        | 包的来源。**酿酒时所用到的水的来源**                                                                    |
| cask       | 木桶          | 安装 macOS native 应用的扩展，可以理解为有图形化界面的应用。**木酒桶**                                  |
| bundle     | 捆            | 描述 Homebrew 依赖的扩展，解决软件依赖问题                                                              |

[小桶和木桶的区别](https://cn.weblogographic.com/difference-between-keg-and-cask-ale-2652)：
cask啤酒被称为桶装啤酒，它是未经高温消毒的，未经过滤的啤酒，由其调理的同一桶提供，包括二次发酵；key小桶啤酒是经过过滤、巴氏灭菌并人工碳酸化，同时在冷却桶的压力下制成并冷藏好。

### 优点
一个是方便集中管理，在删除不再使用的软件包时，省去了大量软件包散落在各处带来日后清理的头疼问题。另外可以方便更集中的权限管理。

#### 例子
> 这一段是纯抄的，原本想拷贝链接地址，发现网站的证书过期了数月了，而且也一年没新文章了，有点担忧前辈是不是没爱发电了，明明文章挺好的...

举个具体的例子——安装的node软件（node.js，一个运行JavaScript的软件，可近似看作jre/jvm）。如果不使用Homebrew，而用它官网提供的安装程序，那么安装完成后，会创建如下文件：
```shell
/usr/local/bin
-rwxrwxr-x  1 spring  admin  30994272 10  3 09:55 node
lrwxr-xr-x  1 502     staff        38 10 16 09:57 npm -> ../lib/node_modules/npm/bin/npm-cli.js

/usr/local/include
drwxrwxr-x  42 spring  admin  1344 10  3 09:55 node

/usr/local/lib
drwxrwxr-x  3 spring  admin  96 10  3 09:54 node_modules

/usr/local/lib/dtrace
-rw-rw-r--  1 spring  admin  12324 10  3 09:50 node.d

/usr/local/share/doc
drwxrwxr-x  5 spring  admin  160 10  3 09:54 node

/usr/local/share/man/man1
-rw-rw-r--  1 spring  admin  7849 10  3 09:48 node.1

/usr/local/share/systemtap/tapset
-rw-rw-r--  1 spring  admin  2361 10  3 09:48 node.stp
```
一个nodejs环境，安装看起来不难，但它却会在这么多的位置创建文件。当然，如果你一直在同样的环境使用它，没什么问题。但如果有一天你不需要它了，想删除掉，就比较麻烦了，除非官方能提供方便的卸载工具。也就是说，你必须像我现在这样清楚的列出它所有文件的位置，才能干净的将它删除掉。这个现象存在于大多数我们常用的工具软件中。

再来看看 Homebrew 是如何解决这个问题的，下面是使用 Homebrew 安装 node 后文件的存放位置：
```
/usr/local/bin
lrwxr-xr-x  1 marspro  admin  29 10 16 12:03 node -> ../Cellar/node/8.7.0/bin/node
lrwxr-xr-x  1 marspro  admin  46 10 16 12:03 npm -> /usr/local/lib/node_modules/npm/bin/npm-cli.js

/usr/local/include
lrwxr-xr-x  1 marspro  admin  33 10 16 12:03 node -> ../Cellar/node/8.7.0/include/node

/usr/local/lib
drwxr-xr-x  4 marspro  admin  128 10 17 14:01 node_modules

/usr/local/lib/dtrace
lrwxr-xr-x  1 marspro  admin  41 10 16 12:03 node.d -> ../../Cellar/node/8.7.0/lib/dtrace/node.d

/usr/local/share/doc
lrwxr-xr-x  1 marspro  admin  38 10 16 12:03 node -> ../../Cellar/node/8.7.0/share/doc/node

/usr/local/share/man/man1
lrwxr-xr-x  1 marspro  admin     48 10 16 12:03 node.1 -> ../../../Cellar/node/8.7.0/share/man/man1/node.1

/usr/local/share/systemtap/tapset
-rw-r--r--  1 marspro  admin  3495 10 11 20:36 node.stp
```
之前目录中的文件都变成了符号链接，并且指向 Cellar 目录中。

也就是说，虽然 node 安装后的目录结构没变，但相应的位置已经变成了符号链接，而 node 实际的文件都存放在了 /usr/local/Cellar 目录中。也就是我们前面说的 Homebrew 模块主目录。当你想删除 nodejs 的时候，只需要运行：
```
brew uninstall node
```
这一条命令就可以完整的删除之前安装的 nodejs 所有内容，这里也包括在 /usr/local/ 中创建的那些符号链接。

这就是包管理的魅力，因此有必要好好认识一下这位大管家。

## 安装
开始安装前需要安装 macOS 命令行工具，请确保该工具存在，否则需要执行如下命令：
```shell
xcode-select -v  # 查看版本，以确认工具存在

xcode-select —-install  # 如果工具不存在则执行该命令
```

通过运行以下的命令安装HomeBrew：
```shell
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
该命令包括两个命令，下载install文件，然后用系统的ruby工具安装HomeBrew。

安装过程中会提示你执行了哪些动作。
```
==> This script will install:
/usr/local/bin/brew  # 运行命令(软链)
/usr/local/share/doc/homebrew  # 帮助文档相关
/usr/local/share/man/man1/brew.1  # 帮助文档相关
/usr/local/share/zsh/site-functions/_brew  # 帮助文档相关
/usr/local/etc/bash_completion.d/brew  # bash环境中,brew命令补齐
/usr/local/Homebrew  # 安装目录
==> The following new directories will be created:
/usr/local/bin
/usr/local/etc
/usr/local/include
/usr/local/lib
/usr/local/sbin
/usr/local/share
/usr/local/var
/usr/local/opt
/usr/local/share/zsh
/usr/local/share/zsh/site-functions
/usr/local/var/homebrew
/usr/local/var/homebrew/linked
/usr/local/Cellar
/usr/local/Caskroom
/usr/local/Homebrew
/usr/local/Frameworks
==> The Xcode Command Line Tools will be installed.
```

安装完成后输入brew -v 即可显示是否安装成功。

## 最常用的命令

### 安装&卸载
1. brew install <formula> 安装指定软件
    * 安装指定版本，例如 brew install node@6
    * 即<formula>可以表示为“软件名称@版本编号”
2. brew uninstall <formula> 卸载指定软件
3. brew uninstall <fromula> --force 彻底卸载指定软件，包括旧版本
4. brew list 显示所有的已安装的软件
5. brew search text 搜索本地远程仓库的软件，已安装会显示绿色的勾
   * 在搜索时应当遵循宁可少字，不能错字的原则来搜索。
   * 也可以直接访问 https://formulae.brew.sh/ 来进行搜索
6. brew search /text/ 使用正则表达式搜软件
7. brew info <formula> 显示指定软件信息
8. brew reinstall <formula> 重新安装指定软件，先卸载后安装
9.  brew install <formula> --build-from-source 源码安装指定软件，可以给定指定参数


通过brew安装的文件会自动设置环境变量，所以不用担心命令行不能启动的问题。

### 升级
1. brew update 自动升级homebrew（从github下载最新版本）
2. brew outdated 检测已经过时的软件
3. brew upgrade 升级所有已过时的软件，即列出的以过时软件
4. brew upgrade <formula>升级指定的软件
5. brew pin <formula> 禁止指定软件升级
6. brew unpin <formula> 解锁禁止升级
7. brew upgrade --all 升级所有的软件包，包括未清理干净的旧版本的包

### 清理
homebrew再升级软件时候不会清理相关的旧版本，在软件升级后我们可以使用如下命令清理
1. brew cleanup -n 列出需要清理的内容
2. brew cleanup <formula> 清理指定的软件过时包
3. brew cleanup 清理所有的过时软件

### 依赖
1. brew deps <formula> 查看指定软件依赖于哪些软件
2. brew uses <formula> 查看指定软件被哪些软件所依赖

### 软链
Homebrew 会把软件安装到 /usr/local/Cellar，并且通过软链链接到 /usr/local/bin。我们可以通过 brew unlink 和 brew link 删除或创建链接。
1. brew link <formula> 建立指定软件的软链接
2. brew unlink <formula> 删除指定软件的软链接

### 通用
* brew commands 列出所有可用命令
* brew doctor 环境检查，修复error，提示warning

## service
macOS使用 launchctl 命令加载开机自动运行的服务，brew service 可以简化 lauchctl 的操作。

以MySQL为例，使用launchctl启动:
```shell
ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

如使用 brew service 可以简化为:
```shell
brew services start mysql
```

### 常用命令
1. brew services list  # 查看使用brew安装的服务列表
2. brew services run formula|--all  # 启动服务（仅启动不注册）
3. brew services start formula|--all  # 启动服务，并注册
4. brew services stop formula|--all   # 停止服务，并取消注册
5. brew services restart formula|--all  # 重启服务，并注册
6. brew services cleanup  # 清除已卸载应用的无用的配置

### 配置文件目录
```shell
/Library/LaunchDaemons # 开机自启，需要sudo
~/Library/LaunchAgents # 用户登录后自启
```
在这个目录里面，可以查看配置文件的服务路径、启动参数、日志路径等配置参数。

## cask
Homebrew Cask 是 Homebrew 的扩展，借助它可以方便地在 macOS 上**安装图形界面程序**，即我们常用的各类应用。Homebrew 中文含义为自制、自酿酒，Cask 中文含义为桶、木桶，桶装酒是一种成品，也就是说每一个 homebrew cask 都可以直接使用的，比如atom和chrome，可以使用如下命令安装：
```shell
brew install --cask atom
brew install --cask google-chrome
```

### 常用命令
1. brew install --cask <formula> 安装指定图形界面软件
2. brew uninstall --cask <formula> 卸载软件
3. brew uninstall --cask --force <formula> 卸载软件，带参数
4. brew search --cask text 搜索软件
5. brew list --cask 列出所有通过cask安装的软件

### 搜索软件
* https://formulae.brew.sh/cask/
* http://macappstore.org/


## tap
tap是水龙头的意思，酿酒的时候肯定要用到水。

通常一户人家只有一根水管（入户的那个管道，上面有一个水表），但是作为专业的酿酒户，有多根水管也是合情合理的，在这些水管里面里面可以承载蒸馏水、矿泉水等不同种类的水。  
通过水龙头，可以方便酿酒时获取到水。

用专业的术语去描述，这里的tap应该被称为仓库——存放软件包的仓库。

### 查看当前仓库
```shell
cd "$(brew --repo)"  # 进入仓库目录
git remote -v  # 查看仓库地址
```

### 换源
这里的换源，指的是更换镜像源。

还是用酒的例子来解释：德国黑啤，诞生于德国，但在全球各地都有它的工厂，全球各地都有销售。你想要想用它，并非要跑到德国去，在当地就可以买到。  
这里的换源，就相当于你从本地采购德国黑啤，当然你也可以原装进口。

替换为[中科大源](http://mirrors.ustc.edu.cn/):
* [Homebrew 源代码仓库](https://mirrors.ustc.edu.cn/help/brew.git.html)
* [Homebrew 核心软件仓库](https://mirrors.ustc.edu.cn/help/homebrew-core.git.html)
* [Homebrew 预编译二进制软件包](https://mirrors.ustc.edu.cn/help/homebrew-bottles.html)
* [Homebrew cask 软件仓库](https://mirrors.ustc.edu.cn/help/homebrew-cask.git.html)，提供 macOS 应用和大型二进制文件
* [Homebrew cask 其他版本软件仓库](https://mirrors.ustc.edu.cn/help/homebrew-cask-versions.git.html)，提供使用人数多的、需要的版本不在 cask 仓库中的应用。

### 常用命令
1. brew tap 列出本地资源仓库，其中 homebrew 是默认仓库，其它都是第三方仓库
2. brew tap <user/repo> 添加仓库
3. brew untap <user/repo> 删除仓库

这里的添加/删除仓库，和上述的换源不太一样。

还是德国黑啤的例子：你想要来点伏特加了！（同一种产品的不同采购方式-对比-不同产品的采购方式）


## 辅助软件
除了命令行，还有两款GUI软件可以帮助我们更好的使用 Homebrew ，他们分别是 Cakebrew 和 launchrocket。

### Cakebrew
Cakebrew 是 Homebrew 的 GUI 管理器，在 Cakebrew 中，你可以看到当前所有已经安装的软件，并可以在 Caskbrew 中对其他软件执行升级等操作。

你只需要执行`brew install --cask cakebrew`就可以完成 Cakebrew 的安装。

安装完成后，在 LaunchPad 中打开即可。

> 亲测体验很差

### launchrocket
launchrocket 可以用于管理 Homebrew 安装的服务，在使用时，你需要先添加对应的tap，然后再安装软件。

```shell
brew tap jimbojsb/launchrocket
brew install --cask launchrocket
```

安装完成后，在 LaunchPad 中打开即可。

## 解酒药
有时候喝酒可能会上头，这里有一些我曾经吃过的解酒药。

### JDK
* http://asing1elife.com/homebrew/2019/04/10/Homebrew-%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE-JDK/

### Tomcat
臭猫是真的烦
* https://newsn.net/say/brew-install-tomcat.html
* https://www.cnblogs.com/kerrycode/p/3586098.html
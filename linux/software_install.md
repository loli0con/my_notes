# 软件安装
截取自《鸟哥的Linux私房菜》

## 安装方式
* 使用源代码进行软件编译
* RPM & YUM

## 软件管理机制
Linux 开发商先在固定的硬件平台与操作系统平台上面将需要安装或升级的软件编译好，然后将这个软件的所有相关文件打包成为一个特殊格式的文件，在这个软件文件内还包含了预先侦测系统与相依软件的脚本，并提供记载该软件提供的所有文件信息等，最终将这个软件文件释出。

用户端取得这个文件后，只要通过特定的指令来安装，那么该软件文件就会依照内部的脚本来侦测相依的前驱软件是否存在，若安装的环境符合需求，那就会开始安装，安装完成后还会将该软件的信息写入软件管理机制中，以达成未来可以进行升级、移除等动作。

### 属性相依
每个软件文件都有提供相依属性的检查，通过将相依属性的数据做成列表，等到实际软件安装时，若发生有相依属性的软件状况时，例如*安装 A 需要先安装 B 与 C ，而安装 B 则需要安装 D 与 E 时，那么当你要安装 A ，通过相依属性列表，管理机制自动去取得 B, C, D, E 来同时安装*，以此解决属性相依的问题。

目前新的 Linux 开发商都有提供这样的“线上升级”机制，通过这个机制，原版光盘就只有第一次安装时需要用到而已，其他时候只要有网络，就能够取得原本开发商所提供的任何软件。

### 总结
|distribution 代表|软件管理机制|使用指令|线上升级机制(指令)|
|---|---|---|---|
|Red Hat/Fedora|RPM|rpm, rpmbuild|YUM (yum)|
|Debian/Ubuntu|DPKG|dpkg|APT (apt-get)|

Centos：使用的软件管理机制为 RPM 机制，而用来作为线上升级的方式则为 YUM 。

## RPM
RPM 全名是“ RedHat Package Manager ”简称则为 RPM 。

RPM 是以一种数据库记录的方式来将你所需要的软件安装到你的 Linux 系统的一套管理机制。

### 特点
他最大的特点就是将要安装的软件先编译过，并且打包成为 RPM 机制的包装文件。通过包装好的软件里头默认的数据库记录，记录这个软件要安装的时候必须具备的相依属性软件，当执行安装时，RPM 会先依照软件里头的数据查询 Linux 主机的相依属性软件是否满足，若满足则予以安装，若不满足则不予安装。  
安装的时候会将该软件的信息整个写入 RPM 的数据库中，以便未来的查询、验证与反安装。

### 优点
1. RPM 内含已经编译过的程序与配置文件等数据，可以让使用者免除重新编译的困扰，在软件传输与安装上很方便
2. RPM 在被安装之前，会先检查系统的硬盘容量、操作系统版本等，可避免文件被错误安装
3. RPM 文件本身提供软件版本信息、相依属性软件名称、软件用途说明、软件所含文件等信息，便于了解软件
4. RPM 管理的方式使用数据库记录 RPM 文件的相关参数，便于升级、移除、查询与验证

### 缺点
1. 软件文件安装的环境必须与打包时的环境需求一致或相当;
2. 需要满足软件的相依属性需求;
3. 反安装时需要特别小心，最底层的软件不可先移除，否则可能造成整个系统的问题!

## SRPM
顾名思义，他是 Source RPM 的意思，也就是这个 RPM 文件里面含有源代码。

通常 SRPM 的扩展名是以 *.src.rpm 这种格式来命名的。

SRPM 虽然内容是源代码，但是他仍然含有该软件所需要的相依性软件说明、以及所有 RPM 文件所提供的数据。同时，他与 RPM 不同的是，他提供了参数配置文件(就是 configure 与 makefile)。所以，如果下载的是 SRPM ，那么要安装该软件时，就必须要:
1. 先将该软件以 RPM 管理的方式编译，此时 SRPM 会被编译成为 RPM 文件;
2. 然后将编译完成的 RPM 文件安装到 Linux 系统当中。

### 对比
|文件格式|文件名格式|直接安装与否|内含程序类型|可否修改参数并编译|
|---|---|---|---|---|
|RPM|xxx.rpm|可|已编译|不可|
|SRPM|xxx.src.rpm|不可|未编译之源代码|可|

### 文件名
文件名通常为：
`软件名称-软件的版本信息- 释出的次数.适合的硬件平台.扩展名`。


## YUM
CentOS先将释出的软件放置到 YUM 服务器内，然后分析这些软件的相依属性问题，将软件内的记录信息写下来(header)，然后再将这些信息分析后记录成软件相关性的清单列表。这些列表数据与软件所在的本机或网络位置可以称呼为容器或软件仓库或软件库(repository)。

当用户端有软件安装的需求时，用户端主机会主动的向网络上面的 yum 服务器的软件库网址下载清单列表，然后通过清单列表的数据与本机 RPM 数据库已存在的软件数据相比较，就能够一口气安装所有需要的具有相依属性的软件了。

![software_install+20210819174502](https://raw.githubusercontent.com/loli0con/picgo/master/images/software_install%2B20210819174502.png%2B2021-08-19-17-45-03)

软件仓库内的清单会记载每个文件的相依属性关系，以及所有文件的网络位置(URL)。由于记录了详细的软件网络位置，所以当有需要的时候，就会自动的从网络下载该软件。

当用户端有升级、安装的需求时， yum 会向软件库要求清单的更新，等到清单更新到本机的 /var/cache/yum 里面后，等一下更新时就会用这个本机清单与本机的 RPM 数据库进行比较，这样就知道该下载什么软件。接下来 yum 会跑到软件库服务器 (yum server) 下载所需要的软件 (因为有记录软件所在的网址)，然后再通过 RPM 的机制开始安装软件啦!


## RPM 软件管理程序: rpm

### 相关目录
* RPM 的数据库：/var/lib/rpm/
* 软件内的文件：
  * 可执行文件：/usr/bin
  * 动态函数库：/usr/lib
  * 软件使用手册与说明文档：/usr/share/doc
  * man page 文件：/usr/share/man

### 相关指令

#### 安装
```sh
rpm -ivh package_name
```
选项与参数:
* -i :install的意思
* -v :查看更细部的安装信息画面
* -h :以安装信息列显示安装进度

package_name:
* 绝对路径：/mnt/Packages/rp-pppoe-3.11-5.el7.x86_64.rpm
* 相对路径&多个：a.i386.rpm b.i386.rpm *.rpm
* 网址：http://website.name/path/pkgname.rpm

##### 可选参数
除了test和justdb，其他谨慎使用。  
replacepkgs可用于覆盖安装。
![software_install+20210819180102](https://raw.githubusercontent.com/loli0con/picgo/master/images/software_install%2B20210819180102.png%2B2021-08-19-18-01-04)

#### 升级
```sh
rpm -Uvh package_name
# 后面接的软件即使没有安装过，则系统将予以直接安装; 若后面接的软件有安装 Uvh 过旧版，则系统自动更新至新版;

rpm -Fvh package_name
# 如果后面接的软件并未安装到你的 Linux 系统上，则该软件不会被安装;亦即只 Fvh 有已安装至你 Linux 系统内的软件会被“升级”!
```

##### 可选参数
同“安装”

#### 查询
RPM 在查询的时候，其实查询的地方是在 /var/lib/rpm/ 这个目录下的数据库文件。另外， RPM 也可以查询未安装的 RPM 文件内的信息。

```sh
rpm -q 软件名称(版本号非必要)
```

选项与参数:
* 查询已安装软件的信息:
  * -q :仅查询，后面接的软件名称是否有安装
  * -qa :列出所有的，已经安装在本机 Linux 系统上面的所有软件名称
  * -qi :列出该软件的详细信息 (information)，包含开发商、版本与说明等
  * -ql :列出该软件所有的文件与目录所在完整文件名 (list)
  * -qc :列出该软件的所有配置文件 (找出在 /etc/ 下面的文件名而已)
  * -qd :列出该软件的所有说明文档 (找出与 man 有关的文件而已)
  * -qR :列出与该软件有关的相依软件所含的文件 (Required 的意思)
  * -qf :由后面接的文件名称，找出该文件属于哪一个已安装的软件
  * -q --scripts:列出是否含有安装后需要执行的脚本档，可用以debug
* 查询某个 RPM 文件内含有的信息:
  * -qp\[icdlR]:注意 -qp 后面接的所有参数以上面的说明一致。但用途仅在于找出某个 RPM 文件内的信息，而非已安装的软件信息。


#### 验证
验证 (Verify) 的功能主要在于提供系统管理员一个有用的管理机制，作用的方式是“使用 /var/lib/rpm 下面的数据库内容来比对目前 Linux 系统的环境下的所有软件文件”。也就是说，当你有数据不小心遗失，或者是因为你误杀了某个软件的文件，或者是不小心不知道修改到某一个软件的文件内容，就用这个简单的方法来验证一下原本的文件系统吧!

```sh
rpm -V
```

选项与参数:
* -V :后面加的是软件名称，若该软件所含的文件被更动过，才会列出来;
* -Va :列出目前系统上面所有可能被更动过的文件;
* -Vp :后面加的是文件名称，列出该软件内可能被更动过的文件;
* -Vf :列出某个文件是否被更动过，同时列出被更动过的信息类型。
 
##### 改动信息
示例，检查一下 logrotate 这个软件：
```sh
查询一下， /etc/crontab 是否有被更动过
rpm -V logrotate
..5....T.  c /etc/logrotate.conf
```

最前面的几个信息是：
* S :(file Size differs) 文件的容量大小是否被改变
* M :(Mode differs) 文件的类型或文件的属性 (rwx) 是否被改变
* 5 :(MD5 sum differs) MD5 这一种指纹码的内容已经不同
* D :(Device major/minor number mis-match) 设备的主/次代码已经改变
* L :(readLink(2) path mis-match) Link 路径已被改变
* U :(User ownership differs) 文件的所属人已被改变
* G :(Group ownership differs) 文件的所属群组已被改变
* T :(mTime differs) 文件的创建时间已被改变
* P :(caPabilities differ) 功能已经被改变

后面的字母代表文件类型：
* c :配置文件 (config file)
* d :文件数据文件 (documentation)
* g :鬼文件~通常是该文件不被某个软件所包含，较少发生!(ghost file)
* l :授权文件 (license file)
* r :读我文件 (read me)


#### 数码签章
数码签章被用来检验软件的来源。厂商使用数码签章系统产生一个专属于该软件的签章，并将该签章的公钥 (public key) 释出。当你要安装一个 RPM 文件时:
1. 首先你必须要先安装原厂释出的公钥文件
2. 实际安装原厂的RPM软件时，rpm指令会去读取RPM文件的签章信息，与本机系统内的签章信息比对
3. 若签章相同则予以安装，若找不到相关的签章信息时，则给予警告并且停止安装

CentOS 使用的数码签章系统为GPG，CentOS 的数码签章位于：`/etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7`。

使用`rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7`命令可以安装该数码签章。

#### 反安装
反安装就是将软件解除安装啦!要注意的是，“解除安装的过程一定要由最上层往下解除”。

移除的选项很简单，就通过 -e 即可移除。
```sh
rpm -e 软件名称
```

#### 重建数据库
由于 RPM 文件常常会安装/移除/升级等，某些动作或许可能会导致 RPM 数据库 /var/lib/rpm/ 内的文件破损。  
可以使用 --rebuilddb 这个选项来重建一下数据库。
```sh
rpm --rebuilddb
```

## yum

### 常用指令

#### 查询

如果想要查询利用 yum 来查询原版 distribution 所提供的软件，或已知某软件的名称，想知道该软件的功能，可以利用 yum 相关的参数为:`yum [option] [查询工作项目] [相关参数]`。

选项与参数:
* \[option]:主要的选项，包括有:
  * -y :当 yum 要等待使用者输入时，这个选项可以自动提供 yes 的回应;
  * --installroot=/some/path :将该软件安装在 /some/path 而不使用默认路径
* \[查询工作项目] \[相关参数]:这方面的参数有:
  * search :搜寻某个软件名称或者是描述 (description) 的重要关键字;
  * list :列出目前 yum 所管理(即已经安装在本机)的所有的软件名称与版本;
  * info :列出软件的详细信息;
  * provides:从文件去搜寻软件!类似 rpm -qf 的功能!

#### 安装和升级

`yum [option] [安装与升级的工作项目] [相关参数]`

选项与参数:
* install :后面接要安装的软件!
* update :后面接要升级的软件，若要整个系统都升级，就直接 update 即可

#### 移除

`yum [remove] 软件`


### 配置文件
手动的修改 yum 的配置文件，可以配置软件库、 yum server 、映射站台(镜像站)等。

#### 配置文件解读
以base软件库为例，运行命令`vim /etc/yum.repos.d/CentOS-Base.repo`，可以查看如下内容：
```
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

其中：
* \[base]:代表软件库的名字!中括号一定要存在，里面的名称则可以随意取。但是不能有两个相同的软件库名称，否则 yum 会不晓得该到哪里去找软件库相关软件清单文件。
* name:只是说明一下这个软件库的意义而已，重要性不高!
* mirrorlist=:列出这个软件库可以使用的映射站台，如果不想使用，可以注解到这行;
* baseurl=:这个最重要，因为后面接的就是软件库的实际网址! mirrorlist 是由 yum 程序自行去捉映射站台，baseurl 则是指定固定的一个软件库网址!
* enable=1:让这个软件库被启动。如果不想启动可以使用enable=0!
* gpgcheck=1:指定是否需要查阅 RPM 文件内的数码签章!
* gpgkey=:数码签章的公钥档所在位置!使用默认值即可。

默认情况下，只有base、updates、extras这三个软件仓库被启用。

#### 列出所有软件库
```sh
yum repolist all
```

#### 清单整理
由于 yum 会先下载软件库的清单到本机的 /var/cache/yum 里面去，当修改了网址却没有修改软件库名称 (中括号内的文字)，可能就会造成本机的清单与 yum 服务器的清单不同步，此时就会出现无法更新的问题了!

此时需要清除掉本机上面的旧数据，并构建新的数据：
```
yum clean all
yum makecache
```


### 软件群组
通过 yum 的软件群组功能，可以安装的是一个大型专案（一系列的软件）。

```sh
yum [群组功能] [软件群组]
```

选项与参数:
* grouplist :列出所有可使用的“软件群组组”
* groupinfo :后面接 group_name，则可了解该 group 内含的所有软件名
* groupinstall :安装一整组的软件群组
* groupremove :移除某个软件群组


#### Optional Packages
在使用groupinfo查看软件群组的信息的时候，会出现“Optional Packages”，即可选择的软件包。默认情况下，这些软件包不会安装。

如果想要让 groupinstall 默认安装好所有的 optional 软件就得要修改配置文件，更改选 groupinstall 选择的软件项目：
1. 运行命令`vim /etc/yum.conf`，
2. distroverpkg=centos-release # 找到这一行，下面新增一行!
3. group_package_types=default, mandatory, optional


### 第三方协力软件
在 Linux 上面经常需要安装第三方协力软件，可以增加额外软件库去满足自己需求（以epel软件库为例）。

首先，要针对 epel 进行 yum 的配置文件处理，`vim /etc/yum.repos.d/epel.repo`。
```
[epel]
name = epel packages
baseurl = https://dl.fedoraproject.org/pub/epel/7/x86_64/
gpgcheck = 0
enabled = 0
```
此处配置默认不要去找这个软件库，当需要的时候才进行安装：
```
yum --enablerepo=epel install 软件
```

#### 原版光盘
上述的配置方式，可以使用原版的光盘来作为主要的软件来源。首先将光盘挂载到某个目录（假设在 /mnt ），然后设置如下的 yum 配置文件（`vim /etc/yum.repos.d/cdrom.repo`）：
```
[mycdrom]
name = mycdrom
baseurl = file:///mnt
gpgcheck = 0
enabled = 0
```
最后执行安装命令：
```
yum --enablerepo=mycdrom install 软件
```
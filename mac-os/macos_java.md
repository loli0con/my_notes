# MacOs里面的Java

## 前言
需在在Mac上构建Java开发环境，我这台Mac从16年用到现在，上面的环境有点乱，所以需要整理一下，发现还挺麻烦的

## 背景
在我的Mac上，涉及的Java的有几个地方：
* /usr/bin/java~~(symlink to /System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java)~~
* /System/Library/Frameworks/JavaVM.framework/Versions/A/~~Commands/java~~
* /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java
* /Library/Java/JavaVirtualMachines/jdk-xxxxxx.jdk/Contents/Home/bin/java

## 状况分析
### 1
这个`/usr/bin/java`有点奇怪
![macos_java+截屏2021-04-19 15.51.44](https://raw.githubusercontent.com/loli0con/picgo/master/images/macos_java%2B%E6%88%AA%E5%B1%8F2021-04-19%2015.51.44.png%2B2021-04-19-15-52-48)
这个java不是软连接，而且inode号码是1。和网上说的不太一样，网上说的是“这个java软连接到`/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/java`”。

这里我推测macOS采取了某种机制，让`/usr/bin/java`指向了`/usr/libexec/java_home -V`结果列表的第一项。
![macos_java+截屏2021-04-19 16.02.49](https://raw.githubusercontent.com/loli0con/picgo/master/images/macos_java%2B%E6%88%AA%E5%B1%8F2021-04-19%2016.02.49.png%2B2021-04-19-16-04-34)

### 2
`/System/Library/Frameworks/JavaVM.framework/Versions/A/`里面没有Commands子目录，而且
![macos_java+截屏2021-04-19 16.05.59](https://raw.githubusercontent.com/loli0con/picgo/master/images/macos_java%2B%E6%88%AA%E5%B1%8F2021-04-19%2016.05.59.png%2B2021-04-19-16-07-08)

这里我就不管它了，关闭系统保护程序（System Integrity Protection）的步骤有些繁琐，备忘一下有这个东西，放着它不管算了。

### 3
`/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home/bin/java`好像是用于Safari的，包含了Java8，Safari用它来运行[applet](https://baike.baidu.com/item/Applet/2723730)。

### 4
`/Library/Java/JavaVirtualMachines/jdk-xxxxxx.jdk`是我看着最像开发环境的目录，我打算基于这个目录构建我的Java开发环境。

## 关于/usr/libexec/java_home
在Mac OS X 10.5之后，在 /usr/libexec/ 路径下多了一个叫java_home文件，这是Mac上专门用来管理JAVA_HOME的文件，我们可以靠它轻松得到不同版本的JAVA_HOME。

这个程序不太友善，我不知道它如何搜寻Java环境的，也不知道如何改变找到的Java环境列表的排序。

通过man java_home，你可以查看这个命令的说明文档。

## 清理环境
把这陈年老垢全部清理掉，再重新构建开发环境。
```zsh
sudo rm -rf /Library/Java/*
sudo rm -rf /Library/PreferencePanes/Java*
sudo rm -rf /Library/Internet\ Plug-Ins/Java*

sudo rm -rf /Library/PreferencePanes/JavaControlPanel.prefPane
sudo rm -rf /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin
sudo rm -rf /Library/LaunchAgents/com.oracle.java.Java-Updater.plist
sudo rm -rf /Library/LaunchDaemons/com.oracle.java.Helper-Tool.plist
sudo rm -rf /Library/Preferences/com.oracle.java.Helper-Tool.plist
sudo rm -rf /System/Library/Frameworks/JavaVM.framework

sudo rm -rf /var/db/receipts/com.oracle.j*
sudo rm -rf /var/root/Library/Preferences/com.oracle.javadeployment.plist
sudo rm -rf ~/Library/Preferences/com.oracle.java.JavaAppletPlugin.plist
sudo rm -rf ~/Library/Preferences/com.oracle.javadeployment.plist
sudo rm -rf ~/.oracle_jre_usage
```

`sudo rm -rf /System/Library/Frameworks/JavaVM.framework`会失败，不动它了，怕翻车。
有兴趣可以看看[这个帖子](https://www.zhihu.com/question/403361335)。

## 构建环境
网上有很多方案可以参考的，大体上分为自行安装jdk，或者使用homebrew安装jdk。

我这边选择的是自行安装jdk，出了问题的时候就可以不去和homebrew里面的逻辑打交道。  
去[oracle官网](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html)下载，我用的是jdk8u201。需要用到一个[账号](https://blog.csdn.net/LinBilin_/article/details/50217541)。

![macos_java+jdk8](https://raw.githubusercontent.com/loli0con/picgo/master/images/macos_java%2Bjdk8.jpeg%2B2021-04-21-19-09-52)

下载回来一个dmg，执行安装后，配置了JAVA_HOME为`/usr/libexec/java_home -v1.8.0_201`，暂时没配置CLASS_PATH，在idea中成功打印hello world。

## 参考
https://www.coder.work/article/4720975  
https://developer.aliyun.com/article/523980  
https://www.barretlee.com/blog/2016/04/06/operation-not-permitted-problem-in-linux-or-unix-system/  
https://qastack.cn/programming/12757558/installed-java-7-on-mac-os-x-but-terminal-is-still-using-version-6  
https://blog.csdn.net/Mr_OOO/article/details/60339919  
https://segmentfault.com/a/1190000020834358  
https://www.coder.work/article/1515513  
https://www.javaroad.cn/articles/2519  
https://www.codenong.com/19039752/  
https://github.com/huataihuang/cloud-atlas-draft/blob/master/develop/mac/multiple_versions_java_in_os_x.md  

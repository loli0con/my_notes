# MacOs里的python

## 起因
某天，脑子被门夹了，然后，我......  
```shell
brew upgrade
```
再然后打开了一个陈年屎坑

忆往昔懵懂岁月，入手这台MacBook是2016年年中的时候，因此这里存在一个是我迷惑的开发环境。
这波升级，让我的python直接升到了3.9，pip也出了点问题

**もうやるしかない**

## 开坑
> 动手删除多余python，开始整理开发环境

mac上安装python的方式有很多，常见的如下：
* 官网下载python安装包
* brew install
* anaconda

其实一开始我真不知道竟然有如此多的安装方法的，我只熟悉前两者。
由于我没有装过anaconda，所以我检查的范围主要是前两者的

对自己有一点的了解，相信自己不会骚操作，所以就没用```locate```和```find```，然后执行如下操作：
```shell
$ which -a python
/usr/bin/python  # macos自带的，不能删的，系统依赖它，将来会被python3取代
$ which -a python3
/usr/local/bin/python3  # brew安装的
/usr/bin/python3 # 这是什么鬼？
```
运行这个/usr/bin/python3
```shell
$ python3  # 这是用brew装的python3
Python 3.9.1 (default, Dec 16 2020, 01:57:40)
[Clang 12.0.0 (clang-1200.0.32.27)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
$ ./python3
Python 3.8.2 (default, Nov  4 2020, 21:23:28)
[Clang 12.0.0 (clang-1200.0.32.28)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import sys
>>> sys.path
['', '/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.8/lib/python38.zip', '/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8', '/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8/lib-dynload', '/Applications/Xcode.app/Contents/Developer/Library/Frameworks/Python3.framework/Versions/3.8/lib/python3.8/site-packages']
>>>
```
很神奇的发现，这是xcode里面的python。
好了，顺手把xcode给删了吧，好久没用而且估计也很难再用它了，要的时候再装就好了。
再删东西的时候，建议用**AppClean**去删除，免得一堆残留。

虽然在网上检索时，也知道Xcode会引入python，但是这个位置还是和[网上说的][1]不同的。
[这篇文章][2]也提供了类似的思路。

## 填坑
>重新安装Command Lines Tool

由于删掉了Xcode，在我印象中好像Xcode里面有一些东西是被依赖的。
所幸在查阅网上资料时，已经知道了这个东西是**Command Lines Tool**了。
参考[教程][3]进行对应的处理，如下：
```shell
$ tree /Library/Developer/CommandLineTools
/Library/Developer/CommandLineTools
└── usr
    └── share
        └── man
            └── whatis

3 directories, 1 file
```
缺少bin文件夹，应该是需要重新安装Command Lines Tool了。
有兴趣的话，可以[深入浅出一下Command Lines Tool][4]。


[1]: https://segmentfault.com/q/1010000022576249
[2]: https://www.cnblogs.com/jing99/p/13963048.html
[3]: https://segmentfault.com/a/1190000018045211
[4]: https://juejin.cn/post/6844904052271087624
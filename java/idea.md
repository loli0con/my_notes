# idea教程

## link
* [项目结构](https://mp.weixin.qq.com/s/j_yIiXvNlp7JMZaVO1FSrA)
* [debug](https://mp.weixin.qq.com/s/uOSgKtcyS-Qy0LSDROJ-xw)
* [大全\(稍旧)](https://blog.csdn.net/qq_27093465/article/details/77449117)
* [idea破解](https://gitee.com/pengzhile/ide-eval-resetter)
* [代码审查](https://mp.weixin.qq.com/s/tD0_q3xdU0pISVmN-bcZPg)

## vmoptions
```
-server
-Xms1024m
-Xmn1024m
-Xmx2048m
-XX:ReservedCodeCacheSize=1024m


# https://www.jianshu.com/p/ba2d613df94f
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

-XX:+UseConcMarkSweepGC
-XX:+UseParNewGC
-XX:+CMSParallelRemarkEnabled
-XX:+CMSScavengeBeforeRemark
-XX:CMSInitiatingOccupancyFraction=75
-XX:+UseCMSInitiatingOccupancyOnly
-XX:+UseCMSCompactAtFullCollection
-XX:CMSFullGCsBeforeCompaction=3
-XX:SoftRefLRUPolicyMSPerMB=50

-ea
-XX:CICompilerCount=2
-Dsun.io.useCanonPrefixCache=false
-Djdk.http.auth.tunneling.disabledSchemes=""
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Djdk.attach.allowAttachSelf=true
-Dkotlinx.coroutines.debug=off
-Djdk.module.illegalAccess.silent=true
-XX:+UseCompressedOops
-Dfile.encoding=UTF-8

-XX:ErrorFile=$USER_HOME/java_error_in_idea_%p.log
-XX:HeapDumpPath=$USER_HOME/java_error_in_idea.hprof
```
# idea教程

## link
* [项目结构](https://mp.weixin.qq.com/s/j_yIiXvNlp7JMZaVO1FSrA)
* [debug](https://mp.weixin.qq.com/s/uOSgKtcyS-Qy0LSDROJ-xw)
* [大全\(稍旧)](https://blog.csdn.net/qq_27093465/article/details/77449117)
* [idea破解](https://gitee.com/pengzhile/ide-eval-resetter)
* [代码审查](https://mp.weixin.qq.com/s/tD0_q3xdU0pISVmN-bcZPg)

## vmoptions（旧版）
```
-server
# 促使内存
-Xms1536m
# 最大可用内存
-Xmx2048m
# 年轻代大小
-Xmn768m

# 设置持久代初始值
-XX:MetaspaceSize=256m
# 设置持久代最大值
-XX:MaxMetaspaceSize=512m

-XX:ReservedCodeCacheSize=256m
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



## vmoptions（新版）
参考：
1. https://juejin.cn/post/6963835326821302303#heading-10
2. https://juejin.cn/post/6883641230664663047

```
-ea
-server
-Xms2048m
-Xmx3072m
-Xss16m
-XX:MaxMetaspaceSize=2G
-XX:MetaspaceSize=1G
-XX:ConcGCThreads=8
-XX:ParallelGCThreads=8
-XX:NewRatio=2
-XX:ReservedCodeCacheSize=240m
-XX:+AlwaysPreTouch
-XX:+UseG1GC
-XX:+DoEscapeAnalysis
-XX:+TieredCompilationUseG1GC
-XX:SoftRefLRUPolicyMSPerMB=50
-XX:+UnlockExperimentalVMOptions
-Dsun.io.useCanonPrefixCache=false
-Djava.net.preferIPv4Stack=true
-Dsun.io.useCanonCaches=false
-XX:LargePageSizeInBytes=256m
-XX:+UseCodeCacheFlushing
-XX:+DisableExplicitGC
-XX:+ExplicitGCInvokesConcurrent
-XX:+AggressiveOpts
-XX:+CMSClassUnloadingEnabled
-XX:CMSInitiatingOccupancyFraction=60
-XX:+CMSParallelRemarkEnabled
-XX:+UseAdaptiveGCBoundary
-XX:+UseSplitVerifier
-XX:CompileThreshold=10000
-XX:+OptimizeStringConcat
-XX:+UseStringCache
-XX:+UseFastAccessorMethods
-XX:+UnlockDiagnosticVMOptions
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-Djdk.attach.allowAttachSelf=true
-Dkotlinx.coroutines.debug=off
-Djdk.module.illegalAccess.silent=true
-XX:+UseCompressedOops
-Dfile.encoding=UTF-8
-XX:CICompilerCount=2
-Xverify:none
```
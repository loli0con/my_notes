# lsof
netstat在mac上不好用了,试试lsof

## 用途
lsof显示了当前在任何应用程序中打开的任何文件，您还可以使用它来检查与应用程序相关的开放端口(port)。

## 参数
* -n 禁止将网络号转换为主机名
* -P 禁用端口号到端口名的转换
* 不加 sudo 只能查看以当前用户运行的程序
* -s 强制显示文件大小(file size)。但是和-i成对出现时，它的含义就不同了：它允许用户**指定要返回的命令的协议和状态**
* -i 展示了所有打开的网络连接(open network connections) 和使用这个连接(connection)的进程(process)的名称。
  * -i4 将展示IPv4连接
  * -i6 将展示IPv6连接
  * -TCP 将展示TCP连接
  * -UDP 将展示UDP连接

## 实例
* lsof -iTCP -sTCP:LISTEN
  * 此命令返回状态为LISTEN的每个TCP连接，显示Mac上所有打开的TCP端口。它还列出了与那些打开的端口关联的进程。
* sudo lsof -nP -iTCP:端口号 -sTCP:LISTEN

顺带，kill -9 <PID>

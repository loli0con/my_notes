# profiling
通过profiling可以对一条SQL语句在MySQL不同模块中的执行时间进行分析。

##  确认profiling 是否开启
通过`select @@profiling;`或`show variables like '%profiling%'`查看是否是开启。开启profiling后，MySQL会收集SQL执行时使用资源的情况.

profiling=0代表关闭，我们需要把 profiling 打开，即设置为1：`set profiling=1;`

## 多次执行相同SQL查询
```SQL
select * from employees;
select * from employees;
```

## 查看profiles
查看当前会话所产生的所有profiles：
```MySQL
show profiles; # 显示最近的几次查询
```

![profiling+20240228134237](https://raw.githubusercontent.com/loli0con/picgo/master/images/profiling+20240228134237.png+2024-02-28-13-42-38)

## 查看profile
显示执行计划，查看程序的执行步骤：
```MySQL
show profile;
```
![profiling+20240228134358](https://raw.githubusercontent.com/loli0con/picgo/master/images/profiling+20240228134358.png+2024-02-28-13-44-00)

* 以查询指定的 Query ID，比如：`show profile for query 7;`。
* 还可以查询更丰富的内容：`show profile cpu,block io for query 6;`。

具体语法：
```
Syntax:
SHOW PROFILE [type [, type] ... ]
    [FOR QUERY n]
    [LIMIT row_count [OFFSET offset]]

type: {
    ALL              -- 显示所有参数的开销信息 
  | BLOCK IO         -- 显示IO的相关开销
  | CONTEXT SWITCHES -- 上下文切换相关开销
  | CPU              -- 显示CPU相关开销信息
  | IPC              -- 显示发送和接受相关开销信息
  | MEMORY           -- 显示内存相关开销信息
  | PAGE FAULTS      -- 显示页面错误相关开销信息
  | SOURCE           -- 显示和Source_function,Source_file,Source_line相关的开销信息
  | SWAPS            -- 显示交换次数相关的开销信息
}
```
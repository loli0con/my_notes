# MySQL8程序



## 安装（red hat）
|安装方式|特点|
|---|---|
|rpm|安装简单，灵活性差，无法灵活选择版本、升级|
|rpm repository|安装包极小，版本安装简单灵活，升级方便，需要联网安装|
|通用二进制包|安装比较复杂，灵活性高，平台通用性好|
|源码包|安装最复杂，时间长，参数设置灵活，性能好|

[官方下载地址](https://downloads.mysql.com/archives/community/) 直接点Download下载RPM Bundle全量包。



## 服务配置

### 服务的初始化
为了保证数据库目录与文件的所有者为 mysql 登录用户，如果你是以 root 身份运行 mysql 服务，需要执
行下面的命令初始化：`mysqld --initialize --user=mysql`。

`--initialize` 选项默认以“安全”模式来初始化，则会为 root 用户生成一个密码并将该密码标记为过期，登录后你需要设置一个新的密码。生成的**临时密码**会往日志中记录一份。查看密码：`cat /var/log/mysqld.log`。

### 服务的管理
加不加.service后缀都可以：
* 启动: `systemctl start mysqld.service`
* 关闭: `systemctl stop mysqld.service`
* 重启: `systemctl restart mysqld.service`
* 查看状态: `systemctl status mysqld.service`
* 设置自启动: `systemctl enable mysqld.service`
* 禁止自启动: `systemctl disable mysqld.service`



## 远程访问

### 调整防火墙

#### （方式1）关闭防火墙
```sh
systemctl start firewalld.service
systemctl status firewalld.service
systemctl stop firewalld.service

# 设置开机启用防火墙
systemctl enable firewalld.service
# 设置开机禁用防火墙
systemctl disable firewalld.service
```

#### （方式2）开放端口
```sh
# 查看开放的端口号
firewall-cmd --list-all

# 设置开放的端口号    
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=3306/tcp --permanent

# 重启防火墙
firewall-cmd --reload
```

### 调整root账号

#### 查看root账号
登录MySQL后输入如下命令：
```sql
use mysql;
select Host,User from user;
```

![MySQL8安装+20231113162206](https://raw.githubusercontent.com/loli0con/picgo/master/images/MySQL8%E5%AE%89%E8%A3%85%2B20231113162206.png%2B2023-11-13-16-22-07)
可以看到root用户的当前主机配置信息为localhost。

#### 修改Host为通配符%
Host列指定了允许用户登录所使用的IP。比如`Host=192.168.1.1 | User=root`，这里的意思就是说root用户只能通过192.168.1.1的客户端去访问。`Host=localhost | User=root`，表示只能通过本机客户端去访问。而`%`是个**通配符**，如果`Host=192.168.1.%`，那么就表示只要是IP地址前缀为“192.168.1.”的客户端都可以连接。如果`Host=%`，表示所有IP都有连接权限。

Host设置了“%”后便可以允许远程访问：
```SQL
update user set host = '%' where user ='root';
```

![MySQL8安装+20231113162621](https://raw.githubusercontent.com/loli0con/picgo/master/images/MySQL8%E5%AE%89%E8%A3%85%2B20231113162621.png%2B2023-11-13-16-26-22)

Host修改完成后记得执行`flush privileges`使配置立即生效。



## 密码策略
MySQL8 引入了服务器组件(Components)这个特性，validate_password插件已用服务器组件重新实现。

当validate_password组件已安装时：
```SQL
mysql> SELECT * FROM mysql.component;
+--------------+--------------------+------------------------------------+
| component_id | component_group_id | component_urn                      |
+--------------+--------------------+------------------------------------+
|            1 |                  1 | file://component_validate_password  |
+--------------+--------------------+------------------------------------+
1 row in set (0.00 sec)

mysql> show variables like 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file     |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.01 sec)
```

关于 **validate_password** 组件对应的系统变量说明:
|选项|默认值|参数描述|
|---|---|---|
|check_user_name|ON|设置为ON的时候表示能将密码设置成当前用户名。|
|dictionary_file||用于检查密码的字典文件的路径名，默认为空|
|length|8|密码的最小长度，也就是说密码长度必须大于或等于8|
|mixed_case_count|1|如果密码策略是中等或更强的，validate_password要求密码具有的小写和大写字符的最小数量。对于给定的这个值密码必须有那么多小写字符和那么多大写字符。|
|number_count|1|密码必须包含的数字个数|
|policy|MEDIUM|密码强度检验等级，可以使用数值0、1、2或相应的符号值LOW、MEDIUM、STRONG来指定。</br>**0/LOW**：只检查长度。</br>**1/MEDIUM**：检查长度、数字、大小写、特殊字符。</br>**2/STRONG**：检查长度、数字、大小写、特殊字符、字典文件。|
|special_char_count|1|密码必须包含的特殊字符个数|



## 字符集

### 字符集的比较规则
MySQL版本一共支持41种字符集，其中的`Default collation`列表示这种字符集中一种默认的比较规则，里面包含着该比较规则主要作用于哪种语言，比如*utf8_polish_ci*表示以波兰语的规则比较，*utf8_spanish_ci*是以西班牙语的规则比较，*utf8_general_ci*是一种通用的比较规则。

后缀表示该比较规则是否区分语言中的重音、大小写。具体如下:
|后缀|英文释义|描述|
|---|---|---|
|_ai|accent insensitive|不区分重音|
|_as|accent sensitive|区分重音|
|_ci|case insensitive|不区分大小写|
|_cs|case sensitive|区分大小写|
|_bin|binary|以二进制方式比较|

### 请求到响应过程中字符集的变化
|系统变量|描述|
|---|---|
|**character_set_client**|服务器解码请求时使用的字符集|
|**character_set_connection**|服务器处理请求时会把请求字符串从**character_set_client**转为**character_set_connection**|
|**character_set_results**|服务器向客户端返回数据时使用的字符集|

![MySQL8安装+20231113194725](https://raw.githubusercontent.com/loli0con/picgo/master/images/MySQL8%E5%AE%89%E8%A3%85%2B20231113194725.png%2B2023-11-13-19-47-27)



## SQL大小写规范
在 SQL 中，关键字和函数名是不用区分字母大小写的，比如 SELECT、WHERE、ORDER、GROUP BY等关键字，以及 ABS、MOD、ROUND、MAX 等函数名。

### 配置参数
lower_case_table_names参数值的设置:
* 默认为0，大小写敏感。
* 设置1，大小写不敏感。创建的表和数据库都是以小写形式存放在磁盘上，对于sql语句都是转换为小写后再对表和数据库进行查找。
* 设置2，创建的表和数据库依据语句上格式存放，凡是查找都是转换为小写进行。

### Windows和Linux的默认配置
在 Windows 和 Linux 环境下，**windows系统默认大小写不敏感**，**linux系统是大小写敏感的**。

MySQL在Linux下：
1. 数据库名、表名、表的别名、变量名是严格区分大小写的
2. 关键字、函数名称在 SQL 中不区分大小写
3. 列名(字段名)与列的别名(字段别名)在所有的情况下均是忽略大小写的

MySQL在Windows的环境下全部不区分大小写。

### 命名规范
1. 关键字和函数名称全部大写;
2. 数据库名、表名、表别名、字段名、字段别名等全部小写;
3. SQL 语句必须以分号结尾。



## SQL_MODE

### 介绍
sql_mode会影响MySQL支持的SQL语法以及它执行的数据验证检查。通过设置sql_mode，可以完成不同严格程度的数据校验，有效地保障数据准确性。

MySQL服务器可以在不同的SQL模式下运行，并且可以针对不同的客户端以不同的方式应用这些模式，具体取决于sql_mode系统变量的值。
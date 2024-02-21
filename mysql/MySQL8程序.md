# MySQL8程序



## MySQL8安装（red hat系）
|安装方式|特点|
|---|---|
|rpm|安装简单，灵活性差，无法灵活选择版本、升级|
|rpm repository|安装包极小，版本安装简单灵活，升级方便，需要联网安装|
|通用二进制包|安装比较复杂，灵活性高，平台通用性好|
|源码包|安装最复杂，时间长，参数设置灵活，性能好|

[官方下载地址](https://downloads.mysql.com/archives/community/) 直接点Download下载RPM Bundle全量包。



## mysqld服务配置

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



## 远程登录

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

### 修改MySQL配置

#### 查看配置
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
|validate_password_check_user_name|ON|设置为ON的时候表示能将密码设置成当前用户名。|
|validate_password_dictionary_file||用于检查密码的字典文件的路径名，默认为空|
|validate_password_length|8|密码的最小长度，也就是说密码长度必须大于或等于8|
|validate_password_mixed_case_count|1|如果密码策略是中等或更强的，validate_password要求密码具有的小写和大写字符的最小数量。对于给定的这个值密码必须有那么多小写字符和那么多大写字符。|
|validate_password_number_count|1|密码必须包含的数字个数|
|validate_password_policy|MEDIUM|密码强度检验等级，可以使用数值0、1、2或相应的符号值LOW、MEDIUM、STRONG来指定。</br>**0/LOW**：只检查长度。</br>**1/MEDIUM**：检查长度、数字、大小写、特殊字符。</br>**2/STRONG**：检查长度、数字、大小写、特殊字符、字典文件。|
|validate_password_special_char_count|1|密码必须包含的特殊字符个数|



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



## 默认数据库
* **mysql**：MySQL 系统自带的核心数据库，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等。
* **information_schema**：MySQL 系统自带的数据库，这个数据库保存着MySQL服务器**维护的所有其他数据库的信息**，比如有哪些表、哪些视图、哪些触发器、哪些列、哪些索引。这些信息并不是真实的用户数据，而是一些描述性信息，有时候也称之为**元数据**。在系统数据库 information_schema 中提供了一些以 innodb_sys 开头的表，用于表示内部系统表。
* **performance_schema**：MySQL 系统自带的数据库，这个数据库里主要保存MySQL服务器运行过程中的一些状态信息，可以用来**监控 MySQL 服务的各类性能指标**。包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等信息。
* **sys**：MySQL 系统自带的数据库，这个数据库主要是通过**视图**的形式把 information_schema 和 performance_schema 结合起来，帮助系统管理员和开发人员监控 MySQL 的技术性能。

## 主要目录结构
* 相关命令目录: /usr/bin(mysqladmin、mysqlbinlog、mysqldump等命令)和/usr/sbin
* 配置文件目录: /usr/share/mysql-8.0(命令及配置文件)和/etc/mysql(如my.cnf)
* MySQL数据库文件的存放路径: /var/lib/mysql/

### 数据库文件

#### 数据库在文件系统中的表示
除了 information_schema 这个系统数据库外，其他的数据库在**数据目录（/var/lib/mysql/）**下都有对应的子目录。

#### 表在文件系统中的表示（InnoDB存储引擎）

##### 表结构
为了保存表结构，InnoDB在**数据目录**下对应的数据库子目录下创建了一个专门用于**描述表结构的文件**，文件名为：`表名.frm`。frm文件的格式在不同的平台上都是相同的，以**二进制格式**存储。

MySQL8中不再单独提供frm文件，而是合并在[ibd文件](#独立表空间file-per-table-tablespace)中。

##### 表中数据和索引

###### 系统表空间(system tablespace)
默认情况下，InnoDB会在数据目录下创建一个名为**idbdata**、大小为**12M**的文件，这个文件就是对应的**系统表空间**在文件系统上的表示。这个文件是**自扩展文件**，当不够用的时候它会自己增加文件大小。

如果想让系统表空间对应文件系统上多个实际文件，或者想修改文件名，那可以在MySQL启动时配置对应的文件路径以及它们的大小，比如修改my.cnf配置文件：
```cnf
[server]
innodb_data_file_path=data1:512M;data2:512M:autoextend
```

###### 独立表空间(file-per-table tablespace)
InnoDB为**一个表建立一个独立表空间**，使用**独立表空间**来存储表中的数据和索引。在表所属数据库对应的子目录下存在一个表示该独立表空间的文件，文件名和表名相同，扩展名为**ibd**，完整的文件名称为：`表名.ibd`。
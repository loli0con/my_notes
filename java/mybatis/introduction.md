# 简介

## 什么是 MyBatis？

### 官方回答
* MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。
* MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
* MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 解读
1. *持久层*的*框架*
2. 功能强大（依赖JDBC驱动程序，见⬇️）
   1. 映射——将 *Java中的**类/对象*** 映射为 *数据库中的**表/记录***
   2. 自定义SQL
   3. 存储过程
3. 封装JDBC（提供强大的功能，见⬆️）
4. 通过简单的**XML**来进行配置

![introduction+mybatis-mybaits简介](https://raw.githubusercontent.com/loli0con/picgo/master/images/introduction%2Bmybatis-mybaits%E7%AE%80%E4%BB%8B.png%2B2021-07-05-23-06-10)

## 开发过程
![introduction+mybatis-demo](https://i.loli.net/2021/08/18/rihTQBog6PSKtIx.png)

## 核心概念
每个基于 MyBatis 的应用都是以一个 **SqlSessionFactory** 的实例为核心的。  
SqlSessionFactory 的实例可以通过 **SqlSessionFactoryBuilder** 获得。  
而 SqlSessionFactoryBuilder 则可以从 **XML 配置文件**或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。


既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 **SqlSession** 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 **SQL 语句**。
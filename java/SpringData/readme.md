# Spring Data
Spring Data 项目是从 2010 年开发发展起来的，从创立之初 Spring Data 就想提供一个大家熟悉的、一致的、基于 Spring 的**数据访问编程模型**，同时仍然保留底层数据存储的特殊特性。它可以轻松地让开发者使用数据访问技术包括：关系数据库、非关系数据库（NoSQL）和基于云的数据服务。

Spring Data Common 是 Spring Data 所有模块的公用部分，该项目提供跨 Spring 数据项目的共享基础设施，它包含了技术中立的库接口以及一个坚持 Java 类的元数据模型。

Spring Data 不仅对传统的数据库访问技术：JDBC、Hibernate、JDO、TopLick、JPA、MyBatis 做了很好的支持和扩展、抽象、提供方便的 API，还对 NoSQL 等非关系数据做了很好的支持：MongoDB 、Redis、Apache Solr 等。

## 子项目
Spring Data 的子项目：
* 主要项目（Main Modules）：
  * Spring Data Commons
  * Spring Data Gemfire
  * Spring Data JPA
  * Spring Data KeyValue
  * Spring Data LDAP
  * Spring Data MongoDB
  * Spring Data REST
  * Spring Data Redis
  * Spring Data for Apache Cassandra
  * Spring Data for Apache Solr
* 社区支持的项目（Community Modules）：
  * Spring Data Aerospike
  * Spring Data Couchbase
  * Spring Data DynamoDB
  * Spring Data Elasticsearch
  * Spring Data Hazelcast
  * Spring Data Jest
  * Spring Data Neo4j
  * Spring Data Vault
* 其他（Related Modules）：
  * Spring Data JDBC Extensions
  * Spring for Apache Hadoop
  * Spring Content

![readme+20211202193326](https://raw.githubusercontent.com/loli0con/picgo/master/images/readme%2B20211202193326.png%2B2021-12-02-19-33-27)

## 主要特性
Spring Data 项目旨在为大家提供一种通用的编码模式，数据访问对象实现了对物理数据层的抽象，为编写查询方法提供了方便。通过对象映射，实现域对象和持续化存储之间的转换，而模板提供的是对底层存储实体的访问实现，操作上主要有如下特征：
* 提供模板操作，如 Spring Data Redis 和 Spring Data Riak；
* 强大的 Repository 和定制的数据储存对象的抽象映射；
* 对数据访问对象的支持（Auting 等）。

![readme+20211202193635](https://raw.githubusercontent.com/loli0con/picgo/master/images/readme%2B20211202193635.png%2B2021-12-02-19-36-35)
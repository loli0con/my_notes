# 简介

## 什么是 MyBatis？

### 官方回答
* MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。
* MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
* MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

### 解读
1. **持久层框架**
2. 功能强大（依赖JDBC驱动程序，见⬇️）
   1. 映射——将 *Java中的**类/对象*** 映射为 *数据库中的**表/记录***
   2. 自定义SQL
   3. 存储过程
3. 封装JDBC（提供强大的功能，见⬆️）
4. 通过简单的**XML**来进行配置

![introduction+mybatis-mybaits简介.drawio](https://raw.githubusercontent.com/loli0con/picgo/master/images/introduction%2Bmybatis-mybaits%E7%AE%80%E4%BB%8B.drawio.png%2B2022-08-05-16-42-10)

## 开发过程
![introduction+mybatis-demo](https://i.loli.net/2021/08/18/rihTQBog6PSKtIx.png)

### SqlSessionFactory - 会话工厂
每个基于 MyBatis 的应用都是以一个 **SqlSessionFactory** 的实例为核心的。SqlSessionFactory 的实例可以通过**SqlSessionFactoryBuilder** 获得。而 SqlSessionFactoryBuilder 则可以从 **XML 配置文件**或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。

```Java
// 全局配置文件：mybatis-config.xml（习惯性命名，非强制要求）
String resource = "org/mybatis/example/mybatis-config.xml";
// Mybatis 自带的 Resources工具类
InputStream inputStream = Resources.getResourceAsStream(resource);
// SqlSessionFactoryBuilder 通过 InputStream 实例构建 sqlSessionFactory，也可采任意方式获得该 InputStream 实例
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

### SqlSession - 会话
既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 **SqlSession** 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 **SQL 语句**。

```Java
// 使用 try 语法，自动关闭该回话
try (SqlSession session = sqlSessionFactory.openSession()) {
   /*
    * 此处的 Mapper 接口为 BlogMapper，Entity 实体为 Blog
    * 假定已通过 org.mybatis.example.BlogMapper 接口的 selectBlog 方法完成了SQL语句的映射
    * 此处直接使用该方法执行SQL命令，该方法的入参为‘101’，返回类型为 Blog
    */


  // 方式1（不推荐）
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);

  // 方式2（推荐）
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

## 作用域（选读）

### SqlSessionFactoryBuilder - 用完即丢
这个类可以被实例化、使用和丢弃，一旦创建了 SqlSessionFactory，就不再需要它了。 因此 SqlSessionFactoryBuilder 实例的**最佳作用域是方法作用域**（也就是局部方法变量）。 你可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

### SqlSessionFactory - 全局唯一
SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例。 使用 SqlSessionFactory 的最佳实践是在应用运行期间不要重复创建多次，多次重建 SqlSessionFactory 被视为一种代码“坏习惯”。因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

### SqlSession - 方法作用域
每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的**最佳的作用域是请求或方法作用域**。 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 Servlet 框架中的 HttpSession。 如果你现在正在使用一种 Web 框架，考虑将 SqlSession 放在一个和 HTTP 请求相似的作用域中。 换句话说，每次收到 HTTP 请求，就可以打开一个 SqlSession，返回一个响应后，就关闭它。 这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 finally 块中。 下面的示例就是一个确保 SqlSession 关闭的标准模式：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  // 你的应用逻辑代码
}
```

在所有代码中都遵循这种使用模式，可以保证所有数据库资源都能被正确地关闭。

### 映射器实例 - 方法作用域
映射器是一些绑定映射语句的接口。映射器接口的实例是从 SqlSession 中获得的。虽然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但**方法作用域才是映射器实例的最合适的作用域**。 也就是说，映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃。 映射器实例并不需要被显式地关闭。尽管在整个请求作用域保留映射器实例不会有什么问题，但是你很快会发现，在这个作用域上管理太多像 SqlSession 的资源会让你忙不过来。 因此，最好将映射器放在方法作用域内。就像下面的例子一样：

```java
try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  // 你的应用逻辑代码
}
```
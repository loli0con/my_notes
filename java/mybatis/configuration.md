# 配置
此处描述的配置是mybatis的全局配置。
MyBatis的配置文件包含了会深深影响MyBatis行为的设置和属性信息。

配置文档的顶层结构如下：
* configuration（配置）
  * properties（属性）
  * settings（设置）
  * typeAliases（类型别名）
  * typeHandlers（类型处理器）
  * objectFactory（对象工厂）
  * plugins（插件）
  * environments（环境配置）
    * environment（环境变量）
      * transactionManager（事务管理器）
      * dataSource（数据源）
  * databaseIdProvider（数据库厂商标识）
  * mappers（映射器）

## properties（属性）
通过引入典型的 Java 属性文件，或者在 properties 元素的子元素中设置属性，然后使用这些设置好的属性就可以在整个配置文件中替换需要动态配置的属性值。

### 示例
```xml
<!-- 一个例子 -->

<!-- 设置属性 -->
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性，还可以修改通过value属性修改默认值的分隔符(默认为“:”) -->
</properties>
<!-- 使用属性 -->
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username:ut_user}"/> <!-- 默认值特性(需要手动开启)：如果属性 'username' 没有被配置，'username' 属性的值将为 'ut_user' -->
  <property name="password" value="${password}"/>
</dataSource>
```
### 顺序
如果一个属性在不只一个地方进行了配置，那么，MyBatis 将按照下面的顺序来加载（越往下，优先级越高）：
1. 首先读取在 properties 元素体内指定的属性。
2. 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性。
3. 最后读取作为方法参数（构建SqlSessionFactory的方法）传递的属性，并覆盖之前读取过的同名属性。

## settings（设置）
|设置名|描述|有效值|默认值|
|---|---|---|---|
|cacheEnabled|全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。|true或false|true|
|lazyLoadingEnabled|延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。|true或false|false|
|aggressiveLazyLoading|开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 lazyLoadTriggerMethods)。|true或false|false （在 3.4.1 及之前的版本中默认为 true）|
|multipleResultSetsEnabled|是否允许单个语句返回多结果集（需要数据库驱动支持）。|true或false|true|
|useColumnLabel|使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。|true或false|true|
|useGeneratedKeys|允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。|true或false|false|
|autoMappingBehavior|指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。|NONE, PARTIAL, FULL|PARTIAL|
|autoMappingUnknownColumnBehavior|指定发现自动映射目标未知列（或未知属性类型）的行为。 NONE: 不做任何反应 WARNING: 输出警告日志（'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN） FAILING: 映射失败 (抛出 SqlSessionException)|NONE, WARNING, FAILING|NONE|
|defaultExecutorType|配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。|SIMPLE REUSE BATCH|SIMPLE|
|defaultStatementTimeout|设置超时时间，它决定数据库驱动等待数据库响应的秒数。|任意正整数|未设置 (null)|
|defaultFetchSize|为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。|任意正整数|未设置 (null)|
|defaultResultSetType|指定语句默认的滚动策略。（新增于 3.5.2）|FORWARD_ONLY或SCROLL_SENSITIVE或SCROLL_INSENSITIVE或DEFAULT（等同于未设置）|未设置 (null)|
|safeRowBoundsEnabled|是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。|true或false|false|
|safeResultHandlerEnabled|是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。|true或false|true|
|mapUnderscoreToCamelCase|是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。|true或false|false|
|localCacheScope|MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。|SESSION或STATEMENT|SESSION|
|jdbcTypeForNull|当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。|JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。|OTHER|
|lazyLoadTriggerMethods|指定对象的哪些方法触发一次延迟加载。|用逗号分隔的方法列表。|equals,clone,hashCode,toString|
|defaultScriptingLanguage|指定动态 SQL 生成使用的默认脚本语言。|一个类型别名或全限定类名。|org.apache.ibatis.scripting.xmltags.XMLLanguageDriver|
|defaultEnumTypeHandler|指定 Enum 使用的默认 TypeHandler 。（新增于 3.4.5）|一个类型别名或全限定类名。|org.apache.ibatis.type.EnumTypeHandler|
|callSettersOnNulls|指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。|true或false|false|
|returnInstanceForEmptyRow|当返回行的所有列都是空时，MyBatis默认返回 null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2）|true或false|false|
|logPrefix|指定 MyBatis 增加到日志名称的前缀。|任何字符串|未设置|
|logImpl|指定 MyBatis 所用日志的具体实现，未指定时将自动查找。|SLF4J或LOG4J或LOG4J2或JDK_LOGGING或COMMONS_LOGGING或STDOUT_LOGGING或NO_LOGGING|未设置|
|proxyFactory|指定 Mybatis 创建可延迟加载对象所用到的代理工具。|CGLIB或JAVASSIST|JAVASSIST （MyBatis 3.3 以上）|
|vfsImpl|指定 VFS 的实现|自定义 VFS 的实现的类全限定名，以逗号分隔。|未设置|
|useActualParamName|允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 -parameters 选项。（新增于 3.4.1）|true或false|true|
|configurationFactory|指定一个提供 Configuration 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为static Configuration getConfiguration() 的方法。（新增于 3.2.3）|一个类型别名或完全限定类名。|未设置|
shrinkWhitespacesInSql|从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5)|true或false|false|
|defaultSqlProviderType|Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the type(or value) attribute on sql provider annotation(e.g. @SelectProvider), when these attribute was omitted.|A type alias or fully qualified class name|Not set|

## typeAliases（类型别名）
类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置，意在降低冗余的全限定类名书写。

```xml
<typeAliases>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <!-- 当这样配置时，Blog 可以用在任何使用 domain.blog.Blog 的地方。 -->

  <package name="domain.blog"/>
  <!-- 也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean -->
  <!-- 每一个在包 domain.blog 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 -->
  <!-- 若有注解(@Alias)，则别名为其注解值。 -->
</typeAliases>
```

常见的 Java 类型内建的类型别名，都是使用大写开头，例如：
* list(内建类) -> List(别名)
* int(内建类) -> 	Integer(别名)

## typeHandlers（类型处理器）
MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。

你可以重写已有的类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler， 并且可以（可选地）将它映射到一个 JDBC 类型。

## objectFactory（对象工厂）
每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。 如果想覆盖对象工厂的默认行为，可以通过创建自己的对象工厂来实现。

## plugins（插件）
MyBatis 允许你在映射语句执行过程中的某一点进行拦截调用。

## environments（环境配置）
MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中。

尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。为了指定创建哪种环境，只要将它作为可选的参数传递给 SqlSessionFactoryBuilder 即可。如果忽略了环境参数，那么将会加载默认环境。

### 示例
environments 元素定义了如何配置环境。
```xml
<!-- default设置默认环境为“development” -->
<environments default="development"> 
  <!-- 设置开发(development)环境 -->
  <environment id="development">
    <!-- 设置事务管理器为JDBC -->
    <transactionManager type="JDBC">
      <!-- 配置事务管理器的属性参数 -->
      <property name="..." value="..."/>
    </transactionManager>
    <!-- 配置数据源，一般为连接池 -->
    <dataSource type="POOLED">
      <!-- 配置数据源的属性参数 -->
      <!-- 配置数据库驱动程序 -->
      <property name="driver" value="${driver}"/>
      <!-- 配置数据库的url -->
      <property name="url" value="${url}"/>
      <!-- 配置访问数据库的用户名 -->
      <property name="username" value="${username}"/>
      <!-- 配置访问据库的密码 -->
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

### transactionManager（事务管理器）
在 MyBatis 中有两种类型的事务管理器（也就是 type="\[JDBC|MANAGED]"）：
* JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
* MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器（例如Spring容器）来管理事务的整个生命周期。

通常这两种事务管理器类型都不需要设置任何属性。

### dataSource（数据源）
有三种内建的数据源类型（也就是 type="\[UNPOOLED|POOLED|JNDI]"）：
* UNPOOLED：这个数据源的实现会每次请求时打开和关闭连接。
* POOLED：这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。
* JNDI：这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。

实际项目中，一般使用外建的、性能更好的数据源：
* C3P0
* druid


## databaseIdProvider（数据库厂商标识）
MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。 MyBatis 会加载带有匹配当前数据库 databaseId 属性和所有不带 databaseId 属性的语句。 如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃。 


## mappers（映射器）
定义 SQL 映射语句后，需要告诉 MyBatis 到哪里去找到这些语句。这个配置配置会告诉 MyBatis 去哪里找映射文件。

### 示例
```xml
<mappers>
  <!-- 使用相对于类路径的资源引用 -->
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  
  <!-- 使用完全限定资源定位符（URL） -->
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>

  <!-- 使用映射器接口实现类的完全限定类名 -->
  <mapper class="org.mybatis.builder.AuthorMapper"/>

  <!-- 将包内的映射器接口实现全部注册为映射器 -->
  <package name="org.mybatis.builder"/>
</mappers>
```
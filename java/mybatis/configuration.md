# 全局配置

## 结构
XML配置文件的结构如下：
* configuration（配置）
  * properties（属性）
  * settings（设置）
  * typeAliases（类型别名）
  * typeHandlers（类型处理器）
  * objectFactory（对象工厂）
  * objectWrapperFactory（对象包装工厂）
  * reflectorFactory（反射器工厂）
  * plugins（插件）
  * environments（环境配置）
    * environment（环境变量）
      * transactionManager（事务管理器）
      * dataSource（数据源）
  * databaseIdProvider（数据库厂商标识）
  * mappers（映射器）

XML配置文件中的标签必须按照上述的顺序。



## demo
一个简单的🌰
```XML
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//MyBatis.org//DTD Config 3.0//EN"
        "http://MyBatis.org/dtd/MyBatis-3-config.dtd">

<configuration>
  <!--引入properties文件，此时就可以${属性名}的方式访问属性值-->
  <properties resource="jdbc.properties"></properties>
  
  <settings>
    <!--将表中字段的下划线自动转换为驼峰-->
    <setting name="mapUnderscoreToCamelCase" value="true"/> 
    <!--开启延迟加载-->
    <setting name="lazyLoadingEnabled" value="true"/>
  </settings>
  
  <typeAliases>
    <!--
        typeAlias:设置某个具体的类型的别名
        属性:
          type:需要设置别名的类型的全类名
          alias:设置此类型的别名，
                若不设置此属性，该类型拥有默认的别名，即类名且不区分大小写
                若设置此属性，此时该类型的别名只能使用alias所设置的值 
    -->
    <!--<typeAlias type="com.atguigu.mybatis.bean.User"></typeAlias>-->
    <!--<typeAlias type="com.atguigu.mybatis.bean.User" alias="abc"></typeAlias>-->

    <!--以包为单位，设置改包下所有的类型都拥有默认的别名，即类名且不区分大小写-->
    <package name="com.atguigu.mybatis.bean"/>
  </typeAliases>

  <!--
    environments:设置多个连接数据库的环境
    属性:
      default:设置默认使用的环境的id
  -->
  <environments default="mysql_test">
    <!--
        environment:设置具体的连接数据库的环境信息
        属性:
          id:设置环境的唯一标识
    -->
    <environment id="mysql_test">
      <!--
          transactionManager:设置事务管理方式
          属性:
            type:设置事务管理方式，type="JDBC|MANAGED"
              当type="JDBC":设置当前环境的事务管理都必须手动处理
              当type="MANAGED":设置事务被管理，例如通过spring中的AOP进行管理
      -->
      <transactionManager type="JDBC"/>
      <!--
          dataSource:设置数据源
          属性:
            type:设置数据源的类型type="POOLED|UNPOOLED|JNDI"
              当type="POOLED":使用数据库连接池，即会将创建的连接进行缓存，下次使用可以从缓存中直接获取，不需要重新创建
              当type="UNPOOLED":不使用数据库连接池，即每次使用连接都需要重新创建
              当type="JNDI":调用上下文中的数据源
      -->
      <dataSource type="POOLED">
        <!--设置驱动类的全类名-->
        <property name="driver" value="${jdbc.driver}"/>
        <!--设置连接数据库的连接地址-->
        <property name="url" value="${jdbc.url}"/>
        <!--设置连接数据库的用户名-->
        <property name="username" value="${jdbc.username}"/>
        <!--设置连接数据库的密码-->
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>

  <!--引入映射文件-->
  <mappers>
    <!-- 以单个mapper配置文件为单位 -->
    <mapper resource="UserMapper.xml"/>

    <!--
        以包为单位，将包下所有的映射文件引入核心配置文件
        注意:此方式必须保证mapper接口和mapper映射文件必须在相同的包下
    -->
    <package name="com.atguigu.mybatis.mapper"/>
  </mappers>
</configuration>
```



## properties（属性）
通过引入典型的 Java 属性文件，或者在 properties 元素的子元素中设置属性，然后使用这些设置好的属性就可以在整个配置文件中替换需要动态配置的属性值。

### 示例
```xml
<!-- 一个例子 -->

<!-- 设置属性 -->
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性 -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value=":"/> <!-- 默认值的分隔符 -->
</properties>

<!-- 使用属性 -->
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username:ut_user}"/> <!-- 默认值特性(需要手动开启，见上面的enable-default-value，属性和默认值之间的分隔符由default-value-separator控制)：如果属性 'username' 没有被配置，'username' 属性的值将为 'ut_user' -->
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
|aggressiveLazyLoading|开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 lazyLoadTriggerMethods）。|true或false|false （在 3.4.1 及之前的版本中默认为 true）|
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
|logImpl|指定 MyBatis 所用日志的具体实现，未指定时将自动查找。|SLF4J或LOG4J（3.5.9 起废弃）或LOG4J2或JDK_LOGGING或COMMONS_LOGGING或STDOUT_LOGGING或NO_LOGGING|未设置|
|proxyFactory|指定 Mybatis 创建可延迟加载对象所用到的代理工具。|CGLIB（3.5.10 起废弃）或JAVASSIST|JAVASSIST （MyBatis 3.3 以上）|
|vfsImpl|指定 VFS 的实现|自定义 VFS 的实现的类全限定名，以逗号分隔。|未设置|
|useActualParamName|允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 -parameters 选项。（新增于 3.4.1）|true或false|true|
|configurationFactory|指定一个提供 Configuration 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为static Configuration getConfiguration() 的方法。（新增于 3.2.3）|一个类型别名或完全限定类名。|未设置|
|shrinkWhitespacesInSql|从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5)|true或false|false|
|defaultSqlProviderType|指定一个拥有 provider 方法的 sql provider 类（新增于 3.5.6）。这个类适用于指定 sql provider 注解上的type（或 value） 属性（当这些属性在注解中被忽略时）。 (例如@SelectProvider)|类型别名或者全限定名|未设置|
|nullableOnForEach	为 'foreach' 标签的 'nullable' 属性指定默认值。（新增于 3.5.9）|true或false|false|
|argNameBasedConstructorAutoMapping|当应用构造器自动映射时，参数名称被用来搜索要映射的列，而不再依赖列的顺序。（新增于 3.5.10）|true或false|false|

## typeAliases（类型别名）
类型别名可为 Java 类型设置一个缩写名字。 它仅用于 XML 配置（**全局配置mybatis-config.xml**和**SQL映射XML配置**），意在降低冗余的全限定类名书写。

```xml
<typeAliases>
  <!-- 当这样配置时，author1 可以用在任何使用 domain.blog.Author 的地方。 -->
  <typeAlias alias="author1" type="domain.blog.Author"/>

  <!-- 也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean -->
  <!-- 每一个在包 domain.blog 中的 Java Bean，在没有注解的情况下，会使用 Bean 的首字母小写的非限定类名来作为它的别名。 -->
  <!-- 例如 domain.blog.Author 的别名为 author  -->
  <package name="domain.blog"/>
</typeAliases>
```

若在类上有注解@Alias，则别名为其注解值。
```Java
// 表明为注解值author
@Alias("author")
public class Author {
    ...
}
```

常见的 Java 类型内建的类型别名如下：
|别名|映射的类型|
|---|---|
|_byte|byte|
|_char|char|
|_character|char|
|_long|long|
|_short|short|
|_int|int|
|_integer|int|
|_double|double|
|_float|float|
|_boolean|boolean|
|string|String|
|byte|Byte|
|char|Character|
|character|Character|
|long|Long|
|short|Short|
|int|Integer|
|integer|Integer|
|double|Double|
|float|Float|
|boolean|Boolean|
|date|Date|
|decimal|BigDecimal|
|bigdecimal|BigDecimal|
|biginteger|BigInteger|
|object|Object|
|date\[]|Date[]|
|decimal\[]|BigDecimal[]|
|bigdecimal\[]|BigDecimal[]|
|biginteger\[]|BigInteger[]|
|object\[]|Object[]|
|map|Map|
|hashmap|HashMap|
|list|List|
|arraylist|ArrayList|
|collection|Collection|
|iterator|Iterator|

### demo
在XML映射配置文件中使用别名：
```xml
<!--方法签名为：int getCount();-->
<!-- Java的int类型在xml映射配置文件中，写成_int或者_integer -->
<select id="getCount" resultType="_integer">
    select count(id) from t_user
</select>
```

## typeHandlers（类型处理器）
MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时，都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。

|类型处理器|Java 类型|JDBC 类型|
|---|---|---|
|BooleanTypeHandler|java.lang.Boolean, boolean|数据库兼容的 BOOLEAN|
|ByteTypeHandler|java.lang.Byte, byte|数据库兼容的 NUMERIC 或 BYTE|
|ShortTypeHandler|java.lang.Short, short|数据库兼容的 NUMERIC 或 SMALLINT|
|IntegerTypeHandler|java.lang.Integer, int|数据库兼容的 NUMERIC 或 INTEGER|
|LongTypeHandler|java.lang.Long, long|数据库兼容的 NUMERIC 或 BIGINT|
|FloatTypeHandler|java.lang.Float, float|数据库兼容的 NUMERIC 或 FLOAT|
|DoubleTypeHandler|java.lang.Double, double|数据库兼容的 NUMERIC 或 DOUBLE|
|BigDecimalTypeHandler|java.math.BigDecimal|数据库兼容的 NUMERIC 或 DECIMAL|
|StringTypeHandler|java.lang.String|CHAR, VARCHAR|
|ClobReaderTypeHandler|java.io.Reader|-|
|ClobTypeHandler|java.lang.String|CLOB, LONGVARCHAR|
|NStringTypeHandler|java.lang.String|NVARCHAR, NCHAR|
|NClobTypeHandler|java.lang.String|NCLOB|
|BlobInputStreamTypeHandler|java.io.InputStream|-|
|ByteArrayTypeHandler|byte[]|数据库兼容的字节流类型|
|BlobTypeHandler|byte[]|BLOB, LONGVARBINARY|
|DateTypeHandler|java.util.Date|TIMESTAMP|
|DateOnlyTypeHandler|java.util.Date|DATE|
|TimeOnlyTypeHandler|java.util.Date|TIME|
|SqlTimestampTypeHandler|java.sql.Timestamp|TIMESTAMP|
|SqlDateTypeHandler|java.sql.Date|DATE|
|SqlTimeTypeHandler|java.sql.Time|TIME|
|ObjectTypeHandler|Any|OTHER 或未指定类型|
|EnumTypeHandler|Enumeration Type|VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值）|
|EnumOrdinalTypeHandler|Enumeration Type|任何兼容的 NUMERIC 或 DOUBLE 类型，用来存储枚举的序数值（而不是名称）|
|SqlxmlTypeHandler|java.lang.String|SQLXML|
|InstantTypeHandler|java.time.Instant|TIMESTAMP|
|LocalDateTimeTypeHandler|java.time.LocalDateTime|TIMESTAMP|
|LocalDateTypeHandler|java.time.LocalDate|DATE|
|LocalTimeTypeHandler|java.time.LocalTime|TIME|
|OffsetDateTimeTypeHandler|java.time.OffsetDateTime|TIMESTAMP|
|OffsetTimeTypeHandler|java.time.OffsetTime|TIME|
|ZonedDateTimeTypeHandler|java.time.ZonedDateTime|TIMESTAMP|
|YearTypeHandler|java.time.Year|INTEGER|
|MonthTypeHandler|java.time.Month|INTEGER|
|YearMonthTypeHandler|java.time.YearMonth|VARCHAR 或 LONGVARCHAR|
|JapaneseDateTypeHandler|java.time.chrono.JapaneseDate|DATE|


你可以重写已有的类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。 具体做法为：实现 org.apache.ibatis.type.TypeHandler 接口， 或继承一个很便利的类 org.apache.ibatis.type.BaseTypeHandler， 并且可以（可选地）将它映射到一个 JDBC 类型。

```java
// ExampleTypeHandler.java
// 该类型处理器将会覆盖已有的处理 Java String 类型的属性以及 VARCHAR 类型的参数和结果的类型处理器
@MappedJdbcTypes(JdbcType.VARCHAR) // JDBC类型
public class ExampleTypeHandler extends BaseTypeHandler<String> { // Java类型

  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    return rs.getString(columnName);
  }

  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    return rs.getString(columnIndex);
  }

  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    return cs.getString(columnIndex);
  }
}
```

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
</typeHandlers>
```

MyBatis 不会通过检测数据库元信息来决定使用哪个类型处理器，必须在参数和结果映射中指明字段的类型，以使其能够绑定到正确的类型处理器上，这是因为 MyBatis 直到语句被执行时才清楚数据类型。

当在 `ResultMap` 中决定使用哪种类型处理器时，此时 Java 类型是已知的（从结果类型中获得），但是 JDBC 类型是未知的。因此 Mybatis 使用 `javaType=[Java 类型], jdbcType=null` 的组合来选择一个类型处理器。这意味着使用 @MappedJdbcTypes 注解可以限制类型处理器的作用范围，并且可以确保，除非显式地设置，否则类型处理器在 ResultMap 中将不会生效。 如果希望能在 ResultMap 中隐式地使用类型处理器，那么设置 @MappedJdbcTypes 注解的 includeNullJdbcType=true 即可。

## objectFactory（对象工厂）
每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。 如果想覆盖对象工厂的默认行为，可以通过创建自己的对象工厂来实现。

```java
// ExampleObjectFactory.java
public class ExampleObjectFactory extends DefaultObjectFactory {
  @Override
  public <T> T create(Class<T> type) {
    return super.create(type);
  }

  @Override
  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
    return super.create(type, constructorArgTypes, constructorArgs);
  }

  @Override
  public void setProperties(Properties properties) {
    super.setProperties(properties);
  }

  @Override
  public <T> boolean isCollection(Class<T> type) {
    return Collection.class.isAssignableFrom(type);
  }
}
```
```xml
<!-- mybatis-config.xml -->
<objectFactory type="org.mybatis.example.ExampleObjectFactory">
  <property name="someProperty" value="100"/>
</objectFactory>
```

ObjectFactory 接口包含两个创建实例用的方法，一个是处理默认无参构造方法的，另外一个是处理带参数的构造方法的。 另外，setProperties 方法可以被用来配置 ObjectFactory，在初始化你的 ObjectFactory 实例后， objectFactory 元素体中定义的属性会被传递给 setProperties 方法。

## plugins（插件）
MyBatis 允许你在映射语句执行过程中的某一点进行拦截调用。MyBatis 允许使用插件来拦截的方法调用包括：
* Executor(update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
* ParameterHandler(getParameterObject, setParameters)
* ResultSetHandler(handleResultSets, handleOutputParameters)
* StatementHandler(prepare, parameterize, batch, update, query)

这些类中方法的细节可以通过查看每个方法的签名来发现，或者直接查看 MyBatis 发行包中的源代码。

使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可:
```java
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  private Properties properties = new Properties();

  @Override
  public Object intercept(Invocation invocation) throws Throwable {
    // implement pre processing if need
    Object returnObject = invocation.proceed();
    // implement post processing if need
    return returnObject;
  }

  @Override
  public void setProperties(Properties properties) {
    this.properties = properties;
  }
}
```

## environments（环境配置）//
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

如果使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

### dataSource（数据源）
dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

有三种内建的数据源类型（也就是 type="\[UNPOOLED|POOLED|JNDI]"）：
* UNPOOLED：这个数据源的实现会每次请求时打开和关闭连接。
  * driver – 这是 JDBC 驱动的 Java 类全限定名。
    * 作为可选项，可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：driver.encoding=UTF8，这将通过 `DriverManager.getConnection(url, driverProperties)` 方法传递值为 UTF8 的 encoding 属性给数据库驱动。
  * url – 这是数据库的 JDBC URL 地址。
  * username – 登录数据库的用户名。
  * password – 登录数据库的密码。
  * defaultTransactionIsolationLevel – 默认的连接事务隔离级别。
  * defaultNetworkTimeout – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。
* POOLED：这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：
  * poolMaximumActiveConnections – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
  * poolMaximumIdleConnections – 任意时间可能存在的空闲连接数。
  * poolMaximumCheckoutTime – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
  * poolTimeToWait – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
  * poolMaximumLocalBadConnectionTolerance – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance 之和。 默认值：3
  * poolPingQuery – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
  * poolPingEnabled – 是否启用侦测查询。若开启，需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
  * poolPingConnectionsNotUsedFor – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。
* JNDI：这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：
  * initial_context – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
  * data_source – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

可以通过实现接口 org.apache.ibatis.datasource.DataSourceFactory 来使用第三方数据源实现：
```java
public interface DataSourceFactory {
  void setProperties(Properties props);
  DataSource getDataSource();
}
```

## databaseIdProvider（数据库厂商标识）
MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。 MyBatis 会加载带有匹配当前数据库 databaseId 属性和所有不带 databaseId 属性的语句。如果同时找到带有 databaseId 和不带 databaseId 的相同语句，则后者会被舍弃。 


## mappers（映射器）
定义 SQL 映射语句后，需要告诉 MyBatis 到哪里去找到这些语句。可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 file:/// 形式的 URL），或类名和包名等配置会告诉 MyBatis 去哪里找映射文件。

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
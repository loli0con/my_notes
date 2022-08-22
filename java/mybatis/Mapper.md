# 映射器
映射是mybatis最强大的功能，映射器主要是对这一过程的描述。

![Mapper+mybatis-结果映射](https://i.loli.net/2021/08/18/YhywX8IgRDnoxCA.png)

## 基本概念
### 获得一个映射器
通过SqlSession.getMapper(xxxMapper.class)方法，可以获得一个映射器对象mapper。  
其中：
* xxxMapper通常为程序员编写的接口，该接口定义了一个映射器；  
  在其中可以定义许多抽象方法，用以描述其具备的映射功能
* `mapper instanceof xxxMapper`的结果为`true`

### 映射器的具体实现

由于xxxMapper只是一个接口，mybatis还需要知道这个接口中的抽象方法是如何实现的（即如何完成映射的过程）。

可以通过两种方式去告知mybatis关于接口的实现：
* Mapper XML 配置文件（功能强大）
* 在抽象方法上进行注解（使用简单）


## 配置文件
SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：
* cache – 该命名空间的缓存配置。
* cache-ref – 引用其它命名空间的缓存配置。
* resultMap – 描述如何从数据库结果集中加载对象，是最复杂也是最强大的元素。
* sql – 可被其它语句引用的可重用语句块。
* insert – 映射插入语句。
* update – 映射更新语句。
* delete – 映射删除语句。
* select – 映射查询语句。


## select
select元素用于配置查询和结果映射，select元素可以配置很多属性来控制每条语句的行为细节，它有属性如下表所示：
|属性|描述|
|---|---|
|id|在命名空间中唯一的标识符，可以被用来引用这条语句。|
|parameterType|将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。|
|resultType|期望从这条语句中返回结果的类全限定名或别名。注意，如果返回的是集合，那应该设置为集合包含的类型，而不是集合本身的类型。resultType 和 resultMap 之间只能同时使用一个。|
|resultMap|对外部 resultMap 的命名引用。resultType 和 resultMap 之间只能同时使用一个。|
|flushCache|将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：false。|
|useCache|将其设置为 true 后，将会导致本条语句的结果被二级缓存缓存起来，默认值：对 select 元素为 true。|
|timeout|这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。|
|fetchSize|这是一个给驱动的建议值，尝试让驱动程序每次批量返回的结果行数等于这个设置值。 默认值为未设置（unset）（依赖驱动）。|
|statementType|可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。|
|resultSetType|FORWARD_ONLY，SCROLL_SENSITIVE, SCROLL_INSENSITIVE 或 DEFAULT（等价于 unset） 中的一个，默认值为 unset （依赖数据库驱动）。|
|databaseId|如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。|
|resultOrdered|这个设置仅针对嵌套结果 select 语句：如果为 true，将会假设包含了嵌套结果集或是分组，当返回一个主结果行时，就不会产生对前面结果集的引用。 这就使得在获取嵌套结果集的时候不至于内存不够用。默认值：false。|
|resultSets|这个设置仅适用于多结果集的情况。它将列出语句执行后返回的结果集并赋予每个结果集一个名称，多个名称之间以逗号分隔。|


## insert, update 和 delete 元素
数据变更语句 insert，update 和 delete 的实现非常接近，它们一些有共同的属性：
|属性|描述|
|id|在命名空间中唯一的标识符，可以被用来引用这条语句。|
|parameterType|将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。|
|flushCache|将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。|
|timeout|这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。|
|statementType|可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。|
|useGeneratedKeys|（仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。|
|keyProperty|（仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（unset）。如果生成列不止一个，可以用逗号分隔多个属性名称。|
|keyColumn|（仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。|
|databaseId|如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。|

### 主键的生成
如果数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置为（目标对象的）目标属性就 OK 了。

如果数据库支持多行插入, 可以传入一个数组或集合，并返回自动生成的主键。

```xml
<!-- 实体Author的属性id对应数据库中的主键 -->
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>

<insert id="insertAuthors" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

对于不支持自动生成主键列的数据库和可能不支持自动生成主键的 JDBC 驱动，MyBatis 有另外一种方法来生成主键——使用selectKey 元素，selectKey 元素的属性如下：
|属性|描述|
|keyProperty|selectKey 语句结果应该被设置到的目标属性。如果生成列不止一个，可以用逗号分隔多个属性名称。|
|keyColumn|返回结果集中生成列属性的列名。如果生成列不止一个，可以用逗号分隔多个属性名称。|
|resultType|结果的类型。通常 MyBatis 可以推断出来，但是为了更加准确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果生成列不止一个，则可以使用包含期望属性的 Object 或 Map。|
|order|可以设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它首先会生成主键，设置 keyProperty 再执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句 - 这和 Oracle 数据库的行为相似，在插入语句内部可能有嵌入索引调用。|
|statementType|和前面一样，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 类型的映射语句，分别代表 Statement, PreparedStatement 和 CallableStatement 类型。|

```xml
<!-- 首先会运行 selectKey 元素中的语句，并设置 Author 的 id，然后才会调用插入语句。 -->
<insert id="insertAuthor">
  <selectKey keyProperty="id" resultType="int" order="BEFORE">
    select CAST(RANDOM()*1000000 as INTEGER) a from SYSIBM.SYSDUMMY1
  </selectKey>
  insert into Author
    (id, username, password, email,bio, favourite_section)
  values
    (#{id}, #{username}, #{password}, #{email}, #{bio}, #{favouriteSection,jdbcType=VARCHAR})
</insert>
```

## sql 元素
这个元素可以用来定义可重用的 SQL 代码片段，以便在其它语句中使用。

参数可以静态地（在加载的时候）确定下来，并且可以在不同的 include 元素中定义不同的参数值：
```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password </sql>
<select id="selectUsers" resultType="map">
  select
    <!-- 在不同的include元素中定义不同的参数值 -->
    <include refid="userColumns"><property name="alias" value="t1"/></include>,
    <include refid="userColumns"><property name="alias" value="t2"/></include>
  from some_table t1
    cross join some_table t2
</select>
```

可以在 include 元素的 refid 属性或内部语句中使用属性值：
```xml
<sql id="sometable">
  ${prefix}Table
</sql>

<sql id="someinclude">
  from
    <include refid="${include_target}"/>
</sql>

<select id="select" resultType="map">
  select
    field1, field2, field3
  <include refid="someinclude">
    <property name="prefix" value="Some"/>
    <property name="include_target" value="sometable"/>
  </include>
</select>
```


## 参数
参数是 MyBatis 非常强大的元素，参数在映射文件中表示为：`#{参数定义}`

### demo
一个简单的例子：
```xml
<!-- 传入原始类型或简单数据类型的情况 -->
<select id="selectUsers" resultType="User">
  select id, username, password
  from users
  where id = #{id}
</select>

<!-- 传入复杂的对象 -->
<insert id="insertUser" parameterType="User">
  insert into users (id, username, password)
  values (#{id}, #{username}, #{password})
</insert>
```
解析：
* 对于原始类型和简单数据类型（比如Integer和String）：它们可以随意命名，且会自动设置类型（例子中为int类型），因为没有其他属性，会用它们的值作为参数。
* 对于复杂的对象（例子中的User类型的参数对象）：Mybatis会查找相应的属性（例子中的 id、username 和 password 属性），然后将它们的值传入预处理语句的参数中。

### type
参数可以指定一个特殊的数据类型：
```xml
<!-- 假设参数对象为{name:xiaoming,age:21,height:1.72} -->
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
<!-- 对于数值类型，可以设置 numericScale 指定小数点后保留的位数。 -->
```

#### javaType
除非该对象是一个HashMap，Mybatis基本可以根据参数对象的类型确定javaType。该对象是一个HashMap，需要显式指定 javaType 来确保正确的类型处理器（TypeHandler）被使用。

> 在运行期间，HashMap的key类型无法确定。

#### jdbcType
JDBC 要求，如果一个列允许使用 null 值，并且会使用值为 null 的参数，就必须要指定 JDBC 类型（jdbcType）。

#### 自定义类型处理方式
要更进一步地自定义类型处理方式，可以指定一个特殊的类型处理器类（或别名），比如：
```xml
<!-- 假设参数对象为{name:xiaoming,age:21,height:1.72} -->
#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}
<!-- 例如在MyTypeHandler里面实现对属性age进行四舍五入的特殊处理逻辑 -->
```

#### mode
mode 属性允许你指定 IN，OUT 或 INOUT 参数。如果参数的 mode 为 OUT 或 INOUT，将会修改参数对象的属性值，以便作为输出参数返回。当使用 OUT 或 INOUT 参数时，你必须显式设置类型的名称。

如果 mode 为 OUT（或 INOUT），而且 jdbcType 为 CURSOR（也就是 Oracle 的 REFCURSOR），你必须指定一个 resultMap 引用来将结果集 ResultSet 映射到参数的类型上。要注意这里的 javaType 属性是可选的，如果留空并且 jdbcType 是 CURSOR，它会被自动地被设为 ResultSet。
```xml
#{department, mode=OUT, jdbcType=CURSOR, javaType=ResultSet, resultMap=departmentResultMap}

<!-- MyBatis 也支持很多高级的数据类型，比如结构体（structs） -->
#{middleInitial, mode=OUT, jdbcType=STRUCT, jdbcTypeName=MY_TYPE, resultMap=departmentResultMap}
```


### 字符串替换
默认情况下，使用 `#{}` 参数语法时，MyBatis 会创建 PreparedStatement 参数占位符，并通过占位符安全地设置参数（就像使用 ? 一样）。 

有时就是想直接在 SQL 语句中直接插入一个不转义的字符串，使用 `${}` 参数语法，MyBatis 就不会修改或转义该字符串了。

```java
// 其中 ${column} 会被直接替换，而 #{value} 会使用 ? 预处理。
@Select("select * from user where ${column} = #{value}")
User findByColumn(@Param("column") String column, @Param("value") String value);
```



## 结果映射
resultMap元素具备从JDBC ResultSets（结果集）提取数据的功能，即resultMap元素控制了**结果集**到**Java对象**的映射过程。

### resultType
使用`resultType`属性指定一个JavaBean：Mybatis会在幕后自动创建一个 `ResultMap`，再根据属性名来映射列到JavaBean的属性上，这样可以无需显式地配置`ResultMap`。

### resultMap

#### 属性
ResultMap的属性列表
|属性|描述|
|id|当前命名空间中的一个唯一标识，用于标识一个结果映射。|
|type|类的完全限定名, 或者一个类型别名（关于内置的类型别名，可以参考上面的表格）。|
|autoMapping|如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。默认值：未设置（unset）。|

#### 子元素
resultMap 元素有很多子元素和一个值得深入探讨的结构：
* constructor - 用于在实例化类时，注入结果到构造方法中
  * idArg - ID 参数；标记出作为 ID 的结果可以帮助提高整体性
  * arg - 将被注入到构造方法的一个普通结果
* id – 一个 ID 结果；标记出作为 ID 的结果可以帮助提高整体性能
* result – 注入到字段或 JavaBean 属性的普通结果
* association – 一个复杂类型的关联；许多结果将包装成这种类型
  * 嵌套结果映射 – 关联可以是 resultMap 元素，或是对其它结果映射的引用
* collection – 一个复杂类型的集合
  * 嵌套结果映射 – 集合可以是 resultMap 元素，或是对其它结果映射的引用
* discriminator – 使用结果值来决定使用哪个 resultMap
  * case – 基于某些值的结果映射
    * 嵌套结果映射 – case 也是一个结果映射，因此具有相同的结构和元素；或者引用其它的结果映射

##### id & result
id 和 result 元素通过修改对象属性(property)/修改字段(field)的方式，将一个列的值映射到一个简单数据类型（String, int, double, Date等）的属性或字段。

这两者之间的唯一不同是，id 元素对应的属性会被标记为对象的标识符，在比较对象实例时使用。这样可以提高整体的性能，尤其是进行缓存和嵌套结果映射（也就是连接映射）的时候。

两个元素都有一些属性：
|属性|描述|
|property|映射到列结果的字段或属性。如果 JavaBean 有这个名字的属性（property），会先使用该属性。否则 MyBatis 将会寻找给定名称的字段（field）。 无论是哪一种情形，你都可以使用常见的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。|
|column|数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。|
|javaType|一个 Java 类的全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。|
|jdbcType|JDBC 类型，所支持的 JDBC 类型参见这个表格之后的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可以为空值的列指定这个类型。|
|typeHandler|我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的全限定名，或者是类型别名。|

###### 支持的 JDBC 类型
MyBatis 通过内置的 jdbcType 枚举类型支持下面的 JDBC 类型：
1. TINYINT、SMALLINT、INTEGER、BIGINT
2. FLOAT、REAL、DOUBLE、NUMERIC、DECIMAL
3. CHAR、VARCHAR、LONGVARCHAR、NVARCHAR、NCHAR
4. TIMESTAMP、DATE、TIME
5. BIT、BINARY、VARBINARY、LONGVARBINARY、BLOB、NCLOB、CLOB
6. OTHER、UNDEFINED、BOOLEAN、NULL、CURSOR、ARRAY

##### constructor
constructor元素通过构造方法进行注入的方式来将结果集注入到对象中，在constructor元素的内部使用idArg元素和arg元素来对应对象的构造方法的形参列表。当在处理一个带有多个形参的构造方法时，可以通过如下方式确定相应的构造方法：
1. 根据构造方法形参的顺序，定义相应的idArg元素和arg元素来完成构造方法的定位。注意参数类型也需要一一匹配。
2. 可以使用 @Param 注解构造方法签名中的参数，然后通过constructor元素中的`name`属性来引用构造方法参数。使用这种方式，构造方法的形参顺序不必与 constructor 元素中参数声明的顺序匹配。如果存在名称和类型相同的属性，那么可以省略 javaType 。

###### idArg & arg
idArg & arg 的属性和规则如下。

|属性|描述|
|column|数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。|
|javaType|一个 Java 类的完全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。如果存在名称和类型相同的属性，那么可以省略 javaType 。|
|jdbcType|JDBC 类型，所支持的 JDBC 类型参见这个表格之前的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。|
|typeHandler|我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。|
|select|用于加载复杂类型属性的映射语句的 ID，它会从 column 属性中指定的列检索数据，作为参数传递给此 select 语句。具体请参考关联元素。|
|resultMap|结果映射的 ID，可以将嵌套的结果集映射到一个合适的对象树中。 它可以作为使用额外 select 语句的替代方案。它可以将多表连接操作的结果映射成一个单一的 ResultSet。这样的 ResultSet 将会将包含重复或部分数据重复的结果集。为了将结果集正确地映射到嵌套的对象树中，MyBatis 允许你 “串联”结果映射，以便解决嵌套结果集的问题。想了解更多内容，请参考下面的关联元素。|
|name|构造方法形参的名字。从 3.4.3 版本开始，通过指定具体的参数名，你可以以任意顺序写入 arg 元素。参看上面的解释。|

##### association
关联（association）元素处理“有一个”类型的关系。 你需要指定目标属性名以及属性的javaType（很多时候 MyBatis 可以自己推断出来），在必要的情况下你还可以设置 JDBC 类型，如果你想覆盖获取结果值的过程，还可以设置类型处理器。

MyBatis 有两种不同的方式加载关联：
* 嵌套 Select 查询：通过执行另外一个 SQL 映射语句来加载期望的复杂类型。
* 嵌套结果映射：使用嵌套的结果映射来处理连接结果的重复子集。

|属性|描述|
property|映射到列结果的字段或属性。如果用来匹配的 JavaBean 存在给定名字的属性，那么它将会被使用。否则 MyBatis 将会寻找给定名称的字段。 无论是哪一种情形，你都可以使用通常的点式分隔形式进行复杂属性导航。 比如，你可以这样映射一些简单的东西：“username”，或者映射到一些复杂的东西上：“address.street.number”。|
|javaType|一个 Java 类的完全限定名，或一个类型别名（关于内置的类型别名，可以参考上面的表格）。 如果你映射到一个 JavaBean，MyBatis 通常可以推断类型。然而，如果你映射到的是 HashMap，那么你应该明确地指定 javaType 来保证行为与期望的相一致。|
|jdbcType|JDBC 类型，所支持的 JDBC 类型参见这个表格之前的“支持的 JDBC 类型”。 只需要在可能执行插入、更新和删除的且允许空值的列上指定 JDBC 类型。这是 JDBC 的要求而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可能存在空值的列指定这个类型。|
|typeHandler|我们在前面讨论过默认的类型处理器。使用这个属性，你可以覆盖默认的类型处理器。 这个属性值是一个类型处理器实现类的完全限定名，或者是类型别名。|

###### 关联的嵌套 Select 查询

|属性|描述|
|---|---|
|column|数据库中的列名，或者是列的别名。一般情况下，这和传递给 resultSet.getString(columnName) 方法的参数一样。 注意：在使用复合主键的时候，你可以使用 column="{prop1=col1,prop2=col2}" 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 prop1 和 prop2 作为参数对象，被设置为对应嵌套 Select 语句的参数。|
|select|用于加载复杂类型属性的映射语句的 ID，它会从 column 属性指定的列中检索数据，作为参数传递给目标 select 语句。 具体请参考下面的例子。注意：在使用复合主键的时候，你可以使用 column="{prop1=col1,prop2=col2}" 这样的语法来指定多个传递给嵌套 Select 查询语句的列名。这会使得 prop1 和 prop2 作为参数对象，被设置为对应嵌套 Select 语句的参数。|
|fetchType|可选的。有效值为 lazy 和 eager。 指定属性后，将在映射中忽略全局配置参数 lazyLoadingEnabled，使用属性的值。|


###### 关联的嵌套结果映射

|属性|描述|
|---|---|
|resultMap|结果映射的 ID，可以将此关联的嵌套结果集映射到一个合适的对象树中。 它可以作为使用额外 select 语句的替代方案。它可以将多表连接操作的结果映射成一个单一的 ResultSet。这样的 ResultSet 有部分数据是重复的。 为了将结果集正确地映射到嵌套的对象树中, MyBatis 允许你“串联”结果映射，以便解决嵌套结果集的问题。使用嵌套结果映射的一个例子在表格以后。|
|columnPrefix|当连接多个表时，你可能会不得不使用列别名来避免在 ResultSet 中产生重复的列名。指定 columnPrefix 列名前缀允许你将带有这些前缀的列映射到一个外部的结果映射中。 详细说明请参考后面的例子。|
|notNullColumn|默认情况下，在至少一个被映射到属性的列不为空时，子对象才会被创建。 你可以在这个属性上指定非空的列来改变默认行为，指定后，Mybatis 将只在这些列中任意一列非空时才创建一个子对象。可以使用逗号分隔来指定多个列。默认值：未设置（unset）。|
|autoMapping|如果设置这个属性，MyBatis 将会为本结果映射开启或者关闭自动映射。 这个属性会覆盖全局的属性 autoMappingBehavior。注意，本属性对外部的结果映射无效，所以不能搭配 select 或 resultMap 元素使用。默认值：未设置（unset）。|


###### 关联的多结果集
|属性|描述|
|---|---|
|column|当使用多个结果集时，该属性指定结果集中用于与 foreignColumn 匹配的列（多个列名以逗号隔开），以识别关系中的父类型与子类型。|
|foreignColumn|指定外键对应的列名，指定的列将与父类型中 column 的给出的列进行匹配。|
|resultSet|指定用于加载复杂类型的结果集名字。在映射语句中，必须通过 resultSets 属性为每个结果集指定一个名字，多个名字使用逗号隔开。|

某些数据库允许存储过程返回多个结果集，或一次性执行多个语句，每个语句返回一个结果集。可以利用这个特性，在不使用连接的情况下，只访问数据库一次就能获得相关数据。


##### 集合
集合元素和关联元素几乎是一样的，**关联**用于处理“有一个”的类型，**集合**用于处理“有很多个”的类型。

要把嵌套结果集合映射到一个 List 中，可以使用集合元素：可以使用嵌套 Select 查询，或基于连接的嵌套结果映射集合。

###### 集合的嵌套 Select 查询
“ofType” 属性：它用来将 JavaBean（或字段）属性的类型和集合存储的类型区分开来。在一般情况下，MyBatis 可以推断 javaType 属性，因此并不需要填写。

###### 集合的嵌套结果映射
除了新增的 “ofType” 属性，它和关联的完全相同。

###### 集合的多结果集
在映射语句中，必须通过 resultSets 属性为每个结果集指定一个名字，多个名字使用逗号隔开。


### 鉴别器
鉴别器（discriminator）元素被设计来应对“一个数据库查询可能会返回多个不同的结果集”的情况，也能处理“类的继承层次结构”的情况。

鉴别器的概念很好理解——它很像 Java 语言中的 switch 语句。如果结果值匹配任意一个鉴别器的 case，就会使用这个 case 指定的结果映射。这个过程是互斥的，也就是说，剩余的结果映射将被忽略（除非它是扩展的，通过使用extends属性实现扩展）。如果不能匹配任何一个 case，MyBatis 就只会使用鉴别器块外定义的结果映射。 

一个鉴别器的定义需要指定 column 和 javaType 属性。column 指定了 MyBatis 查询被比较值的地方。 而 javaType 用来确保使用正确的相等测试（虽然很多情况下字符串的相等测试都可以工作）。

## 自动映射
在简单的场景下，MyBatis 可以为你自动映射查询结果。但如果遇到复杂的场景，你需要构建一个结果映射。  
你可以混合使用这两种策略，对于每一个结果映射，在 ResultSet 出现的列，如果没有设置手动映射，将被自动映射。在自动映射处理完毕后，再处理手动映射。

### 命名转换
当自动映射查询结果时，MyBatis 会获取结果中返回的列名并在 Java 类中查找相同名字的属性（忽略大小写）。   
通常数据库列使用大写字母组成的单词命名，单词间用下划线分隔；而 Java 属性一般遵循驼峰命名法约定。为了在这两种命名方式之间启用自动映射，需要将 mapUnderscoreToCamelCase 设置为 true。

### 映射等级
有三种自动映射等级：
* NONE - 禁用自动映射。仅对手动映射的属性进行映射。
* PARTIAL - 对除在内部定义了嵌套结果映射（也就是连接的属性）以外的属性进行映射
* FULL - 自动映射所有属性。

自动映射等级是哪种，你都可以通过在结果映射上设置 autoMapping 属性来为指定的结果映射设置启用/禁用自动映射。
# JDBC

## 基本概念
### 定义
JDBC的全称是Java Database Connectivity，即Java数据库连接，它是一种可以执行SQL语句的Java API。  

### 特点
JDBC为数据库开发提供了标准的API，使用JDBC开发的数据库应用可以跨平台运行，而且可以跨数据库。
![JDBC+20210628210009](https://raw.githubusercontent.com/loli0con/picgo/master/images/JDBC%2B20210628210009.png%2B2021-06-28-21-00-10)

### 用途
程序可通过JDBC API连接到关系数据库，并使用结构化查询语言（SQL，数据库标准的查询语言）来完成对数据库的查询、更新。

### 驱动程序
数据库驱动程序是JDBC程序和数据库之间的转换层，数据库驱动程序负责将JDBC调用映射成特定的数据库调用。

![JDBC+20210628210240](https://raw.githubusercontent.com/loli0con/picgo/master/images/JDBC%2B20210628210240.png%2B2021-06-28-21-02-41)

当需要连接某个特定的数据库时，必须有相应的数据库驱动程序。

## 常用接口和类
### DriverManager
用于管理JDBC驱动的服务类。  
程序中使用该类的主要功能是获取Connection对象，该类包含如下方法：
* public static synchronized Connection getConnection(String url, String user, String pass) throws SQLException：该方法获得url对应数据库的连接。

### Connection
代表数据库连接对象，每个Connection代表一个物理连接会话。要想访问数据库，必须先获得数据库连接。该接口的常用方法如下：
* Statement createStatement() throws SQLExcetpion：该方法返回一个Statement对象。
* PreparedStatement prepareStatement(String sql) throws SQLExcetpion：该方法返回预编译的Statement对象，即将SQL语句提交到数据库进行预编译。
* CallableStatement prepareCall(String sql) throws SQLExcetpion：该方法返回CallableStatement对象，该对象用于调用存储过程。

上面三个方法都返回用于执行SQL语句的Statement对象，PreparedStatement、CallableStatement是Statement的子类，只有获得了Statement之后才可执行SQL语句。

### Statement
用于执行SQL语句的工具接口。该对象既可用于执行DDL、DCL语句，也可用于执行DML语句，还可用于执行SQL查询。当执行SQL查询时，返回查询到的结果集。它的常用方法如下：
* ResultSet executeQuery(String sql) throws SQLException：该方法用于执行查询语句，并返回查询结果对应的ResultSet对象。该方法只能用于执行查询语句。
* int executeUpdate(String sql) throws SQLExcetion：该方法用于执行DML语句，并返回受影响的行数；该方法也可用于执行DDL语句，执行DDL语句将返回0。
* boolean execute(String sql) throws SQLException：该方法可执行任何SQL语句。如果执行后第一个结果为ResultSet对象，则返回true；如果执行后第一个结果为受影响的行数或没有任何结果，则返回false。

### ResultSet
结果集对象。该对象包含访问查询结果的方法，ResultSet可以通过列索引或列名获得列数据。它包含了如下常用方法来移动记录指针：
* void close()：释放ResultSet对象。
* boolean absolute（int row）：将结果集的记录指针移动到第row行，如果row是负数，则移动到倒数第row行。如果移动后的记录指针指向一条有效记录，则该方法返回true。
* void beforeFirst()：将ResultSet的记录指针定位到首行之前，这是ResultSet结果集记录指针的初始状态—记录指针的起始位置位于第一行之前。
* boolean first()：将ResultSet的记录指针定位到首行。如果移动后的记录指针指向一条有效记录，则该方法返回true。
* boolean previous()：将ResultSet的记录指针定位到上一行。如果移动后的记录指针指向一条有效记录，则该方法返回true。
* boolean next()：将ResultSet的记录指针定位到下一行，如果移动后的记录指针指向一条有效记录，则该方法返回true。
* boolean last()：将ResultSet的记录指针定位到最后一行，如果移动后的记录指针指向一条有效记录，则该方法返回true。
* void afterLast()：将ResultSet的记录指针定位到最后一行之后。

当把记录指针移动到指定行之后，ResultSet可通过getXxx(int columnIndex)或getXxx(String columnLabel)方法来获取当前行、指定列的值，前者根据列索引获取值，后者根据列名获取值。

## 编程步骤
JDBC编程大致按如下步骤进行：
1. 加载数据库驱动
2. 通过DriverManager获取数据库连接
3. 通过Connection对象创建Statement对象
4. 使用Statement执行SQL语句
5. 操作结果集
6. 回收数据库资源

## 执行SQL语句
### Statement
* 使用executeQuery()来执行DQL语句
* 使用executeUpdate()来执行DDL和DML语句
* 使用execute()来执行SQL语句，Statement提供了如下两个方法来获取执行结果：
  * getResultSet()：获取该Statement执行查询语句所返回的ResultSet对象。
  * getUpdateCount()：获取该Statement()执行DML语句所影响的记录行数。

### PreparedStatement
创建PreparedStatement对象使用Connection的prepareStatement()方法，该方法需要传入一个SQL字符串，该SQL字符串可以包含占位符参数`?`。

PreparedStatement也提供了execute()、executeUpdate()、executeQuery()三个方法来执行SQL语句，不过这三个方法**无须参数**，因为PreparedStatement已存储了预编译的SQL语句。

使用PreparedStatement预编译SQL语句时，该SQL语句可以带占位符参数，因此**在执行SQL语句之前必须为这些参数传入参数值**，PreparedStatement提供了一系列的setXxx(int index, Xxx value)方法来传入参数值。

#### 优点
总体来看，使用PreparedStatement比使用Statement多了如下三个好处：
* PreparedStatement预编译SQL语句，**性能**更好。
* PreparedStatement无须“拼接”SQL语句，编程更**简单**。
* PreparedStatement可以防止SQL注入，**安全**性更好。

### CallableStatement
调用存储过程使用CallableStatement，可以通过Connection的prepareCall()方法来创建CallableStatement对象，创建该对象时需要传入调用存储过程的SQL语句。调用存储过程的SQL语句总是这种格式：{call过程名(?, ?, ?...)}，其中的问号`?`作为存储过程参数的占位符。

存储过程的参数既有传入参数，也有传出参数。所谓传入参数就是Java程序必须为这些参数传入值，可以通过CallableStatement的setXxx()方法为传入参数设置值；所谓传出参数就是Java程序可以通过该参数获取存储过程里的值，CallableStatement需要调用registerOutParameter()方法来注册该参数。

调用CallableStatement的execute()方法来执行存储过程，执行结束后通过CallableStatement对象的getXxx（int index）方法来获取指定传出参数的值。

#### 示例代码
```Java
// 假定存在一个存储过程 my_add(in a int, in b int, out sum int)
// 使用Connection来创建一个CallableStatement对象
cstmt = conn.prepareCall("{call my_add(?, ?, ?)}");

// 传入参数
cstmt.setInt(1, 4);
cstmt.setInt(2, 5);

// 注册CallableStatement的第三个参数是int类型
cstmt.registerOutParameter(3, Types.INTEGER);

// 执行存储过程
cstmt.execute();

// 获取存储过程传出参数的值
int sum = cstmt.getInt(3);
```

## 管理结果集
JDBC使用ResultSet来封装执行查询得到的查询结果，然后通过移动ResultSet的记录指针来取出结果集的内容。除此之外，JDBC还允许通过ResultSet来更新记录，并提供了ResultSetMetaData来获得ResultSet对象的相关信息。

### 创建结果集
Connection在创建Statement或PreparedStatement时还可额外传入如下两个参数：
* resultSetType：控制ResultSet的类型，该参数可以取如下三个值。
  * ResultSet.TYPE_FORWARD_ONLY：该常量控制记录指针只能向前移动。
  * ResultSet.TYPE_SCROLL_INSENSITIVE：该常量控制记录指针可以自由移动（可滚动结果集），但底层数据的改变不会影响ResultSet的内容。
  * ResultSet.[TYPE_SCROLL_SENSITIVE](https://blog.csdn.net/l912943297/article/details/52292211)：该常量控制记录指针可以自由移动（可滚动结果集），而且底层数据的改变会影响ResultSet的内容。
* resultSetConcurrency：控制ResultSet的并发类型，该参数可以接收如下两个值。
  * ResultSet.CONCUR_READ_ONLY：该常量指示ResultSet是只读的并发模式（默认）。
  * ResultSet.CONCUR_UPDATABLE：该常量指示ResultSet是可更新的并发模式。

#### 更新
可更新的结果集需要满足如下两个条件：
* 所有数据都应该来自一个表。
* 选出的数据集必须包含主键列。

程序可调用ResultSet的updateXxx(int columnIndex，Xxx value)方法来修改记录指针所指记录、特定列的值，最后调用ResultSet的updateRow()方法来提交修改。

### 处理Blob类型数据
Blob（Binary Long Object）是二进制长对象的意思，Blob列通常用于存储大文件，典型的Blob内容是一张图片或一个声音文件，由于它们的特殊性，必须使用特殊的方式来存储。

#### 存
将Blob数据插入数据库需要使用PreparedStatement，该对象有一个方法：setBinaryStream(int parameterIndex，InputStream x)，该方法可以为指定参数传入二进制输入流，从而可以实现将Blob数据保存到数据库的功能。

#### 取
当需要从ResultSet里取出Blob数据时，可以调用ResultSet的getBlob(int columnIndex)方法，该方法将返回一个Blob对象，Blob对象提供了getBinaryStream()方法来获取该Blob数据的输入流，也可以使用Blob对象提供的getBytes()方法直接取出该Blob对象封装的二进制数据。

### ResultSetMetaData
通过ResultSetMetaData可以获取关于ResultSet的描述信息。

ResultSet里包含一个getMetaData()方法，该方法返回该ResultSet对应的ResultSetMetaData对象。一旦获得了ResultSetMetaData对象，就可通过ResultSetMetaData提供的大量方法来返回ResultSet的描述信息。  
常用的方法有如下三个：
* int getColumnCount()：返回该ResultSet的列数量。
* String getColumnName(int column)：返回指定索引的列名。
* int getColumnType(int column)：返回指定索引的列类型。

### RowSet包装结果集
RowSet接口继承了ResultSet接口，RowSet接口下包含JdbcRowSet、CachedRowSet、FilteredRowSet、JoinRowSet和WebRowSet常用子接口。

RowSet规范的接口类图：
![JDBC+20210629094838](https://raw.githubusercontent.com/loli0con/picgo/master/images/JDBC%2B20210629094838.png%2B2021-06-29-09-48-41)

与ResultSet相比，RowSet默认是可滚动、可更新、可序列化的结果集，而且作为JavaBean使用，因此能方便地在网络上传输，用于同步两端的数据。


#### 离线
对于离线RowSet而言，程序在创建RowSet时已把数据从底层数据库读取到了内存，因此可以充分利用计算机的内存，从而降低数据库服务器的负载，提高程序性能。  
离线的RowSet，无须保持与数据库的连接；在线的RowSet，必须保持与数据库的连接。

离线RowSet会直接将底层数据读入内存中，封装成RowSet对象，而RowSet对象则完全可以当成Java Bean来使用。
为了将程序对离线RowSet所做的修改同步到底层数据库，程序在调用RowSet的acceptChanges()方法时必须传入Connection。


#### RowSetFactory
##### 创建
RowSetProvider负责创建RowSetFactory，而RowSetFactory则提供了如下方法来创建RowSet实例：
* CachedRowSet createCachedRowSet()：创建一个默认的CachedRowSet。
* FilteredRowSet createFilteredRowSet()：创建一个默认的FilteredRowSet。
* JdbcRowSet createJdbcRowSet()：创建一个默认的JdbcRowSet。
* JoinRowSet createJoinRowSet()：创建一个默认的JoinRowSet。
* WebRowSet createWebRowSet()：创建一个默认的WebRowSet。

##### 使用
为了让RowSet能抓取到数据库的数据，需要为RowSet设置数据库的URL、用户名、密码等连接信息。因此，RowSet接口中定义了如下常用方法：
* setUrl(String url)：设置该RowSet要访问的数据库的URL。
* setUsername(String name)：设置该RowSet要访问的数据库的用户名。
* setPassword(String password)：设置该RowSet要访问的数据库的密码。
* setCommand(String sql)：设置使用该sql语句的查询结果来装填该RowSet。
* execute()：执行查询。

#### 分页
CachedRowSet提供了分页功能。所谓分页功能就是一次只装载ResultSet里的某几条记录，这样就可以避免CachedRowSet占用内存过大的问题。   
CachedRowSet提供了如下方法来控制分页：
* populate(ResultSet rs, int startRow)：使用给定的ResultSet装填RowSet，从ResultSet的第startRow条记录开始装填。
* setPageSize(int pageSize)：设置CachedRowSet每次返回多少条记录。
* previousPage()：在底层ResultSet可用的情况下，让CachedRowSet读取上一页记录。
* nextPage()：在底层ResultSet可用的情况下，让CachedRowSet读取下一页记录。


## 事务
JDBC连接也提供了事务支持，JDBC连接的事务支持由Connection提供，Connection默认打开自动提交。可以调用Connection的setAutoCommit()方法来关闭自动提交，开启事务。

一旦事务开始之后，程序可以像平常一样创建Statement对象，创建了Statement对象之后，可以执行任意多条DML语句。这些语句所做的修改不会生效，因为事务还没有结束。如果所有的语句都执行成功，程序可以调用Connection的commit()方法来提交事务。

如果任意一条SQL语句执行失败，则应该用Connection的rollback()方法来回滚事务。  
实际上，当Connection遇到一个未处理的SQLException异常时，系统将会非正常退出，事务也会自动回滚。但如果程序捕获了该异常，则需要在异常处理块中显式地回滚事务。

Connection提供了setSavepoint()方法设置中间点，提供了rollback(Savepoint savepoint)方法回滚到指定中间点。

### 批量更新
JDBC还提供了一个批量更新的功能，使用批量更新时，多条SQL语句将被作为一批操作被同时收集，并同时提交。

使用批量更新也需要先创建一个Statement对象，然后利用该对象的addBatch()方法将多条SQL语句同时收集起来，最后调用executeBatch()方法同时执行这些SQL语句。

执行executeBatch()方法将返回一个int\[]数组，因为使用Statement执行DDL、DML语句都将返回一个long值，而执行多条DDL、DML语句将会返回多个long值，多个long值就组成了这个int[]数组。如果在批量更新的addBatch()方法中添加了select查询语句，程序将直接出现错误。

为了让批量操作可以正确地处理错误，必须把批量执行的操作视为单个事务，如果批量更新在执行过程中失败，则让事务回滚到批量操作开始之前的状态。

## DatabaseMetaData
JDBC提供了DatabaseMetaData来封装数据库连接对应数据库的信息，通过Connection提供的getMetaData()方法就可以获取数据库对应的DatabaseMetaData对象。

许多DatabaseMetaData方法以ResultSet对象的形式返回查询信息，然后使用ResultSet的常规方法（如getString()和getInt()）即可从这些ResultSet对象中获取数据。如果查询的信息不可用，则将返回一个空ResultSet对象。  
DatabaseMetaData的很多方法都需要传入一个xxxPattern模式字符串，这里的xxxPattern不是正则表达式，而是SQL里的模式字符串，即用百分号（%）代表任意多个字符，使用下画线（_）代表一个字符。在通常情况下，如果把该模式字符串的参数值设置为null，即表明该参数不作为过滤条件。

## 连接池
数据库连接池是Connection对象的工厂，JDBC的数据库连接池使用javax.sql.DataSource来表示。DataSource通常被称为数据源，它包含连接池和连接池管理两个部分，但习惯上也经常把DataSource称为连接池。

数据库连接池的常用参数如下：
* 数据库的初始连接数。
* 连接池的最大连接数。
* 连接池的最小连接数。
* 连接池每次增加的容量。


### C3P0数据源
C3P0连接池可以自动清理不再使用的Connection，还可以自动清理Statement和ResultSet。

```Java
// 创建连接池实例
var ds = new ComboPooledDataSource();
// 设置连接池连接数据库的所需驱动
ds.setDriverClass("com.mysql.cj.jdbc.Driver");
// 设置连接数据库的URL
ds.setJdbcUrl("jdbc:mysql://localhost:3306/db?useSSL=false&serverTimezone=UTC");
// 设置连接数据库的用户名
ds.setUser("root");
// 设置连接数据库的密码
ds.setPassword("root");
// 设置连接池的最大连接数
ds.setMaxPoolSize(40);
// 设置连接池的最小连接数
ds.setMinPoolSize(2);
// 设置连接池的初始连接数
ds.setInitialPoolSize(10);
// 设置连接池的缓存Statement的最大数
ds.setMaxStatements(180);


// 通过数据源获取数据库连接
Connection conn = ds.getConnection();
// 执行业务
doSomething(conn);
// 释放数据库连接，归还给连接池
conn.close();
```
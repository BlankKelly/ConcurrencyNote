###DBCP
####简介
![commons dbcp](https://commons.apache.org/proper/commons-dbcp/images/dbcp-logo-white.png)
许多apache项目都支持与关系型数据库交互。为了执行一个可能只需要几毫秒的数据库事务，而要为每个用户创建一个新的数据库连接会造成大量时间消耗。对打存在大量并发访问的互联网应用来说，为每一个用户打开一个连接是很困难的。于是，工程师们常常希望共享一个连接池共所有并发访问应用的用户使用。在某一个时间段执行一个请求的用户数通常占所有活跃用户数很小的一部分，在请求处理过程中，一个数据库连接的时间至少是需要的。应用自身记录日志到关系型数据库管理系统，在内部处理任何用户报告的问题。

已经有几个数据库连接池发布，包括apache发布的和其他的。这个 Commons 包提供了一个机会去感受需要去创建和操作一个高效的、富有特性的包带来的影响。

<code>commons-dbcp2</code>依赖<code>commons-pool2</code>包下的代码，它操作的底层对象池机制。

DBCP现在一共有3个不同的版本去支持不同版本的JDBC。如下所示：
- DBCP2 仅能在Java7下编译和运行（JDBC 4.1）
- DBCP 1.4 仅能在Java5下编译和运行（JDBC 4）
- DBCP 1.3 仅能在Java 1.4 - 1,5下编译和运行（JDBC 3）

DBCP 2 应该被运行在Java7下的应用使用。

DBCP 1.4 应该被运行在Java6下的应用使用。

DBCP 1,3应该被运行在Java1.4或Java1.5下的应用使用。

DBCP 2基于Commons Pool 2，与DBCP 1.x 相比提供了更好的性能，支持JMX和其它一些新特性。迁移到2.x的用户应该认识到包的名称和Maven 坐标银镜发生了改变。从DBCP 2.x以后，不再二进制兼容 DBCP 1.x。这些用户也应该意识到一些配置选型应该被重命名，要和Commons Pool 2的新名称保持一致。

####基本数据源配置参数
| 参数 | 描述 |
| :----- | :----- |
| username | 连接用户名，传递给我们的JDBC驱动，建立一个连接 |
| password | 连接密码，传递给我们的JDBC驱动，建立一个连接 |
| url | 连接URL，床底给我们的JDBC驱动，建立一个连接 |
| driverClassName | 所使用的数据库驱动类的全限定名 |
| connectionProperties | 当建立一个连接时，将会传递给我们的JDBC驱动的连接属性。字符串格式必须为[属性名=属性;]*，注意："user"和"password"属性会被显式的传递，所以它们不需要被包含在这里 |

| 参数 | 默认值 | 描述 |
| :----  | :-------- | :----: |
| defaultAutoCommit | driver default | 被这个连接池创建的连接的默认的自动提交状态。如果没有设置，setAutoCommit方法将不会被调用 |
| defaultReadOnly | dirver default | 被这个数据库连接池创建的连接的默认只读状态。如果没有设置，setReadOnly方法将不会被调用。（一些数据驱动不支持只读模式，比如，Informix）|
| defaultTransactionlsolation | driver default | 被这个数据库连接池创建的连接的默认事务隔离状态。可以为下列值之一：NONE， READ_COMMITTED，READ_UNCOMMITED， REPEATABLE_READ ，SERIALIZE  |
| defaultCatalog | | 被这个数据库连接创建的连接的默认catalog |
| cacheState | true | 如果为true，连接池连接将会在第一次读或者写以及随后的所有写操作时缓存当前的readOnly和autoCommit设置。这样，在再次获取数据时不再需要额外的数据库查询。如果可以直接访问底层的数据库连接，readOnly 和/或 autoCommit设置改变当前的缓存值将不会对当前状态造成影响。在这案例中，将这个属性设置为false，缓存将会被取消 |

| 参数 | 默认值 | 描述 |
| :--- | :---  | :-- |
| initialSize | 0 | 当连接池被启动时，创建的初始化连接数 |
| maxTotal | 8 | 在同一时刻，连接池可以分配的最大活跃连接数，负值没有限制 |
| maxIdle | 8 | 在不额外的创建连接情况下，连接池中最大空闲连接数，负值没有限制 |
| minIdle | 0 | 在不额外的创建连接的情况下，连接池中最小的空闲连接数，在不创建任何连接时为0 |
| maxWaitMillis | indefinitely | 在抛出一个异常之前，连接池等待一个连接返回等待的最长时间（单位为毫秒），或者设为-1无限等待 |

**注意：**如果在高负载的系统中maxIdle的值被设的很低，你可能会发现一些连接被关闭，几乎同时新的连接的被打开。这是活跃的线程关闭连接的时间快于它们打开连接的时间，导致空闲连接数上升到maxIdle带来的结果。对于高负载系统来说，maxIdle最合适的值是不确定的，不过默认值是一个好的起点。

| 参数 | 默认值 | 描述 |
| :---- | :---- | :---- |
| validationQuery | | 在此连接池返回给调用者连接之前用于验证此连接的SQL查询如果指定，这个查询必须是SQL中的SELECT语句且至少返回一行数据。如果未指定，连接将通过<code>isValid()</code>来验证。 |
| testOnCreate | false | 用于标识对象在被创建后是否会被验证。如果对象验证失败，触发对象创建的担保尝试（borrow attempt）也会失败。 |
| testOnBorrow | true | 用于标识从此连接池借用（borrow）对象时，此对象是否会别验证。如果对象验证失败，它将会被从连接池中删除，然后我们尝试借用另一个。 |
| testOnReturn | false | 用于标识对象在返回到连接池时是否需要验证 |
| testWhileIdle | false | 用于标识对象是否会被空闲对象监视器验证。如果对象验证失败，它将会被从连接池中删除 |
| timeBetweenEvictionRunsMillis | -1 | 空闲对象监视器线程运行期间休眠的毫秒数。当此值为负数时，没有空闲对象监视器线程运行 |
| numTestsPerEvictionRun | 3 | 在每个空闲对象监视器线程运行期间测试的对象的个数 |
| minEvictableIdleTimeMillis | 1000*60*30 | 一个对象在空闲对象监视器线程判定为可以被清除之前可以在连接池中保持空闲的最小时长 |
| softMiniEvictableIdleTimeMillis | -1 | 一个连接在空闲连接监视器认定为可以被清除之前可以在连接池中保持空闲的最小时长，额外的条件就是至少 “最小空闲”（minIdle）连接保留在此连接池中。当<code>miniEvictableIdle</code>被设为一个整值时，<code>miniEvictableTimeMillis</code>首次被空闲连接监视器测试-比如，当空闲连接被监视器访问的时候，首先比较空闲时间和miniEvictableIdleTimeMillis（不考虑连接池中的空闲连接数）,然后再比较<code>softMinEvictableIdleTimeMillis</code>，包括最小空闲约束。 |
| maxConnLifetimeMillis | -1 | 一个连接的最长存活时间（单位为毫秒）。超过这个时间，这个连接将会在下次激活（activation)、钝化（passivation）或者验证（validation）测试时失败。此值如果小于或等于0，连接永久存活。 |
| logExpiredConnections | true | 设置在一个连接因为<code>maxConnLifetimeMillis</code>超时被连接池关闭时是否记录一条日志。将这个属性设为false，来关闭默认开启的过期连接日志。 |
| connectionInitSqls | null | 当物理连接被创建时，用来初始化这些连接的一些SQL语句。这些语句仅被执行一次——当配置连接工厂创建连接时。 |
| lifo | true | 如果设为true，则从连接池中“借用”对象时将返回最近被使用的对象（如果有空闲对象可用）。如果设为false，则来连接池行为上类似于一个FIFO队列，连接将会被从空闲实例池中以他们返回连接池中时的顺序被取走。 |


| 参数 | 默认值 | 描述 |
| :----- | :----- | :----- |
| poolPreparedStatements | false | 允许在此连接池中使用预编译语句池 |
| maxOpenPreparedStatements | unlimited | 在同一时刻，此语句池可以分配的打开语句的最大个数，负值将没有限制。 |

这个组件也可以池化（pool）PreparedStatement。当允许为每一个连接创建一个语句池时，PreparedStatement会被连接池通过下列方法之一来创建：
- public PreparedStatemet prepareStatement(String sql)
- public PreparedSattement prepareSattement(String sql, int resultSetType, int resultSetConcurrency)

**注意：**-确保你的连接为其它连接保留了一些资源。Pooling PreparedSattements 可能会让它们的游标在数据中一直打开，导致一个连接用尽游标，特别，如果<code>maxPreparedStatements</code>保留默认值（没有限制）且一个程序在每一个连接上打开了大量的PreparedSattements。为了避免这个问题，<code>maxOpenPreparedStatements</code>应该被设为一个小于在一个连接上可以打开的最大游标数的值。

| 参数 | 默认值 | 描述 |
| :--- | :--- | :--- |
| accessToUnderlyingConnectionAllowed | false | 是否允许PoolGuard访问底层连接 |

当允许你访问底层连接的时候，可以通过如下方式：

    Connection conn = ds.getConnection();
    Connection dconn = ((DelegatingConnection)conn).getInnermostDelegate();
    ...
    conn.close();

**提示：**默认是false，这个一个存在潜在风险的操作，恶意程序可以做一些对系统有害的事情。（当 guarded connection 已经关闭的时候，关闭底层连接或者继续使用）时刻关注且仅在你需要直接访问驱动指定扩展时才使用。

**注意：**不要关闭底层连接，只有一个。

| 参数 | 默认值 | 描述 |
| :--- | :--- | :--- |
| removeAbandonedOnMaintenance/removeAbandonedOnBorrow | false |是否移除被丢弃的对象在它们超出<code>removeAbandonedTimout</code>时。一个连接在大于<code>removeAbandonedTimeout</code>时长内没有被使用就可以考虑丢弃并认定为可移除。创建一个<code>Statement</code>，<code>PreparedStatement</code>或者<code>CallableSatement</code>或者使用它们其中之一执行一个查询会重置其父连接的lastUsed属性。Setting one or both of these to true can recover db connections from poorly written applications which fail to close connections.removeAbandonedOnMaintenance to true removes abandoned connections on the maintenance cycle (when eviction ends). This property has no effect unless maintenance is enabled by setting timeBetweenEvicionRunsMillis to a positive value. If removeAbandonedOnBorrow is true, abandoned connections are removed each time a connection is borrowed from the pool, with the additional requirements that: getNumActive() > getMaxTotal() - 3; and getNumIdle() < 2 |
| removeAbandonedTimeout | 300 | 一个可丢弃连接被移除之前的时长（单位：秒） |
| logAbandoned | false | Flag to log stack traces for application code which abandoned a Statement or Connection.Logging of abandoned Statements and Connections adds overhead for every Connection open or new Statement because a stack trace has to be generated. |

**提示：**If you have enabled removeAbandonedOnMaintenance or removeAbandonedOnBorrow then it is possible that a connection is reclaimed by the pool because it is considered to be abandoned. This mechanism is triggered when (getNumIdle() < 2) and (getNumActive() > getMaxTotal() - 3) and removeAbandonedOnBorrow is true; or after eviction finishes and removeAbandonedOnMaintenance is true. For example, maxTotal=20 and 18 active connections and 1 idle connection would trigger removeAbandonedOnBorrow, but only the active connections that aren't used for more then "removeAbandonedTimeout" seconds are removed (default 300 sec). Traversing a resultset doesn't count as being used. Creating a Statement, PreparedStatement or CallableStatement or using one of these to execute a query (using one of the execute methods) resets the lastUsed property of the parent connection.

| 参数 | 默认值 | 描述 |
| :----| :---- | :---- |
| fastFailValidation | false | When this property is true, validation fails fast for connections that have thrown "fatal" SQLExceptions. Requests to validate disconnected connections fail immediately, with no call to the driver's isValid method or attempt to execute a validation query.The SQL_STATE codes considered to signal fatal errors are by default the following: 57P01 (ADMIN SHUTDOWN) ， 57P02 (CRASH SHUTDOWN)，57P03 (CANNOT CONNECT NOW)，01002 (SQL92 disconnect error)，JZ0C0 (Sybase disconnect error)，JZ0C1 (Sybase disconnect error)，Any SQL_STATE code that starts with "08" To override this default set of disconnection codes, set the disconnectionSqlCodes property. |
| disconnectionSqlCodes | null | Comma-delimited list of SQL_STATE codes considered to signal fatal disconnection errors. Setting this property has no effect unless fastFailValidation is set to true. |

####开发者指南
**BasicDataSource**
![BasicDataSource](https://commons.apache.org/proper/commons-dbcp/images/uml/BasicDataSource.gif)

**ConnectionFactory**
![ConnectionFactory](https://commons.apache.org/proper/commons-dbcp/images/uml/ConnectionFactory.gif)

#####JNDI Howto
JNDI是Java平台的一部分，提供给基于Java技术的应用一个统一的接口映射到多个命名和目录服务。你可以使用这项工业标准构建强大的、可移植的 directory-enabled 引用。

当你在一个应用服务器中部署你的程序时，容器将会为启动 JNDI 树。但是，如果你写的是一个框架或者仅仅是一个 standalone application ，那么下面的例子将会展示给你如何构造和将引用绑定到 DBCP 数据源上。

下面的例子使用的是sun文件系统JNDI服务提供器。你可以从 [JNDI software sownload](http://www.oracle.com/technetwork/java/index.html)页面下载它。

**BasicDataSource**

	    System.setProperty(Context.INITIAL_CONTEXT_FACTORY,
	    "com.sun.jndi.fscontext.RefFSContextFactory");
		  System.setProperty(Context.PROVIDER_URL, "file:///tmp");
	  InitialContext ic = new InitialContext();
	
	  // Construct BasicDataSource
	  BasicDataSource bds = new BasicDataSource();
	  bds.setDriverClassName("org.apache.commons.dbcp2.TesterDriver");
	  bds.setUrl("jdbc:apache:commons:testdriver");
	  bds.setUsername("username");
	  bds.setPassword("password");
	
	  ic.rebind("jdbc/basic", bds);
	   
	  // Use
	  InitialContext ic2 = new InitialContext();
	  DataSource ds = (DataSource) ic2.lookup("jdbc/basic");
	  assertNotNull(ds);
	  Connection conn = ds.getConnection();
	  assertNotNull(conn);
	  conn.close();

**PerUserPoolDataSource**

	    System.setProperty(Context.INITIAL_CONTEXT_FACTORY,
	    "com.sun.jndi.fscontext.RefFSContextFactory");
		  System.setProperty(Context.PROVIDER_URL, "file:///tmp");
	  InitialContext ic = new InitialContext();
	
	  // Construct DriverAdapterCPDS reference
	  Reference cpdsRef = new Reference("org.apache.commons.dbcp2.cpdsadapter.DriverAdapterCPDS",
	    "org.apache.commons.dbcp2.cpdsadapter.DriverAdapterCPDS", null);
	  cpdsRef.add(new StringRefAddr("driver", "org.apache.commons.dbcp2.TesterDriver"));
	  cpdsRef.add(new StringRefAddr("url", "jdbc:apache:commons:testdriver"));
	  cpdsRef.add(new StringRefAddr("user", "foo"));
	  cpdsRef.add(new StringRefAddr("password", "bar"));
	  ic.rebind("jdbc/cpds", cpdsRef);
	     
	  // Construct PerUserPoolDataSource reference
	  Reference ref = new Reference("org.apache.commons.dbcp2.datasources.PerUserPoolDataSource",
	    "org.apache.commons.dbcp2.datasources.PerUserPoolDataSourceFactory", null);
	  ref.add(new StringRefAddr("dataSourceName", "jdbc/cpds"));
	  ref.add(new StringRefAddr("defaultMaxTotal", "100"));
	  ref.add(new StringRefAddr("defaultMaxIdle", "30"));
	  ref.add(new StringRefAddr("defaultMaxWaitMillis", "10000"));
	  ic.rebind("jdbc/peruser", ref);
	     
	  // Use
	  InitialContext ic2 = new InitialContext();
	  DataSource ds = (DataSource) ic2.lookup("jdbc/peruser");
	  assertNotNull(ds);
	  Connection conn = ds.getConnection("foo","bar");
	  assertNotNull(conn);
	  conn.close();

####类图
**PoolingDataSource **
![PoolingDataSource](https://commons.apache.org/proper/commons-dbcp/images/uml/PoolingDataSource.gif)

**PoolingDataConnection**
![PoolingDataSource](https://commons.apache.org/proper/commons-dbcp/images/uml/PoolingConnection.gif)

**Delegating**
![Delegating](https://commons.apache.org/proper/commons-dbcp/images/uml/Delegating.gif)

**AbandonedObjectPool**
![AbandonedObjectPool](https://commons.apache.org/proper/commons-dbcp/images/uml/AbandonedObjectPool.gif)

####序列图
**createDataSource**
![createDataSource](https://commons.apache.org/proper/commons-dbcp/images/uml/createDataSource.gif)

**getConnection**
![getConnection](https://commons.apache.org/proper/commons-dbcp/images/uml/getConnection.gif)

**prepareStatement**
![prepareStatement](https://commons.apache.org/proper/commons-dbcp/images/uml/prepareStatement.gif)

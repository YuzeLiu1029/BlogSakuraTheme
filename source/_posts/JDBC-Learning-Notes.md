---
title: PGSQL_JDBC_Learning_Notes
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-11-21 11:21:59
tags:
    - tips
    - 技术
    - 学习笔记
keywords: PGSQL
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123869.jpg
---
# JDBC API介绍
## JDBC主要API简介
JDBC API是一系列接口，使程序与数据库连接，执行SQL语句，得到返回结果。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/jdbc1.jpg)

## Driver接口
Java.sql.Driver 接口是所有 JDBC 驱动程序需要实现的接口。这个接口是提供给数据库厂商使用的，不同数据库厂商提供不同的实现。在程序中不需要直接去访问实现了 Driver 接口的类，而是由驱动程序管理器类(java.sql.DriverManager)去调用这些Driver实现

### 加载与注册JDBC驱动
指定驱动 ：```Class.forName(driverName)```。向其传递要加载的 JDBC 驱动的类名```DriverManager``` 类是驱动程序管理器类，负责管理驱动程序。通常不用显式调用 DriverManager 类的 registerDriver() 方法来注册驱动程序类的实例，因为 Driver 接口的驱动程序类都包含了静态代码块，在这个静态代码块中，会调用DriverManager.registerDriver() 方法来注册自身的一个实例
（java反射机制 ：```static Class forName(String className)``` : 返回与带有给定字符串名的类或接口相关联的class对象。driverName其实是需要反射的类的全路径，已得到想反射的类。)
```java
Class.forName("org.postgresql.Driver"); //加载JDBC驱动
```
## 建立连接
```Connection conn = DriverManager.getConnection(url, props)```。
调用DriverManager类的```getConnection()```建立数据库连接。
JDBC URL 用于标识一个被注册的驱动程序，驱动程序管理器通过这个 URL 选择正确的驱动程序，从而建立到数据库的连接。JDBC URL的标准由三部分组成，各部分间用冒号分隔。

* ```jdbc:<子协议>:<子名称> ：jdbc:postgresql://localhost:5432;```
*  协议：JDBC URL中的协议总是jdbc
*  子协议：标识一个数据库的驱动程序
*  子名称：标识数据库的方法，子名称可以依不同的子协议而变化，用子名称的目的是为了定位数据库提供足够的信息。

### 常用数据库JDBC URL格式：
* 对于 Oracle 数据库连接，采用如下形式:
 ```jdbc:oracle:thin:@localhost:1521:sid```

* 对于 SQLServer 数据库连接，采用如下形式：
```jdbc:microsoft:sqlserver//localhost:1433; DatabaseName=sid```

* 对于 MYSQL 数据库连接，采用如下形式：
```jdbc:mysql://localhost:3306/sid```

## 访问数据库
数据库连接被用于向数据库服务器发送命令和 SQL 语句，在连接建立后，需要对数据库进行访问，执行 sql 语句。
在java.sql 包中有 3 个接口分别定义了对数据库的调用的不同方式：

* Statement ：普通的不带参的查询SQL
* PreparedStatement ：支持可变参数的SQL
* CallableStatement ：支持调用存储过程,提供了对输出和输入/输出参数(INOUT)的支持

带参查询：在查询过程共如果加入了参数，那么这种情况就叫带参查询。参数化查询(Parameterized Query 或 Parameterized Statement)是指在设计与数据库链接并访问数据时，在需要填入数值或数据的地方，使用参数 (Parameter) 来给值，这个方法目前已被视为最有效可预防SQL注入攻击 (SQL Injection) 的攻击手法的防御方式。（https://www.cnblogs.com/kdp0213/p/8554032.html）
不带参查询：在查询过程共如果没有加入了参数，那么这种情况就叫不带参查询。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/statement.jpg)

Statement, PreparedStatement, CallableStatement都是接口。

* Statement继承自Wrapper
* PreparedStatement继承自Statement
* CallableStatement继承自PreparedStatement。

### Statement
```Statement stmt = (Connection) conn.createStatement(); ```
通过调用```Statement```对象```createStatement```方法来创建对象. stmt用于执行静态的SQL语句，并且返回执行结果。
Statement中定义了两种方法用于执行SQL语句。

* ResultSet excuteQuery(String sql):   ```stmt.excuteQuery(sql);``` 
  用来执行```SELECT```查询语句。

* int excuteUpdate(String sql): ```stmt.excuteUpdate;```
用来执行```Update```,```Insert```,```CREATE TABLE```,```TRUNCATE```

## ResultSet
通过调用 Statement 对象的 excuteQuery() 方法创建该对象。
ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商实现。
ResultSet 对象维护了一个指向当前数据行的游标，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的 next() 方法移动到下一行。

## 总结
* java.sql.DriverManager用来装载驱动程序，获取数据库连接。
* java.sql.Connection完成对某一指定数据库的联接
* java.sql.Statement在一个给定的连接中作为SQL执行声明的容器，他包含了两个重要的子类型　　　　　
  * java.sql.PreparedSatement 用于执行预编译的sql声明
  * java.sql.CallableStatement用于执行数据库中存储过程的调用
* java.sql.ResultSet对于给定声明取得结果的途径

# PGSQL源码阅读
调用JDBC通用api执行数据库操作时，最基本的流程和步骤是：

```java
Class.forName("org.postgresql.Driver"); //注册驱动
Connection conn = DriverManager.getConnection(url, props) //建立连接 (PGConnection)
Statement stmt = conn.createStatement(); //建立Statement
stmt.excuteUpdate(sql) //执行建表，增删改，清表操作
stmt.excuteQuery(sql); //执行查询操作
stmt.close(); //关闭Statement
conn.close(); //关闭连接
```
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/SequenceDiagram1.jpg)

## 1. 注册驱动
首先通过```class.forName("org.postgresql.Driver");```。```class.forName```会要求JVM查找并加载指定的类，也就是说在这个过程中肯定的JVM会执行该类的静态代码段，JAVA规范中明确规定：所有的驱动程序必须在静态初始化代码块中将驱动注册到驱动程序管理器中，这样通过这个反射的方式，我们就把驱动给注册进来。
1. 通过class.forName("org.postgresql.Driver")反射加载这个类.
2. 会自动执行Driver类的静态代码块中的register这个静态方法.
3. register方法中先判断是否注册过，如果没有则执行下面三行代码.(单例模式)。
4. 将Driver注册到DriverManager中。

### Driver.java
```java
public static void register() throws SQLException {
    if (isRegistered()) {
      throw new IllegalStateException(
          "Driver is already registered. It can only be registered once.");
    }
    Driver registeredDriver = new Driver();
    DriverManager.registerDriver(registeredDriver);//执行时会加载java.sql.DriverManager类，并执行它的静态代码块。
    Driver.registeredDriver = registeredDriver;//Driver类定义的静态变量，引用指向自己，deregister时可以用到。
  }
  public static void deregister() throws SQLException {
    if (!isRegistered()) {
      throw new IllegalStateException(
          "Driver is not registered (or it has not been registered using Driver.register() method)");
    }
    DriverManager.deregisterDriver(registeredDriver);
    registeredDriver = null;
  }
```
## 2. 获取连接
* DriverManager中有4个```getConnection()```方法，前三个都是对访问进行转接，将传入参数处理成同一格式后调用最后一个```getConnection()```的方法。前三个方法都是public的，最后一个是private的。前三个开放的方法，传入的参数可以是单独的url，也可以是url+property，或者url+user_name+password.

```java
public static Connection getConnection(String url,java.util.Properties info);
public static Connection getConnection(String url,String user, String password);
public static Connection getConnection(String url);              
```

* 前三个方法最终向私有```getConnection```方法传入的参数为```String url```, ```java.util.Properties info```以及```Reflection.getCallerClass()``` ，传入这个参数的目的是在方法里面获取该类的类加载器，用于判断注册驱动列表中哪一个是需要获取连接的驱动。
* ```Reflection.getCallerClass()```，是反射中的一个方法，这个方法是native的，用来返回他的调用类。
 * **说明** ：也就说是哪个类调用了这个方法，Reflection类位于调用栈中的0帧位置，在JDK7以前，该方法可以传入int n返回调用栈中从0帧开始的第n帧中的类，在JDK7中，需要设置java命令行选项```Djdk.reflect.allowGetCallerClass```来使用该方法，到了JDK8时，再调用该方法会导致```UnsupportedOperationException```异常。
 JDK8中```getCallerClass```使用方法变更为```getCallerClass(),Reflection.getCallerClass()```方法调用所在的方法必须用@CallerSensitive进行注解，通过此方法获取class时会跳过链路上所有的有```@CallerSensitive```注解的方法的类，直到遇到第一个未使用该注解的类，避免了用```Reflection.getCallerClass(int n)``` 这个过时方法来自己做判断。
    在这里每个```getConnection```都是用```CallerSensitive```修饰的，调用```getCallerClass```应该是获取外面使用```DriverManager.getConnection()```的类的名称，即在class A中调用了```DriverManager.getConnection()```，则返回class A。
* ```getConnection```方法调用 ```caller.getClassLoader()```方法获取传入类的类加载器，这里还涉及到了委派链的相关知识，如果无法获取到类加载器，将通过```Thread.currentThread().getContextClassLoader();```获取线程上下文类加载器
然后在对驱动注册列表中注册的驱动类进行遍历，通过```isDriverAllowed(aDriver.driver, callerCL)```方法判断列表中哪个驱动是当前请求获取连接的类所调用的驱动。
```isDriverAllowed```方法中获取了传入的驱动类的类加载器，首先通过```aClass = Class.forName(driver.getClass().getName(), true, classLoader);```方法获得类，该方法返回与给定字符串名的类或接口的Class对象，使用给定的类加载器进行加载，然后通过
```result = ( aClass == driver.getClass() ) ? true : false;```进行对比，只有同一个类加载器中的Class使用==比较时才会相等，此处就是校验用户注册Driver时该Driver所属的类加载器与调用时的是否为同一个
通过判断得到需要的驱动类，并调用Driver类的connect方法获取连接。
* Driver类的connect方法中会先判断url前缀是否为jdbc:postgresql，然后通过将传入的properties中的属性信息，初始化给connect定义的Properties props，并getDefaultProperties方法为driver类中定义的properties进行扩权操作，简单来说就是保证其有足够权限读取资源。
* 调用acceptsURL方法，该方法对URL进行了校验与解析
* 解析无误后会调用makeConnection方法获取连接。如果设置了等待连接时间，那么还会开启一个线程来调用makeConnection方法。
* makeConnection方法中只有一个简单的return new PgConnection(hostSpecs(props), user(props), database(props), props, url)，返回的Connection类就是通常来说获取的连接。
* PgConnection类实现了BaseConnection接口，该接口继承于PGConnection, Connection接口，在这个类的构造方法中，调用了QueryExecutor org.postgresql.core.ConnectionFactory.openConnection(hostSpecs, user, database, info)工厂方法返回连接：
this.queryExecutor = ConnectionFactory.openConnection(hostSpecs, user, database, info);
* ConnectionFactory类的openConnection方法中，返回了org.postgresql.core.QueryExecutor queryExecutor = org.postgresql.core.v3.ConnectionFactory.openConnectionImpl( hostSpecs, user, database, info);
* 最终在org.postgresql.core.v3.ConnectionFactory类中的openConnectionImpl方法中真正实现了与数据库的交互，在这里完成了与数据库的连接。

### DriverManager.class
```java
private static Connection getConnection(
        String url, java.util.Properties info, Class<?> caller) throws SQLException {
        /*
         * When callerCl is null, we should check the application's
         * (which is invoking this class indirectly)
         * classloader, so that the JDBC driver class outside rt.jar
         * can be loaded from here.
         */
        ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
        synchronized(DriverManager.class) {
            // synchronize loading of the correct classloader.
            if (callerCL == null) {
                callerCL = Thread.currentThread().getContextClassLoader();
            }
        }

        if(url == null) {
            throw new SQLException("The url cannot be null", "08001");
        }

        println("DriverManager.getConnection(\"" + url + "\")");

        // Walk through the loaded registeredDrivers attempting to make a connection.
        // Remember the first exception that gets raised so we can reraise it.
        SQLException reason = null;

        for(DriverInfo aDriver : registeredDrivers) {
            // If the caller does not have permission to load the driver then
            // skip it.
            if(isDriverAllowed(aDriver.driver, callerCL)) {
                try {
                    println("    trying " + aDriver.driver.getClass().getName());
                    Connection con = aDriver.driver.connect(url, info);
                    if (con != null) {
                        // Success!
                        println("getConnection returning " + aDriver.driver.getClass().getName());
                        return (con);
                    }
                } catch (SQLException ex) {
                    if (reason == null) {
                        reason = ex;
                    }
                }

            } else {
                println("    skipping: " + aDriver.getClass().getName());
            }

        }

        // if we got here nobody could connect.
        if (reason != null)    {
            println("getConnection failed: " + reason);
            throw reason;
        }

        println("getConnection: no suitable driver found for "+ url);
        throw new SQLException("No suitable driver found for "+ url, "08001");
    }
    
    private static boolean isDriverAllowed(Driver driver, ClassLoader classLoader) {
        boolean result = false;
        if(driver != null) {
            Class<?> aClass = null;
            try {
                aClass =  Class.forName(driver.getClass().getName(), true, classLoader);
            } catch (Exception ex) {
                result = false;
            }

             result = ( aClass == driver.getClass() ) ? true : false;
        }

        return result;
}
```

### Driver.java
```java
/*
Driver.java 需要改写的部分:
将makeConnection改写，我们将模仿PGConnection重新写一个Connection的子类继承Baseonnection，那么makeConnection将需要new一个我们自定义的类。
*/
public Connection connect(String url, Properties info) throws SQLException {
    if (url == null) {
      throw new SQLException("url is null");
    }
    // get defaults
    Properties defaults;

    if (!url.startsWith("jdbc:postgresql:")) {
      return null;
    }
    try {
      defaults = getDefaultProperties();
    } catch (IOException ioe) {
      throw new PSQLException(GT.tr("Error loading default settings from driverconfig.properties"),
          PSQLState.UNEXPECTED_ERROR, ioe);
    }

    // override defaults with provided properties
    Properties props = new Properties(defaults);
    if (info != null) {
      Set<String> e = info.stringPropertyNames();
      for (String propName : e) {
        String propValue = info.getProperty(propName);
        if (propValue == null) {
          throw new PSQLException(
              GT.tr("Properties for the driver contains a non-string value for the key ")
                  + propName,
              PSQLState.UNEXPECTED_ERROR);
        }
        props.setProperty(propName, propValue);
      }
    }
    // parse URL and add more properties
    if ((props = parseURL(url, props)) == null) {
      return null;
    }
    try {
      // Setup java.util.logging.Logger using connection properties.
      setupLoggerFromProperties(props);

      LOGGER.log(Level.FINE, "Connecting with URL: {0}", url);

      long timeout = timeout(props);
      if (timeout <= 0) {
        return makeConnection(url, props);
      }

      ConnectThread ct = new ConnectThread(url, props);
      Thread thread = new Thread(ct, "PostgreSQL JDBC driver connection thread");
      thread.setDaemon(true); // Don't prevent the VM from shutting down
      thread.start();
      return ct.getResult(timeout);
    } catch (PSQLException ex1) {
      LOGGER.log(Level.FINE, "Connection error: ", ex1);
      // re-throw the exception, otherwise it will be caught next, and a
      // org.postgresql.unusual error will be returned instead.
      throw ex1;
    } catch (java.security.AccessControlException ace) {
      throw new PSQLException(
          GT.tr(
              "Your security policy has prevented the connection from being attempted.  You probably need to grant the connect java.net.SocketPermission to the database server host and port that you wish to connect to."),
          PSQLState.UNEXPECTED_ERROR, ace);
    } catch (Exception ex2) {
      LOGGER.log(Level.FINE, "Unexpected connection error: ", ex2);
      throw new PSQLException(
          GT.tr(
              "Something unusual has occurred to cause the driver to fail. Please report this exception."),
          PSQLState.UNEXPECTED_ERROR, ex2);
    }
  }
  //TODO:改写
  private static Connection makeConnection(String url, Properties props) throws SQLException {
    return new PgConnection(hostSpecs(props), user(props), database(props), props, url);
  }
```
### PGConnection.java

主要函数：```ConnectionFactory.openConnection(hostSpecs, user, database, info);```

```java
/*
PGConnection：
YsConnection: YsConnection将用来取代PGConnnection类
在该类中会增加两个重要的变量：

YsConnectionID ： 这个ID由连接Server端后提供，在之后的SQL语句运行中进行传递。

GRpcClient ：新建一个GRpcclient类，该类遵循GRPC协议，并对连接进行存根以便实现复用。

传递参数的改变：
String url由 ： 
jdbc:postgresql://localhost/db_name 
->
jdbc:xxxxx://XXX.X.X.X:PORT NUMBER/db_name
其中：
xxxxx为我们jdbc驱动的子协议名称， 
IP地址为GRPC client端和server端通信地址， 
PORT NUMBER为GRPC通信端口， 
db_name为数据库名称（指定的名称？）。

Properties info中将包含：
user -> user_name;
password -> password;
ssl -> default-false(不需要）
*/
//TODO: 主要改写的代码部分：
public PgConnection(HostSpec[] hostSpecs,
                      String user,
                      String database,
                      Properties info,
                      String url) throws SQLException {
    // Print out the driver version number
    LOGGER.log(Level.FINE, org.postgresql.util.DriverInfo.DRIVER_FULL_NAME);

    this.creatingURL = url;

    this.readOnlyBehavior = getReadOnlyBehavior(PGProperty.READ_ONLY_MODE.get(info));

    setDefaultFetchSize(PGProperty.DEFAULT_ROW_FETCH_SIZE.getInt(info));

    setPrepareThreshold(PGProperty.PREPARE_THRESHOLD.getInt(info));
    if (prepareThreshold == -1) {
      setForceBinary(true);
    }

    // Now make the initial connection and set up local state
    this.queryExecutor = ConnectionFactory.openConnection(hostSpecs, user, database, info);

    // WARNING for unsupported servers (8.1 and lower are not supported)
    if (LOGGER.isLoggable(Level.WARNING) && !haveMinimumServerVersion(ServerVersion.v8_2)) {
      LOGGER.log(Level.WARNING, "Unsupported Server Version: {0}", queryExecutor.getServerVersion());
    }

    setSessionReadOnly = createQuery("SET SESSION CHARACTERISTICS AS TRANSACTION READ ONLY", false, true);
    setSessionNotReadOnly = createQuery("SET SESSION CHARACTERISTICS AS TRANSACTION READ WRITE", false, true);

    // Set read-only early if requested
    if (PGProperty.READ_ONLY.getBoolean(info)) {
      setReadOnly(true);
    }

    Set<Integer> binaryOids = getBinaryOids(info);

    // split for receive and send for better control
    Set<Integer> useBinarySendForOids = new HashSet<Integer>(binaryOids);

    Set<Integer> useBinaryReceiveForOids = new HashSet<Integer>(binaryOids);

    /*
     * Does not pass unit tests because unit tests expect setDate to have millisecond accuracy
     * whereas the binary transfer only supports date accuracy.
     */
    useBinarySendForOids.remove(Oid.DATE);

    queryExecutor.setBinaryReceiveOids(useBinaryReceiveForOids);
    queryExecutor.setBinarySendOids(useBinarySendForOids);

    if (LOGGER.isLoggable(Level.FINEST)) {
      LOGGER.log(Level.FINEST, "    types using binary send = {0}", oidsToString(useBinarySendForOids));
      LOGGER.log(Level.FINEST, "    types using binary receive = {0}", oidsToString(useBinaryReceiveForOids));
      LOGGER.log(Level.FINEST, "    integer date/time = {0}", queryExecutor.getIntegerDateTimes());
    }

    //
    // String -> text or unknown?
    //

    String stringType = PGProperty.STRING_TYPE.get(info);
    if (stringType != null) {
      if (stringType.equalsIgnoreCase("unspecified")) {
        bindStringAsVarchar = false;
      } else if (stringType.equalsIgnoreCase("varchar")) {
        bindStringAsVarchar = true;
      } else {
        throw new PSQLException(
            GT.tr("Unsupported value for stringtype parameter: {0}", stringType),
            PSQLState.INVALID_PARAMETER_VALUE);
      }
    } else {
      bindStringAsVarchar = true;
    }

    // Initialize timestamp stuff
    timestampUtils = new TimestampUtils(!queryExecutor.getIntegerDateTimes(), new Provider<TimeZone>() {
      @Override
      public TimeZone get() {
        return queryExecutor.getTimeZone();
      }
    });

    // Initialize common queries.
    // isParameterized==true so full parse is performed and the engine knows the query
    // is not a compound query with ; inside, so it could use parse/bind/exec messages
    commitQuery = createQuery("COMMIT", false, true).query;
    rollbackQuery = createQuery("ROLLBACK", false, true).query;

    int unknownLength = PGProperty.UNKNOWN_LENGTH.getInt(info);

    // Initialize object handling
    typeCache = createTypeInfo(this, unknownLength);
    initObjectTypes(info);

    if (PGProperty.LOG_UNCLOSED_CONNECTIONS.getBoolean(info)) {
      openStackTrace = new Throwable("Connection was created at this point:");
    }
    this.disableColumnSanitiser = PGProperty.DISABLE_COLUMN_SANITISER.getBoolean(info);

    if (haveMinimumServerVersion(ServerVersion.v8_3)) {
      typeCache.addCoreType("uuid", Oid.UUID, Types.OTHER, "java.util.UUID", Oid.UUID_ARRAY);
      typeCache.addCoreType("xml", Oid.XML, Types.SQLXML, "java.sql.SQLXML", Oid.XML_ARRAY);
    }

    this.clientInfo = new Properties();
    if (haveMinimumServerVersion(ServerVersion.v9_0)) {
      String appName = PGProperty.APPLICATION_NAME.get(info);
      if (appName == null) {
        appName = "";
      }
      this.clientInfo.put("ApplicationName", appName);
    }

    fieldMetadataCache = new LruCache<FieldMetadata.Key, FieldMetadata>(
            Math.max(0, PGProperty.DATABASE_METADATA_CACHE_FIELDS.getInt(info)),
            Math.max(0, PGProperty.DATABASE_METADATA_CACHE_FIELDS_MIB.getInt(info) * 1024 * 1024),
        false);

    replicationConnection = PGProperty.REPLICATION.get(info) != null;
}
```

```java
//TODO: 主要改写的代码部分：
public Statement createStatement() throws SQLException {
// We now follow the spec and default to TYPE_FORWARD_ONLY.
    return createStatement(ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
}


public PreparedStatement prepareStatement(String sql) throws SQLException {
    return prepareStatement(sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
}


public CallableStatement prepareCall(String sql) throws SQLException {
    return prepareCall(sql, ResultSet.TYPE_FORWARD_ONLY, ResultSet.CONCUR_READ_ONLY);
}
```

```java
public Statement createStatement(int resultSetType, int resultSetConcurrency,
      int resultSetHoldability) throws SQLException {
    checkClosed();
    return new PgStatement(this, resultSetType, resultSetConcurrency, resultSetHoldability);
}
```
### PgStatement.java
```java

PgStatement(PgConnection c, int rsType, int rsConcurrency, int rsHoldability)
      throws SQLException {
    this.connection = c;
    forceBinaryTransfers |= c.getForceBinary();
    resultsettype = rsType;
    concurrency = rsConcurrency;
    setFetchSize(c.getDefaultFetchSize());
    setPrepareThreshold(c.getPrepareThreshold());
    this.rsHoldability = rsHoldability;
  }
```
## 3. 执行SQL语句
### PgStatement.java
```java
public ResultSet executeQuery(String sql) throws SQLException {
    if (!executeWithFlags(sql, 0)) {
      throw new PSQLException(GT.tr("No results were returned by the query."), PSQLState.NO_DATA);
    }

    return getSingleResultSet();
}

public int executeUpdate(String sql) throws SQLException {
    executeWithFlags(sql, QueryExecutor.QUERY_NO_RESULTS);
    checkNoResultUpdate();
    return getUpdateCount();
}

public boolean executeWithFlags(String sql, int flags) throws SQLException {
    return executeCachedSql(sql, flags, NO_RETURNING_COLUMNS);
}

private boolean executeCachedSql(String sql, int flags, String[] columnNames) throws SQLException {
    PreferQueryMode preferQueryMode = connection.getPreferQueryMode();
    // Simple statements should not replace ?, ? with $1, $2
    boolean shouldUseParameterized = false;
    QueryExecutor queryExecutor = connection.getQueryExecutor();
    Object key = queryExecutor
        .createQueryKey(sql, replaceProcessingEnabled, shouldUseParameterized, columnNames);
    CachedQuery cachedQuery;
    boolean shouldCache = preferQueryMode == PreferQueryMode.EXTENDED_CACHE_EVERYTHING;
    if (shouldCache) {
      cachedQuery = queryExecutor.borrowQueryByKey(key);
    } else {
      cachedQuery = queryExecutor.createQueryByKey(key);
    }
    if (wantsGeneratedKeysOnce) {
      SqlCommand sqlCommand = cachedQuery.query.getSqlCommand();
      wantsGeneratedKeysOnce = sqlCommand != null && sqlCommand.isReturningKeywordPresent();
    }
    boolean res;
    try {
      res = executeWithFlags(cachedQuery, flags);
    } finally {
      if (shouldCache) {
        queryExecutor.releaseQuery(cachedQuery);
      }
    }
    return res;
}

public boolean executeWithFlags(CachedQuery simpleQuery, int flags) throws SQLException {
    checkClosed();
    if (connection.getPreferQueryMode().compareTo(PreferQueryMode.EXTENDED) < 0) {
      flags |= QueryExecutor.QUERY_EXECUTE_AS_SIMPLE;
    }
    execute(simpleQuery, null, flags);
    synchronized (this) {
      checkClosed();
      return (result != null && result.getResultSet() != null);
    }
 }
 
 protected final void execute(CachedQuery cachedQuery, ParameterList queryParameters, int flags)
      throws SQLException {
    try {
      executeInternal(cachedQuery, queryParameters, flags);
    } catch (SQLException e) {
      // Don't retry composite queries as it might get partially executed
      if (cachedQuery.query.getSubqueries() != null
          || !connection.getQueryExecutor().willHealOnRetry(e)) {
        throw e;
      }
      cachedQuery.query.close();
      // Execute the query one more time
      executeInternal(cachedQuery, queryParameters, flags);
    }
 }

private void executeInternal(CachedQuery cachedQuery, ParameterList queryParameters, int flags)
      throws SQLException {
    closeForNextExecution();

    // Enable cursor-based resultset if possible.
    if (fetchSize > 0 && !wantsScrollableResultSet() && !connection.getAutoCommit()
        && !wantsHoldableResultSet()) {
      flags |= QueryExecutor.QUERY_FORWARD_CURSOR;
    }

    if (wantsGeneratedKeysOnce || wantsGeneratedKeysAlways) {
      flags |= QueryExecutor.QUERY_BOTH_ROWS_AND_STATUS;

      // If the no results flag is set (from executeUpdate)
      // clear it so we get the generated keys results.
      //
      if ((flags & QueryExecutor.QUERY_NO_RESULTS) != 0) {
        flags &= ~(QueryExecutor.QUERY_NO_RESULTS);
      }
    }

    if (isOneShotQuery(cachedQuery)) {
      flags |= QueryExecutor.QUERY_ONESHOT;
    }
    // Only use named statements after we hit the threshold. Note that only
    // named statements can be transferred in binary format.

    if (connection.getAutoCommit()) {
      flags |= QueryExecutor.QUERY_SUPPRESS_BEGIN;
    }
    if (connection.hintReadOnly()) {
      flags |= QueryExecutor.QUERY_READ_ONLY_HINT;
    }

    // updateable result sets do not yet support binary updates
    if (concurrency != ResultSet.CONCUR_READ_ONLY) {
      flags |= QueryExecutor.QUERY_NO_BINARY_TRANSFER;
    }

    Query queryToExecute = cachedQuery.query;

    if (queryToExecute.isEmpty()) {
      flags |= QueryExecutor.QUERY_SUPPRESS_BEGIN;
    }

    if (!queryToExecute.isStatementDescribed() && forceBinaryTransfers
        && (flags & QueryExecutor.QUERY_EXECUTE_AS_SIMPLE) == 0) {
      // Simple 'Q' execution does not need to know parameter types
      // When binaryTransfer is forced, then we need to know resulting parameter and column types,
      // thus sending a describe request.
      int flags2 = flags | QueryExecutor.QUERY_DESCRIBE_ONLY;
      StatementResultHandler handler2 = new StatementResultHandler();
      connection.getQueryExecutor().execute(queryToExecute, queryParameters, handler2, 0, 0,
          flags2);
      ResultWrapper result2 = handler2.getResults();
      if (result2 != null) {
        result2.getResultSet().close();
      }
    }

    StatementResultHandler handler = new StatementResultHandler();
    synchronized (this) {
      result = null;
    }
    try {
      startTimer();
      connection.getQueryExecutor().execute(queryToExecute, queryParameters, handler, maxrows,
          fetchSize, flags);
    } finally {
      killTimerTask();
    }
    synchronized (this) {
      checkClosed();
      result = firstUnclosedResult = handler.getResults();

      if (wantsGeneratedKeysOnce || wantsGeneratedKeysAlways) {
        generatedKeys = result;
        result = result.getNext();

        if (wantsGeneratedKeysOnce) {
          wantsGeneratedKeysOnce = false;
        }
      }
    }
  }
```

### QueryExecutorImpl.java
主要函数：
1. ```sendQuery(query, (V3ParameterList) parameters,maxRows, fetchSize, flags,handler, null);```
2. ```processResults(handler, flags);```          


```java
public synchronized void execute(Query query, ParameterList parameters, ResultHandler handler,
      int maxRows, int fetchSize, int flags) throws SQLException {
    waitOnLock();
    if (LOGGER.isLoggable(Level.FINEST)) {
      LOGGER.log(Level.FINEST, "  simple execute, handler={0}, maxRows={1}, fetchSize={2}, flags={3}",
          new Object[]{handler, maxRows, fetchSize, flags});
    }

    if (parameters == null) {
      parameters = SimpleQuery.NO_PARAMETERS;
    }

    flags = updateQueryMode(flags);

    boolean describeOnly = (QUERY_DESCRIBE_ONLY & flags) != 0;

    ((V3ParameterList) parameters).convertFunctionOutParameters();

    // Check parameters are all set..
    if (!describeOnly) {
      ((V3ParameterList) parameters).checkAllParametersSet();
    }

    boolean autosave = false;
    try {
      try {
        handler = sendQueryPreamble(handler, flags);
        autosave = sendAutomaticSavepoint(query, flags);
        sendQuery(query, (V3ParameterList) parameters, maxRows, fetchSize, flags,
            handler, null);
        if ((flags & QueryExecutor.QUERY_EXECUTE_AS_SIMPLE) != 0) {
          // Sync message is not required for 'Q' execution as 'Q' ends with ReadyForQuery message
          // on its own
        } else {
          sendSync();
        }
        processResults(handler, flags);
        estimatedReceiveBufferBytes = 0;
      } catch (PGBindException se) {
        // There are three causes of this error, an
        // invalid total Bind message length, a
        // BinaryStream that cannot provide the amount
        // of data claimed by the length argument, and
        // a BinaryStream that throws an Exception
        // when reading.
        //
        // We simply do not send the Execute message
        // so we can just continue on as if nothing
        // has happened. Perhaps we need to
        // introduce an error here to force the
        // caller to rollback if there is a
        // transaction in progress?
        //
        sendSync();
        processResults(handler, flags);
        estimatedReceiveBufferBytes = 0;
        handler
            .handleError(new PSQLException(GT.tr("Unable to bind parameter values for statement."),
                PSQLState.INVALID_PARAMETER_VALUE, se.getIOException()));
      }
    } catch (IOException e) {
      abort();
      handler.handleError(
          new PSQLException(GT.tr("An I/O error occurred while sending to the backend."),
              PSQLState.CONNECTION_FAILURE, e));
    }

    try {
      handler.handleCompletion();
      if (cleanupSavePoints) {
        releaseSavePoint(autosave, flags);
      }
    } catch (SQLException e) {
      rollbackIfRequired(autosave, e);
    }
  }

```

```java
protected ResultSet getSingleResultSet() throws SQLException {
    synchronized (this) {
      checkClosed();
      if (result.getNext() != null) {
        throw new PSQLException(GT.tr("Multiple ResultSets were returned by the query."),
            PSQLState.TOO_MANY_RESULTS);
      }

      return result.getResultSet();
    }
}

public int getUpdateCount() throws SQLException {
    synchronized (this) {
      checkClosed();
      if (result == null || result.getResultSet() != null) {
        return -1;
      }

      long count = result.getUpdateCount();
      return count > Integer.MAX_VALUE ? Statement.SUCCESS_NO_INFO : (int) count;
    }
}

```



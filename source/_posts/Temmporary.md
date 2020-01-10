---
title: Statement和PreparedStatement
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-12-09 22:30:10
tags: 
    - jdbc
    - 技术
    - 学习笔记
keywords: JDBC
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123843.jpg
---

# 关于Statement和PreparedStatement实现时的差别

# JDBC接口：PreparedStatement预处理

PreparedStatement：

* 是Statement接口的子接口
* 强大之处：
    * 防止SQL攻击；
    * 提高代码的可读性，维护性
    * 提高效率
* 用法：
    * 给出SQL模板
    * 调用Connection的PreparedStatement prepareStatement(String sql模板)，得到PreparedStatement对象pstmt；
    * 调用pstmt的setXxx()系列方法sql模板中的?赋值！如：pstmt.setString(1,"zhangsan");
    * 调用pstmt的executeUpdate()或executeQuery()，但它的方法都没有参数。
* 预处理的原理
    * 服务器的工作：
          * 校验sql语句的语法！
          * 编译：一个与函数相似的东西！
          * 执行：调用函数,
    * PreparedStatement：
          * 前提：连接的数据库必须支持预处理！几乎没有不支持的！
          * 每个pstmt都与一个sql模板绑定在一起，先把sql模板给数据库，数据库先进行校验，再进行编译。执行时只是把参数传递过去而已。若二次执行时，就不用再次校验语法，也不用再次编译！直接执行。
（上述文字转自：https://blog.csdn.net/weixin_42472048/article/details/82827147）

使用实例：

```java
// 准备四大参数
        String driverClassName = "com.mysql.jdbc.Driver";
        String url = "jdbc:mysql://localhost:3306/mydb1";
        String sqlusername = "root";
        String sqlpassword = "123456";
        // 加载驱动类
        Class.forName(driverClassName);
        // 得到连接
        Connection con = DriverManager.getConnection(url, sqlusername, sqlpassword);

        ////////////////////////////////
        ////////////////////////////////

        // 一、得到PreparedStatement
        //    a.给出SQL模  板：所用参数用"？" 替代
        //    b.调用Connection.PrepareStatement();方法  
        String sql = "select * from t_user where username=? and password=?";
        PreparedStatement pstmt = con.prepareStatement(sql);

        //二、 使用setXxx();方法给参数？ 赋值。
        pstmt.setString(1, username);  //给第一个问号赋值。值为username
        pstmt.setString(2, password);  //给第二个问号赋值。值为password

        //三、调用pstmt的executeQuery();  此方法无参
        ResultSet rs = pstmt.executeQuery();  //调用查询方法，向数据库发送查询语句。

        return rs.next();
```
(https://blog.csdn.net/qq_41307491/article/details/81448326)


在Connection创建Statement和PreparedStatement时候，所需要的参数不同：

PreparedStatemnt和Statement相比增加了一个sql语句的参数,具体如下代码：


```java

public Statement createStatement(int resultSetType, int resultSetConcurrency,
      int resultSetHoldability) throws SQLException {
    checkClosed();
    return new PgStatement(this, resultSetType, resultSetConcurrency, resultSetHoldability);
  }

  @Override
public PreparedStatement prepareStatement(String sql, int resultSetType, int resultSetConcurrency,
      int resultSetHoldability) throws SQLException {
    checkClosed();
    return new PgPreparedStatement(this, sql, resultSetType, resultSetConcurrency,
        resultSetHoldability);
  }
```

在新建```PreparedStatement```时，会执行下面两个函数，生成```preparedParameters```.

```java
 PgPreparedStatement(PgConnection connection, String sql, int rsType, int rsConcurrency,
      int rsHoldability) throws SQLException {
    this(connection, connection.borrowQuery(sql), rsType, rsConcurrency, rsHoldability);
  }

  PgPreparedStatement(PgConnection connection, CachedQuery query, int rsType,
      int rsConcurrency, int rsHoldability) throws SQLException {
    super(connection, rsType, rsConcurrency, rsHoldability);

    this.preparedQuery = query;
    this.preparedParameters = this.preparedQuery.query.createParameterList();
    // TODO: this.wantsGeneratedKeysAlways = true;

    setPoolable(true); // As per JDBC spec: prepared and callable statements are poolable by
  }
```
在使用preparedStatement执行```executeQuery(String sql)```时，会报错，具体代码如下，其中明确指出，```"Can''t use query methods that take a query string on a PreparedStatement."```

```java
  @Override
  public ResultSet executeQuery(String sql) throws SQLException {
    throw new PSQLException(
        GT.tr("Can''t use query methods that take a query string on a PreparedStatement."),
        PSQLState.WRONG_OBJECT_TYPE);
  }
```
执行```PreparedStatement```的```executeQuery```时,是不需要带参数的，具体代码如下：

```java
 @Override
  public ResultSet executeQuery() throws SQLException {
    if (!executeWithFlags(0)) {
      throw new PSQLException(GT.tr("No results were returned by the query."), PSQLState.NO_DATA);
    }

    return getSingleResultSet();
  }
```
此时，会执行```executeWithFlags(0)```：

```java
public boolean executeWithFlags(int flags) throws SQLException {
    try {
      checkClosed();

      if (connection.getPreferQueryMode() == PreferQueryMode.SIMPLE) {
        flags |= QueryExecutor.QUERY_EXECUTE_AS_SIMPLE;
      }

      execute(preparedQuery, preparedParameters, flags);

      synchronized (this) {
        checkClosed();
        return (result != null && result.getResultSet() != null);
      }
    } finally {
      defaultTimeZone = null;
    }
  }
```
请注意这段代码中的```execute(preparedQuery, preparedParameters, flags);```函数，这个```execute```也是```Statement```执行的函数，只是```Statement```执行时，```preparedParameters```这个参数是```null```.

下面是Statement调用execute函数时的代码:

```java
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

```
可以看见这个参数是```null```.

接下来，两个```statement```的```execute```已经合并，可以继续向下看,```eecute```会调用```executeInternal```函数，代码如下：

```java
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
```

其中```executeInternal```中会执行到：``` connection.getQueryExecutor().execute(queryToExecute, queryParameters, handler, maxrows,
          fetchSize, flags);```
重点来看这一段代码：

```java
public synchronized void execute(Query query, ParameterList parameters, ResultHandler handler,
      int maxRows, int fetchSize, int flags) throws SQLException {
    waitOnLock();
    if (LOGGER.isLoggable(Level.FINEST)) {
      LOGGER.log(Level.FINEST, "  simple execute, handler={0}, maxRows={1}, fetchSize={2}, flags={3}",
          new Object[]{handler, maxRows, fetchSize, flags});
    }

    if (parameters == null) {
      parameters = SimpleQuery.NO_PARAMETERS; //对于Statement来说， parameters == null
    }

    flags = updateQueryMode(flags);  //Get prefered query mode

    boolean describeOnly = (QUERY_DESCRIBE_ONLY & flags) != 0;

    ((V3ParameterList) parameters).convertFunctionOutParameters(); //预处理Parameters

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

最后调用sendExecute以及processResults方法来拉取数据:
```java
sendQuery(query, (V3ParameterList) parameters, maxRows, fetchSize, flags, handler, null);
 processResults(handler, flags);
```
其中在sendQuery中，如果parameter是不为null， 会遍历按单个sql语句发送，这一部分代码如下：

```java
for (int i = 0; i < subqueries.length; ++i) {
        final Query subquery = subqueries[i];
        flushIfDeadlockRisk(subquery, disallowBatching, resultHandler, batchHandler, flags);

        // If we saw errors, don't send anything more.
        if (resultHandler.getException() != null) {
          break;
        }

        // In the situation where parameters is already
        // NO_PARAMETERS it cannot know the correct
        // number of array elements to return in the
        // above call to getSubparams(), so it must
        // return null which we check for here.
        //
        SimpleParameterList subparam = SimpleQuery.NO_PARAMETERS;
        if (subparams != null) {
          subparam = subparams[i];
        }
        sendOneQuery((SimpleQuery) subquery, subparam, maxRows, fetchSize, flags);
      }
```

所以，对于参数化的查询，jdbc内部先将参数化的查询语句转化成非参数的SQL语句，再发给数据库请求数据。






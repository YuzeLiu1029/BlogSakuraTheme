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




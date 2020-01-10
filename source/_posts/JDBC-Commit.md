---
title: JDBC_Commit
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-12-10 13:05:15
tags:
    - tips
    - Java
    - 学习笔记
keywords:
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123866.jpg
---

PGSQL JDBC执行commit命令时，首先在new PgSqlConnection的时候，会执行：```commitQuery = createQuery("COMMIT", false, true).query;```提前创建一条commit的SQL语句。

```java
public void commit() throws SQLException {
    checkClosed();

    if (autoCommit) {
      throw new PSQLException(GT.tr("Cannot commit when autoCommit is enabled."),
          PSQLState.NO_ACTIVE_SQL_TRANSACTION);
    }

    if (queryExecutor.getTransactionState() != TransactionState.IDLE) {
      executeTransactionCommand(commitQuery);
    }
}
```
执行```commit```命令时，会执行上述代码。
```executeTransactionCommand(commitQuery);```：

```java
private void executeTransactionCommand(Query query) throws SQLException {
    int flags = QueryExecutor.QUERY_NO_METADATA | QueryExecutor.QUERY_NO_RESULTS
        | QueryExecutor.QUERY_SUPPRESS_BEGIN;
    if (prepareThreshold == 0) {
      flags |= QueryExecutor.QUERY_ONESHOT;
    }

    try {
      getQueryExecutor().execute(query, null, new TransactionCommandHandler(), 0, 0, flags);
    } catch (SQLException e) {
      // Don't retry composite queries as it might get partially executed
      if (query.getSubqueries() != null || !queryExecutor.willHealOnRetry(e)) {
        throw e;
      }
      query.close();
      // retry
      getQueryExecutor().execute(query, null, new TransactionCommandHandler(), 0, 0, flags);
    }
  }
```



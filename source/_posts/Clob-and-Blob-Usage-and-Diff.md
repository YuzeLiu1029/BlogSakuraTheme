---
title: Clob_and_Blob_Usage_and_Diff
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-01-13 11:32:10
tags:
    - tech
    - DataBase
    - JDBC
keywords:
    - tech
    - DataBase
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123866.jpg
---
<style>
table, th, td{
  border: 1px solid #1e1b26;
}

th{
  background: #CCEEFF;
  color: #FBFBEF;
}
</style>
# CLOB与BLOB的区别和用途
BLOB和CLOB都是大字段类型，BLOB是按二进制来存储的，而CLOB是可以直接存储文字的。其实两个是可以互换的的，或者可以直接用LOB字段代替这两个。
为了更好的管理，比如ORACLE数据库，通常像图片、文件、音乐等信息就用BLOB字段来存储，先将文件转为二进制再存储进去。而像文章或者是较长的文字，就用CLOB存储，这样对以后的查询更新存储等操作都提供很大的方便。
## CLOB
数据库中的一种保存文件所使用的类型。全称为：```Character Large Object```。

SQL 类型 CLOB 在 JavaTM 编程语言中的映射关系：
SQL CLOB 是内置类型，它将字符大对象 (Character Large Object) 存储为数据库表某一行中的一个列值。默认情况下，驱动程序使用 SQL locator(CLOB) 实现 Clob 对象，这意味着 CLOB 对象包含一个指向 SQL CLOB 数据的逻辑指针而不是数据本身。Clob 对象在它被创建的事务处理期间有效。
在一些数据库系统里，也使用Text 作为CLOB的别名，比如SQL Server。

## BLOB
BLOB (binary large object)，二进制大对象，是一个可以存储二进制文件的容器。
在计算机中，BLOB常常是数据库中用来存储二进制文件的字段类型。
BLOB是一个大文件，典型的BLOB是一张图片或一个声音文件，由于它们的尺寸，必须使用特殊的方式来处理（例如：上传、下载或者存放到一个数据库）。
根据Eric Raymond的说法，处理BLOB的主要思想就是让文件处理器（如数据库管理器）不去理会文件是什么，而是关心如何去处理它。
但也有专家强调，这种处理大数据对象的方法是把双刃剑，它有可能引发一些问题，如存储的二进制文件过大，会使数据库的性能下降。在数据库中存放体积较大的多媒体对象就是应用程序处理BLOB的典型例子。

| Type | size | Case Sensitive|
| :----: | :----: | :--:|
|```BLOB```|Max:64k|True|
|```TinyBlob```|Max:255K|True|
|```MediumBlob```|Max:16M|True|
|```LongBlob```|Max:4G|True|

## CLOB和BLOB的区别
CLOB使用CHAR来保存数据。 如：保存XML文档。
BLOB就是使用二进制保存数据。 如：保存位图。

## JAVA里对CLOB的操作
在绝大多数情况下，使用2种方法使用CLOB
1. 相对比较小的，可以用```String```进行直接操作，把CLOB看成字符串类型即可
2. 如果比较大，可以用```getAsciiStream```或者```getUnicodeStream```以及对应的```setAsciiStream```和```setUnicodeStream```即可。

### 读取数据(SELECT)
```java
ResultSet rs = stmt.executeQuery("SELECT TOP 1 * FROM Test1");
rs.next();
Reader reader = rs.getCharacterStream(2);
```
### 插入数据(INSERT)
```java
PreparedStatement pstmt = con.prepareStatement("INSERT INTO test1 (c1_id, c2_vcmax) VALUES (?, ?)");
pstmt.setInt(1, 1);
pstmt.setString(2, htmlStr);
pstmt.executeUpdate();
```

### 更新数据(UPDATE)
```java
Statement stmt = con.createStatemet();
ResultSet rs = stmt.executeQuery("SELECT * FROM test1");
rs.next();
Clob clob = rs.getClob(2);
long pos = clob.position("dog", 1);
clob.setString(1, "cat", len, 3);
rs.updateClob(2, clob);
rs.updateRow();
```
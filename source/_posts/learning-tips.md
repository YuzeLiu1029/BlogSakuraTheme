---
title: learning_tips
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-11-14 14:12:34
tags:
    - tips
    - 技术
    - 学习笔记
keywords:
description:
photos: https://static.2heng.xin/wp-content/uploads//2019/02/wallhaven-672007-1-1024x576.png
---
#Java 运行报错解决方法
Error:Maven Resources Compiler: Maven project required for model XXX isn't available. Compilation of Maven projects is supported only if external build is started from an IDE.

解决方法： Build -> rebuild

##Intellij IDEA控制台输出中文乱码解决方案

## JDBC连接ORACLE的两种URL格式
JDBC连接数据库有两种方式，使用sid和servicename的写法略有区别。

* 格式1 ： Oracle JDBC Thin using an SID:
    * ```jdbc:oracle:thin:@host:port:SID```
    
    * ex: ```jdbc:oracle:thin:@localhost:1521:orcl```
* 格式2 ： Oracle JDBC Thin using a ServiceName:
    * ```jdbc:oracle:thin:@//host:port/service_name```
    * ex:```jdbc:oracle:thin:@//localhost:1521/orcl.city.com```
第二种格式是Oracle推荐的格式，因为对于集群来说，每个节点的SID是不一样的，但是Service_name可以包含所有的节点。
# 基于Microsoft SQLServe 2017相关

##SQLserve 2017 Install Guide

### 产品密钥？？？
输入产品密钥 

6GPYM-VHN83-PHDM2-Q9T2R-KBV83

22222-00000-00000-00000-00000

TDKQD-PKV44-PJT4N-TCJG2-3YJ6B
参考网址：https://www.cnblogs.com/Williamls/p/10506243.html

### bin文件的丢失与生成
Oracle_jar_compile 中的bin文件夹会被过滤掉，解决方法：在Oracle_jar_compile文件中新建bin文件夹，右键，将bin文件夹设置成exclude模式。
点击project structure按钮，将output path设置为新建bin文件夹的的绝对路径（D:\DOE.........\bin）.
这样编译的时候会在bin文件夹中自动生成，class文件，再按照文档所叙述后续步骤操作即可



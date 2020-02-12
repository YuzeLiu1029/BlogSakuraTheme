---
title: Modelio_erro_solution
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-01-14 15:35:03
tags:
 - tech
 - software
 - java
keywords:
 - Modelio
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123841.jpg
---
# Modelio error handeling
mac在安装Modelio时会发生出现一些错误。部分错误在官网上有所说明，但有一些错误并没有说明，但在社区中有很多用户提出并讨论出解决方案，在此汇总。
## Java version problem
首先关于java version的问题，Modelio目前只能在jdk1.8环境运行，因此需要修改该安装目录目录下ini文件:```Contents/Eclipse/modelio.ini```。
在```-clearPersistedState```之后添加：
```java
-vm 
/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/bin/java
```
在```-vmargs```后添加：
```java
-Dosgi.requiredJavaVersion=1.8
```
## 运行后提示错误，错误信息保存在log文件中
其中log文件报错类似于：
```java
!ENTRY org.eclipse.equinox.launcher 4 0 2019-03-08 12:46:54.621
!MESSAGE Could not find extension: reference:file:org.eclipse.osgi.compatibility.state_1.1.0.v20170516-1513.jar
!ENTRY org.eclipse.equinox.launcher 4 0 2019-03-08 12:46:54.629
!MESSAGE Exception launching the Eclipse Platform:
!STACK
java.lang.ClassNotFoundException: org.eclipse.core.runtime.adaptor.EclipseStarter
at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
at org.eclipse.equinox.launcher.Main.invokeFramework(Main.java:650)
at org.eclipse.equinox.launcher.Main.basicRun(Main.java:590)
at org.eclipse.equinox.launcher.Main.run(Main.java:1499)
```
1. 删除根目录下隐藏文件夹```.modelio```。
2. 将```modelio.ini```文件中该行：```--add-modules=ALL-SYSTEM```更改为：```--add-modules=java.se.ee```。
3. 注意Modelio文件的安装位置，mac用户不需要安装，通常会把Modelio文件直接放在桌面或Download文件夹中运行，可能会导致错误，将Modelio放到Application中运行可以避免错误发生。


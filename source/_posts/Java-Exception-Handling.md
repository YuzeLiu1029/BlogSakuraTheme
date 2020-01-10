---
title: Java_Exception_Handling
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-12-23 15:40:25
tags: 
    - tips
    - Java
    - 学习笔记
keywords:
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123832.jpg
---
# Java 异常处理

## 定义

* 异常：异常阻止当前方法或作用域继续执行的问题。

* throw：就是自己进行异常处理，throw一旦进入被执行，程序立即会转入异常处理阶段，后面的语句就不再执行， 而且所在的方法不再返回有意义的值！

* throws：用来声明一个方法可能抛出的所有异常信息，将异常声明但是不处理，而是将异常往上传，谁调用我就 交给谁处理。

* try：出现在方法体中，它自身是一个代码块，表示尝试执行代码块的语句。如果在执行过程中有某条语句抛出异 常，那么代码块后面的语句将不被执行。

* catch：出现在try代码块的后面，自身也是一个代码块，用于捕获异常try代码块中可能抛出的异常。catch关键字 后面紧接着它能捕获的异常类型，所有异常类型的子类异常也能被捕获。

* finally：表示不管异常是否发生，都得进行此方法体中处理，关闭输入流输出流。

* throw 与 throws区别：
    * throws 出现在方法函数头，表示该方法可能会抛出异常，允许throws后面跟多个异常类型；而throw出现在函数体，当方法在执行过程中遇到异常情况时，将异常信息封装为异常对象，然后throw。
    * throws表示出现异常的可能性，并不一定会发生异常；throw则是一定抛出了某种异常对象。
* try, catch, finally用法
    * ```try{...}```中放置可能会发生异常的语句块，入可能出现异常的函数，也可以是一般的程序语句；```catch{...}```用于抓住异常，```(Exception e)```中```Exception```是异常的类型，必须是```Exception```(```Exception```是所有异常类的父类)的子类。```{...}```定义当出现异常时的处理方法。```finally{...}```表示不管异常是否发生，都得进行```finally{}```中的处理。
    * 在捕捉异常的```try{...}```语句块中，如果出现了异常，则该语句(出现异常的语句)后的程序语句都不执行，而是跳到```catch{...}```语句块中执行异常的处理。

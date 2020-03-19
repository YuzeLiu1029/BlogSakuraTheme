---
title: Java_multi_thread
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-02-25 13:14:23
tags:
 - tech
keywords:
 - process
 - thread
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/cyber3.jpg
---
# 进程与线程
## 概念
**进程**：代码在数据集上的的一次运行活动，实习用进行资源分配和调度的基本单位。
**线程**：进程的一个执行路径，一个进程至少有一个线程，进程中的多个线程共享进程的资源。
虽然系统把资源分配给进程，但是CPU很特殊，是被分配到线程的，线程是CPU分配的基本单位。
## 二者关系
一个进程有多个线程，多个线程共享进程的堆和方法区资源，每个线程有自己的程序计数器和栈区域。
**程序计数器**：是一块内存区域，用来记录当前要执行的指令地址。
**栈**：用于存储该线程的局部变量，这些局部变量是该线程私有的，除此之外还用来存档线程的调用栈祯。
**堆**：一个进程中最大的一块内存，堆是被进程中所有内存共享的。
**方法区**：用来存放NM加载类，常量及静态变量等信息，也是线程共享。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/jmtpic1.png)
## 二者区别
**进程**：有独立的地址空间，一个进程崩溃后，在保护模式下不会对其他进程造成影响。
**线程**：一个进程中的不同执行路径。线程有自己的堆栈和局部变量，单线程之间没有单独的地址空间。


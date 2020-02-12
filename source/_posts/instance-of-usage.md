---
title: instance_of_usage
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-01-13 13:42:37
tags:
 - java
 - tech 
keywords:
 - java
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123832.jpg
---
# ```instnaceof```用法

java中，instanceof运算符的前一个操作符是一个引用变量，后一个操作数通常是一个类（可以是接口），用于判断前面的对象是否是后面的类，或者其子类、实现类的实例。如果是返回true，否则返回false。
也就是说：
**使用instanceof关键字做判断时， instanceof 操作符的左右操作数必须有继承或实现关系，也就是：```object instanceof constructor```**

```instanceof```严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例，用法为：
```java
boolean result = obj instanceof Class
```
其中 obj 为一个对象，Class 表示一个类或者一个接口，当 obj 为 Class 的对象，或者是其直接或间接子类，或者是其接口的实现类，结果result 都返回 true，否则返回false。
注意：编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。

## obj必须为引用类型，不能是基本类型
```java
int i = 0;
System.out.println(i instanceof Integer);//编译不通过
System.out.println(i instanceof Object);//编译不通过
```
instanceof 运算符只能用作对象的判断。

## obj为null
```java
System.out.println(null instanceof Object);//false
```
一般我们知道Java分为两种数据类型，一种是基本数据类型，有八个分别是 ```byte```,```short```,```int```,```long```,```float```,```double```,```char```,```boolean```,一种是引用类型，包括类，接口，数组等等。而Java中还有一种特殊的```null```类型，该类型没有名字，所以不可能声明为null类型的变量或者转换为```null```类型,```null```引用是```null```类型表达式唯一可能的值,```null```引用也可以转换为任意引用类型。我们不需要对```null```类型有多深刻的了解，我们只需要知道```null```是可以成为任意引用类型的特殊符号。
在 JavaSE规范 中对```instanceof```运算符的规定就是：如果```obj```为```null```，那么将返回```false```。
## obj为class类的实例对象
```java
Integer integer = new Integer(1);
System.out.println(integer instanceof  Integer);//true
```
这没什么好说的，最普遍的一种用法。
## obj为class接口的实现类
了解Java集合的，我们知道集合中有个上层接口```List```，其有个典型实现类 ```ArrayList```。
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```
所以我们可以用```instanceof```运算符判断 某个对象是否是```List```接口的实现类，如果是返回```true```，否则返回```false```。
```java
ArrayList arrayList = new ArrayList();
System.out.println(arrayList instanceof List);//true
//反过来也成立
List list = new ArrayList();
System.out.println(list instanceof ArrayList);//true
```
##obj为class类的直接或间接子类
我们新建一个父类```Person.class```，然后在创建它的一个子类```Man.class```。
```java
public class Person {
 
}
```
```java
Man.class
public class Man extends Person{
     
}
```
```java
Person p1 = new Person();
Person p2 = new Man();
Man m1 = new Man();
System.out.println(p1 instanceof Man);//false
System.out.println(p2 instanceof Man);//true
System.out.println(m1 instanceof Man);//true
```




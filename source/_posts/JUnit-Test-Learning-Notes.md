---
title: JUnit_Test_Learning_Notes
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2019-12-28 15:42:29
tags:
    - Java
    - Tech
keywords:
    - JUnit_Test
    - Java
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/cyber4.jpg
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
# JUnit Test Learning Notes
## What is JUnit Test
JUnit是Java编程语言的单元测试框架，用于编写和可重复运行的自动化测试。

## Features
* JUnit 是一个开放的资源框架，用于编写和运行测试。
* 提供注解来识别测试方法。
* 提供断言来测试预期结果。
* JUnit 测试允许你编写代码更快，并能提高质量。
* JUnit 优雅简洁。没那么复杂，花费时间较少。
* JUnit测试可以自动运行并且检查自身结果并提供即时反馈。所以也没有必要人工梳理测试结果的报告。
* JUnit测试可以被组织为测试套件，包含测试用例，甚至其他的测试套件。
* JUnit在一个条中显示进度。如果运行良好则是绿色；如果运行失败，则变成红色。

## JUnit Annotation

|Annotation|Description|
|:----------:|:-------------:|
|``@Test``|测试注解，标记一个方法可以作为一个测试用例|
|``@Before``|Before注解表示，该方法必须在类中的每个测试之前执行,以便执行某些必要的先决条件|
|``@BeforeClass``|BeforeClass注解指出这是附着在静态方法必须执行一次并在类的所有测试之前，这种情况一般用于测试计算、共享配制方法(如数据库连接)|
|``@After``|After注释表示，该方法在每项测试后执行（如执行每一个测试后重置某些变量，删除临时变量等)|
|``@AfterClass``|当需要执行所有测试在JUnit测试用例类后执行，AlterClass注解可以使用以清理一些资源（如数据库连接），注意：方法必须为静态方法|
|``@Ignore``|当想暂时禁用特定的测试执行可以使用这个注解，每个被注解为@Ignore的方法将不再执行|
|``@Runwith``|@Runwith就是放在测试类名之前，用来确定这个类怎么运行的。也可以不标注，会使用默认运行器|
|``@Parameters``|	用于使用参数化功能|
|``@SuitClasses``|用于套件测试|

## JUnit Assertion

|Assertion|Description|
|----------|-------------|
|```void assertEquals([String message],expected value,actual value)```|断言两个值相等。值类型可能是```int```，```short```，```long```，```byte```，```char```，```Object```，第一个参数是一个可选字符串消息|
|```void assertTrue([String message],boolean condition)```|	断言一个条件为真|
|```void assertFalse([String message],boolean condition)```|	断言一个条件为假|
|```void assertNotNull([String message],java.lang.Object object)```|	断言一个对象不为空(null)|
|```void assertNull([String message],java.lang.Object object)```	|断言一个对象为空(null)|
|```void assertSame([String message],java.lang.Object expected,java.lang.Object actual)```|	断言两个对象引用相同的对象|
|```void assertNotSame([String message],java.lang.Object unexpected,java.lang.Object actual)```|	断言两个对象不是引用同一个对象|
|```void assertArrayEquals([String message],expectedArray,resultArray)```|	断言预期数组和结果数组相等，数组类型可能是```int```，```short```，```long```，```byte```，```char```，```Object```|

```java
public class AssertionTest {
 
    @Test
    public void test() {
        String obj1 = "junit";
        String obj2 = "junit";
        String obj3 = "test";
        String obj4 = "test";
        String obj5 = null;
        
        int var1 = 1;
        int var2 = 2;
 
        int[] array1 = {1, 2, 3};
        int[] array2 = {1, 2, 3};
 
        Assert.assertEquals(obj1, obj2);
 
        Assert.assertSame(obj3, obj4);
        Assert.assertNotSame(obj2, obj4);
        
        Assert.assertNotNull(obj1);
        Assert.assertNull(obj5);
 
        Assert.assertTrue(var1 < var2);
        Assert.assertFalse(var1 > var2);
 
        Assert.assertArrayEquals(array1, array2);
 
    }
}
```
## JUnit执行过程

```java
public class JunitTest {
 
    @BeforeClass
    public static void beforeClass() {
        System.out.println("in before class");
    }
 
    @AfterClass
    public static void afterClass() {
        System.out.println("in after class");
    }
 
    @Before
    public void before() {
        System.out.println("in before");
    }
 
    @After
    public void after() {
        System.out.println("in after");
    }
 
    @Test
    public void testCase1() {
        System.out.println("in test case 1");
    }
 
    @Test
    public void testCase2() {
        System.out.println("in test case 2");
    }
 
}
```

Running Result:

```java
in before class
in before
in test case 1
in after
in before
in test case 2
in after
in after class
```
## Ignore Test
* 一个带有@Ignore注解的测试方法不会被执行
* 如果一个测试类带有@Ignore注解，则它的测试方法将不会被执行

## Time Test
JUnit提供了一个暂停的方便选项，如果一个测试用例比起指定的毫秒数花费了更多的时间，那么JUnit将自动将它标记为失败，timeout参数和```@Test```注解一起使用，例如```@Test(timeout=1000)```。

```java
@Test(timeout = 1000)
    public void testCase1() throws InterruptedException {
        TimeUnit.SECONDS.sleep(5000);
        System.out.println("in test case 1");
    }
```
## Exception Test
Junit 用代码处理提供了一个追踪异常的选项。你可以测试代码是否它抛出了想要得到的异常。```expected``` 参数和 ```@Test``` 注释一起使用。

```java
@Test(expected = ArithmeticException.class)
    public void testCase3() {
        System.out.println("in test case 3");
        int a = 0;
        int b = 1 / a;
    }
```
## Parameterized Test
Junit 4 引入了一个新的功能参数化测试。参数化测试允许开发人员使用不同的值反复运行同 一个测试。你将遵循 5 个步骤来创建参数化测试：


* 为准备使用参数化测试的测试类指定特殊的运行器 ```org.junit.runners.Parameterized```。
* 为测试类声明几个变量，分别用于存放期望值和测试所用数据。
* 为测试类声明一个带有参数的公共构造函数，并在其中为第二个环节中声明的几个变量赋值。
* 为测试类声明一个使用注解 ```org.junit.runners.Parameterized.Parameters``` 修饰的，返回值为 ```java.util.Collection``` 的公共静态方法，并在此方法中初始化所有需要测试的参数对。
* 编写测试方法，使用定义的变量作为参数进行测试。

## Test Suite
“套件测试”是指捆绑了几个单元测试用例并运行起来。在```JUnit```中，```@RunWith``` 和 ```@Suite``` 这两个注解是用来运行套件测试。先来创建几个测试类.

```java

public class JunitTest1 { 
    @Test
    public void printMessage(){
        System.out.println("in JunitTest1");
    }
}
public class JunitTest2 { 
    @Test
    public void printMessage(){
        System.out.println("in JunitTest2");
    }
}

@RunWith(Suite.class)
@Suite.SuiteClasses({
        /**
         * 此处类的配置顺序会影响执行顺序
         */
        JunitTest1.class,
        JunitTest2.class
})
public class JunitSuite {
 
}
```
### 关于```@RunWith```
首先要分清几个概念：测试方法、测试类、测试集、测试运行器。

* 其中测试方法就是用@Test注解的一些函数。
* 测试类是包含一个或多个测试方法的一个```**Test.java```文件.
* 测试集是一个```suite```，可能包含多个测试类。
* 测试运行器则决定了用什么方式偏好去运行这些测试集/类/方法。

而```@Runwith```就是放在测试类名之前，用来确定这个类怎么运行的。也可以不标注，会使用默认运行器。常见的运行器有：

* ```@RunWith(Parameterized.class)``` 参数化运行器，配合```@Parameters```使用```JUnit```的参数化功能
* ```@RunWith(Suite.class)```,```@SuiteClasses({ATest.class,BTest.class,CTest.class})``` 测试集运行器配合使用测试集功能。
* ```@RunWith(JUnit4.class)```， junit4的默认运行器
* ```@RunWith(JUnit38ClassRunner.class)```，用于兼容junit3.8的运行器,一些其它运行器具备更多功能。例如```@RunWith(SpringJUnit4ClassRunner.class)```集成了spring的一些功能。




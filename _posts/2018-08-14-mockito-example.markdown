---
layout: post
title: mockito example
date: 2018-08-14
categories: mockito
---

## 关于Mockito的例子，翻译自官网：[javaDoc](http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/Mockito.html)

### mock
下面这个例子mock一个List对象作为演示，实际上，请不要mock一个List,尽量使用一个真实的List对象来完成我们的测试用例 。

```java
//为了使代码更加整洁可读，把Mockito类静态方法都引入来。
import static org.mockito.Mockito.*;

//生成一个mock实例
List mockedList = mock(List.class);

//使用mock实例
mockedList.add("one");
mockedList.clear();

//验证
verify(mockedList).add("one");
verify(mockedList).clear();

```

一旦创建一个mock实例，这个mock实例会记录所有与外部对象之间的交互过程，包括：方法调用、传入参数、调用次数等等。在这些基础上，你可以选择验证你所期望的交互。

### stubbing

```java
//类和接口都可以mock
LinkedList mockedList = mock(LindList.class);

//stubbing
when(mockedList.get(0).thenReturn("first");
when(mockedList.get(1).thenReturn(new RuntimeException());

//打印查看结果:"first"
System.out.println(mockedList.get(0));
//下面语句会抛出一个runtime excpetion
System.out.println(mockedList.get(1));

//下面语句会打印"null",因为我们没有为get(999)提供stub
System.out.println(mockedList.get(999));

//通常不需要验证一个stubbed的调用
//stubbed的前提是，你的测试代码需要关注并基于get(0)的返回值继续运行，如果你的代码不需要关注get(0)的返回值，则不需要stubbed
verify(mockedList).get(0);

```

1. 默认情况下，所有mock实例的方法都会有一个默认的返回值:null、基本类型/基本包装类型、一个空的集合。mockito自动判断什么情况下使用哪种返回值，比如：int/Integer 返回0 ; boolean/Boolean 返回false
2. stubbing是可以被覆盖的，例如，你可以把common的stubbing代码抽出来统一初始化，在具体某个test case再覆盖某个stubbing。但是需要注意的是：这些common的stubbing代码被覆盖太多的话，有可能是潜在的代码坏味道。
3. 一旦对某个mock方法stubbed后，那么这个方法调用总是返回一个stubbed值，不管调用多少次，都会返回固定的stubbed值。
4. 对同一个方法，用相同的参数stubbing，以最后一次以准。

### Argument matchers

当我们对某个方法构建stubbing时，需要指定传入参数来定义来返回特定stubbed对象 , 当test case调用stubbing方法时，mockito据实际传入的传参数来判断需要返回哪个stubbed对象。 mockito使用java内置的equals()方法来对比参数。 但是，有时我们需要再加灵活的参数匹配方式。

```java
//stubbing使用内置的anyInt()参数匹配器
when(mockedList.get(andyInt())).thenReturn("object");


```


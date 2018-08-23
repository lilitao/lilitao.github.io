---

layout: post
title: mockito example
date: 2018-08-14
categories: mockito
author: AndyLi
description: 关于Mockito的常见的使用例子，翻译自官网javadoc

---

## 关于Mockito的例子，翻译自官网：[javaDoc](http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/Mockito.html)

### 1. mock
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

### 2. stubbing

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

### 3. Argument matchers

当我们对某个方法构建stubbing时，需要指定传入参数来定义来返回特定stubbed对象 , 当test case调用stubbing方法时，mockito据实际传入的传参数来判断需要返回哪个stubbed对象。 mockito使用java内置的equals()方法来对比参数。 但是，有时我们需要再加灵活的参数匹配方式。

```java
//stubbing使用内置的anyInt()参数匹配器
when(mockedList.get(andyInt())).thenReturn("object");

//stubbing时使用自定义的参数匹配器,假设 isValid()方法返回你定义的org.hamcrest.Matcher实现
when(mockedList.contains(argThat(isValid()))).thenRenturn("element);

//下面代码将会打印 "object"
System.out.pringln(mockedList.get(999));

//使用参argument matcher来验证
verify(mockedList).get(anyInt());
//使用java 8 lambdas写argument matchers
verify(mockedList).add(argThat(someString -> someString.length() > 5));

```

参数匹配（argument matchers）提供更加灵活的验证方式和stubbing . [点这里](http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/ArgumentMatchers.html) 查看关于客制化Argument matchers，或者 [这里](http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/hamcrest/MockitoHamcrest.html) hamcrest matchers的例子，

更多关于如何实现自定义的argument matchers,查看 java doc [ArgumentMatcher](http://static.javadoc.io/org.mockito/mockito-core/2.21.0/org/mockito/ArgumentMatcher.html)

要适度，合理地使用复杂的argument matchers；尽量常用equals()，或者偶尔使用anyX() matchers 以便于编写简单，干净，便以理解的测试代码。如果可以，尽量重构那些使用复杂argument matchers的代码，尽量应用equals()方法，甚至实现自定义的equals()方法来完成测试代码。

* 使用argument matchers需要注意的地方:
  * 如果某个方法调用使用了argument matchers,那么其他使用调用这个方法的地方的参数也要由matchrees来提供
  * 下面的例子演示了在verification时使用了两个不同的参数提供方式的错误用法。

```java
 verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
   //above is correct - eq() is also an argument matcher

   verify(mock).someMethod(anyInt(), anyString(), "third argument");
   //above is incorrect - exception will be thrown because third argument is given without an argument matcher.

```
anyObject(),eq()这两个匹配方法是不会返回一个matchers对象的，他们只是在内部的stack里记一个matcher并且返回一个dummy value(通常是一个null),这样实现的原因是为了应对java编绎器的静态类型安全检查，所以这样的话，我们不能在verified/stubbed方法之外使用anyObject(),eq()两个方法。

### 4. 清确地校验方法调用的次数，或至少多少次，或者一次也没有调用

```java
//使用mock
mockedList.add("once");

mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

//下面验证调用1次, 默认使用time(1)验证调用一次
verify(mockedList).add("once");
verify(mockedList,times(1)).add("once");

//精确验证调用次数
verify(mockedList,times(2)),add("twice");
verify(mockedList,times(3)).add("three times");

//校验一次都没有调用,never()相当于times(0)
verify(mockedList,never()),add("never happened");

//校验调用次数的边界值，atLeast()或者atMost()
verify(mockedList,atLeastOnce()).add("three times");
verify(mockedList,atLeast(2)).add("three times");
verify(mockedList,atMost(5)).add("three times");
```

默认使用times(1)来verify，因此不需要显示使用times(1)

### 5. 通过exceptons来生成没有返回值的方法的stubbing

```java
doThrow(new RuntimeException()).when(mockedList).clear();
//下面的代码执行会抛出异常
mockedList.clear();
```

第12节是关于doThrow()|doAnswer()等一类方法的详细介绍

### 6. 校验顺序
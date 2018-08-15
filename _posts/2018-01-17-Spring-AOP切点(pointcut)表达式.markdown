---
layout: post
title: Spring AOP切点(pointcut)表达式
date: 2018-04-17 23:55:21 +0800
categories: translate
---

原文地址：http://www.baeldung.com/spring-aop-pointcut-tutorial

[上一篇]({{ site.baseurl }}{% post_url 2018-01-16-Spring-AOP通知类型介绍 %})

#概括

这遍文章将介绍Spring AOP切点表达式(下称表达式)语言，首先介绍两个面向切面编程中使用到的术语。

1.连接点(Joint Point)：广义上来讲，方法、异常处理块、字段这些程序调用过程中可以抽像成一个执行步骤（或者说执行点）的单元。从Spring AOP来讲，就是指java的方法和异常处理代码块。

2.切点(Pointcut)：是连接点的描述定义，Spring AOP通过切点来定位到哪些连接点。切点表达式语言就是切点用来定义连接点的语法。

#用例

表达式会出现在以下几种场景
*作为@Pointcut的参数，用以定义连接点

{% highlight java %}

	@Pointcut("within(@org.springframework.stereotype.Repository *)")
	public void repositoryClassMethods() {}

{% endhighlight %}

在上面的代码片段中的注解@Pointcut的参数"within(@org.springframework.stereotype.Reposity *)"就是使用的切点表达式。而上代码中的repositoryClassMethods()方法被AOP AspectJ定义为切点签名方法，作用是使得通知的注解可以通过这个切点签名方法连接到切点，通过解释切点表达式找到需要被切入的连接点。最终的目的都是为了找到需要被切入的连接点。像下面这段代码片段

{% highlight java %}

	@Around("repositoryClassMethods()")
	public Object measureMethodExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
	    ...
	}

{% endhighlight %}

*如果你的项目是基于xml配置的，可以在<aop:pointcut>标签里配置表达式来定位连接点，参考以下代码片段

{% highlight xml %}

	<aop:config>
	    <aop:pointcut id="anyDaoMethod"
	      expression="@target(org.springframework.stereotype.Repository)"/>
	</aop:config>

{% endhighlight %}

#切点指示符

切点指示符是切点定义的关键字，切点表达式以切点指示符开始。开发人员使切点指示符来告诉切点将要匹配什么，有以下9种切点指示符：execution、within、this、target、args、@target、@args、@within、@annotation，下面一一介结这9种切点指示符。

#execution

execution是一种使用频率比较高比较主要的一种切点指示符，用来匹配方法签名，方法签名使用全限定名，包括访问修饰符（public/private/protected）、返回类型，包名、类名、方法名、参数，其中返回类型，包名，类名，方法，参数是必须的，如下面代码片段所示：
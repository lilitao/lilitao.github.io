---
layout: post
title:  "Spring AOP通知类型介绍"
date:   2018-04-15 11:46:21 +0800
categories: translate
---
原文地址：http://www.baeldung.com/spring-aop-advice-tutorial

# 概述 #
这遍文章将要讨论Spring AOP中使用到的各种通知类型，
通知是切面的具体逻辑实现，由切面负责执行，具体的某个通知对应具体的某些由切点描述的连接点。通知类型包括：前置、后置、环绕。切面即在由对象构建成的分层结构中横向关注点的抽像，比如：日志、系统配置、缓存、数据库事务管理等。
如果你想深入了解切点表达式，请查看我的上一篇翻译文章：

启用通知
在Spring里，在使用AspectJ提供的注解声明通知之前，首先在Spring的配置类里声明@EnableAspectJAutoProxy 启用AspectJ组件，之后，Spring AOP将扫描和管理被AspectJ的@Aspect注解过的切面bean。

{% highlight java %}

    @Configuration
    @ComponentScan(basePackages = {"org.baeldung.dao", "org.baeldung.aop"})
    @EnableAspectJAutoProxy
    public class TestConfig {
    ...
    }

{% endhighlight %}

# 前置通知 #
顾明思义，前置通知在连接点被执行前就被执行了，除非抛出异常，否则前置通知不能够阻止方法的继续执行。
请看下面的代码片段，这个切面简单地打印了方法名称，在连接点执行之前。

{% highlight java %}

    @Component
    @Aspect
    public class LoggingAspect {
    
	    private Logger logger = Logger.getLogger(LoggingAspect.class.getName());
	    
	    @Pointcut("@target(org.springframework.stereotype.Repository)")
	    public void repositoryMethods() {};
	    
	    @Before("repositoryMethods()")
	    public void logMethodCall(JoinPoint jp) {
	    String methodName = jp.getSignature().getName();
	    logger.info("Before " + methodName);
	    }
    }

{% endhighlight %}

在切点表达式repositoryMethods匹配的repository method连接点执行之前，前置通知logMethodCall方法将会首先被执行。

# 后置通知 #
使用@After注解来声明后置通知。在被匹配的连接点执行完后，执行后置通知，不管连接点执行是否有抛出异常.从另一个角度来看，后置通知跟finally语句块相似。如果你只是想在连接点方法正常返回时执行通知(异常时不执行通知)，应该使用@AfterReturning注解声明的返回型通知。如果你只是想在连接点方法抛出异常后执行通知（正常返回时不执行），应该使用@AfterThrowing注解声明的异常通知。

假设当一个新的Foo的实例被创建时，我们希望触发一个通知事件，通知监控的组件，我们可以从FooDao发布一个事件，但是这会破坏对象的单一责任原则，这时面向切面遍程可以提供另一种方式，我们可以通过定义下面的这样一个切面来达到这种效果。

{% highlight java %}

    @Component
    @Aspect
    public class PublishingAspect {
    
	    private ApplicationEventPublisher eventPublisher;
	    
	    @Autowired
	    public void setEventPublisher(ApplicationEventPublisher eventPublisher) {
	   		 this.eventPublisher = eventPublisher;
	    }
	    
	    @Pointcut("@target(org.springframework.stereotype.Repository)")
	    public void repositoryMethods() {}
	    
	    @Pointcut("execution(* *..create*(Long,..))")
	    public void firstLongParamMethods() {}
	    
	    @Pointcut("repositoryMethods() && firstLongParamMethods()")
	    public void entityCreationMethods() {}
	    
	    @AfterReturning(value = "entityCreationMethods()", returning = "entity")
	    public void logMethodCall(JoinPoint jp, Object entity) throws Throwable {
	    	eventPublisher.publishEvent(new FooCreationEvent(entity));
	    }
    }

{% endhighlight %}

请注意：首先，@AfterReturning注解，我们可以访问方法的返回值。其实，通过声明JoinPoint参数，我们可以使用目标方法调用的参数。

下一步，我们定义一个监控器来简单地记录事件，更多关于事件的文档，请查看：http://www.baeldung.com/spring-events

{% highlight java %}

    @Component
    public class FooCreationEventListener implements ApplicationListener<FooCreationEvent> {
    
	    private Logger logger = Logger.getLogger(getClass().getName());
	    
	    @Override
	    public void onApplicationEvent(FooCreationEvent event) {
	    	logger.info("Created foo instance: " + event.getSource().toString());
	    }
    }

{% endhighlight %}

# 环绕通知 #
环绕通知包围着连接点，例如在目标方法调用前后执行切面逻辑。

环绕通知是最强大的一种通知，在目标方法执行的前后可以执行用户自定义的切面逻辑，也可以在环绕通知里决定是否继续执行连接点方法或者跳过连接点直接在环绕通里返回值或者抛出异常。

为了演示环绕通知的使用例子，现在假如你想要测量连接点方法的执行时间，为了达到这样的目的，你可以建立以下这个的切面：

{% highlight java %}

    @Aspect
    @Component
    public class PerformanceAspect {
    
	    private Logger logger = Logger.getLogger(getClass().getName());
	    
	    @Pointcut("within(@org.springframework.stereotype.Repository *)")
	    public void repositoryClassMethods() {};
	    
	    @Around("repositoryClassMethods()")
	    public Object measureMethodExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
		    long start = System.nanoTime();
		    Object retval = pjp.proceed();
		    long end = System.nanoTime();
		    String methodName = pjp.getSignature().getName();
		    logger.info("Execution of " + methodName + " took " + 
		      TimeUnit.NANOSECONDS.toMillis(end - start) + " ms");
		    return retval;
	    }
    }

{% endhighlight %}

当通过repositoryClassMethods 这个切点匹配的所有连接点被执行时，上面切面所示的通知就会被触发。这个方法接收一个ProceedingJointPoint类型的参数，我们可以在使用这个参数执行目标方法之前，执行我们的切面逻辑。在上面的例子里，我们只时简单地保存了目标方法调用的开始时间。其次，由于目标方法可以返回任意类型的对象，所以这个通知的返回值类型是Object，有一点需要注意的是，如果目标方法返回类型是void,通知的必须返回一个null。当目标方法调用完成后，我们可以计算调用时间并打印，然后把返回值返回给调用者。

总结
在这篇文单里，我们学习了Spring中的不同类型的通知和这些通知的声明和实现方式。我们使用基于模式定义的方法和java注解来定义切面。我们提供了几种合适的通知应用场景。

上面所有的例子实现和代码可以在原作者的github里找到：https://github.com/eugenp/tutorials/tree/master/spring-mvc-java ，那是基于eclipse的工程结构，所以很容易导入和运行。 
 

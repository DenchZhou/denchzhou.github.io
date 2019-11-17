---
layout: post
title: Spring之面向切面AOP
categories: Spring
description: Spring之面向切面AOP
keywords: SpringAop Spring AOP
---

### AOP概念
AOP的定义：AOP，Aspect Oriented Programming的缩写，意为面向切面编程，是通过预编译或运行期动态代理实现程序功能处理的统一维护的一种技术。

系统中往往存在非常多的Controller匹配各种请求。现在，如果我们想在每一个Controller的RequestMapping请求识别到的时候做个记录。正常的处理逻辑是在每个Request请求到的时候写个Log。然后执行相应的方法。但是这样的话要把所有的RequestMapping方法都加上这个Log，显然非常麻烦。面向切面就是希望只写一遍Log就让所有的请求都能够执行。横向的将代码嵌入到各个请求中。如下图那样，并不影响原本程序的原有逻辑。    


![image.png](https://upload-images.jianshu.io/upload_images/14607771-fe338f4044edce65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来看一下几个名词

切面（Aspect）：横切关注点可以被模块化为特殊的类被称为切面（aspect）。

通知（Advice）：通知定义了切面是什么以及合适使用。通常有五种类型：
- 前置通知（Before）：在目标方法被调用前调用通知功能；
- 后置通知（After）：在目标方法完成之后调用通知
- 返回通知（After-returning）：在目标方法成功执行后调用通知；
- 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
- 环绕通知（Around）：通知包裹了被通知的方法，在通知的方法调用之前和调用之后执行自定义的行为。

连接点（Join point）：通知执行的时机。例如高速路上有非常多的出口，这些出口都相当于连接点。

切入点（Poincut）：通知所要织入的具体位置。还是以高速路为例子，众多出口（连接点）中，我们只需要找到一个出口，这个出口就是切入点。

引入（Introduction）：允许我们向现有的类添加新方法和属性。

织入（Weaving）：把切面用到目标对象并创建新的代理对象的过程。

下图为《Spring实战》中关于各个名词结合的图。

![image.png](https://upload-images.jianshu.io/upload_images/14607771-b9470bae340f3baa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Spring对AOP的支持
- 基于代理的Spring AOP
- 纯POJO切面
- @AspectJ注解驱动的切面
- 注入式AspectJ切面（适用于Spring各版本）

注：Spring只支持方法级别的连接点

#### 通过切点选择连接点
Spring支持Aspectj的指示器中只有execution指示器是实际执行匹配的。
首先，我们有一个发送短信的接口
```
public interface MessageService {

    void sendMessage(String msg);

}
```
我们希望在MessageService进行sendService方法时候触发通知。就有了如下的表达式。
```
execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))
```
* 表示可以返回任意类型，接下来是全限定类名，方法。方法中的参数为 .. 表明可以使用任意参数。

#### 使用注解创建切面
我们定义一个切面记录方法的开始执行的时间与执行结束的时间，以及失败的时间，此外还有@AfterRetrun和@Around
```
@Aspect
public class TimeLogging {

    @Before("execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))")
    public void recordBeforeExecute() {
        System.out.println("start to send message at " + LocalDateTime.now());
    }

    @After("execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))")
    public void recordAfterExecute() {
        System.out.println("message has sent seccess at " + LocalDateTime.now());
    }

    @AfterThrowing("execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))")
    public void recordFailure() {
        System.out.println("fail to send message at " + LocalDateTime.now());
    }

}
```
显然，上面同一个切入点表达式写了三遍非常难看。因此，可以使用@Pointcut注解定义命名的切点
```
@Aspect
public class TimeLogging {

    // 定义命名的切点
    @Pointcut("execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))")
    public void sendMessage() {}

    @Before("sendMessage()")
    public void recordBeforeExecute() {
        System.out.println("start to send message at " + LocalDateTime.now());
    }

    @After("sendMessage()")
    public void recordAfterExecute() {
        System.out.println("message has sent seccess at " + LocalDateTime.now());
    }

    @AfterThrowing("sendMessage()")
    public void recordFailure() {
        System.out.println("fail to send message at " + LocalDateTime.now());
    }

}
```
接下来需要配置Bean及启用AspectJ注解的自动代理，JavaConfig可以使用@EnableAspectJ-AutoProxy注解启用；Xml可以使用Spring aop命名空间中的<aop:aspectj-autoproxy>元素。XML配置如下
```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="messageService" class="com.dqzhou.spring.service.impl.MessageServiceImpl"/>
    <bean id="timeLogging" class="com.dqzhou.spring.aspectj.TimeLogging"/>

    <aop:aspectj-autoproxy/>
</beans>
```
最后，测试一下调用messageService的sendMessage方法
```
public class SpringApplicationContext {
    
    public static void main(String[] args) {
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
        MessageService messageService = (MessageService) applicationContext.getBean("messageService");
        messageService.sendMessage("hello world!");
    }

}
```
结果如下，Spring通过代理的方式增强方法
```
start to send message at 2019-10-28T00:16:11.746
hello world!
message has sent seccess at 2019-10-28T00:16:11.748
```
##### 创建环绕通知
环绕通知相当于在通知方法中同时编写前置和后置通知。    
更改切面类如下，ProceedingJoinPoint作为参数。通过它能够调用被通知的方法。显然，环绕通知可以轻易实现其它几个注解所实现的功能，但是如果proceed()方法没有调用，被通知的方法将无法访问。不过可以通过多次调用达到重试场景。
```
@Aspect
public class TimeLogging {

    // 定义命名的切点
    @Pointcut("execution(* com.dqzhou.spring.service.MessageService.sendMessage(..))")
    public void sendMessage() {}

    @Around("sendMessage()")
    public void watchMessage(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("start to send message at " + LocalDateTime.now());
            joinPoint.proceed();
            System.out.println("message has sent seccess at " + LocalDateTime.now());
        } catch (Throwable throwable) {
            System.out.println("fail to send message at " + LocalDateTime.now());
        }
    }

}
```
##### 通过切面引入新功能
之前，Spring AOP通过代理已经能够对被通知的方法进行增强。那么，有没有可能在不破坏，没有嵌入原有类的结构上增加新的方法。
@DeclareParents注解，做个标记，暂不展开

#### 使用XML声明切面
Spring的aop命名空间
- <aop:advisor>：定义AOP通知器
- <aop:after>：定义AOP后置通知（不管被通知的方法是否执行成功）
- <aop:after-returning>
- <aop:after-throwing>
- <aop:around>
- <aop:aspect>：定义一个切面
- <aop:aspectj-autoproxy>：启用@AspectJ注解驱动的切面
- <aop:before>
- <aop:config>：顶层的AOP配置元素。大多数的<aop:*>元素必须包含
在<aop:config>元素内
- <aop:declareparents>：以透明的方式为被通知的对象引入额外的接口
- <aop:pointcut>：定义一个切点

创建一个切面类，不加任何注解
```
public class LogAspect {
    
    public void logBeforeExecute() {
        System.out.println("method ready to execute at" + LocalDateTime.now());
    }

    public void logAfterExecute() {
        System.out.println("method has executed at" + LocalDateTime.now());
    }

    public void logFailure() {
        System.out.println("execute fail at " + LocalDateTime.now());
    }
    
    public void logAroundMessage(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("method ready to execute at" + LocalDateTime.now());
            joinPoint.proceed();
            System.out.println("method has executed at" + LocalDateTime.now());
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }
}
```
配置xml
```
    <bean id="logAspect" class="com.dqzhou.spring.aop.aspectj.LogAspect"/>
    <aop:config>
        <aop:aspect ref="logAspect">
            <aop:before pointcut="execution(* com.dqzhou.spring.aop.service.MessageService.sendMessage(..))" method="logBeforeExecute"/>
            <aop:after pointcut="execution(* com.dqzhou.spring.aop.service.MessageService.sendMessage(..))" method="logAfterExecute"/>
        </aop:aspect>
    </aop:config>
```
大多数aop配置必须在<aop:config>元素的上下文使用。然后<aop:config>可以声明多个通知。同样的，写多遍切点表达式很麻烦，可以使用<aop:pointcut>指定切点，然后通知通过pointcut-ref属性来引用命名切点，如下：
```
    <aop:config>
        <aop:aspect ref="logAspect">
            <aop:pointcut expression="execution(* com.dqzhou.spring.aop.service.MessageService.sendMessage(..))" id="logMessage"/>
            <aop:before pointcut-ref="logMessage" method="logBeforeExecute"/>
            <aop:after pointcut-ref="logMessage" method="logAfterExecute"/>
        </aop:aspect>
    </aop:config>
```
如果使用环绕通知，xml配置如下：
```
    <aop:config>
        <aop:aspect ref="logAspect">
            <aop:pointcut expression="execution(* com.dqzhou.spring.aop.service.MessageService.sendMessage(..))" id="logMessage"/>
            <aop:around pointcut-ref="logMessage" method="logAroundMessage"/>
        </aop:aspect>
    </aop:config>
```

#### 总结

面向对象编程通过AOP把应用各处的行为放入可重用的模块中。减少了代码的冗余。Spring提供了AOP的框架，可以通过XML或注解的方式快速进行配置。但是如果SpringAOP不能满足需求的时候，需要专项更为强大的AspectJ。
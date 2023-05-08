# spring全家桶

## spring  、  mvc、springboot、springcloud区别联系

Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架。Spring使你能够编写更干净、更可管理、并且更易于测试的代码。

Spring MVC是Spring的一个模块，一个web框架。通过Dispatcher Servlet, ModelAndView 和 View Resolver，开发web应用变得很容易。主要针对的是网站应用程序或者服务开发——URL路由、Session、模板引擎、静态Web资源等等。

Spring配置复杂，繁琐，所以推出了Spring boot，约定优于配置，简化了spring的配置流程。

Spring Cloud构建于Spring Boot之上，是一个关注全局的服务治理框架。

**Spring VS SpringMVC：**

Spring是一个一站式的轻量级的java开发框架，核心是控制反转（IOC）和面向切面（AOP），针对于开发的WEB层(springMvc)、业务层(Ioc)、持久层(jdbcTemplate)等都提供了多种配置解决方案；

SpringMVC是Spring基础之上的一个MVC框架，主要处理web开发的路径映射和视图渲染，属于Spring框架中WEB层开发的一部分；

**SpringMVC VS SpringBoot：**

SpringMVC属于一个企业WEB开发的MVC框架，涵盖面包括前端视图开发、文件配置、后台接口逻辑开发等，XML、config等配置相对比较繁琐复杂；

SpringBoot框架相对于SpringMVC框架来说，更专注于开发微服务后台接口，不开发前端视图；

**SpringBoot和SpringCloud：**

SpringBoot使用了默认大于配置的理念，集成了快速开发的Spring多个插件，同时自动过滤不需要配置的多余的插件，简化了项目的开发配置流程，一定程度上取消xml配置，是一套快速配置开发的脚手架，能快速开发单个微服务；

SpringCloud大部分的功能插件都是基于SpringBoot去实现的，SpringCloud关注于全局的微服务整合和管理，将多个SpringBoot单体微服务进行整合以及管理；SpringCloud依赖于SpringBoot开发，而SpringBoot可以独立开发；

- Spring是核心，提供了基础功能；
- Spring MVC 是基于Spring的一个 MVC 框架 ；
- Spring Boot 是为简化Spring配置的快速开发整合包；
- Spring Cloud是构建在Spring Boot之上的服务治理框架。

## Spring中Bean的生命周期

> 回答参考模板 

1、实例化一个Bean－－也就是我们常说的new；

2、按照Spring上下文对实例化的Bean进行配置－－也就是IOC注入；

3、如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，此处传递的就是Spring配置文件中Bean的id值

4、如果这个Bean已经实现了BeanFactoryAware接口，会调用它实现的setBeanFactory(setBeanFactory(BeanFactory)传递的是Spring工厂自身（可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以）；

5、如果这个Bean已经实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文（同样这个方式也可以实现步骤4的内容，但比4更好，因为ApplicationContext是BeanFactory的子接口，有更多的实现方法）；

6、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术；

7、如果Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。

8、如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法、；

> 注：以上工作完成以后就可以应用这个Bean了，那这个Bean是一个Singleton的，所以一般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中也可以配置非Singleton，这里我们不做赘述。

9、当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用那个其实现的destroy()方法；

10、最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

## 结合代码理解

### Bean的定义

Spring通常通过配置文件定义Bean。如：

```
<?xml version=”1.0″ encoding=”UTF-8″?>

<beans xmlns=”http://www.springframework.org/schema/beans”
xmlns:xsi=”http://www.w3.org/2001/XMLSchema-instance”
xsi:schemaLocation=”http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.0.xsd”>

<bean id=”HelloWorld” class=”com.pqf.beans.HelloWorld”>
    <property name=”msg”>
       <value>HelloWorld</value>
    </property>
</bean>

</beans>
```

这个配置文件就定义了一个标识为 HelloWorld 的Bean。在一个配置文档中可以定义多个Bean。

### Bean的初始化

1. #### 在配置文档中通过指定init-method 属性来完成
   
   在Bean的类中实现一个初始化Bean属性的方法，如init()，如：
   
   ```
   public class HelloWorld{
      public String msg=null;
      public Date date=null;
   
       public void init() {
         msg=”HelloWorld”;
         date=new Date();
       }
       …… 
   }
   ```

   SpringBoot使用@Bean 注解 配置bean

   然后，在配置文件中设置init-mothod属性：

2. #### 实现 org.springframwork.beans.factory.InitializingBean接口
   
   Bean实现InitializingBean接口，并且增加 afterPropertiesSet() 方法：

```
public class HelloWorld implement InitializingBean {
   public String msg=null;
   public Date date=null;

   public void afterPropertiesSet() {
       msg="向全世界问好！";
       date=new Date();
   }
    …… 
```

那么，当这个Bean的所有属性被Spring的BeanFactory设置完后，会自动调用afterPropertiesSet()方法对Bean进行初始化，于是，配置文件就不用指定 init-method属性了。    

### Bean的调用

三种方式进行Bean的调用 



1、 使用 BeanWrapper

``` 
elloWorld hw=new HelloWorld();
BeanWrapper bw=new BeanWrapperImpl(hw);
bw.setPropertyvalue(”msg”,”HelloWorld”);
system.out.println(bw.getPropertyCalue(”msg”));
```



2、 使用BaenFactory

```
InputStream is=new FileInputStream(”config.xml”);
XmlBeanFactory factory=new XmlBeanFactory(is);
HelloWorld hw=(HelloWorld) factory.getBean(”HelloWorld”);
system.out.println(hw.getMsg());
```



3、使用ApplicationContext

``` 
ApplicationContext actx=new FleSystemXmlApplicationContext(”config.xml”);
HelloWorld hw=(HelloWorld) actx.getBean(”HelloWorld”);
System.out.println(hw.getMsg());
```

### Bean的销毁

1、使用配置文件中的destory-method 属性 

与初始化属性 init-methods类似，在Bean的类中实现一个撤销Bean的方法，然后在配置文件中通过 destory-method指定，那么当bean销毁时，Spring将自动调用指定的销毁方法。



2、实现org.springframwork.bean.factory.DisposebleBean接口

如果实现了DisposebleBean接口，那么Spring将自动调用bean中的Destory方法进行销毁，所以，Bean中必须提供Destory方法。

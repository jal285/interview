# Spring Ioc 源码学习

<!--more-->

## spirng问题

1. 聊聊spring 
2. 说明bean的声明周期
3. 循环依赖
4. 三级缓存
5. FactoryBean和beanFactory 
6. ApplicationContext和Beanfactory 的区别
7. 设计模式 

Spring是spring家族的基石

## Ioc

基本概念

好莱坞原则: Don't call us, we will call you

IOC容器装着 bean对象   ->  数据结构 -> map-> 三级缓存   (map结构存储对象 )  hashmap   

对象存储时都存储的为 K-V格式的数据 -> 创建对象 -> 获取对象  ->beanName /Bean实例对象 

### 源码中的三级缓存

![image-20210416214130584](../images/SpringIoc/image-20210416214130584.png)  文件中的 

![image-20210416214109290](../images/Spring%E5%88%86%E6%9E%90/image-20210416214109290.png)

三级缓存

从 singletonObjects 开始的三个 Map结构 

### AOP

分析AOP源码中找到createProxy 

 在after 方法中 到 WrapIfNecessary 方法再到  createProxy  ,创建代理对象 

![image-20210417140637111](../images/Spring%E5%88%86%E6%9E%90/image-20210417140637111.png)

**DefaultListableBeanFactory**

bean工厂管理着bean ,现有工厂 ,才有 bean 

对spring源码 参照几个接口  

## 使用框架的意义

### 控制反转(思想)

 利用工厂和容器 将对象的管理变得有条理起来,  提供了Ioc 用来将 组件之间解耦 , 提高了程序的可维护性 

在程序规模不断变大情况下依然可以有着较低的维护难度,

在组件比较多的系统当中, 如果耦合度过强,那么程序中出现一点小问题就会导致整个项目的停转, 修复的时间也会随着耦合度的提高而提高, 在解决错误的时候无疑会变得更加具有难度. 所以应该降低对象之间的耦合度 

spring提供了 控制反转, 借用与第三方,也就是IOC容器进行 对象之间的解耦, 对象的控制权上交给IOC容器,  彼此之间就没有了耦合关系,  

让对象按照IOC容器的规则 来,  控制权交到了IOC容器当中, 也就实现了控制反转 

控制权从 object -> IOc

### 依赖注入(设计模式)

将实例变量传入到一个对象当中去

(Dependency injection means giving an object its instance variables)

IoC使用依赖注入作为实现控制反转的方式, 控制反转还有其它的实现方式, 例如 ServiceLoacter

## springboot自动配置

从![image-20210421122757898](../images/Spring%E5%88%86%E6%9E%90/image-20210421122757898.png) 走到

![image-20210421122822091](../images/Spring%E5%88%86%E6%9E%90/image-20210421122822091.png)

到

![image-20210421122835172](../images/Spring%E5%88%86%E6%9E%90/image-20210421122835172.png)

到

![image-20210421122908458](../images/Spring%E5%88%86%E6%9E%90/image-20210421122908458.png)

在AutoAutoConfigurationImportSelector.class中 向下看

![image-20210421123106539](../images/Spring%E5%88%86%E6%9E%90/image-20210421123106539.png)

the auto-configuration  should be imported   

AutoConfigurationEntry 方法会筛选出有效的自动配置类

看注释

![image-20210421123301679](../images/Spring%E5%88%86%E6%9E%90/image-20210421123301679.png)

getCandidateConfigurations 方法加载spring boot 配置的自动配置类

下断点,查看筛选过后的 自动配置 , debug

<img src="../images/Spring%E5%88%86%E6%9E%90/image-20210421124705636.png" alt="image-20210421124705636" style="zoom:200%;" />

从maven下管理的lib

****

![image-20210421133027565](../images/Spring%E5%88%86%E6%9E%90/image-20210421133027565.png)

向下找到web的package

![image-20210421133054326](../images/Spring%E5%88%86%E6%9E%90/image-20210421133054326.png)

查看源码

![image-20210421133132993](../images/Spring%E5%88%86%E6%9E%90/image-20210421133132993.png)

@EnableConfigurationProperties(SerProperties.class) 启动指定类的

ConfigurationProperties 功能, 将配置文件中的值和`ServerProperties`绑定起来并把 ServerProperties 加入到IOC容器

看到ServerProperties

```java
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {

    /**
     * Server HTTP port.
     */
    private Integer port;
```

@ConfigurationProperties 绑定属性映射文件中的server开头的属性,结合默认配置 

```java
# 路径spring-boot-autoconfigure-2.1.1.RELEASE.jar*
*# /META-INF/spring-configuration-metadata.json*

  {
   "name": "server.port",
   "type": "java.lang.Integer",
   "description": "Server HTTP port.",
   "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties",
   "defaultValue": 8080
  }
```

![image-20210421135256852](../images/Spring%E5%88%86%E6%9E%90/image-20210421135256852.png)

达到了自动配置的目的

### 总结

springboot启动时加载主配置类, 开启自动配置功能@EnableAutoConfiguration 

@EnableAutoConfiguration 给容器导入 META-INF/spring.factories 里定义的自动配置类

### XML配置

定义 helloService Bean.

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="net.codingme.boot.service.HelloService"></bean>

</beans>
```

引入配置。

```
@ImportResource(value = "classpath:spring-service.xml")
@SpringBootApplication
public class BootApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootApplication.class, args);
    }
}
```

### 5.2 注解配置

此种方式和上面的XML配置是等效的，也是官方推荐的方式。`@Configuration` 注解的类（要在扫描的包路径中）会被扫描到。

```
/**
 * <p>
 * 配置类，相当于传统Spring 开发中的 xml-> bean的配置
 *
 * @Author niujinpeng
 * @Date 2018/12/7 0:04
 */
@Configuration
public class ServiceConfig {

    /**
     * 默认添加到容器中的 ID 为方法名（helloService）
     *
     * @return
     */
    @Bean
    public HelloService helloService() {
        return new HelloService();
    }
}
```

| @Conditional扩展注解                | 作用（判断是否满足当前指定条件）               |
|:------------------------------- |:------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                |
| @ConditionalOnBean              | 容器中存在指定Bean；                   |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                  |
| @ConditionalOnExpression        | 满足SpEL表达式指定                    |
| @ConditionalOnClass             | 系统中有指定的类                       |
| @ConditionalOnMissingClass      | 系统中没有指定的类                      |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                 |
| @ConditionalOnWebApplication    | 当前是web环境                       |
| @ConditionalOnNotWebApplication | 当前不是web环境                      |
| @ConditionalOnJndi              | JNDI存在指定项                      |
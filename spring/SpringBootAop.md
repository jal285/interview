# Aop在boot中使用





## 定义切面

使用@Aspect 注解加在 public class 上方 , 使得类内为一个切面  





### 定义切点

@Pointcut 注解可以在切面内定义可重用的切点

spring切面粒度最小达到方法级别, execution表达式可以用于明确指定方法返回类型, 类名, 方法名和参数名等于方法相关的部件, 并且实际中,大部分需要使用AOP的业务场景也只需要达到方法级别即可 

因而execution表达式的使用是最为广泛的。下面是execution表达式的语法：

![image-20210620085853506](../images/SpringBootAop/image-20210620085853506.png)

execution表示在方法执行的时候触发. 以" "开头, 表明方法返回值类型为任意类型, 然后是全限定的类名和方法名, ""可以表示任意类和任意方法. 对于方法参数列表, 可以使用 " .. " 表示参数为任意类型. 如果需要多个表达式, 可以使用"&&" 、“||” 和 “ ! ”完成与、或、非的操作 





### 定义通知

前置通知（@Before）：在目标方法调用之前调用通知

后置通知（@After）：在目标方法完成之后调用通知

环绕通知（@Around）：在被通知的方法调用之前和调用之后执行自定义的方法

返回通知（@AfterReturning）：在目标方法成功执行之后调用通知

异常通知（@AfterThrowing）：在目标方法抛出异常之后调用通知

代码中定义了三种类型的通知，使用@Before注解标识前置通知，打印“beforeAdvice...”，使用@After注解标识后置通知，打印“AfterAdvice...”，使用@Around注解标识环绕通知，在方法执行前和执行之后分别打印“before”和“after”。这样一个切面就定义好了，代码如下：

```java
@Aspect
@Component
public class AopAdvice {

    //指定具体类和方法
    @Pointcut("execution (* com.shangguan.aop.controller.*.*(..))")
    public void test() {

    }

    @Before("test()")
    public void beforeAdvice() {
        System.out.println("beforeAdvice...");
    }

    @After("test()")
    public void afterAdvice() {
        System.out.println("afterAdvice...");
    }

    @Around("test()")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) {
        System.out.println("before");
        try {
            proceedingJoinPoint.proceed();
        } catch (Throwable t) {
            t.printStackTrace();
        }
        System.out.println("after");
    }

}
```

使用上面切面去拦截一个controller

```java
@RestController
public class AopController {

    @RequestMapping("/hello")
    public String sayHello(){
        System.out.println("hello");
        return "hello";
    }
}
```



使用别人的测试结果，结果如图 

![image-20210620090556518](../images/SpringBootAop/image-20210620090556518.png)



## AOP 做简单权限认证

[参考](https://juejin.cn/post/6844904160983285774#heading-3)

利用AOP 在controller 登陆方法调用前后加上aop  , 比如执行方法后进行权限认证, 验证是否有权限

使用url转发的形式,调用权限认证后登陆界面, 

在index界面直接做权限,未登录用户直接登出,切换登陆页面 


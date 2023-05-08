# 后端开发之路

<!-- more -->

[TOC]

## Servlet 基础

Servlet规范描述了Http请求以及响应处理过程相关的对象及其作用。

### Servlet 过滤器

为特定的URL请求添加过滤器，

```
<``filter``>
  ``<``filter-name``>LoggingFilter</``filter-name``>
  ``<``filter-class``>LoggingFilter</``filter-class``>
</``filter``>
<``filter-mapping``>
  ``<``filter-name``>LogingFilter</``filter-name``>
  ``<``url-pattern``>/*</``url-pattern``>
</filter-mapping``>
```

### 侦听Servlet容器事件

创建一个基于容器事件执行操作的侦听器，必须开发一个实现该ServletContextListemer接口的类。

需要实现的方法是contextInitialized()和contextDestoryed()

要向容器注册侦听器，可以使用以下技术之一：

1）利用`@WebListener`注释。
2）在web.xml应用程序部署描述符中注册侦听器。
3）使用上`addListener()`定义的方法`ServletContext`。

请注意，`ServletContextListener`它不是Servlet API中的唯一列表器。还有更多例如

- javax.servlet.ServletRequestListener
- javax.servlet.ServletRequestAttrbiteListener
- javax.servlet.ServletContextListener
- javax.servlet.ServletContextAttributeListener
- javax.servlet.HttpSessionListener
- javax.servlet.HttpSessionAttributeListener

可以由列表器类根据想听事件的类型来选择实现，HttpSessionListenser每次创建或销毁新用户会话时，都会收到通知

### 将请求转发到另一个Servlet

#### 使用RequestDispatcher.forward()

有时，您的应用程序要求servlet应该将请求移交给其他servlet，以完成需要完成的任务。此外，应在不将客户端重定向到另一个URL的情况下移交请求，即浏览器中的URL不应更改。

这样做的功能就内置在中`ServletContext`，因此一旦获得对的引用`ServletContext`，则只需调用该`getRequestDispatcher()`方法即可获得一个RequestDispatcher对象，该对象可用于调度请求。调用该`getRequestDispatcher()`方法时，传递一个String，其中包含要将您的请求传递到的servlet的名称。`RequestDispatcher`获取对象后，通过将`HttpServletRequest`和`HttpServletResponse`对象传递给它来调用其前向方法。转发方法执行移交请求的任务。

```java
RequestDispatcher rd = servletContext.getRequestDispatcher("/NextServlet");
rd.forward(request,response);
```

#### 使用HttpServletResponse.sendRedirect()

当您访问应用程序中的特定URL时，想将浏览器重定向到另一个URL。 

为此，将需要调用HttpServletResponse object的sendRedirect()方法

```java
httpServletResponse,sendredirect("/anotherURL");
```

于servlet链接相反，这种简单的重定向不会将HttpRequest对象传递到目标地址

### Servlet编写和读取cookie

```java
Cookie cookie = new Cookie("sessionId","123456789");
cookie.setHttpOnly(true);
cookie.setMaxAge(-30);
response.addCookie(cookie);
```

响应HttpServletResponse 传递给doXXX()方法的实例

```java
Cookie[] cookies = request.getCookies();
for(Cookie cookie : cookies)
{
    //cookie.getName();
    //cookie.getValue()
}  //回读服务器父项上的cookie信息
```

## JDBC

### 驱动程序

四种类型驱动程序

- JDBC-ODBC桥驱动程序  //有纯Java驱动程序替代的话，不建议此
- 本机API驱动程序
- 所有Java+中间件翻译驱动程序
- 纯Java驱动程序

#### 本机API驱动程序

JDBC驱动程序类似于类型1驱动程序，不同之处在于**ODBC部分被替换为本机代码部分**。本机代码部分针对特定的数据库产品，即使用数据库产品的客户端库。驱动程序将JDBC方法调用转换为数据库本机API的本机调用。

这种体系结构消除了对ODBC驱动程序的需求，而直接称为数据库供应商提供的本机客户端库。数据库供应商很快就采用了这种方法，因为它可以快速，廉价地实现，因为他们可以重用现有的基于C / C ++的本机库。

#### 所有Java+ 中间件翻译驱动程序

 JDBC驱动程序是一种全Java驱动程序，它将**JDBC接口调用发送到中间服务器**。然后，中间服务器代表JDBC驱动程序连接到数据库。中间层（应用程序服务器）将JDBC调用直接或间接转换为供应商特定的数据库协议。

Type 3驱动程序试图成为100％Java解决方案，但并没有真正获得太大的吸引力。Type 3驱动程序具有Java客户端组件和Java Server组件，后者实际上是在与数据库对话。尽管从技术上讲这是一个完整的Java解决方案，但是数据库供应商不喜欢这种方法，因为这种方法成本高昂–他们将不得不重写全部为C / C ++的本机客户端库。另外，这并没有提高体系结构的效率，因为我们实际上仍然是3层体系结构，因此很容易看出为什么它从来都不是流行的选择。

![JDBC驱动程序类型3](../images/java_web/JDBC-driver-type-3.png)

#### 纯Java驱动程序

![JDBC驱动程序类型4](../images/java_web/JDBC-driver-type-4.png)

JDBC 4类驱动程序，也称为直接数据库纯Java驱动程序，是一种数据库驱动程序实现，可**将JDBC调用直接转换为特定于供应商的数据库协议**。它是针对特定数据库产品实现的。如今，大多数JDBC驱动程序都是4类驱动程序。

Type 4驱动程序完全用Java编写，因此与平台无关。它们安装在客户端的Java虚拟机内部。这提供了比类型1和类型2驱动程序更好的性能，因为它没有将调用转换为ODBC或数据库API调用的开销。与类型3驱动程序不同，它不需要关联的软件即可工作。

该体系结构将整个JDBC API实现以及用于直接与数据库通信的所有逻辑封装在
单个驱动程序中。通过在100％java程序包中都包含一个单独的层和一个小的驱动程序，这可以简化部署并简化开发过程。

例如，这种类型包括广泛使用的Oracle瘦驱动程序。

### 问题

java使用Stament对象使用execute()方法传递sql语句时，sql语句用的是" " 字符串式，所以在用value传递参数时，不能直接将id，age这样的变量直接使用

需要 '"+id +"','"user"  这样的形式将变量传递进，否则则将传进空值 

jdbc中insert语句 例

或者用字符串拼接的方法传递参数，例

```Java
statement.execute(String.format("INSERT INTO employees(id,age,first,last)"+"VALUES('"+ id +"', '"+age +"','"+first +"','"+last+"')"));
```

注意VALUE 前有双引号，一条语句分开 双引号 

### Spring 中JdbcTemplate 类

JDBC模板类执行SQL查询,更新语句,存储过程调用 , 对ResultSet执行迭代并提取返回的参数值, 它还捕获JDBC异常并将其转换为org.springframework.dao包中定义的通用,信息量更大的异常层次结构 

配置后, JdbcTemplate类的实例时线程安全的,因此,可以配置JdbcTemplate 的单个实例,然后安全地将此共享引用注入到多个DAO中

需要为JDBC模板提供一个数据源, 使得它可以对其进行配置以获取数据库访问权限, 可以是在XML文件中配置DataSource ,

```xml
<bean id = "dataSource" 
   class = "org.springframework.jdbc.datasource.DriverManagerDataSource">
   <property name = "driverClassName" value = "com.mysql.jdbc.Driver"/>
   <property name = "url" value = "jdbc:mysql://localhost:3306/TEST"/>
   <property name = "username" value = "root"/>
   <property name = "password" value = "password"/>
</bean
```

#### 数据访问对象(DAO)

DAO代表数据访问对象,通常用于数据库交互,DAO的存在

执行sql语句

查询整数

```java
String Sql = "select count(*) from Student";'
int rowCount = jdbcTemplateObject.queryForInt(Sql);
```

**查询很长时间**

```
String SQL = "select count(*) from Student";
long rowCount = jdbcTemplateObject.queryForLong( SQL );
```

**使用绑定变量的简单查询**

```
String SQL = "select age from Student where id = ?";
int age = jdbcTemplateObject.queryForInt(SQL, new Object[]{10});
```

**查询字符串**

```
String SQL = "select name from Student where id = ?";
String name = jdbcTemplateObject.queryForObject(SQL, new Object[]{10}, String.class);
```

**查询和返回对象**

```
String SQL = "select * from Student where id = ?";
Student student = jdbcTemplateObject.queryForObject(
   SQL, new Object[]{10}, new StudentMapper());

public class StudentMapper implements RowMapper<Student> {
   public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
      Student student = new Student();
      student.setID(rs.getInt("id"));
      student.setName(rs.getString("name"));
      student.setAge(rs.getInt("age"));

      return student;
   }
}
```

**查询并返回多个对象**

```
String SQL = "select * from Student";
List<Student> students = jdbcTemplateObject.query(
   SQL, new StudentMapper());

public class StudentMapper implements RowMapper<Student> {
   public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
      Student student = new Student();
      student.setID(rs.getInt("id"));
      student.setName(rs.getString("name"));
      student.setAge(rs.getInt("age"));

      return student;
   }
}
```

**在表格中插入一行**

```
String SQL = "insert into Student (name, age) values (?, ?)";
jdbcTemplateObject.update( SQL, new Object[]{"Zara", 11} );
```

**更新表中的一行**

```
String SQL = "update Student set name = ? where id = ?";
jdbcTemplateObject.update( SQL, new Object[]{"Zara", 10} );
```

**从表格中删除一行**

```
String SQL = "delete Student where id = ?";
jdbcTemplateObject.update( SQL, new Object[]{20} );
```

执行DDL语句

您可以使用*jdbcTemplate中*的**execute（..）**方法执行任何SQL语句或DDL语句。以下是使用CREATE语句创建表的示例-

```
String SQL = "CREATE TABLE Student( " +
   "ID   INT NOT NULL AUTO_INCREMENT, " +
   "NAME VARCHAR(20) NOT NULL, " +
   "AGE  INT NOT NULL, " +
   "PRIMARY KEY (ID));"

jdbcTemplateObject.execute( SQL );
```

## 从Spring开始学SpringBoot

Spring的环境配过并解决了一些问题，但仅运行了hello boot 的示例程序

因为没有系统的学习spring，仅仅听了一遍视频，没有任何记忆，所以参考学习网站将Spring的一些基础知识点系统学习并进行记录

### bean 标签的使用

#### bean使用的第一个example程序

<bean>中的bean id 返回调用类的对象，并使用 <property/>标签为变量传递值

例 HelloSpring类为 bean标签内的类  bean id返回HelloSpring类对象传递给HelloSpring variable_name

variable_name将可以调    用HelloSpring类中方法

bean id标签通过类内函数为类成员变量赋值

![image-20210403204446620](../images/java_web/image-20210403204446620.png)

![image-20210403204453493](../images/java_web/image-20210403204453493.png)

message将赋值为"Hello Spring"

#### 定义

构成应用程序主干并由Spring IoC容器管理的对象成为bean，Bean是由Spring IoC容器示例化，组装和以其他方式管理的对象。

属性和说明

| 序号                                | 属性和说明                                                  |
| --------------------------------- | ------------------------------------------------------ |
| 1  class 类                        | 必须的，用于指定创建Bean的类                                       |
| 2 name 姓名                         | 此属性唯一地指示Bean标识符。在基于XML的配置元数据中，使用id和/或name属性来指定Bean 标识符 |
| 3 scope 范围                        | 指定从特定bean 定义创建的对象的范围                                   |
| 4 constructor-arg 构造函数            |                                                        |
| 5 properties 特性                   |                                                        |
| 6 autpwiring mode 自动连线模式          |                                                        |
| 7 lazy-initalization mode 延迟初始化模式 |                                                        |
| 8 initalization mehtod 初始化方法      | 容器设置完Bean的所有必须属性后，将调用此回调                               |
| 9 destruction mehtod 销毁方法         |                                                        |

#### Scopes

强制Spring每次需要一个新的bean实例时，将scope属性声明为prototype，

Spring每次需要一个实例时都返回一个实例时都返回相同的bean实例，则scope属性声明为singleton

| Sr.No. | Scope & Description                                                                                                                           |
| ------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| 1      | **singleton**This scopes the bean definition to a single instance per Spring IoC container (default).                                         |
| 2      | **prototype**This scopes a single bean definition to have any number of object instances.                                                     |
| 3      | **request**This scopes a bean definition to an HTTP request. Only valid in the context of a web-aware Spring ApplicationContext.              |
| 4      | **session**This scopes a bean definition to an HTTP session. Only valid in the context of a web-aware Spring ApplicationContext.              |
| 5      | **global-session**This scopes a bean definition to a global HTTP session. Only valid in the context of a web-aware Spring ApplicationContext. |

##### 单例范围

如果将范围设置为单例，则Spring IoC容器将创建该bean定义所定义的对象的一个实例。该单个实例存储在此类单例bean的高速缓存中，并且对该命名bean的所有后续请求和引用都返回该高速缓存的对象。

#### bean生命周期

定于bean的setup and teardown ，使用initmethod和/或destroy-method参数声明<bean>

##### 初始化回调

org.springframework.beans.factory.InitializingBean接口指定一个方法-

```
void afterPropertiesSet() throws Exception;
```

可以创建一个类去实现Initializing。并且可以在afterPropertiesSet()方法内部完成初始化工作

```java
public class ExampleBean implements InitializingBean {
   public void afterPropertiesSet() {
      // do some initialization work
   }
}
```

对于基于XML的配置元数据，可以使用init-method属性指定具有无效无参数签名的方法的名称

```xml
<bean id = "exampleBean" class = "examples.ExampleBean" init-method = "init"/>
```

```java
public class ExampleBean{
    public void init(){
        //do some initialization work
    }
}
```

##### 销毁回调

所述*org.springframework.beans.factory.DisposableBean*接口指定一个单一的方法-

```
void destroy() throws Exception;
```

因此，您可以简单地实现上述接口，并且可以在destroy（）方法内完成终结工作，如下所示：

```
public class ExampleBean implements DisposableBean {
   public void destroy() {
      // do some destruction work
   }
}
```

对于基于XML的配置元数据，可以使用**destroy-method**属性指定具有无效无参数签名的方法的名称。例如-

```
<bean id = "exampleBean" class = "examples.ExampleBean" destroy-method = "destroy"/>
```

以下是类定义-

```
public class ExampleBean {
   public void destroy() {
      // do some destruction work
   }
}
```

非Web应用程序环境中使用Spring的Ioc容器，不建议使用InitiazingBean或DisposableBean，因为XML文件配置在命名方法方法具有很大的灵活性

示例程序

HelloSpring 类中

![image-20210404091623731](../images/java_web/image-20210404091623731.png)

MainApp 类中内容

![image-20210404091351005](../images/java_web/image-20210404091351005.png)

bena中配置

![image-20210404091721445](../images/java_web/image-20210404091721445.png)

 默认的初始化和销毁方法

如果您有太多具有相同名称的初始化和/或销毁方法的bean，则无需在每个单独的bean上声明**init-method**和**destroy-method**。相反，该框架提供了灵活性，可以使用<beans>元素上的**default-init-method**和**default-destroy-method**属性配置这种情况，如下所示-

```java
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
   default-init-method = "init" 
   default-destroy-method = "destroy">

   <bean id = "..." class = "...">
      <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```

#### bean Post Processors

BeanPostProcesser的接口定义回调方法，你可以实现提供自己的实例化逻辑

可以配置多个BeanPostProcesser接口，并且可以通过设置BeanProProcesser实现Ordered接口的order属性来控制这些BenPostProcesser接口的执行顺序

BeanPostProcessor在Bean（或对象）实例上运行，Spring IoC容器实例化bean实例，然后BeanPostProcessor接口完成其工作。

一个**ApplicationContext的**自动检测与该执行中定义的任何bean**的BeanPostProcessor**接口，并注册这些豆类如后处理器，被然后通过在容器创建bean的适当调用。

![image-20210404094704632](../images/java_web/image-20210404094704632.png)

![image-20210404094717165](../images/java_web/image-20210404094717165.png)

#### Spring-Bean定义继承

从父定义继承配置数据，自定义根据需要覆盖某些值，或添加其他值

![image-20210404123441729](../images/java_web/image-20210404123441729.png)

可以创建一个Bean定义模板，该模板由其它子bean定义使用，在定义Bean定义模板时，不应指定class属性，而应指定abtract属性 并且指定值为true的abstract属性

```xml
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "beanTeamplate" abstract = "true">
      <property name = "message1" value = "Hello World!"/>
      <property name = "message2" value = "Hello Second World!"/>
      <property name = "message3" value = "Namaste India!"/>
   </bean>

   <bean id = "helloIndia" class = "com.tutorialspoint.HelloIndia" parent = "beanTeamplate">
      <property name = "message1" value = "Hello India!"/>
      <property name = "message3" value = "Namaste India!"/>
   </bean>

</beans>
```

父bean不能单独实例化，因为它是不完整的，而且还被明确标记为abstract，这样的定义很抽象时，它只能用作纯模板定义，用作自定义的父定义

#### 依赖注入

写复杂的应用程序时，应用程序类应尽可能独立于其它Java类， 依赖注入（或有时成为接线）有助于将这些类粘合在一起，同时保持他们的独立性

例子

```Java
public class TextEditor {
   private SpellChecker spellChecker;

   public TextEditor() {
      spellChecker = new SpellChecker();
   }
}
```

我们在这里所做的是在TextEditor和SpellChecker之间创建一个依赖项。在控制方案反转的情况下，我们将改为执行以下操作：

```java
public class TextEditor {
   private SpellChecker spellChecker;

   public TextEditor(SpellChecker spellChecker) {
      this.spellChecker = spellChecker;
   }
}
```

SpellChecker将独立实现，并将在TextEditor实例化时提供给TextEditor，整个过程由Spring框架控制

在这里，我们从TextEditor中删除了总控制权，并将其保留在其他地方（即XML配置文件），并且依赖项（即SpellChecker类）已通过**Class Constructor**注入到TextEditor**类中**。因此，控制流已通过依赖注入`（DI）“反转”`，因为您已将依赖有效地委派给了某些外部系统。

注入依赖项的第二种方法是通过TextEditor类的**Setter方法**，在该**方法**中，我们将创建SpellChecker实例。此实例将用于调用setter方法以初始化TextEditor的属性。

| 序号               | 依赖注入类型和描述                                                           |
| ---------------- | ------------------------------------------------------------------- |
| 1   基于构造函数的依赖注入  | 当容器调用带有多个参数的类构造函数时，将完成基于构造函数的DI，每个参数表示另一个类的依赖；                      |
| 2  基于Setter的依赖注入 | 基于设置器的DI是通过调用无参数构造函数或无参数静态工厂方法以实例化您的bean之后，在您的bean上调用setter()方法来完成的 |

可以混合使用基于构造函数的DI和基于Setter的DI

根据经验来说，对于强制性依赖项使用构造函数参数，对于可选的依赖项使用setter方法

#### 内部bean

<protery/>或<construcotr-arg/>元素内的<bean/>元素称为内部bean

example:

```java
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id = "outerBean" class = "...">
      <property name = "target">
         <bean id = "innerBean" class = "..."/>
      </property>
   </bean>

</beans>
```

#### Injecting Collection

传递`Java Collection类型(例如List,Set,Map和Properties)之类的复数值, 为了处理这种情况,Spring提供了四种类型的集合配置元素,

| Sr.No | Element & Description                                                                                             |
| ----- | ----------------------------------------------------------------------------------------------------------------- |
| 1     | **<list>**This helps in wiring ie injecting a list of values, allowing duplicates.                                |
| 2     | **<set>**This helps in wiring a set of values but without any duplicates.                                         |
| 3     | **<map>**This can be used to inject a collection of name-value pairs where name and value can be of any type.     |
| 4     | **<props>**This can be used to inject a collection of name-value pairs where the name and value are both Strings. |

可以手机用<list>或<set>连接java.util.Collection或数组的任何实现

(a)传递集合的直接值

(b)传递bean的引用作为集合元素之一

在<property></property> 标签中使用<list> 或<set>等标签

##### 注入空和空字符串值

如果您需要传递一个空字符串作为值，则可以按以下方式传递它：

```
<bean id = "..." class = "exampleBean">
   <property name = "email" value = ""/>
</bean>
```

上面的示例等效于Java代码：exampleBean.setEmail（“”）

如果需要传递NULL值，则可以按以下方式传递它-

```
<bean id = "..." class = "exampleBean">
   <property name = "email"><null/></property>
</bean>
```

上面的示例等效于Java代码: exampleBean.setEmail(null)

#### auto-Wiring

自动连线 

Spring自动装备协作bean之间的关系,而无需是由<constructor-arg> 和<property> 标签

#### 基于注解的配置

使用注释配置依赖项注入

注释注入在XML注入之间执行,因此,对于通过两种方法俩连接的属性,后一种配置将覆盖前者

默认情况下,Spring容器中的注释接线未打开.因此,在使用基于注解的连接之前,需要在Spring配置文件之中启用它 

```xml
<?xml version = "1.0" encoding = "UTF-8"?>

<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context = "http://www.springframework.org/schema/context"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:annotation-config/>
   <!-- bean definitions go here -->

</beans>
```

配置了<contex:annotation-config/> 就可以开始注释代码

Let us look at a few importment annotations to understand how they work -

|                        | 注释于说明                                                              |
| ---------------------- | ------------------------------------------------------------------ |
| 1  @Required           | 用于bean属性设置器方法                                                      |
| 2  @Autowried          | 应用于属性设置器方法,非设置器方法,构造函数和属性                                          |
| 3  @Qualifier          | 与@Autowried一起使用,可以通过要连接的确切bean来消除混淆                                |
| 4  JSR_250 Annotations | Spring支持基于JSR-250的注释,其中包括@Resource, @PostConstruct和 @PreDestory 注释 |

##### @Required

在属性上添加@Required 将必须为此属性(变量)传递值(在Bean.xml文件中)

##### @Autowired

设置在 setter方法上则不用在bean.xml文件中添加其<bean>标签

比如用在bean中bean里,则不用再写到内部bean中 ,

![image-20210404172934053](../images/java_web/image-20210404172934053.png)

使用后

![image-20210404172921819](../images/java_web/image-20210404172921819.png)

[剩下用法参考](https://www.tutorialspoint.com/spring/spring_autowired_annotation.htm)

直接用在属性上 , 摆脱setter方法 当您使用<property>传递自动装配属性的值时，Spring会自动为这些属性分配传递的值或引用。

用在构造函数上  构造函数@Autowired批注知识即使在XML文件中配置Bean时不适用任何<contructor-arg>元素,也应在创建Bean时自动构造该构造函数

可以在构造函数内直接创建对象

@Autowired with(required=false)选项

默认情况下，@ Autowired批注表示与@Required批注类似，需要依赖项，但是，可以通过对@Autowired使用**（required = false）**选项来关闭默认行为。

即使您没有为age属性传递任何值，但仍将要求name属性，下面的示例仍然有效。您可以自己尝试该示例，因为除了仅更改了**Student.java**文件之外，它与@Required注释示例相似。

```java
package com.tutorialspoint;

import org.springframework.beans.factory.annotation.Autowired;

public class Student {
   private Integer age;
   private String name;

   @Autowired(required=false)
   public void setAge(Integer age) {
      this.age = age;
   }
   public Integer getAge() {
      return age;
   }

   @Autowired
   public void setName(String name) {
      this.name = name;
   }
   public String getName() {
      return name;
   }
}
```

#### Configuration和@Bean批注

用@Configuration注释一个类表示该类可以被Spring Ioc容器用作定义Bean的原 @Bean注释告诉Spring使用@Bean注释的方法将返回一个对象,该对象应在Sprign应用程序的上下文中注册为Bean,最简单的@Configuration类如下

```java
@Configuration
public class HelloConfiguration {
    @Bean
    public HelloSpring hello(){
        return new HelloSpring();
        //这里的HelloSpring 是已经写好的类 ,定义好了setter方法和属性
        //也可以直接创建本类的对象  return  HelloConfiguration() 
    }
}
```

等效于在XML中配置

<bean id="hello" class="location.HelloConfiguration"/>

方法名称用@Bean标注为Bean ID,它创建并返回实际的Bean,类中可以有多个@Bean的声明,一旦定义了配置类,就可以使用 *AnnotationConfigApplicationContex* 将他们加载出来并提供给Spring容器 

使用

```java
   public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(HelloConfiguration.class);

        HelloSpring helloSpring = context.getBean(HelloSpring.class);
        helloSpring.setMessage1("你好 @bean 注释");
        helloSpring.getMessage1();
    }
```

您可以按如下方式加载各种配置类：

```java
public static void main(String[] args) {
   AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();

   ctx.register(AppConfig.class, OtherConfig.class);
   ctx.register(AdditionalConfig.class); 
   ctx.refresh();

   MyService myService = ctx.getBean(MyService.class);
   myService.doStuff();
}
```

@Import  Annotation

@Import annotation 允许直接从其他 Configuration类中加载@Bean定义(denfinition)

as follows(如下)

```java
@Configuration
public class ConfigA {
   @Bean
   public A a() {
      return new A(); 
   }
}
```

You can import above Bean declaration in another Bean Declaration as follows −

```java
@Configuration
@Import(ConfigA.class)
public class ConfigB {
   @Bean
   public B b() {
      return new B(); 
   }
}
```

现在,不需要实例化时去指定 ConfigA和Config ,只需要提供ConfigB

as follows 

```java
public static void main(String[] args) {
   ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

   // now both beans A and B will be available...
   A a = ctx.getBean(A.class);
   B b = ctx.getBean(B.class);
}
```

##### 生命周期回调

@Bean注释支持指定任意的初始化和销毁方法 ,像咋Spring XML 上的 init-mehtod 和 destroy-method 属性一样

```java
public class Foo {
   public void init() {
      // initialization logic
   }
   public void cleanup() {
      // destruction logic
   }
}
@Configuration
public class AppConfig {
   @Bean(initMethod = "init", destroyMethod = "cleanup" )
   public Foo foo() {
      return new Foo();
   }
}
```

##### 指定Bean范围

默认范围为单例,但可以使用@Bean 注释(annotation)重写它

```java
@Configuration
public class AppConfig{
     @Bean
     @Scope("property")
     public Foo foo(){
         reuturn Foo();
     }
}
```

### Event Handing in Spring

事件处理

The  core(核心) of Spring is the ApplicationContext ,which manages the complete life cycle(周期) of the beans.

loding beans时, ApplicationContext 发布某些类型的事件 . 例如 a ContextStartedEvent is published when the context is started and ContextStoppedEvent is published when the context is stopped

ApplicationContext 通过ApplicationEvent class和 ApplicationListen interface 提供事件处理,如果bean implements the ApplicationListener

| Sr.No. | Spring Built-in Events & Description                                                                                                                                                                                                                                  |
|:------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| 1      | **ContextRefreshedEvent**This event is published when the *ApplicationContext* is either initialized or refreshed. This can also be raised using the refresh() method on the *ConfigurableApplicationContext* interface.                                              |
| 2      | **ContextStartedEvent**This event is published when the *ApplicationContext* is started using the start() method on the *ConfigurableApplicationContext* interface. You can poll your database or you can restart any stopped application after receiving this event. |
| 3      | **ContextStoppedEvent**This event is published when the *ApplicationContext* is stopped using the stop() method on the *ConfigurableApplicationContext* interface. You can do required housekeep work after receiving this event.                                     |
| 4      | **ContextClosedEvent**This event is published when the *ApplicationContext* is closed using the close() method on the *ConfigurableApplicationContext* interface. A closed context reaches its end of life; it cannot be refreshed or restarted.                      |
| 5      | **RequestHandledEvent**This is a web-specific event telling all beans that an HTTP request has been serviced.                                                                                                                                                         |

Spring的事件处理是单线程的,因此,如果事件被发布,则直到并且除非所有接收者都收到消息,否则流程将被阻塞并且流程不会继续. 因此,如果要使用事件处理,则在设计应用程序时十分小心 be careful

#### 监听上下文事件

要侦听上下文事件,bean应该实现ApplicationListener接口, 该接口只有一个方法 onApplicationEvent(), 

![image-20210404212310883](../images/java_web/image-20210404212310883.png)

![image-20210404212316459](../images/java_web/image-20210404212316459.png)

上面实现了两个Spring中定义的事件 ,进行了简单写入

![image-20210404212517061](../images/java_web/image-20210404212517061.png)

ApplicationContex对象通过 start()方法和stop()方法 创建 开始和结束事件

### 自定义事件

编写和发布自己的自定义事件需要采取许多步骤,  将跟随教程发布和处理Cutstom Spring Events

|     | 描述                                                                                                                                                                      |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | 创建一个名称为*SpringExample*的项目，并在创建的项目的**src**文件夹下创建一个*com.tutorialspoint*包。所有类都将在此程序包下创建。                                                                                   |
| 2   | 使用“*添加外部JAR”*选项添加所需的Spring库，如“ *Spring Hello World示例”*一章中所述。                                                                                                            |
| 3   | 通过扩展**ApplicationEvent**创建事件类*CustomEvent*。此类必须定义一个默认构造函数，该构造函数应继承ApplicationEvent类的构造函数。                                                                               |
| 4   | 一旦定义了事件类，就可以从任何类中发布它，让我们说一下实现*ApplicationEventPublisherAware的**EventClassPublisher*。您还需要在XML配置文件中将该类声明为Bean，以便容器可以将Bean标识为事件发布者，因为它实现了ApplicationEventPublisherAware接口。 |
| 5   | 可以在一个类中处理已发布的事件，比方说*EventClassHandler*，它实现*ApplicationListener*接口，并为自定义事件实现*onApplicationEvent*方法。                                                                      |
| 6   | 在**src**文件夹下创建bean配置文件*Beans.xml*，并在*MainApp*类中创建Spring应用程序。                                                                                                            |
| 7   | 最后一步是创建所有Java文件和Bean配置文件的内容，然后按以下说明运行应用程序。                                                                                                                              |

通过上述步骤可以创建一个自己的事件并发布

通过教程简单的创建了一个小的自定义事件 ,但对于其原理和用处不是很明白,所以goole了一下

#### Spring中自定义Event事件的使用和浅析

[参考](https://blog.csdn.net/tuzongxun/article/details/53637159)

参考文章内提到了事件驱动模型 

```
事件驱动模型也就是我们常说的观察者，或者发布-订阅模型；理解它的几个关键点：

1 首先是一种对象间的一对多的关系；最简单的如交通信号灯，信号灯是目标（一方），行人注视着信号灯（多方）；
2 当目标发送改变（发布），观察者（订阅者）就可以接收到改变；
3 观察者如何处理（如行人如何走，是快走/慢走/不走，目标不会管的），目标无需干涉；所以就松散耦合了它们之间的关系。
```

该例子将时间的发布比喻为信号灯, 也就是说事件的发布相当于信号灯

自定义事件需要继承Application类, 相当于给信号灯通电,使信号灯可以亮起 

创建一个监听类, 需要实现ApplicationListener接口 ,并且重写 onApplicationEvent()方法, 也就是提供了信号的意思, 比如绿灯行,红灯听 监听类可以是多方的, 因为红绿灯可以被多个人看到

第三步,需要一个控制信号灯变化的东西,也就是控制信号灯的内容  红黄绿灯的程序和电路

通过main方法将静止的灯和人动起来

行人看灯,电路通电

可以通过实验在写一个 监听类

### IoC容器

Spring容器是Spring框架的核心，容器将会创建对象，将他们连接在一起，对其进行配置，并管理从创建到销毁的整个生命周期

Spring容器使用DI来管理组成应用程序的组件。这些对象称为Spring Bean

容器通过读取提供的配置元数据来获取有关要实例化，配置和组装哪些对象的指令。配置元数据可以用XML，Java批注或Java代码表示。下图代表了Spring的工作原理的高级视图。Spring IoC容器利用Java POJO类和配置元数据来生成完全配置且可执行的系统或应用程序。

![春季IoC容器](../images/java_web/spring_ioc_container.jpg)

Spring提供了两种不同类型的容器

| 序号  | 容器及说明                                                                                                                                                                                                                                                                         |
|:---:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| 1个  | [Spring BeanFactory容器](https://www.tutorialspoint.com/spring/spring_beanfactory_container.htm)这是为DI提供基本支持的最简单的容器，由*org.springframework.beans.factory.BeanFactory*接口定义。Spring仍存在BeanFactory及其相关接口，例如BeanFactoryAware，InitializingBean，DisposableBean，目的是向后兼容与Spring集成的大量第三方框架。 |
| 2个  | [Spring ApplicationContext容器](https://www.tutorialspoint.com/spring/spring_applicationcontext_container.htm)此容器添加了更多特定于企业的功能，例如从属性文件解析文本消息的功能以及将应用程序事件发布到感兴趣的事件侦听器的功能。该容器由*org.springframework.context.ApplicationContext*接口定义。                                               |

所述*的ApplicationContext*容器包括所有功能*的BeanFactory*容器，因此，通常建议在*Bean工厂*。BeanFactory仍可用于轻量级应用程序，例如移动设备或基于Applet的应用程序，这些应用程序的数据量和速度非常重要。

#### AOP

Aspect oriented programming(AOP)

| r.No | Terms & Description                                                                                                                                                                                                                          |
|:----:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| 1    | **Aspect**  This is a module which has a set of APIs providing cross-cutting requirements. For example, a logging module would be called AOP aspect for logging. An application can have any number of aspects depending on the requirement. |
| 2    | **Join point ** This represents a point in your application where you can plug-in the AOP aspect. You can also say, it is the actual place in the application where an action will be taken using Spring AOP framework.                      |
| 3    | **Advice**   This is the actual action to be taken either before or after the method execution. This is an actual piece of code that is invoked during the program execution by Spring AOP framework.                                        |
| 4    | **Pointcut ** This is a set of one or more join points where an advice should be executed. You can specify pointcuts using expressions or patterns as we will see in our AOP examples.                                                       |
| 5    | **Introduction**  An introduction allows you to add new methods or attributes to the existing classes.                                                                                                                                       |
| 6    | **Target object **The object being advised by one or more aspects. This object will always be a proxied object, also referred to as the advised object.                                                                                      |
| 7    | **Weaving ** Weaving is the process of linking aspects with other application types or objects to create an advised object. This can be done at compile time, load time, or at runtime.                                                      |

types of Advice

Spring aspects can work with five kinds of advice mentioned as follows −

| Sr.No | Advice & Description                                                                                     |
|:-----:|:--------------------------------------------------------------------------------------------------------:|
| 1     | **before**Run advice before the a method execution.                                                      |
| 2     | **after**Run advice after the method execution, regardless of its outcome.                               |
| 3     | **after-returning**Run advice after the a method execution only if method completes successfully.        |
| 4     | **after-throwing**Run advice after the a method execution only if method exits by throwing an exception. |
| 5     | **around**Run advice before and after the advised method is invoked.                                     |

自定义方面的实施

Spring支持@AspectJ 批注样式方法 和 基于模式的方法来实现自定义方面. 一下各节详细说明了两种方法

|              | 方法与描述                                  |
| ------------ | -------------------------------------- |
| 1 基于XML模式    | 使用常规类以及基于XML的配置来实现切面                   |
| 2 基于@AspectJ | @AspectJ是一种将切面声明为带有Java 5批注的常规Java类的样式 |

### SpringMVC

Spring Web MVC框架提供了Model-View-Controller(MVC)架构和线程的组件,可用于开发灵活且松散耦合的Web应用程序. MVC模式导致分离应用程序的不同方面(输入逻辑, 业务逻辑和UI逻辑) 

- 该**模型**封装了应用程序数据，通常它们将由POJO组成。
- 该**视图**负责呈现模型数据，并在总体上产生HTML输出，客户端的浏览器可以解释。
- 该**控制器**负责处理用户请求，并且建立一个合适的模型，并将其传递到用于渲染的图。

#### DispatcherServlet

Spring Web模型视图控制器（MVC）框架是围绕处理所有HTTP请求和响应的*DispatcherServlet*设计的。下图说明了Spring Web MVC *DispatcherServlet*的请求处理工作流程-

![Spring DispatcherServlet](../images/java_web/spring_dispatcherservlet.png)

以下是与*DispatcherServlet*的传入HTTP请求相对应的事件序列-

- 收到HTTP请求后，*DispatcherServlet*咨询*HandlerMapping* 来调用适当的*Controller*。
- 该*控制器*接受请求，并调用基于所使用GET或POST方法相应的服务的方法。服务方法将基于定义的业务逻辑设置模型数据，并将视图名称返回给*DispatcherServlet*。
- 所述*的DispatcherServlet*将帮助从*的ViewResolver*到拾取该请求的已定义视图。
- 视图完成后，*DispatcherServlet*将模型数据传递到视图，该视图最终在浏览器上呈现。

所需配置

您需要通过使用**web.xml**文件中的URL映射来映射希望*DispatcherServlet*处理的请求。以下是显示**HelloWeb** *DispatcherServlet*示例的声明和映射的示例-

```xml
<web-app id = "WebApp_ID" version = "2.4"
   xmlns = "http://java.sun.com/xml/ns/j2ee" 
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://java.sun.com/xml/ns/j2ee 
   http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

   <display-name>Spring MVC Application</display-name>

   <servlet>
      <servlet-name>HelloWeb</servlet-name>
      <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
      </servlet-class>
      <load-on-startup>1</load-on-startup>
   </servlet>

   <servlet-mapping>
      <servlet-name>HelloWeb</servlet-name>
      <url-pattern>*.jsp</url-pattern>
   </servlet-mapping>

</web-app>
```

在**web.xml中**的文件将被保存在你的Web应用程序的WebContent / WEB-INF目录下。初始化**HelloWeb** DispatcherServlet时，框架将尝试从位于应用程序WebContent / WEB-INF目录中名为**[servlet-name] -servlet.xml**的文件中加载应用程序上下文。在这种情况下，我们的文件将是**HelloWebservlet.xml**。

接下来，<servlet-mapping>标记指示哪个DispatcherServlet将处理哪些URL。在这里，所有以**.jsp**结尾的HTTP请求都将由**HelloWeb** DispatcherServlet处理。

如果不想使用默认文件名作为*[servlet-name] -servlet.xml*和默认位置作为*WebContent / WEB-INF*，则可以通过在web.xml文件中添加Servlet侦听器*ContextLoaderListener*来自定义此文件名和位置。如下-

```xml
<web-app...>

   <!-------- DispatcherServlet definition goes here----->
   ....
   <context-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>/WEB-INF/HelloWeb-servlet.xml</param-value>
   </context-param>

   <listener>
      <listener-class>
         org.springframework.web.context.ContextLoaderListener
      </listener-class>
   </listener>

</web-app>
```

HelloWebservlet.xml必须配置

```xml
<beans xmlns = "http://www.springframework.org/schema/beans"
   xmlns:context = "http://www.springframework.org/schema/context"
   xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation = "http://www.springframework.org/schema/beans     
   http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
   http://www.springframework.org/schema/context 
   http://www.springframework.org/schema/context/spring-context-3.0.xsd">

   <context:component-scan base-package = "location.package" />

   <bean class = "org.springframework.web.servlet.view.InternalResourceViewResolver">
      <property name = "prefix" value = "/WEB-INF/jsp/" />
      <property name = "suffix" value = ".jsp" />
   </bean>

</beans>
```

- The [servle-tname]-servlet.xml file will be used to create the beans defined, overriding the  definitions of any beans defined with the same name in the global scope
- The *<context:component-scan...>* tag will be use to activate Spring MVC annotation scanning capability which allows to make use of annotations like @Controller and @RequestMapping etc.
- The *InternalResourceViewResolver* will have rules defined to resolve the view names. As per the above defined rule, a logical view named **hello** is delegated to a view implementation located at */WEB-INF/jsp/hello.jsp* .

定义解析视图名称规则, 根据上述定义的规则, 名为hello的逻辑视图被委派给/WEB-INF/jsp/hello.jsp

自定义.jsp 文件名称和内容, 

#### 定义控制器

@Controller 注解 表明特定类供应控制器的作用 

在mvc配置文件中通过  **<context:component-scan />** 扫描相应的类包，就可以使一个 POJO 类变成一个可以处理 HTTP 请求的控制器

可以创建数量不限的控制器用于处理不头同的业务请求

@RequestMapping 注解将用于将URl 映射到要么整个类或特定处理程序方法

所以可以根据需要将 @RequestMapping 调整到合适的位置

<mvn:annotationDriven />  标签基本上是将你的Sprign上下文允许请求调度到控制器

 essentially sets you your Spring context to allow for dispatching requests to Controller

#### 一些简单的用法and 基础

##### 接受json数据的四种方式

**ReqeuestParam** 

![image-20210501005141126](../images/java_web/image-20210501005141126.png)

@RequestParam 

**实体类** 

@RequestBody 参数

![image-20210501005224867](../images/java_web/image-20210501005224867.png)

json对象 

**map接收**

json对象![image-20210501005257833](../images/java_web/image-20210501005257833.png)

**List接收**

![image-20210501005315987](../images/java_web/image-20210501005315987.png)

##### @Requesting 下方法的返回值

```java
@Controller
public class HelloController {
    @RequestMapping(value = "/hello" ,method = RequestMethod.GET)
    public String printHello(ModelMap model){
        model.addAttribute("message","Hello Spring MVC Framework !");
        return "hello";
    }
}
```

retrun "hello"; 返回的为hello.jsp页面 

##### 重定向

```java
@Controller
public class WebController {
   @RequestMapping(value = "/index", method = RequestMethod.GET)
   public String index() {
      return "index";
   }
   @RequestMapping(value = "/redirect", method = RequestMethod.GET)
   public String redirect() {
      return "redirect:finalPage";
   }
   @RequestMapping(value = "/finalPage", method = RequestMethod.GET)
   public String finalPage() {
      return "final";
   }
```

将 重定向到 fianl.jsp 

##### 静态页面

```xml
<mvc:resources mapping = "/pages/**" location = "/WEB-INF/pages/" />
   <mvc:annotation-driven/>
```

**<mvc：resources .... />**用于映射静态页面。的**映射**属性必须是一个Ant图案指定一个HTTP请求的URL模式。该**位置**属性必须指定为静态页面，包括images，stylesheets，JavaScript和其它静态内容的一个或多个有效的资源目录位置。可以使用逗号分隔的值列表来指定多个资源位置。

##### @RequestMapping

1. 映射请求  处理URL请求, 注解可以标记在类上或方法上 

2. Dispatcher 截获请求后就通过@RequestMapping 提供的映射信息确定请求所对应的处理方法

3. @RequestMapping中可以指定value（请求URL）、method（请求的方法）、params（请求参数）、heads（请求头），他们之间时    与的关系，使用多个条件联合查询，可使请求精确化。

4. 支持Ant风格的Url , Ant风格地址支持三种匹配符 ?  匹配文件名中的一个字符
   
    匹配文件名中的任意字符   * *: * * 匹配多层路径 

5. @Pathvariable 映射URL绑定的占位符
   
   通过@PathVariable可以将URL中占位符参数绑定到控制器处理方法的入参中

请求的URL为：

```html
<a href="helloworld/1">To success</a>  
```

，可将“1”传入到方法中

```java
  @RequestMapping("/helloworld/{id}")
  public String hello(@PathVariable("id") Integer id){
      System.out.println("helloWorld"+id);
      return "success";
  }
```

###### 映射请求参数、请求头

1. 使用@RequestParam 可以把请求参数传递给请求方法 value : 参数名, required: 是否必须
   
    请求Url: `<a href="helloworld?username=123">To success </a>`
   
   ```java
   @RequestMapping("/helloworld")
   public String hello(@RequestParam(value="username" required=false)String un){
           System.out.println("helloworld :"+ un);
           reuturn "success";
   }
   ```

2. 使用 @RequestHeader绑定请求报头信息，请求头包含了若干属性，服务器可据此获知客户端的信息。
   
   ```java
   @RequestMapping("/helloworld")
   public String hello(@RequestHeader("Accept-Encoding") String encoding){
           System.out.println("helloWorld:"+encoding);
   
            return "success";
   }
   ```

3. 使用@CookieValue绑定请求中的Cookie值
   
   ```java
   @RequestMapping("/helloworld")
   public String hello(@CookieValue("JSESSIONID") String sessionId){
        System.out.println("helloWorld:"+sessionId);
   
        return "success";
   }
   ```

4. 使用POJO对象绑定请求参数, SpringMvc会按照请求参数名和POJO属性名自动进行配置, 自动为该对象填充属性,支持联属性 
    1)表单, 此表单对应一个User 的JavaBean 
   
   ```xml
   <form action="testPojo" method="post">
          username:<input type="text" name="username"> <br>
          password:<input type="password" name="password"> <br>
          age:<input type="text" name="age"><br>
          <input type="submit" name="Submit">
   </form>
   ```
   
   2) 处理方法, 结果在控制台输出user : User[username=adfd, password=585, age=13]

```java
@RequestMapping("/testPojo")
public String testPojo(User user) {
    System.out.println(user);
    return "success";
}
```

5. MVC的Handler 可以接受Servlet API

HttpServletRequest, HttpServletResponse, HttpSession. java.serity.Principal , Locale , InpustSteam . OutputStream. Reader, Writer 

##### ModelMap

声明格式 

```
public class ModelMap extends LinkedHashMap<String,Object> 
```

ModelMap对象主要用于传递控制方法处理数据到结果页面, 也就是说我们把结果页面上需要的数据放到ModelMap对象中 即可, 他的作用类似于request对象的setArrtibute方法,用来在一个请求过程中处理数据 

通过 

addAttribute(String ley, Object value) ;

向页面传递参数    

在页面上可以通过el变量方式$key 或者bboss的一系列数据展示标签获取并展示modelmap中的数据 

##### ModelAndView

ModelAndView 可以作为返回值, 其包含 模型和数据信息, Springmvc 会将model 中的数据放到requests域 中 . 

添加数据模型 : ModelAndView addObject(String attributeName , Object attributeValue)

​                           ModelAndView addAllObject（Map<String,?> modelMap） 

设置视图:        void setView(View view)

​                         void setViewName(String ViewName)

example

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
        ModleAndView mv = new ModelAndiew("success");
        mv.addObject("time",new Date());
        return mv;
}
```

在目标也success.jsp 接收time 

```xml
time:<%=request.getAttribute("time") %>
```

MoudelAndView 对象有两个作用:

(1)设置转向地址, 着也是 ModelAndView 和 ModelMap 的主要区别 

```java
ModelAndview view = new ModelAndView("path:ok")
```

或者通过setViewName 的方法 

```java
public void setViewName(String viewName){.....}
```

(2). 将控制器方法中处理的结果数据传递到结果页面，也就是把在结果页面上需要的数据放到ModelAndView对象中即可，其作用类似于request对象的setAttribute方法的作用，用来在一个请求过程中传递处理的数据。通过以下方法向页面传递参数：

```java
public ModelAndView addObject(String attributeName, Object attributeValue){...}
public ModelAndView addObject(Object attributeValue){...}12
```

在页面上也是可以通过el表达式语言$attributeName等系列数据展示标签获取并展示ModelAndView中的数据。

使用方式

```java
  public ModelAndView  examehtod(String someparam){
      //构建ModelAndView 实例,并设置跳转地址
      ModelAndView view = new ModelAndView("path:handleok");
      //将数据放置到ModelAndView对象view 中, 第二个参数可以是任何java类型
      view.addObejt("可以",someparam);

      //返回ModelAndView 对象View 
      return view;

  }
```

##### Model

  Mode 是一个接口, 实现类为ExtendedMoudelMap , 继承了 ModelMap 类

```java
 public class ExtendedModelMap extens ModelMap implments Model 
```

##### 数据的转换 ,格式话, 校验

SpringMVC定义了3种类型的转换器接口:

- Conveet<S,T> 将S类型转换为T类型对象 
- ConverterFactory : 将相同系列多个"通知" Conterver封装在一起 
- GenericConverter: 会根据源类对象 及目标所在的宿主主类中的上下信息进行类型转换 

##### mvc 视图的流程图

[参考](https://blog.csdn.net/eson_15/article/details/51689023)

<img src="../images/java_web/image-20210407121859423.png" alt="image-20210407121859423" style="zoom:200%;" />

处理映射器 HanderMapping(不需要程序员开发)

作用:  根据请求的url查找Header 

3. 处理器适配器 HandlerAdapter(不需要开发)
   作用: 按照特定规则(HandlerAdapter 要求的规则) 去执行Handler

4. 处理器Handler`(需要开发`)
   注意: 编写Handler 时按照HandlerAdapter 的要求去做, 这样适配器才可以正确执行 Handler 

5. 视图 Vier(需要开发jsp)
   
    View是一个接口, 实现类支持不同的View 类型(jsp, freemarker, pdf)
   
    [注] 不需要开发的, 需要自己做一下配置

##### HandlerMapping

处理器映射器  将bean的name 作为url进行查找, 也就是说在配置 Hander时需要在其 bean 中加入name 

例  

handler 

```xml
<bean name="/my_url" class="location.Controller" />
```

hanlerMapping

```xml
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping/>
```

HandlerAdapter

```xml
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```

 以上三条配置 都要在 mvc-servlet.xml文件中配置 

##### InternalResourceViewResoulver

试图解析器 可以不做任何限制 ,

也可以添加 标签配置 

```xml
 <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp"/>
    </bean>
```

常用配置为上面 

配置"sufiix"后 对jsp文件引用时不需要 在加上.jsp 和 /WEB-INF/jsp/ 文件目录 

#### 文件上传

[参考](https://blog.csdn.net/suifeng3051/article/details/51659731)

对于图片, 视频等二进制数据处理需要用到myltipart表单,  multipart表单和上面介绍的普通表单不同，它会把表单分割成块，表单中的每个字段对应一个块，每个块都有自己的数据类型。也就是说，对于上传字段对应的块，它的数据类型就可以是二进制了：

现文件上传，其实就是解析一个Mutipart请求。DispatchServlet自己并不负责去解析mutipart 请求，而是委托一个实现了`MultipartResolver`接口的类来解析mutipart请求。 spring3.1后提供了两个现成的MultipartResolver 接口的实现类

添加bean

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.support.StandardServletMultipartResolver"/> 
```

配置Dispatchcher ![image-20210415214735719](../images/java_web/image-20210415214735719.png)

@Conroller 层

 File.separator 静态变量   代表 系统目录中的分隔符

pathSeparatorChar
public static final char pathSeparatorChar

与系统有关的路径分隔符。此字段被初始为包含系统属性 path.separator 值的第一个字符。此字符用于分隔以路径列表 形式给定的文件序列中的文件名。在 UNIX 系统上，此字段为 ‘:’；在 Microsoft Windows 系统上，它为 ‘;’。

pathSeparator
public static final String pathSeparator

与系统有关的路径分隔符，为了方便，它被表示为一个字符串。此字符串只包含一个字符，即 pathSeparatorChar。

#### 跨域

同一协议，同一ip，同一端口，三同中有一不同就产生了跨域。

##### 后端解决

在Cpntroller 类上添加一个@CrossOrigin 注解(允许所有ip跨域访问) 就可以实现对当前controller的跨域访问,  也可以在方法上使用

或者 CSRF(Cross-site request forgey) , 跨站请求维造 ,也被称为：one click attack/session riding，缩写为：CSRF/XSRF。

建立跨域配置文件

```java
@Configuration
public class CorsConfiguration {
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**")
                        .allowCredentials(false)
                        .allowedMethods("POST", "GET", "PUT", "OPTIONS", "DELETE")
                        .allowedOrigins("*");
            }
        };
    }

}
```

### 注解

#### @Component

把普通pojo实例化spring容器中

泛指各种组件, 党我们的类不属于各种归类的时候(不属于@Controller . @Service等),我们可以使用Component标注这个类 

#### @ComponentScan

主要用发是扫描指定的包下面的bean对象  , 可以配置多个 

springboot默认扫描启动类所在的包下面的所有bean , 还需要额外可以指定别的package ,需要新增ComponentScan, 手动指定springboot所在类的package的路径, 要不然不会被加载 

### Spring AOP

[参考](https://www.jianshu.com/p/994027425b44)

首先，在面向切面编程的思想里面，把功能分为核心业务功能，和周边功能。

- **所谓的核心业务**，比如登陆，增加数据，删除数据都叫核心业务
- **所谓的周边功能**，比如性能统计，日志，事务管理等等

周边功能在Spring的AOP思想里,被定义为切面

核心功能和切面功能分别独立进行开发, 然后把切面功能和核心业务功能"编织"在一起,叫AOP

AOP 当中的概念：

- 切入点（Pointcut）
     在哪些类，哪些方法上切入（**where**）
- 通知（Advice）
     在方法执行的什么实际（**when:**方法前/方法后/方法前后）做什么（**what:**增强的功能）
- 切面（Aspect）
     切面 = 切入点 + 通知，通俗点就是：**在什么时机，什么地方，做什么增强！**
- 织入（Weaving）
     把切面加入到对象，并创建出代理对象的过程。（由 Spring 来完成）

选择好连接点后可以创建切面, 可以把切面理解为一个拦截器 , 当程序运行到连接点的时候, 被拦截下来, 在开头加入了初始化的方法, 在结尾加入了销毁的方法而已 , 在Spring 中只要使用@Aspect注解一个类 , 那么Spring Ioc容器就会认为这是一个切面了 

被定义为切面的类仍然是一个Bean, 需要@Component 注解标注 

| 注解                | 说明                                        |
|:----------------- |:----------------------------------------- |
| `@Before`         | 前置通知，在连接点方法前调用                            |
| `@Around`         | 环绕通知，它将覆盖原有方法，但是允许你通过反射调用原有方法，后面会讲        |
| `@After`          | 后置通知，在连接点方法后调用                            |
| `@AfterReturning` | 返回通知，在连接点方法执行并正常返回后调用，要求连接点方法在执行过程中没有发生异常 |
| `@AfterThrowing`  | 异常通知，当连接点方法异常时调用                          |

定义切点  execution正则表达式

```java
execution(*pojo.Landlord.service())
```

- execution：代表执行方法的时候会触发
- `*` ：代表任意返回类型的方法
- pojo.Landlord：代表类的全限定名
- service()：被拦截的方法名称

通过execution表达式,Spring会知道应该拦截pojo.Landlord类下的service()方法. 

可以通过使用@Pointcut注解来定义一个切点来解决需要写多个表达式的麻烦

#### 环绕通知

AOP中最强大的通知, 因为它集成了前置通知和后置通知,保留了连接点原有的方法的功能, 强大又灵活 

### Mybatis

[参考](https://juejin.cn/post/6844903555145400328#heading-13)

基本构成

![img](../images/java_web/16128007e21a883f.png)

- SqlSessionFactoryBuilder 

SqlSessionFactoryBuilder通常利用XML或者Java编码来构建SqlSessionFactory，一旦我们构建了SqlSessionFactory，它的作用也就完结了。上面的例子中也就是简单的读取配置文件，转化为流对象，再创建出SqlSessionFactory。因为它的主要目的是创建SqlSessionFactory，所以它一般也只存活在创建的那个方法里，声明周期很短。

- SqlSessionFactory

SqlSessionFactory是一个工厂类，它的作用就是生产SqlSession，而SqlSession你可以想象成JDBC里的一个Connection，每次访问数据库都需要一个SqlSession，所以SqlSessionFactory通常伴随整个Mybatis的生命周期，我们通常为一个数据库创建一个SqlSessionFactory，因为如果创建多个，那么每次创建新的，它都会附带着打开一堆Connection资源，所以通常我们使用单例模式管理SqlSessionFactory。

- Mapper

Mapper也就是上面我们写的`CommodityDao commodityDao = sqlSession.getMapper(CommodityDao.class);`,我们仅仅使用Java接口和一个XML文件（或者注解）去实现Mapper而不用实现类，其实这里运用到了动态代理，Mybatis会为接口生成代理类对象，代理对象会根据“接口路径+方法名”去匹配，找到对应的XML文件（或注解）去完成它所需要的任务，返回我们需要的结果。Mapper是一个方法级别的东西，它的最大生命周期应该和SqlSession是相同的。

#### 输入映射

为 接口方法传入参数, 参数与sql语句中的参数保持一致 

当只有一个参数时 , 只需要在方法中设定参数 , 如果需要传递多个参数, 就需要用到输入映射 

单个参数时 Mybatis 会帮助我们完成自动映射, 只要返回的SQL列名和JavaBean的属性一致, Mybatis就可以帮助我们自动回填这些字段而无需任何配置 , 可以在Mybatis的配置文件里,通过autoMappingBehavior 来设置自动映射, 它包含3个值：

- **NONE**：取消自动映射
- **PARTIAL**：这也是默认值，只会自动映射，没有定义嵌套结果集映射的结果集
- **FULL**：会自动映射任意复杂的结果集（无论是否嵌套）

传递多个参数通常由3种方式 ,通过Map,通过注解, 通过POJO对象 

##### 通过Map

假设我们想查询带有**饼**的食物，并且价格低于80的。那么我们可以这么写SQL语句

```xml
<select id="getCommodityByMap" parameterType="map" resultType="com.shuqing28.pojo.Commodity">
                SELECT * FROM commodity WHERE name like '%${name}%' AND price<#{price}
        </select>
```

mybatis使用xml定义Mapper,所以'<' 使用了转义符 `&lt;`表示 parameterType(参数类型) 定义为map  ,接口也介意Map为参数 

```java
List<Commodity> getCommodityByMap(Map<String, Object> params); 
```

```java
CommodityDao commodityDao = sqlSession.getMapper(CommodityDao.class);
Map<String, Object> paramsMap = new HashMap<String , Object>();
paramsMap.put("name", "饼");
paramsMap.put("price", 80.0);
List<Commodity> commodities = commodityDao.getCommodityByMap(paramsMap);
```

Map 需要一项项指定Key的名字, 不方便 

##### 注解方式

注解方式使用了**@Param**注解 ,

```java
List<Commodity> getCommodityByAnnation(@Param("name") String name, @Param("price") Double price);
```

根据 @Param提供的名称, 把变量值传入到Sql语句种对应的参数中, 可读性提高, 适合于参数不多的情况 

##### POJO 传递

通常我们只需要传递简单的POJO对象, 比如 

```xml
<select id="getCommodityByPOJO" parameterType="com.shuqing28.pojo.Commodity" resultType="com.shuqing28.pojo.Commodity">
    SELECT * FROM commodity WHERE name like '%${name}%' AND price<#{price}
</select>
```

- **id**标识了这条sql，id与接口名保持一致
- **parameterType**定义参数类型，这里就是int
- **resultType**定义了返回值类型，这里是我们定义的POJO类

测试 

```java
@Test
public void getLowPriceCommodityByPOJO(){
    SqlSession sqlSession = sqlSessionFactory.openSession();
    try {
        CommodityDao commodityDao = sqlSession.getMapper(CommodityDao.class);
        Commodity commodity = new Commodity();
        commodity.setName("饼");
        commodity.setPrice(80.0);
        List<Commodity> commodities = commodityDao.getCommodityByPOJO(commodity);
        System.out.println(commodities);
    } finally {
        sqlSession.close();
    }
}
```

可以通过别名减少名称的长度 , 需要在 mybatis-config.xml中的configuration 中定义

```xml
<typeAliases>
        <typeAlias alias="commodity" type="com.shuqing28.pojo.Commodity" />  <!--your.class.location-->
</typeAliases>
```

可以用commodity替换 ,但不能有重名

##### #和$的区别

Mybatis本身是基于JDBC封装的. 所以#{para}是预编译处理(PreparedStatement) . 相当于 原来的 ?, 而${para}是字符串替换, Mybatis在处理#时, 会调用PreparedStatement 的set系列 方法来赋值; 处理$时, 就是把${para}替换成变量的值. #方式能够很大程度防止slq 注入, $方式一般用于传入数据库对象, 例如传入表名, 一般能用#的就别用$

#### 输出映射

Mybatis 的输出映射分两种, resultType和resultMap  

##### resultType

可以是简单类型,或者POJO对象或者列表   

##### 单个对象

```
Commodity getCommodityById(Integer id);
```

Mybatis根据接口返回值判断返回一条对象，如果用过ibatis的可以知道内部调用了session.selectOne。

##### 返回列表

```
List<Commodity> getCommodityByPOJO(Commodity commodity);
```

内部使用session.selectList，Mapper接口使用`List<XXX>`作为返回值。

查询出来的列名和POJO属性只要有一个一致就会创建POJO对象, 最多别的字段为默认值 ,但如果全部不一致, 就不会创建该POJO对象 

##### resultMap

上面的resultType在查询的列名和POJO属性值一致的时候才可以映射成功，如果不一致的话，就需要resultMap登场表演了。

如果查询出来的列名和POJO的属性名不一致，通过定义一个resultMap对列名和POJO属性名之间作一个映射关系。

在使用resultMap前我们需要定义resultMap 假设我们查询商品类的两个字段id和name，但是查询的时候定义为`id_`和`name_`。列名和属性名不一致了，先定义resultMap

```
<resultMap id="commodityResultMap" type="commodity">
        <id column="id_" property="id"/>
        <result column="name_" property="name"/>
    </resultMap>
```

其中

- **id**标识这个resultMap
- **type**标识映射哪个POJO里面的属性，这里因为上面说过别名了，所以就是映射的Commodity这个POJO
- **id**元素标识这个对象的主键，result就是普通字段
- **property**代表POJO的属性名称
- **column**表示数据库SQL的列名

再看我们的select元素：

```
<select id="getCommodityByPrice" parameterType="double" resultMap="commodityResultMap">
       SELECT id id_, name name_ FROM USER WHERE price=#{price}
</select>
```

使用resultType进行输出映射时，只有查询出来的列名和pojo中的属性名一致，该列才可以映射成功。如果查询出来的列名和pojo的属性名不一致，通过定义一个resultMap对列名和pojo属性名之间作一个映射关系。 此外，resultMap还可以做一对多、多对多等高级映射

#### 高级映射

https://juejin.cn/post/6844903558488277006

#### toString()

在使用实体类对象接受到数据库中数据时, 最好把pojo中实体类 都加上toString() 方法, 方便对对象中内容的输出, 否则会输出对象的地址

没加![image-20210421212203811](../images/java_web/image-20210421212203811.png)

加上后

![image-20210421212248441](../images/java_web/image-20210421212248441.png)

通过Debug调试

![image-20210421212418740](../images/java_web/image-20210421212418740.png)

以上没加toString() 方法

加了后 ,方便对数据的读取 

![image-20210421212520909](../images/java_web/image-20210421212520909.png)

#### sql

基于xml的 sql可以加入标签进行 sql语句的定义和 去掉重复的代码, 实现 sql语句的动态性 

#### 在boot中配置xml路径

```
mybatis.mapper-locations=classpath:mapper/*.xml
```

## spring boot

 开始学习时间2021.4.18  立下 flag  5.28之间整体结构理清并写出一个web项目 

完成大作业并 在期末考试前面试暑期实习  4 级考试 

SpringBoot 默认只扫描 启动类() main) 所在包, 

### 文件结构

controller package和其它package在无配置注解扫描的情况下应该和 Application在同级目录下 

#### 配置 yml文件

使用.yml文件作为springboot 中 .properties文件的替代 

pom中加入依赖

![image-20210420195618627](../images/java_web/image-20210420195618627.png)

#### resources

templates 文件夹下存放的为模板文件,  可以通过 @Controller层 return  "name" 访问 

name.html 文件  

需要加依赖

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
```

#### templates文件夹

存放springbot的动态页面 

通过@controller 访问到 template下的html页面 

使用 Thymeleaf 语法写 页面

### 配置注解

#### 将类声明为Bean对象

@Component :  不确定组件   比如pojo 实例化到spring容器  作用域默认为singleton

@Service : 业务逻辑层(Service层)使用 

@Repositpry 在数据访问层(dao层)使用 

@Controller 用于标注控制层组件 

@RestConroller  作用近似于 @Controller + @ResponseBody 

#### Controller

使用@controller 注解需要设定方法返回值为模板类型 

```java
@RestController
```

@Controller 和 @Response 注解的结合 ,返回json对象

后端与前端分离 , 一般后端返回的为 json对象, 前端通过框架处理 json数据显示在页面上 

#### @RequestMapping

配置在类上的映射要在类内方法的前面

如

```java
@RestController
@RequestMapping(value = "index")
public class IndexController {


    @RequestMapping(value = "/test")
    public String index(){
        return "test";
    }

    @RequestMapping(value = {"/Variavle/{id}"} ,method= RequestMethod.GET)
    public String getUrlVariable(@PathVariable("id")Integer id) {
        return "id"+id;
     //   return "hello world";
    }
    @RequestMapping(value = "/hello")
    public String hello(){
        return "hello";
    }


}
```

访问 index() 方法的页面要通过 localhost:8080/index/test   web的根url 为 " / "  , 一般method 选择 get/post

##### 处理web请求参数的注解

- @PathVariable 获取url地址中的数据 /name/id    

- @RequestParam   获取请求参数的值   /name?id= 

```
@RequestParam(name="user",default="xxx初始化")String user
```

使用 map 接收多个map类型 

```
@PostMapping("test")
public  String login(@RequestParam Map <String,Object> params){
    return "name:" + params.get("name");
}
```

- @GetMapping 组合注解  @RequestMapping(method = RequestMethod.GET)缩写 

#### @SpringBootApplication

```
@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan
```

@EnableAutoConfiguration : 启动SpringBoot 自动装配机制 

@Configuration: 允许在上下文中注册额外的bean或导入其它    配置类 

@Component : 扫描被@Commponent(@Service, @Controller)注解的bean , 注解会默认扫描启动类所在的包下所有类 ,可以自定义不扫描某些bean

![img](../images/java_web/bcc73490afbe4c6ba62acde6a94ffdfd~tplv-k3u1fbpfcp-watermark.png)

容器排除 TypeExcludeFilter 和 AutoConfigurtionExcludeFilter 

@ComponentScan

#### @RequestBody

将HttpRequest body 中的JSON类型数据反序列化为合适的Java类型   ,  以实体类方式接受json数据

#### @Configuation

标注在类上,相当于把该类作为Spring的xml文件中的<beans>, 作用为: 配置spring容器

#### @Bean

@Bean标注在返回实例的方法上, 等价于<bean>, 注册为bean对象

```
public class Test{
    @Bean
    @Scope("protorype")
    public  A test(){
        return new A();
    }

}
```

*//@Bean注解注册bean,同时可以指定初始化和销毁方法*    *//@Bean(name="testNean",initMethod="start",destroyMethod="cleanUp")*

#### @Async

异步查询结果 

![image-20210428232251726](../images/java_web/image-20210428232251726.png)

### 读取配置

#### @configurationProperties 读取并于bean绑定

```ag-0-1g2gj2fnjag-0-1g2gjag-1-1g2gj2fnj2fnjag-0-1g2gjag-1-1g2gj2fnj2fnjag-0-1g2gjag-1-1g2gj2fnj2fnjymlag-1-1g2gj2fnj
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

![image-20210420200151167](../images/java_web/image-20210420200151167.png)

spring加载配置文件的优先级

![img](../images/java_web/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392d31312f726561642d636f6e6669672d70726f706572746965732d7072696f726974792e6a7067.png)

### 类

#### ResponseEntity

表示整个HTTP Response : 状态码 ,表头和正文内容. 可以使用它来定义HTTP Response 的内容 

#### spring-boot-starter-Mail 邮件发送

[参考](https://mp.weixin.qq.com/s?__biz=MzI1MDIxNjQ1OQ==&mid=2247483764&idx=1&sn=8cee8b1781b8659b3fdc23c0d650db49&chksm=e984e810def3610640e741eea1f94bbf95d4a2a5b1c68946829a036b4a6bc58b0d23e156474a&cur_album_id=1361542504662908929&scene=189#rd)

```java
JavaMailSender
```

简单用法

配置 文件

```
spring.mail.host=smtp.qq.com
spring.mail.port=587
# 你的邮箱地址
spring.mail.username=2233451206@qq.com
#授权码
spring.mail.password=xvkqcuzqfgorebfa
spring.mail.default-encoding=UTF-8
```

```java
@Service
@Slf4j
public  class MailService {

    @Value("${spring.mail.username}")
    private String from;
    @Autowired
    private JavaMailSender mailsender;

    public void  sendSmipleTextMail(String to , String subject, String content){
        SimpleMailMessage message = new SimpleMailMessage( );
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        message.setFrom(from);
        mailsender.send(message);
        log.info("[文本邮件]发送成功! to={ }",to);
    }
}
```

Test

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {

    @Autowired
    private MailService mailService;
    @Autowired
    private TemplateEngine templateEngine;

    @Test
    public void sendSimpleTextMailTest(){
        String to = "2233451206@qq.com";
        String subject = "Spring boot 发送简单邮件";
        String contentment = "<p>第一封SpringBoot 简单文本邮件</p>";
        mailService.sendSmipleTextMail(to,subject,contentment);
    }
}
```

to为收件人地址 

![image-20210422161537587](../images/java_web/image-20210422161537587.png)

##### 发送html文件

使用 

```
import javax.mail.internet.MimeMessage; 
```

```java
   public void SendHtmlMail(String to,String subject,String content ) throws MessagingException {
        MimeMessage message = mailsender.createMimeMessage();
        MimeMessageHelper messageHelper = new MimeMessageHelper(message,true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        //true 为HTML 邮件
        messageHelper.setText(content,true)
            ;
    }
```

发送图片

```java
  /**
     * 发送带图片的邮件
     *
     * @param to
     * @param subject
     * @param content
     * @param imgMap
     */
    public void sendImgMail(String to, String subject, String content, Map<String, String> imgMap)
        throws MessagingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, true);
        messageHelper.setFrom(from);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        messageHelper.setText(content, true);
        // 添加图片
        for (Map.Entry<String, String> entry : imgMap.entrySet()) {
            FileSystemResource fileResource = new FileSystemResource(new File(entry.getValue()));
            if (fileResource.exists()) {
                String filename = fileResource.getFilename();
                messageHelper.addInline(entry.getKey(), fileResource);
            }
        }
        mailSender.send(mimeMessage);
        log.info("【图片邮件】成功发送！to={}", to);
    }
```

测试类中方法

```java
 @Test
    public void sendImgTest() throws MessagingException {
        String to = "niumoo@126.com";
        String subject = "Springboot 发送 HTML 图片邮件";
        String content =
            "<h2>Hi~</h2><p>第一封 Springboot HTML 图片邮件</p><br/><img src=\"cid:img01\" /><img src=\"cid:img02\" />";
        String imgPath = "apple.png";
        Map<String, String> imgMap = new HashMap<>();
        imgMap.put("img01", imgPath);
        imgMap.put("img02", imgPath);
        mailService.sendImgMail(to, subject, content, imgMap);
    }
```

##### mail模板邮件

模板邮件用处很广泛,注册成功邮件或者是操作通知邮件等都是模板邮件, 模板邮件往往只需更改其中的几个变量, Springboot中的模板邮件首选需要选择一款模板引擎 , 比如 Thymeleaf 

### 微服务

如果将所有微服务放在一台服务器上部署或者在本地运行时, 注意端口号(server.port) 不要重复, 否则会报端口被占用的错误

服务注册就是维护一个登记簿, 它管理系统内所有的服务地址. 当新的服务启动后, 它回想登记簿交代自己的地址信息. 服务的依赖方直接向登记簿索要 Service Provider 地址就行 , 

当前用于服务注册的工具非常多  ooKeeper，Consul，Etcd, 还有 Netflix 家的 eureka 等 

服务注册 

- 客户端注册 
- 第三方注册

微服务就是很小的服务, 小到一个服务只对应一个单一的共能, 只做一件事 .这个服务可以单独部署运行, 服务之间可以通过RPC(远程调用)来相互交互

#### 微服务架构

在做架构设计的时候，先做逻辑架构，再做物理架构，当你拿到需求后，估算过最大用户量和并发量后，计算单个应用服务器能否满足需求，如果用户量只有几百人的小应用，单体应用就能搞定，即所有应用部署在一个应用服务器里，如果是很大用户量，且某些功能会被频繁访问，或者某些功能计算量很大，建议将应用拆解为多个子系统，各自负责各自功能，这就是微服务架构

#### 客户端注册

客户端注册是服务自身要负责注册与注销的工作, 当服务启动后向注册中心注册自身,当服务下线时注销自己. 期间还需要和注册中心保持心跳 

#### 第三方注册(独立的服务Registrar)

第三方注册由一个独立的服务Registar负责注册与注销, 当服务启动后以某种方式通知Registrar,然后Registar 负责向注册中心发起注册工作. 同时注册中心要维护与服务之间的心跳, 当服务不可用时, 向注册中心注销服务, 这种方式的缺点是 Registar 必须是一个高可用的系统, 否则注册工作没法进展 

#### 客户端发现

客户端发现是指负责查询可用服务地址, 以及负载均衡的工作. 这种方式最直接, 而且也方便做负载均衡. 再者一旦发现某个服务不可用立即换另外一个，非常直接。缺点也在于多 语言时的重复工作，每个语言实现相同的逻辑

![image-20210426234601269](../images/java_web/image-20210426234601269.png)

#### 服务端发现

服务端发现需要额外的Router 服务, 请求先打到Router, 然后Router 负责查询服务与负载均衡. 这种方式虽然没有客户端发现的缺点, 但是它的缺点是Router 的高可用

![image-20210426234614452](../images/java_web/image-20210426234614452.png)

#### Eureka

Spring Cloud Eureka 可以快速实现服务注册与发现 

![image-20210426233731896](../images/java_web/image-20210426233731896.png)

##### 客户端

```
eureka:
  instance:
    lease-renewal-interval-in-seconds: 20  #发送心跳, 表示当前服务没有宕机
  client:
    register-with-eureka: true  # 注册到eureka 服务上, 默认为true , 这里是eureka Server
    fetch-registry: true   # 是否从eureka服务上获取注册信息 ,这里是单点, 不用同步其它eureka 服务节点上数据
    initial-instance-info-replication-interval-seconds: 30 # 新实例信息的变化到Eureka服务端的间隔时间，单位为秒
    registry-fetch-interval-seconds: 10  # 获取服务并缓存
    service-url:
      defaultZone: http://localhost:8761/eureka
```

#### 消息队列

将用户请求数据写入消息队列, 可以将订单数据返回给用户, 等消息队列中的请求数据进程处理完订单后, 甚至出库后, 再将订单成功的消息通过邮件或者短信通知用户订单成功

使用消息队列能为我们的系统带来下面三点好处 

1. **通过异步处理提高系统性能（减少响应所需时间）。**
2. **削峰/限流**
3. **降低系统耦合性。**

结合自己项目进行回答

##### **削峰/限流**

**先将短时间高并发产生的事务消息存储在消息队列中，然后后端服务再慢慢根据自己的能力去消费这些消息，这样就避免直接把后端服务打垮掉。**

举例：在电子商务一些秒杀、促销活动中，合理使用消息队列可以有效抵御促销活动刚开始大量订单涌入对系统的冲击。如下图所示：

![削峰](../images/java_web/%E5%89%8A%E5%B3%B0-%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97.png)

##### 降低系统耦合性

 使用消息队列还可以降低系统耦合性。我们知道如果模块之间不存在直接调用，那么新增模块或者修改模块就对其他模块影响较小，这样系统的可扩展性无疑更好一些。还是直接上图吧：

![解耦](../images/java_web/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97-%E8%A7%A3%E8%80%A6.png)

生产者（客户端）发送消息到消息队列中去，接受者（服务端）处理消息，需要消费的系统直接去消息队列取消息进行消费即可而不需要和其他系统有耦合， 这显然也提高了系统的扩展性。

**消息队列使利用发布-订阅模式工作，消息发送者（生产者）发布消息，一个或多个消息接受者（消费者）订阅消息。** 从上图可以看到**消息发送者（生产者）和消息接受者（消费者）之间没有直接耦合**，消息发送者将消息发送至分布式消息队列即结束对消息的处理，消息接受者从分布式消息队列获取该消息后进行后续处理，并不需要知道该消息从何而来。**对新增业务，只要对该类消息感兴趣，即可订阅该消息，对原有系统和业务没有任何影响，从而实现网站业务的可扩展性设计**。

消息接受者对消息进行过滤、处理、包装后，构造成一个新的消息类型，将消息继续发送出去，等待其他消息接受者订阅该消息。因此基于事件（消息对象）驱动的业务架构可以是一系列流程。

另外，为了避免消息队列服务器宕机造成消息丢失，会将成功发送到消息队列的消息存储在消息生产者服务器上，等消息真正被消费者服务器处理后才删除消息。在消息队列服务器宕机后，生产者服务器会选择分布式消息队列服务器集群中的其他服务器发布消息。

**备注：** 不要认为消息队列只能利用发布-订阅模式工作，只不过在解耦这个特定业务环境下是使用发布-订阅模式的。除了发布-订阅模式，还有点对点订阅模式（一个消息只有一个消费者），我们比较常用的是发布-订阅模式。另外，这两种消息模型是 JMS 提供的，AMQP 协议还提供了 5 种消息模型。

##### 带来的问题

- **系统可用性降低：** 系统可用性在某种程度上降低，为什么这样说呢？在加入 MQ 之前，你不用考虑消息丢失或者说 MQ 挂掉等等的情况，但是，引入 MQ 之后你就需要去考虑了！
- **系统复杂性提高：** 加入 MQ 之后，你需要保证消息没有被重复消费、处理消息丢失的情况、保证消息传递的顺序性等等问题！
- **一致性问题：** 我上面讲了消息队列可以实现异步，消息队列带来的异步确实可以提高系统响应速度。但是，万一消息的真正消费者并没有正确消费消息怎么办？这样就会导致数据不一致的情况了!

JMS （JAVA Message Service,java 消息服务）是 java 的消息服务 

 AMQP ，即 Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准 **高级消息队列协议**（二进制应用层协议），是应用层协议的一个开放标准,为面向消息的中间件设计，兼容 JMS。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件同产品，不同的开发语言等条件的限制。

#### oshi(工具包)

监控本机状态, 内存和CPU使用率, 磁盘和 分区使用情况, 设备 , 传感器等

项目监控功能

 引入依赖

Maven

```
<!-- https://mvnrepository.com/artifact/com.github.oshi/oshi-core -->
<dependency>
    <groupId>com.github.oshi</groupId>
    <artifactId>oshi-core</artifactId>
    <version>5.2.5</version>
</dependency>
```

Gradle

```
// https://mvnrepository.com/artifact/com.github.oshi/oshi-core
compile group: 'com.github.oshi', name: 'oshi-core', version: '5.2.5'
```

 功能演示

获取硬件信息对象`HardwareAbstractionLayer` ：

```
//系统信息
SystemInfo si = new SystemInfo();
//操作系统信息
OperatingSystem os = si.getOperatingSystem();
//硬件信息
HardwareAbstractionLayer hal = si.getHardware();
```

有了代表硬件信息的对象`HardwareAbstractionLayer` 之后，我们就可以获取硬件相关的信息了！

下面简单演示一下获取内存和 CPU 相关信息。

**1.获取内存相关信息**

```
//内存相关信息
GlobalMemory memory = hal.getMemory();
//获取内存总容量
String totalMemory = FormatUtil.formatBytes(memory.getTotal());
//获取可用内存的容量
String availableMemory = FormatUtil.formatBytes(memory.getAvailable());
```

有了内存总容量和内存可用容量，你就可以计算出当前内存的利用率了。

**2.获取 CPU 相关信息**

```
//CPU相关信息
CentralProcessor processor = hal.getProcessor();
//获取CPU名字
String processorName = processor.getProcessorIdentifier().getName();
//获取物理CPU数
int physicalPackageCount = processor.getPhysicalPackageCount();
//获取物理核心数
int physicalProcessorCount = processor.getPhysicalProcessorCount();
```

#### EasyExcel (工具包)

快速,简单避免OOM的java 处理Excel 工具

 引入依赖

Maven

```
<!-- https://mvnrepository.com/artifact/com.alibaba/easyexcel -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.2.6</version>
</dependency>
```

Gradle

```
// https://mvnrepository.com/artifact/com.alibaba/easyexcel
compile group: 'com.alibaba', name: 'easyexcel', version: '2.2.6'
```

 功能演示

这里直接分享官方提供读取 Excel 的例子（*在实际项目中我对这部分做了简单的封装，涉及的地方比较多，就不分享出来了*）。

**实体对象** （Excel 导入导出实体对象）

```
@Data
public class DemoData {
    private String string;
    private Date date;
    private Double doubleData;
}
```

**监听器** （自定义 `AnalysisEventListener` 一次读取 5 条数据存储到数据库）

```
// 有个很重要的点 DemoDataListener 不能被spring管理，要每次读取excel都要new,然后里面用到spring可以构造方法传进去
public class DemoDataListener extends AnalysisEventListener<DemoData> {
    private static final Logger LOGGER = LoggerFactory.getLogger(DemoDataListener.class);
    /**
     * 每隔5条存储数据库，实际使用中可以3000条，然后清理list ，方便内存回收
     */
    private static final int BATCH_COUNT = 5;
    List<DemoData> list = new ArrayList<DemoData>();
    /**
     * 假设这个是一个DAO，当然有业务逻辑这个也可以是一个service。当然如果不用存储这个对象没用。
     */
    private DemoDAO demoDAO;
    public DemoDataListener() {
        // 这里是demo，所以随便new一个。实际使用如果到了spring,请使用下面的有参构造函数
        demoDAO = new DemoDAO();
    }
    /**
     * 如果使用了spring,请使用这个构造方法。每次创建Listener的时候需要把spring管理的类传进来
     *
     * @param demoDAO
     */
    public DemoDataListener(DemoDAO demoDAO) {
        this.demoDAO = demoDAO;
    }
    /**
     * 这个每一条数据解析都会来调用
     *
     * @param data
     *            one row value. Is is same as {@link AnalysisContext#readRowHolder()}
     * @param context
     */
    @Override
    public void invoke(DemoData data, AnalysisContext context) {
        LOGGER.info("解析到一条数据:{}", JSON.toJSONString(data));
        list.add(data);
        // 达到BATCH_COUNT了，需要去存储一次数据库，防止数据几万条数据在内存，容易OOM
        if (list.size() >= BATCH_COUNT) {
            saveData();
            // 存储完成清理 list
            list.clear();
        }
    }
    /**
     * 所有数据解析完成了 都会来调用
     *
     * @param context
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 这里也要保存数据，确保最后遗留的数据也存储到数据库
        saveData();
        LOGGER.info("所有数据解析完成！");
    }
    /**
     * 加上存储数据库
     */
    private void saveData() {
        LOGGER.info("{}条数据，开始存储数据库！", list.size());
        demoDAO.save(list);
        LOGGER.info("存储数据库成功！");
    }
}
```

**持久层** (mybatis 或者 jpa 来做都行)

```
/**
 * 假设这个是你的DAO存储。当然还要这个类让spring管理，当然你不用需要存储，也不需要这个类。
 **/
public class DemoDAO {
    public void save(List<DemoData> list) {
        // 如果是mybatis,尽量别直接调用多次insert,自己写一个mapper里面新增一个方法batchInsert,所有数据一次性插入
        System.out.println(list);
    }
}
```

**读取数据**

```
String fileName = "src/test/resources/demo/demo.xlsx";
// 这里 需要指定读用哪个class去读，然后读取第一个sheet 文件流会自动关闭
EasyExcel.read(fileName, DemoData.class, new DemoDataListener()).sheet().doRead();
```

**输出结果** （已过滤非必要数据）

```
2020-09-16 08:14:33.727 DEBUG [main] com.alibaba.excel.context.AnalysisContextImpl:91 - Began to read：ReadSheetHolder{sheetNo=0, sheetName='Sheet1'} com.alibaba.excel.read.metadata.holder.xlsx.XlsxReadSheetHolder@6f3b5d16
2020-09-16 08:14:33.870 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1577811661000,"doubleData":1.0,"string":"字符串0"}
2020-09-16 08:14:33.870 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1577898061000,"doubleData":2.0,"string":"字符串1"}
2020-09-16 08:14:33.871 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1577984461000,"doubleData":3.0,"string":"字符串2"}
2020-09-16 08:14:33.871 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578070861000,"doubleData":4.0,"string":"字符串3"}
2020-09-16 08:14:33.872 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578157261000,"doubleData":5.0,"string":"字符串4"}
2020-09-16 08:14:33.872 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:80 - 5条数据，开始存储数据库！
[DemoData(string=字符串0, date=Wed Jan 01 01:01:01 CST 2020, doubleData=1.0), DemoData(string=字符串1, date=Thu Jan 02 01:01:01 CST 2020, doubleData=2.0), DemoData(string=字符串2, date=Fri Jan 03 01:01:01 CST 2020, doubleData=3.0), DemoData(string=字符串3, date=Sat Jan 04 01:01:01 CST 2020, doubleData=4.0), DemoData(string=字符串4, date=Sun Jan 05 01:01:01 CST 2020, doubleData=5.0)]
2020-09-16 08:14:33.874 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:82 - 存储数据库成功！
2020-09-16 08:14:33.875 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578243661000,"doubleData":6.0,"string":"字符串5"}
2020-09-16 08:14:33.875 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578330061000,"doubleData":7.0,"string":"字符串6"}
2020-09-16 08:14:33.876 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578416461000,"doubleData":8.0,"string":"字符串7"}
2020-09-16 08:14:33.876 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578502861000,"doubleData":9.0,"string":"字符串8"}
2020-09-16 08:14:33.876 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:54 - 解析到一条数据:{"date":1578589261000,"doubleData":10.0,"string":"字符串9"}
2020-09-16 08:14:33.877 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:80 - 5条数据，开始存储数据库！
[DemoData(string=字符串5, date=Mon Jan 06 01:01:01 CST 2020, doubleData=6.0), DemoData(string=字符串6, date=Tue Jan 07 01:01:01 CST 2020, doubleData=7.0), DemoData(string=字符串7, date=Wed Jan 08 01:01:01 CST 2020, doubleData=8.0), DemoData(string=字符串8, date=Thu Jan 09 01:01:01 CST 2020, doubleData=9.0), DemoData(string=字符串9, date=Fri Jan 10 01:01:01 CST 2020, doubleData=10.0)]
2020-09-16 08:14:33.877 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:82 - 存储数据库成功！
2020-09-16 08:14:33.877 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:80 - 0条数据，开始存储数据库！
[]
2020-09-16 08:14:33.877 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:82 - 存储数据库成功！
2020-09-16 08:14:33.877 INFO [main] com.alibaba.easyexcel.test.demo.read.DemoDataListener:73 - 所有数据解析完成！

Process finished with exit code 0
```

### 分布式

分布式服务顾名思义服务是分散部署在不同的机器上的，一个服务可能负责几个功能，是一种面向SOA架构的，服务之间也是通过rpc来交互或者是webservice来交互的。逻辑架构设计完后就该做物理架构设计，系统应用部署在超过一台服务器或虚拟机上，且各分开部署的部分彼此通过各种通讯协议交互信息，就可算作分布式部署，生产环境下的微服务肯定是分布式部署的，分布式部署的应用不一定是微服务架构的，比如集群部署，它是把相同应用复制到不同服务器上，但是逻辑功能上还是单体应用。

![img](../images/java_web/c2VydmljZVR5cGU9MiZzZXJ2aWNlRGF0YT17Im5vdGVHdWlkIjoiMWYzOWNmM2EtNDc5OS00ZDYwLTlmNjUtMzhmNDA0MmFhZDhjIiwicmVzb3VyY0d1aWQiOiI1ZmU1MWU3Ni1mZDFiLTRmYTEtOWNlMS1kZDBlMTQ3MDczY2EifQ==.png)

由于笔记全反到一个md文件内不好管理和详细记录, 所以将 类型给出, 并指出相应的md文件 

#### redis

redis 时一种  开放源代码(BSD许可)的内存中数据结构存储, 用作 数据库, 缓存和 消息代理 , 

Redis 提供的数据结构 

![image-20210501233735056](../images/java_web/image-20210501233735056.png)

因为Redis 的特性 ,所以可以用来作为 分布式缓存 

[Redis详解](D:/program/writing/Redis.md)

#### 缓存

[参考](https://github.com/CyC2018/CS-Notes/blob/master/notes/%E7%BC%93%E5%AD%98.md#%E5%9B%9B%E7%BC%93%E5%AD%98%E9%97%AE%E9%A2%98)

对资源文件的提前加载, 或对用户实现请求时临时作为存储

##### 特征

命中率  

当某个请求能够通过访问缓存而得到响应时，称为缓存命中。

缓存命中率越高，缓存的利用率也就越高。

 最大空间

缓存通常位于内存中，内存的空间通常比磁盘空间小的多，因此缓存的最大空间不可能非常大。

当缓存存放的数据量超过最大空间时，就需要淘汰部分数据来存放新到达的数据。

 淘汰策略

- FIFO（First In First Out）：先进先出策略，在实时性的场景下，需要经常访问最新的数据，那么就可以使用 FIFO，使得最先进入的数据（最晚的数据）被淘汰。
- LRU（Least Recently Used）：最近最久未使用策略，优先淘汰最久未使用的数据，也就是上次被访问时间距离现在最久的数据。该策略可以保证内存中的数据都是热点数据，也就是经常被访问的数据，从而保证缓存命中率。
- LFU（Least Frequently Used）：最不经常使用策略，优先淘汰一段时间内使用次数最少的数据。

##### 位置

浏览器 

当Http响应允许进行缓存时, 浏览器会将HTML, CSS,JavaScript, 图片等静态资源进行缓存 

ISP

网络服务提供商(ISP) 是网络访问的第一跳, 通过将数据缓存在ISP中能够大大提高用户的访问速度 

反向代理

反向代理位于服务器之前，请求与响应都需要经过反向代理。通过将数据缓存在反向代理，在用户请求反向代理时就可以直接使用缓存进行响应。

本地缓存

使用 Guava Cache 将数据缓存在服务器本地内存中，服务器代码可以直接读取本地内存中的缓存，速度非常快。

分布式缓存

使用 Redis、Memcache 等分布式缓存将数据缓存在分布式缓存系统中。

相对于本地缓存来说，分布式缓存单独部署，可以根据需求分配硬件资源。不仅如此，服务器集群都可以访问分布式缓存，而本地缓存需要在服务器集群之间进行同步，实现难度和性能开销上都非常大。

数据库缓存

MySQL 等数据库管理系统具有自己的查询缓存机制来提高查询效率。

Java内部缓存

Java 为了优化空间，提高字符串、基本数据类型包装类的创建效率，设计了字符串常量池及 Byte、Short、Character、Integer、Long、Boolean 这六种包装类缓冲池。

CPU多级缓存

CPU 为了解决运算速度与主存 IO 速度不匹配的问题，引入了多级缓存结构，同时使用 MESI 等缓存一致性协议来解决多核 CPU 缓存数据一致性的问题。

### JPA

[参考](https://zhuanlan.zhihu.com/p/121786644)

springboot  中添加JPA依赖后需要创建实体类实现数据表的建立

在写mapper接口时不需要具体实现, 继承 CrudRepository 类 会自动实现接口内方法 

JPA对象将保存到数据库并从数据库中获取他们, 都不需要编写具体的存储库实现 

**原理**

​    是一种规范, Hibernate 是它的一种是实现

**能力**

定义了独特的JPQL（Java Persistence Query Language）,是EJB QL 的一种扩展, 针对实体的一种查询语言, 操作对象是实体, 而不是关系数据库的表, 支持 批量更新和修改 

 JPA中能够支持面向对象的高级特性 

需要注意的是 ,jpa可以自动创建 sql语句,也支持自定义sql , 自动创建sql是分析函数的命名去自动生成查询 , 

![image-20210429200647233](../images/java_web/image-20210429200647233.png)

 因为基于函数命名 所以再写函数时要注意名字 

,包括其它地方使用的时候也需要注意 

![image-20210501145935863](../images/java_web/image-20210501145935863.png)

jpa 自动生成sql语句时参数基于接口中的参数  JPA中Repository 接口中一般添加的为实例类和 主键为id 的类型, 

不包含没有声明过的类型 . 

jpaRepository 中扩展了其它的jp Repository, 提供了比较的sql 使用 

使用 spring Security时

访问网页会默认带访问限制 ,默认username 为: user  密码在控制台中随机生成

![image-20210424220230138](../images/java_web/image-20210424220230138.png)

#### 配置h2数据库

h2为内存数据库 

H2提供了一个名为H2 Console的Web界面来查看数据。让我们在application.properties中启用h2控制台。

/src/main/resources/application.properties中加入：

```javascript
spring.h2.console.enabled=true
```

访问http://localhost:8080/h2-console/ 直接点击进入

如果使用了spring-security , 则需要输入user 和 passwoed(自动给出)

或者自己配置

就H2而言，只要Spring Boot在类路径中看到H2，它就会自动配置类似于下面所示的数据源：

```javascript
      spring.datasource.url=jdbc:h2:mem:testdb      spring.datasource.driverClassName=org.h2.Driver      spring.datasource.username=sa      spring.datasource.password=
      spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
```

注意：JDBC URL默认是jdbc:h2:~/test，而Spring Boot的默认数据库url应该是jdbc:h2:mem:testdb，否则进去后找不到JPA创建的数据表PRODUCT:

使用SpringSecurity 的时候需要配置Securit 为 H2增加权限 

```java
  @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/h2-conslole/**").permitAll() //放行h2后台
                .and().csrf().ignoringAntMatchers("/h2-console/**")  // 禁用 H2-console csrf防护
                .and().headers().frameOptions().sameOrigin();  // 允许来自同一来源的 H2 控制台的请求
    }
```

#### sql语句

##### @Quety

[官方 ](https://docs.spring.io/spring-data/jpa/docs/2.5.0/reference/html/#jpa.query-methods.at-query)

###### like

![image-20210429000504417](../images/java_web/image-20210429000504417.png)

LIKE分隔符 : %

###### sort

```
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.lastname like ?1%")
  List<User> findByAndSort(String lastname, Sort sort);

  @Query("select u.id, LENGTH(u.firstname) as fn_len from User u where u.lastname like ?1%")
  List<Object[]> findByAsArrayAndSort(String lastname, Sort sort);
}

repo.findByAndSort("lannister", Sort.by("firstname"));                
repo.findByAndSort("stark", Sort.by("LENGTH(firstname)"));            
repo.findByAndSort("targaryen", JpaSort.unsafe("LENGTH(firstname)")); 
repo.findByAsArrayAndSort("bolton", Sort.by("fn_len"));               
```

###### 命名参数

```
public interface UserRepository extends JpaRepository<User, Long> {

  @Query("select u from User u where u.firstname = :firstname or u.lastname = :lastname")
  User findByLastnameOrFirstname(@Param("lastname") String lastname,
                                 @Param("firstname") String firstname);
}
```

带参数 

![image-20210428234407083](../images/java_web/image-20210428234407083.png)

使用 

![image-20210429000733994](../images/java_web/image-20210429000733994.png)

运行正常

###### Spel 表达式

![image-20210428235629062](../images/java_web/image-20210428235629062.png)

```java
@Entity
public class User {

  @Id
  @GeneratedValue
  Long id;

  String lastname;
}

public interface UserRepository extends JpaRepository<User,Long> {

  @Query("select u from #{#entityName} u where u.lastname = ?1")
  List<User> findByLastname(String lastname);
}
```

![image-20210428235834668](../images/java_web/image-20210428235834668.png)

###### 修改查询

![image-20210429162358896](../images/java_web/image-20210429162358896.png)

涉及到 update 或者delete, add等会修改数据的时候, 

需要加上@Modifying注解 

关键字 使用 ,可以在<named-query /> 中使用 

| Keyword                              | Sample                                                                                                               | JPQL snippet                                                                                  |
|:------------------------------------ |:-------------------------------------------------------------------------------------------------------------------- |:--------------------------------------------------------------------------------------------- |
| `And`和                               | `findByLastnameAndFirstname`findByLastnameAndFirstname                                                               | `… where x.lastname = ?1 and x.firstname = ?2`…其中x.lastname =？1和x.firstname =？2               |
| `Or`或者                               | `findByLastnameOrFirstname`findByLastnameOrFirstname                                                                 | `… where x.lastname = ?1 or x.firstname = ?2`…其中x.lastname =？1或x.firstname =？2                |
| `Is,Equals`是，等于                      | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals`findByFirstname，findByFirstnameIs，findByFirstnameEquals | `… where x.firstname = ?1`…其中x.firstname =？1                                                  |
| `Between`之间                          | `findByStartDateBetween`findByStartDateBetween                                                                       | `… where x.startDate between ?1 and ?2`…其中x.startDate在？1和？2之间                                 |
| `LessThan`少于                         | `findByAgeLessThan`findByAgeLessThan                                                                                 | `… where x.age < ?1`…其中x.age <？1                                                              |
| `LessThanEqual`小于等于                  | `findByAgeLessThanEqual`findByAgeLessThanEqual                                                                       | `… where x.age ⇐ ?1`…其中x.age⇐？1                                                               |
| `GreaterThan`比...更棒                  | `findByAgeGreaterThan`findByAgeGreaterThan                                                                           | `… where x.age > ?1`…其中x.age>？1                                                               |
| `GreaterThanEqual`大于等于               | `findByAgeGreaterThanEqual`findByAgeGreaterThanEqual                                                                 | `… where x.age >= ?1`…其中x.age> =？1                                                            |
| `After`后                             | `findByStartDateAfter`findByStartDateAfter                                                                           | `… where x.startDate > ?1`…其中x.startDate>？1                                                   |
| `Before`前                            | `findByStartDateBefore`findByStartDateBefore                                                                         | `… where x.startDate < ?1`…其中x.startDate <？1                                                  |
| `IsNull`一片空白                         | `findByAgeIsNull`findByAgeIsNull                                                                                     | `… where x.age is null`…其中x.age为空                                                             |
| `IsNotNull,NotNull`IsNotNull，NotNull | `findByAge(Is)NotNull`findByAge（Is）NotNull                                                                           | `… where x.age not null`…其中x.age不为null                                                        |
| `Like`喜欢                             | `findByFirstnameLike`findByFirstnameLike                                                                             | `… where x.firstname like ?1`…其中x.firstname像？1                                                |
| `NotLike`不喜欢                         | `findByFirstnameNotLike`findByFirstnameNotLike                                                                       | `… where x.firstname not like ?1`…其中x.firstname不喜欢？1                                          |
| `StartingWith`从...开始                 | `findByFirstnameStartingWith`findByFirstnameStartingWith                                                             | `… where x.firstname like ?1` (parameter bound with appended `%`)…其中x.firstname如？1（参数与附加的％绑定） |
| `EndingWith`结尾于                      | `findByFirstnameEndingWith`findByFirstnameEndingWith                                                                 | `… where x.firstname like ?1` (parameter bound with prepended `%`)…其中x.firstname如？1（参数与前缀％绑定） |
| `Containing`内含                       | `findByFirstnameContaining`findByFirstnameContaining                                                                 | `… where x.firstname like ?1` (parameter bound wrapped in `%`)…其中x.firstname像？1（参数绑定用％包裹）     |
| `OrderBy`按订单                         | `findByAgeOrderByLastnameDesc`findByAgeOrderByLastnameDesc                                                           | `… where x.age = ?1 order by x.lastname desc`…其中x.age =？1，由x.lastname desc排序                  |
| `Not`不是                              | `findByLastnameNot`findByLastnameNot                                                                                 | `… where x.lastname <> ?1`…其中x.lastname <>？1                                                  |
| `In`在                                | `findByAgeIn(Collection<Age> ages)`findByAgeIn（Collection <Age>年龄）                                                   | `… where x.age in ?1`…其中x.age在？1中                                                             |
| `NotIn`不在                            | `findByAgeNotIn(Collection<Age> age)`findByAgeNotIn（Collection <Age> age）                                            | `… where x.age not in ?1`…其中x.age不在？1中                                                        |
| `True`真的                             | `findByActiveTrue()`findByActiveTrue（）                                                                               | `… where x.active = true`…其中x.active = true                                                   |
| `False`错误的                           | `findByActiveFalse()`findByActiveFalse（）                                                                             | `… where x.active = false`…其中x.active = false                                                 |
| `IgnoreCase`忽略大小写                    | `findByFirstnameIgnoreCase`findByFirstnameIgnoreCase                                                                 | `… where UPPER(x.firstame) = UPPER(?1)`…其中UPPER（x.firstame）= UPPER（？1）                        |

![image-20210428231134633](../images/java_web/image-20210428231134633.png)

或者

![image-20210428231246524](../images/java_web/image-20210428231246524.png)

```
@Entity
@NamedQuery(name = "User.findByEmailAddress",
  query = "select u from User u where u.emailAddress = ?1")
public class User {

}
```

使用时需要声明 

```java
public interface UserRepository extends JpaRepository<User, Long> {

  List<User> findByLastname(String lastname);

  User findByEmailAddress(String emailAddress);
}
```

![image-20210428232412226](../images/java_web/image-20210428232412226.png)

###### 使用原生 sql

在spring中 为  native count queries  本机查询 

在@Query中 加上 nativeQuery  = true 

![image-20210429000547981](../images/java_web/image-20210429000547981.png)

![image-20210429000628954](../images/java_web/image-20210429000628954.png)

##### querydsl

[外接框架](https://github.com/querydsl/querydsl)

#### 分页和排序

需要用到 spring data 中的 Page 接口 +和 Pageable 分页实现类  

也可以使用其它类型 

![image-20210429165407881](../images/java_web/image-20210429165407881.png)

Pageable 使用时 需要传入页数, 每页条数和 排序规则 

![image-20210429170208344](../images/java_web/image-20210429170208344.png)

##### @Query中使用 Pageable 参数进行分页

![image-20210501163347938](../images/java_web/image-20210501163347938.png)

使用本地查询需要 加上coutQuerty , nativeQuery= true;

#### 连接 mysql

将datasource 的url和driver 设置为mysql

![image-20210425161812697](../images/java_web/image-20210425161812697.png)

#### 整合Mybatis

### boot 多moudle

作为父moudle 的 将出现![image-20210429193509970](../images/java_web/image-20210429193509970.png)

 作为父moudle的作用为提供pom 文件将多个moudle 中的 依赖集中管理 , packing 方式为 pom 

所以不能去运行, 所以要将需要运行的不同层次的代码放到各个对应层次的子 moudle中, 

且每个moudle 只能有一个application 主类 ,否则在 install 时会爆 ![image-20210429193607814](../images/java_web/image-20210429193607814.png)

父moudle 

![image-20210429215310843](../images/java_web/image-20210429215310843.png)

子moudle 

![image-20210429215239024](../images/java_web/image-20210429215239024.png)

### springCloud

由于springboot 新版本 2020.x.x的发布,所以把netflix 的一些东西给删减了 , 所以 一些东西不匹配

构建框架时 先把springSecurity关闭,  如果yml文件中不行或者没卵用那就properties文件中再加一遍

### 单点登陆

webFlux+JWT+权限验证

## 问题

### bean对象创建和加载顺序

在实践bean中属性时，为了方便，将bean id更改，属性更改，类名不更改的情况，直接在同一个java文件内实现通过不同的bean id 获得有着不同属性的对象，并输出结果进行比较学习。发现后来通过getBean方法创建的bean对象反而先执行实现

通过更改参数，发现未定义参数返回空值的bean对象调用方法要提前执行  

![image-20210404002003162](../images/java_web/image-20210404002003162.png)

![image-20210404002008518](../images/java_web/image-20210404002008518.png)

上面为scope属性的单例范围实现

下面为原型范围实现

经过检验后发现 bean 中的scope属性颠倒 ，且重复调用了objSingle对象，重复为止在objPrototype.setMessage("I'm a objProtype firt one")；下

说明正常来说bean对象执行过程按照创建顺序执行 ，`写代码要更加细心`

学习新的知识点，关于Spring容器加载和实例化bean的顺序

#### 关于Spring容器加载和实例化bean的顺序

[参考](https://my.oschina.net/u/3387320/blog/2995935)

Spring容器在实例化时会加载容器内所有非延迟加载的单例类型Bean。

ApplicationContext内置一个BeanFactory对象，作为实际的Bean工厂，和Bean相关业务都交给BeanFactory去处理。BeanFactory在加载一个BeanDefinition（也就是加载Bean Class）时，将相应的beanName存入beanDefinitionNames属性中，beanDefinitionName 属性是Spring在加载Bean Class生成的BeanDefinition时，为这些Bean预先定义好的名称，在加载完所有的BeanDefinition后，执行Bean实例化工作在BeanFactory实例化所有非延迟加载的单例Bean时，遍历beanDefinitionNames 集合，按顺序实例化指定名称的Bean，此时会依据beanDefinitionNames的顺序来有序实例化Bean，也就是说Spring容器内Bean的加载和实例化是有顺序的，而且近似一致，当然仅是近似。

加载bean过程

先看加载Bean Class过程，零配置下Spring Bean的加载起始于ConfigurationClassPostProcessor类的postProcessBeanDefinitionRegistry（BeanDefinitionRegistry）方法，
其加载解析Bean Class的流程如下：

![img](../images/java_web/9147bdc75798c2deb3f87d6c968142290e7.jpg)

配置类

1. Spring容器起始配置类
2. @ComponentScan扫描得到的类
3. @Import引入的类

如果类上含有@Configuration， @Component ，@ComponentScan , @Import , @ImportResource注解中的一个，或者内部含有@Bean标识的方法，那么这个类就是一个配置类，Spring就会按照一定流畅去解析这个类上的信息，在解析的第一步会验证当前类是否已经被解析过了，如果是，那么按照一定规则处理(ComponeneScan得到的Bean能覆盖@Import得到的Bean，@Bean定义的优先级最高)

如果未解析过，那么开始解析：

1.解析内部类，查看内部类是否应该被定义成一个Bean，如果是，递归解析。
        2.解析@PropertySource，也就是解析被引入的Properties文件。
        3.解析配置类上是否有@ComponentScan注解，如果有则执行扫描动作，通过扫描得到的Bean Class会被立    即解析成BeanDefinition，添加进beanDefinitionNames属性中。之后查看扫描到的Bean Class是否是一个配置类（大部分情况是，因为标识@Component注解），如果是则递归解析这个Bean Class。
        4.解析@Import引入的类，如果这个类是一个配置类，则递归解析。
        5.解析@Bean标识的方法，此种形式定义的Bean Class不会被递归解析
        6.解析父类上的@ComponentScan，@Import，@Bean，父类不会被再次实例化，因为其子类能够做父类的工作，不需要额外的Bean了。

### Spring和SpringMVC扫描注解类冲突

因为在已经配置好了一个Beans.xml文件的项目工程下学习Spring的用法, 一直没有出现问题

所以在学习SpringMVC的时候直接在WEB-INF下创建springmvc-servlet.xml扫描注解类 

在 404报错页面中一直挣扎,包括修改tomcat配置和实验以前留下的servlet文件用来实验tomcat容器下的文件是否能正常配置

在最基础的一个mvc项目(输出hello SpringMVC)报错挣扎两三个小时

一直查找springMVC 404 进而修改 serlvlet.xml扫描注解文件

终于想到是不是因为存在两个Spring 类的xml文件而引起冲突  ,查找到@Controller 失效. 出现404错误

,因为WEB-INF文件夹下的jsp文件本来就不能在浏览器上直接访问(在解决上述404页面时学习到的新知识)     webapp目录下有着自动配置好的安全 配置  

最终解决,因为 创建的maven-web项目都添加了使得maven不自动加载maven插件,使得maven项目中不存在web 目录,需要手动创建, 后来重新建立 maven项目,并 加入maven-web moudle,出现webapp文件夹

并在此配置springmvc-servlet.xml文件  idea可以识别到此配置文件, 出现spring的标识符

![image-20210406142508684](../images/java_web/image-20210406142508684.png)

同时 配置的 base-package 下的@Contoller 文件 会对应的有着spring的符号 

![image-20210406142935810](../images/java_web/image-20210406142935810.png)

![image-20210406142919387](../images/java_web/image-20210406142919387.png)

成功呢

#### 附上一些配置文件

 ![image-20210406144724593](../images/java_web/image-20210406144724593.png)

Moudules中 Spring 中 Springmvc Servlet 为 springmvc-servlet.xml 文件的 配置名称

mvc 的配置文件 为  name-servlet .xml 形式 

文件结构

![image-20210406145207661](../images/java_web/image-20210406145207661.png)

java 文件夹  和 webapp应该在同一分级

### springMVC中JSP页面EL表达式不起作用

${}

在使用 el 表达式的jsp中配置 

<%@page isELIgonored="false" %>

该设置表达式在jsp中使用el表达式,可以解析其中的值, 若isELIgnored 设置为true , 代表在本页不使用el表达式, 当作字符串解析出来显示.  为flase时,el表达式正常工作, 显示正常 

`isLIgnored  默认值为true! 所以才会出现这个问题`

### 使用@Insert 注解时, 数据没有插入数据库,但得到自增id

![image-20210412225000488](../images/java_web/image-20210412225000488.png)

![image-20210412225005593](../images/java_web/image-20210412225005593.png)

进行查询和数据库中表单更新 ,没有出现新添加数据 ,也没有id

#### 解决

需要使用commit()函数手动提交事物

![image-20210413123221058](../images/java_web/image-20210413123221058.png)

![image-20210413123230890](../images/java_web/image-20210413123230890.png)

### 对Mybatis中接口方法的返回值进行处理

对通过id获得数据库中数据的方法进行特定处理, 想实现数据存储进list 

和 调用getAll 方法获得的返回值类型不同, 不能直接使用 list.add()  ,  查找原因后 

 ![image-20210413173041564](../images/java_web/image-20210413173041564.png)

接口内的方法类型决定了方法体的返回值类型, 需要用对应的类型参数取接收

### mvc中写 mybatis 接口实现类调用方法出现异常问题

![image-20210413220755641](../images/java_web/image-20210413220755641.png)

上面程序正确 , 但服务器日志会出现500报错时,  找了很久报错没解决, 最后调整到mybaits 中mapper别名后出现 mysql-connector驱动找不到, mysql-connector为我自己添加的lib,没有经过pom, 所以tomcat服务器不识别, 将mysql-connector 依赖在pom中重新添加, 正确运行

![image-20210413221036462](../images/java_web/image-20210413221036462.png)

### could  not autowrite

Could not autowire. No beans of ‘AreaDao’ type found   

对接口 无法用@Autowired 去自动创建bean, 是spring的一个idea插件找实现类的问题, idea以为找不到实现类给出的提示 

修改error为warnig 

### jsp中c:forEach 标签不输出 list<object> 数据

没添加jstl 依赖 

![image-20210415004400915](../images/java_web/image-20210415004400915.png)

![image-20210415004408538](../images/java_web/image-20210415004408538.png)

添加后并且要在 jsp页面上加入

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

### 遇到属性不可写问题

![image-20210415144118740](../images/java_web/image-20210415144118740.png)

jsp页面中使用到了pageNow作为当前页数的记载  ,  在调用如下操作时候发生 page.PageNow属性不可写错误 

![image-20210415144147666](../images/java_web/image-20210415144147666.png)

将 setPageNow 方法中 Page改为void , 如下 

![image-20210415144254998](../images/java_web/image-20210415144254998.png)

这样便可写, 因为设置时候 使用的Page对象进行 数据写入 

### 文件 上传临时路径无效

用的是在dispatcher中配置 mult 存放临时路径的文件上传实现类

更改存放路径 , 放在idea临时路径的根目录下, 

### @autowired 使用

@autowired使用的类 必须要被spring组件扫描到 

spring中 添加 @mapperScan 扫描 @ComponentScan 扫描包

### Invalid bound statement

跟着练手程序写的时候 没有注意到是写在springboot .yml文件中的 

用了.yml文件的配置格式导致程序运行找不到 mapper 的 xml文件, 所以就有了 Invalid bound statement

### jpa和mybatis 的区别和使用

在逼乎上翻了评论 ,众口不一, 最终得出的结论 

翻找 官方文档, 在官方和 框架底层内去寻找自己想实现的功能, 

在自动化上, mybatis 有mybatis-plus generator 自动实现mapper 接口和mapper xml 文件的创建和配置 

mapper xml 中 ![image-20210428215520376](../images/java_web/image-20210428215520376.png) 

没有sql' 语句

需要配置generatorConfig.xml

```
<generatorConfiguration>
    <context id="MysqlContext" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- ??????Mapper ??????? ,??true??, ????????????-->
        <property name="useMapperCommentGenerator" value="true"/>

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <!--??????Mapper???????????? -->
            <property name="mappers" value="tk.mybatis.mapper.common.MySqlMapper"/>
        </plugin>
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <!--??????Mapper???????????? -->
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
        </plugin>

        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://127.0.0.1:3306/library?charcterEncoding=utf-8&serverTimezone=GMT%2B8&nullCatalogMeansCurrent=true"
                        userId="root"
                        password="root">
        </jdbcConnection>
        <!-- ??pojo???-->
        <javaModelGenerator targetPackage="com.example.springboot.pojo" targetProject="src/main/java"></javaModelGenerator>

        <!-- ???mapper????-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources"></sqlMapGenerator>
        <!--??mapper???java??-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.example.springboot.dao" targetProject="src/main/java"></javaClientGenerator>

        <!--??????(??tableName?domainObjectName)-->
        <table tableName="book" domainObjectName="Book" enableCountByExample="true"
               enableDeleteByExample="true" enableUpdateByExample="true"
               selectByExampleQueryId="true" enableDeleteByPrimaryKey="true"
               enableSelectByPrimaryKey="true" enableSelectByExample="true">
        </table>
    </context>


</generatorConfiguration>
```

并写Class 执行main方法进行 xml文件 导入和 使用

```java
public class MybatisGenerator {
    public void generator() throws Exception{
        ArrayList<String> warnings = new ArrayList<>();
        boolean overwrite = true;
        //指定配置文件
        File configFile = new File("generatorConfig.xml");
        System.out.println(configFile.getAbsolutePath());
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }

    public static void main(String[] args) throws Exception {
        MybatisGenerator mybatisGenerator = new MybatisGenerator();
        mybatisGenerator.generator();
    }
}
```

对于jpa有个问题是 jpa中只需要在 dao层的接口内定义方法 ,jpa 中 extends 对应的类后就会自动实现基本的 查询等基础 sql操作 , 无法实现sql的优化 , 或者对表一些比较复杂的操作会涉及到效率问题 , 

使用的是 @Entity 注解 将 实体类对象创建为表,  面向对象的思想, 将对象看作表进行实现

jpa 支持自定义SQL语句查询 , 官方文档 

![image-20210428223748115](../images/java_web/image-20210428223748115.png)

@Query 

spring中一些比较详细的东西 需要在docs  中查看 

对jpa可以使用xml 文件方式 ,使用命名空间 , 

自定义命名空间属性 ,基于@Entity

对于JPA命名查询 

#### [官方query文档](https://docs.spring.io/spring-data/jpa/docs/1.10.2.RELEASE/reference/html/#jpa.query-methods.at-query)

对表操作 

### 多moudle中关于 relativePath

![image-20210429191614588](../images/java_web/image-20210429191614588.png)

<relativePath> 中需要继承来自父moudle 的pom.xml文件 ,需要检验

### boot 多moudle 启动问题

当依赖存在并可以被项目识别提示, 但运行时会报错不存在 , 找不到符号 ,

需要勾选 

![image-20210429192036651](../images/java_web/image-20210429192036651.png)

#### 子模块互相引用

![image-20210429232630567](../images/java_web/image-20210429232630567.png)

需要在一个子moudle中进行 另一个子模块依赖的引入, 而不能在 父级pom 中引入, 因为这样被依赖模块本身会依赖自己 ,进行 重构在另一个moudle 中引入依赖 

在加上这个

![image-20210429232946672](../images/java_web/image-20210429232946672.png)

在父项目下的maven-plugin 中写入 ,运行 

mvn clean package  , 相当于clean后重新打包, 

![image-20210429233031551](../images/java_web/image-20210429233031551.png)

此时会有两个jar

![image-20210430152728623](../images/java_web/image-20210430152728623.png)

maven 中会加入 moudle 的依赖包

#### 子moudle a中使用另一个moudle b的bean 出现 error  creating

因为moudle a 扫描不到  moudle下的包 , 加上 CpmponentScan 扫描 

![image-20210430120148307](../images/java_web/image-20210430120148307.png)

"{ }" 可以扫描多个package , 用","隔开 

#### 子moudle a 在controller层调用 service 实现报错

![image-20210430152835803](../images/java_web/image-20210430152835803.png)

service

![image-20210430152937772](../images/java_web/image-20210430152937772.png)

CustomerRepisutory调用的为子moudle b 中的 repo 文件 

cotroller

![image-20210430153023238](../images/java_web/image-20210430153023238.png)

最后找到原因为依赖的子moudle b在 运行 applcation时报错, 报错的刚好为 moudle a 所引用的类

将 moudle b中的错误修正后 ,在 moudle a 中编写正确的程序  

*![image-20210430160525751](../images/java_web/image-20210430160525751.png)*

### 依赖重复

不小心在一个moudle中添加了spring框架 ,因为已经有了springboot, 导致以来重复,在

![image-20210501102240688](../images/java_web/image-20210501102240688.png)

删除就ok
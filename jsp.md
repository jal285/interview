# jsp

<!-- more -->

## knowledge

Tomcat 访问任何的资源都是在访问Servlet

JSP第一次被访问时候会被编译为HttpJspPage类（该类是HttpServlet 的一个子类）

JSP本身就是一种Servlet ， jsp比serlvlet 更方便见简单的一个重要原因就是：内置了9个对象！

内置对象有：out、session、response、request、config、page、application、pageContext、exception

### out.print() out.write()

out.print(97) 打印的**97**都为字符串， 

out.wite(97)打印的为ASCII表中的字符a

### getParameter

getParameter得到的都是String类型的。或者是用于读取提交的表单中的值（http://a.jsp?id=123中的123），或者是某个表单提交过去的数据； 
getAttribute则可以是对象Object，需进行转换,可用setAttribute设置成任意对象，使用很灵活，可随时用；

setAttribute 是应用服务器把这个对象放在该页面所对应的一块内存中去，当你的页面服务器重定向到另一个页面时，应用服务器会把这块内存拷贝另一个页面所对应的内存中。这样getAttribute就能取得你所设下的值，当然这种方法可以传对象。session也一样，只是对象在内存中的生命周期不一样而已。 
getParameter只是应用服务器在分析你送上来的request页面的文本时，取得你设在表单或url重定向时的值。 

### jsp:getProperty

[jsp:getProperty](http://www.51gjie.com/javaweb/839.html)

## El表达式

[参考](https://blog.csdn.net/meiyalei/article/details/2127738?utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&dist_request_id=1328767.81997.16177741380994275&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

为了使jsp写起来更简单

    表达式语言的灵感来自于 ECMAScript 和 XPath 表达式语言，它提供了在 JSP 中简化表达式的方法。它是一种简单的语言，基于可用的命名空间（PageContext 属性）、嵌套属性和对集合、操作符（算术型、关系型和逻辑型）的访问符、映射到 Java 类中静态方法的可扩展函数以及一组隐式对象。

语法结构

${expression}

1、语法结构
     ${expression}
2、[ ]与.运算符
     EL 提供“.“和“[ ]“两种运算符来存取数据。
     当要存取的属性名称中包含一些特殊字符，如.或?等并非字母或数字的符号，就一定要使用“[ ]“。例如：
         ${user.My-Name}应当改为${user["My-Name"] }
     如果要动态取值时，就可以用“[ ]“来做，而“.“无法做到动态取值。例如：
         ${sessionScope.user[data]}中data 是一个变量
3、变量
     EL存取变量数据的方法很简单，例如：${username}。它的意思是取出某一范围中名称为username的变量。
     因为我们并没有指定哪一个范围的username，所以它会依序从Page、Request、Session、Application范围查找。
     假如途中找到username，就直接回传，不再继续找下去，但是假如全部的范围都没有找到时，就回传null。
     属性范围在EL中的名称
         Page          PageScope
         Request          RequestScope
         Session          SessionScope
         Application      ApplicationScope

${ pageContext.request.contextPath} 取出部署应用程序的名字

## 标签

### 跳转

<jsp:forward page:"Relative URL">

### bean实例化

<jsp:useBean id="instanceName" sope="page | request | session | application"

class= "packageName.className" type="packageName.className"

beanName="packageName.clssName | <%=expression>"/>

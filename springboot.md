# 记录一些写程序中得到的知识

<!-- more-->

## 使用postman

### post请求传递给@RequestBody 数据

![image-20210420164023668](../images/springboot/image-20210420164023668.png)

这里使用的是 @RequestBody 获得post传递的json数据 

因为是以**简单对象**接收数据, 所以使用 json  , RESTfunl web服务一般会将返回地数据以JSON的形式返回 ,  

第一次发送数据

![image-20210420164355301](../images/springboot/image-20210420164355301.png)

得到的返回

![image-20210420164608039](../images/springboot/image-20210420164608039.png)

再次添加

![image-20210420164652108](../images/springboot/image-20210420164652108.png)

因为 我设置的@RequestBody 接受对象Book还存在着另一个对象, 所以想到为Book对象中的booCase对象传入数据 

,自己直接在postman中尝试发送数据的时候因为参考一些别的文章, 导致发送数据的格式不正确 

所以想到自己添加一个数据进bookCase对象,并利用book对象返回json数据得到 正确格式 

![image-20210420173827462](../images/springboot/image-20210420173827462.png)

返回地数据为 

![image-20210420173851245](../images/springboot/image-20210420173851245.png)

按照这种类型的格式进行json的写入

![image-20210420173939168](../images/springboot/image-20210420173939168.png)

传入结果

![image-20210420173957464](../images/springboot/image-20210420173957464.png)

网上搜到这种为 嵌套json , 还有 json 数组等其他数据格式 

## 处理多moudle

共享resources 文件 

![image-20210525221027455](../images/springboot/image-20210525221027455.png)

上面配置只是让maven 在构建项目的时候将配置文档放到相应classes 目录下 

需要在配置文件中引用需要做相应配置 

![image-20210525221226047](../images/springboot/image-20210525221226047.png)

![image-20210525221235376](../images/springboot/image-20210525221235376.png)

上面配置会在在多个resources 下的不同.properties  文件中用到 

### 项目公用resources 文件夹

[参考](https://my.oschina.net/u/150107/blog/3191606)

共享资源的pom 可以配置如下 

```javascript
<project>
  ...  
  <groupId>org.test</groupId>
  <artifactId>common</artifactId>
  ...
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-remote-resources-plugin</artifactId>
        <version>1.8.0</version>
        <executions>
          <execution>
            <goals>
              <goal>bundle</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <includes>
            <include>**/*.properties</include>
          </includes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

这将在genroate-resources 阶段将共享资源捆版到JAR 文件中, 这意味着其它模块可以在此后的任何阶段使用这些资源 

配置其他模块以使用共享模板 

要在另一个模块中使用共享资源, 需要按如下方式配置插件:

```javascript
<project>
  ...
  <groupId>org.test</groupId>
  <artifactId>resource-consumer</artifactId>
  ...
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-remote-resources-plugin</artifactId>
        <version>1.8.0</version>
        <configuration>
          <resourceBundles>
            <resourceBundle>org.test:common:${project.version}</resourceBundle>
          </resourceBundles>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>process</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
   </plugins>
  </build>
  ...
  <dependencies>
    <dependency>
      <groupId>${project.groupId}</groupId>
      <artifactId>common</artifactId>
      <version>${project.version}</version>
    </dependency>
  </dependencies>
</project>
```

这将检索common模块的捆绑资源, 处理捆绑中的每个资源. 并将他们放入resource-consumer模块的$ {project.build.directory} / maven-shared-archive-resources目录中。

**注: 上述模块名称根据自己使用的进行命名 **

## 对项目之间项目依赖问题

将 项目文件的package 都使用相同的名称 

如

 ![image-20210525222834199](../images/springboot/image-20210525222834199.png)

将依赖都放入主moudle 中集中引用，其余子 moudle之间避免相互引用

将 maven-plugin![image-20210529205922882](../images/springboot/image-20210529205922882.png)添加到主moudle 里， 我这里是web moudle

## 经验之谈

### bean

#### bean重复问题

确保依赖引入没有引入不同jar包, 相同 Java Class , 否则ioc 将抛出bean 重复问题 



### @RestController 实体类不返回json

报406错误 

实体类中需要加上 toString方法修改实体类的返回值类型

可以使用lombock 的@ToString 注解 

# 结合maven学习spring框架的后端之旅

<!-- more -->

## 一些基础要点

### maven 下载依赖问题

maven

配置conf 中setting.xml文件   本地仓库   镜像

idea配置pom.xml    后刷新maven中仓库信息

setting 中找到maven 处修改maven本地目录

maven 依赖不好拉  注意依赖的版本和冲突   

人生建议，有加速器就挂加速器直接从中央仓库拉取，配置阿里云仓库会出现莫名其秒的报错+有些jar文件下载失败

```
    <mirrors>
        <!-- 阿里云仓库 -->
        <mirror>
            <id>alimaven</id>
            <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
        </mirror>

        <!-- 中央仓库1 -->
        <mirror>
            <id>repo1</id>
            <mirrorOf>central</mirrorOf>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://repo1.maven.org/maven2/</url>
        </mirror>

        <!-- 中央仓库2 -->
        <mirror>
            <id>repo2</id>
            <mirrorOf>central</mirrorOf>
            <name>Human Readable Name for this Mirror.</name>
            <url>http://repo2.maven.org/maven2/</url>
        </mirror>
    </mirrors>
```

## 解决maven一些问题

### 解决maven中不能直接创建Servlet文件和filter文件问题

在 Project Structure中设定src/main/java为sourceRoot

再在web项目中iml文件中添加如下

```xml
  <sourceRoots>
          <root url="file://$MODULE_DIR$/src/main/java" />
        </sourceRoots>
```

示例

```3xml
<?xml version="1.0" encoding="UTF-8"?>
<module type="JAVA_MODULE" version="4">
  <component name="FacetManager">
    <facet type="web" name="Web">
      <configuration>
        <descriptors>
          <deploymentDescriptor name="web.xml" url="file://$MODULE_DIR$/web/WEB-INF/web.xml" />
        </descriptors>
        <webroots>
          <root url="file://$MODULE_DIR$/web" relative="/" />
        </webroots>
        <sourceRoots>
          <root url="file://$MODULE_DIR$/src/main/java" />
        </sourceRoots>
      </configuration>
    </facet>
  </component>
</module>
```

### ## 一般maven中依赖文件夹内需要的文件

一个.jar和一个.pom文件, 注意自己手动下载时修改文件名时确认好

## setting 中maven

![image-20210411131622171](../images/maven-ssm/image-20210411131622171.png)

勾选 Delegate IDE build/run actions to Maven 后将会将java类的运行委托给 maven ,

此时需要用到

![image-20210411131721540](../images/maven-ssm/image-20210411131721540.png)

但 ,总会报错, 你懂的, 测试程序类时候不要勾选了还是, 有需要时配置 

配置阿里云 镜像时需要加上忽略https的ssl证书验证 

-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true

![img](../images/maven-ssm/v2-e3bbb645b913485826c4ec85416958f6_720w.jpg)

以防万一,两个地方都加上

这里加的是这条语句

-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true

![img](../images/maven-ssm/v2-e6dcc8da4821243f964262ed5e7ec295_720w.jpg)

依赖出问题, 

配本地仓库, 删了重新下载 

重新创建pom.xml文件

idea 自动还原mavn默认配置  , 在  idea 的  

在 c盘

project.default.xml 文件中添加

```
<component name="MavenImportPerferences">
  <option name="generalSettings">
    <MavenGeneralSettings>
      <option name="localRepository" value="E:\programsoftware\apache-maven-3.6.3\repo" />
      <option name="mavenHome" value="E:\programsoftware\apache-maven-3.6.3"/>
      <option name="userSettingsFile" value="E:\programsoftware\apache-maven-3.6.3\conf\settings.xml"/>
    </MavenGeneralSettings>
  </option>
</component>
```

## 父子模块

以下配置<packaging>pom</packaging>的意思是使用maven分模块管理，都会有一个父级项目，pom文件一个重要的属性就是packaging（打包类型），一般来说所有的父级项目的packaging都为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包。

![image-20210427181728645](../images/maven-ssm/image-20210427181728645.png)

### mvn clean

进行依赖更换和拉取

## 

# SpringBoot 配置文件

```yml
#redis配置
#Redis服务器地址
spring.redis.host=127.0.0.1
  #Redis服务器连接端口
spring.redis.port=6379
  #Redis数据库索引（默认为0）
spring.redis.database=0
  #连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=50
  #连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=3000
  #连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=20
  #连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=2
  #连接超时时间（毫秒）
spring.redis.timeout=5000


spring.main.allow-bean-definition-overriding=true

  # mysql-datasource
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai

spring.datasource.username=root
spring.datasource.password=root
spring.datasource.druid.driver-class-name= com.mysql.cj.jdbc.Driver
mybatis.mapper-locations=/mapper/**.xml
  #spring.datasource.druid.initial-size=5
  #spring.datasource.druid.max-active=30
  #spring.datasource.druid.min-idle=5
spring.profiles.active=local
#redis配置
#Redis服务器地址
spring.redis.host=6379
#Redis服务器连接端口
spring.redis.port=6379
#Redis数据库索引（默认为0）
spring.redis.database=0
#连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=50
#连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=3000
#连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=20
#连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=2
#连接超时时间（毫秒）
spring.redis.timeout=5000


spring.main.allow-bean-definition-overriding=true

# mysql-datasource
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai

spring.datasource.username=root
spring.datasource.password=root
spring.datasource.druid.driver-class-name= com.mysql.cj.jdbc.Driver
mybatis.mapper-locations=/mapper/**.xml
#spring.datasource.druid.initial-size=5
#spring.datasource.druid.max-active=30
#spring.datasource.druid.min-idle=5
spring.profiles.active=local
```

在pom中加上这个保证resources文件下下的xml和 yml被编译

```xml
 <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.yml</include>
            </includes>
            <filtering>false</filtering>
        </resource>

        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.yml</include>
            </includes>
            <filtering>false</filtering>
        </resource>
    </resources>
</build>
```

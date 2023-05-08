# SpringSecurity 结合springboot

<!-- more -->

springSecutrity会对web页面进行限制 , 在查看web界面的时候需要登陆 ,默认 用户名为user 密码在控制台上随机生成 

可以在sringboot中自己配置

## 权限处理

## url

## webflux

### 权限配置

![image-20210626163747666](../images/java_Security/image-20210626163747666.png)

在 SecurityWebFilterChain 方法中添加权限需求 

SecurityExpressionRoot类中的**getAuthoritySet()**  方法获取authentication中的鉴权信息,  在UserDetail中获取getAuthroities()方法返回的roleInfoList 为所有鉴权信息,接口继承重写了getAuthroities()方法使得我们可以定义自己所想要的roleDetail 

![image-20210626163849517](../images/java_Security/image-20210626163849517.png)

UserDatil 实现了UserDatils接口

![image-20210626163935884](../images/java_Security/image-20210626163935884.png)

重写UserDetatils 接口中的getAuthorities()方法获得权限信息, roleInfoList将会添加进roles 中,并在判断是否有权限时通过roles来判断权限有无 

![image-20210626164018432](../images/java_Security/image-20210626164018432.png)

roleInfoList 为 角色信息列表 , 在角色信息中添加role信息 :RoleDetatil 类 

![image-20210626164034689](../images/java_Security/image-20210626164034689.png)

可以利用role_id 划分role规则, 这里用role属性测试 

![image-20210626165258812](../images/java_Security/image-20210626165258812.png)

这里使得getAuthroity 返回的为role属性 , role ="user";  也就是说在这里的userDetail 中的role 都定义为user

RoleDetail 继承GrantedAuthroity接口 

重写String getAuthroity()方法 

GrantedAuthroity 接口中的getAuthority() 方法描述 

```
/**
     * If the <code>GrantedAuthority</code> can be represented as a <code>String</code>
     * and that <code>String</code> is sufficient in precision to be relied upon for an
     * access control decision by an {@link AccessDecisionManager} (or delegate), this
     * method should return such a <code>String</code>.
     * <p>
     * If the <code>GrantedAuthority</code> cannot be expressed with sufficient precision
     * as a <code>String</code>, <code>null</code> should be returned. Returning
     * <code>null</code> will require an <code>AccessDecisionManager</code> (or delegate)
     * to specifically support the <code>GrantedAuthority</code> implementation, so
     * returning <code>null</code> should be avoided unless actually required.
     * @return a representation of the granted authority (or <code>null</code> if the
     * granted authority cannot be expressed as a <code>String</code> with sufficient
     * precision).
     */
    String getAuthority();
```

![image-20210626170719865](../images/java_Security/image-20210626170719865.png)

**上面四个方法最终都是通过hasAnyAuthorityName**方法进行判断

```java
private boolean hasAnyAuthorityName(String prefix, String... roles) {
   Set<String> roleSet = getAuthoritySet();
   for (String role : roles) {
      String defaultedRole = getRoleWithDefaultPrefix(prefix, role);
      if (roleSet.contains(defaultedRole)) {
         return true;
      }
   }
   return false;
}
```

hashAnyAuthorityName 通过传入的role 是否在用户roleSet中判断是否有使用权限

**一般defaultRolePreFix **使用默认值

```java
private String defaultRolePrefix = "ROLE_";
```

也可以自己设置 

```java
public void setDefaultRolePrefix(String defaultRolePrefix) {
   this.defaultRolePrefix = defaultRolePrefix;
}
```

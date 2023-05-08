## 关于token 在web中的使用

 <!--more-->

## 客户端认证

token不存储到服务器任何地方, 只提供给服务端解密 并通过验证, 就认为是合法的 , 比如jwt

### JWT

使用在客户量特别大的时候

#### 使用示例

```java
package com.example.cloud.util.token;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.interfaces.DecodedJWT;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * @author badpoone
 */
public class JWTTokenUtil {
    /**
     *     过期时间
     *     static final 静态, 无法改变,必须初始化
     */
    private static final long EXPIRE_DATE=30*60*100000;
    //利用随机数生成 tokenSecret  保存到缓存
    public static final String TOKEN_SECRET = "ZCfashuaUUhufgudddadadadaeER";

    public static String token(String username, String password){
        String token = "";
        try {
            //过期时间
            Date date = new Date(System.currentTimeMillis()+EXPIRE_DATE);
            //密钥以及加密算法
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            //设置 头部信息
            Map<String,Object> header= new HashMap<>();
            //携带username, password 信息, 生成签名
            token = JWT.create()
                    .withHeader(header)
                    .withClaim("username",username)
                    .withClaim("password",password)
                    .sign(algorithm);
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
        return token;
    }

    public static boolean verify(String token){

        try {
            Algorithm algorithm=Algorithm.HMAC256(TOKEN_SECRET);
            JWTVerifier verifier = JWT.require(algorithm).build();
            DecodedJWT jwt = verifier.verify(token);
            return true;
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }

    }


    public static void main(String[] args) {
        String username ="name1";
        String password = "pw1";
        //注意，一般不会把密码等私密信息放在payload中，这边只是举个列子
        String token = token(username,password);
        System.out.println(token);
        boolean b = verify(token);
        System.out.println(b);
    }

}

```



#### 关于JWT的安全问题

JWT token 的关键过程点就是密钥, 加入这个密钥泄露了, 那么基本就可以伪造token了 

这一点可以在用户登陆时 利用随机数生成token的TOKEN_SECRET , 并保存到缓存, key是登陆账户 , 客户端

访问接口是 : header要带登陆账号 和 token ,服务端拿到登录账号，到缓存去捞相应的SecretKey ，然后再进行token校验。可以防伪造token了（这个方案在一定程度上能防止伪造

，但是不能防止token泄露被劫持）





#### 使用 JWT进行用户认证及Token的刷个牙续保 



## 服务端认证




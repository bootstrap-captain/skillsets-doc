# JWT

- [官网地址](https://jwt.io/)
- Json Web Token
- 将原始的json数据格式进行了安全的封装，从而直接将数据进行传输

## 认证方式

- 可以用来进行用户身份权限的校验

![image-20250326125424082](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250326125424082.png)

```bash
1. 客户端发起登录请求，认证服务的登录接口，查看当前登录用户的身份正确后，返回JWT给客户端
2. 客户端拿到JWT后，将JWT存储起来，在后续每次访问资源服务时，都会将JWT令牌携带过去
3. 资源服务统一拦截请求，线判断当前token是否合法(公钥合法？是否过期？是否是对应的接受者？)
                  - 合法后，才会去解析JWT中的的payload
                  - 一旦token被篡改过，就会验证失败
```

## 组成

- 简洁：JWT就是一个简单的字符串，可以在请求参数或者请求头中直接传递
- 自包含：可以根据自身需求，在JWT中存储自定义的数据内容： 如用户的相关信息和权限

![image-20250327104815356](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250327104815356.png)

```bash
# Header
- 记录令牌类型，签名算法等
- 是将对应的json数据转换为base64

# Payload
- 有效载荷，携带自定义信息，默认信息
- 将对应的json数据转换为base64
- 不要携带敏感信息，因为这部分信息是可以被明文查看的

# Signature
-  数字签名，包含多种不同的签名算法，防止token被篡改，确保安全性
-  一旦token中的任何一个数据被修改了，JWT在后续检验的时候就会失败

  # Signature的生成步骤
  - 将header和payload进行Base64Url编码，并用 . 连接
  - 使用密钥将上述组合字符串进行签名，生成原始二进制数据
  - 将二进制数据转换为Base64Url字符串
```

## 算法

### HMAC

- HS256、HS384、HS512(SHA-256/384/512)
- HMAC(Hash-based Message Authetication Code)算法结合SHA哈希函数

```bash
# 密钥
- 对称加密，需安全共享
# 速度
- 签名和验证速度都很快
# 签名长度
- 固定(HS256的为32个字节)
# 安全性能
- 依赖密钥保密性，密钥泄漏则完全暴露
# 使用场景
- 单服务建构或封闭系统
- 需要高性能的场景
```

### RSA

- RS256/384/512(SHA-256/384/512 + RSA)

```bash
# 密钥
- 非对称密钥， 私钥用于签名，公钥用于验证签名
# 速度
- 签名生成慢，验证快
# 签名长度
- 等于RSA模数， 2048位=256字节
# 安全性
- 密钥长度>=2048
# 使用场景
- 分布式系统(微服务，公钥可公开发布)
- 需要分离签名和验证权限的场景
```

### RSA-PSS

- 改进的RSA替代方案，比传统的RSA更安全可靠(使用概率签名方案)
- PS256/384/512

## 签名/验签

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>4.5.0</version>
</dependency>
```

### RSA

- 对应的claim，过期时间等，如果匹配不上，则就会验证失败

```java
package com.princess.kitchen.utils;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.interfaces.DecodedJWT;
import com.fasterxml.jackson.databind.ObjectMapper;
import lombok.extern.slf4j.Slf4j;

import java.security.PrivateKey;
import java.security.PublicKey;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.util.Date;

@Slf4j
public class JWTUtil {

    public static String singJWT() throws Exception {
        Phone phone = new Phone("001", "Iphone", "Iphone-15");
        ObjectMapper objectMapper = new ObjectMapper();
        String phonePayload = objectMapper.writeValueAsString(phone);

        /*自己写的工具类，从pem中加载私钥*/
        PrivateKey privateKey = RSAEncryptDecryptUtil.loadPrivateKey("/Users/shuzhan/Documents/princess-kitchen/api-service/src/main/resources/private.pem");
        Algorithm algorithm = Algorithm.RSA256(null, (RSAPrivateKey) privateKey);

        return JWT.create()
                .withIssuer("erick-issuer") /*签发方*/
                .withSubject("kitchen-subject") /*主题*/
                .withAudience("lucy-audience")/*接受方*/
                .withExpiresAt(new Date(System.currentTimeMillis() + 1000 * 60 * 30))/*过期时间：30min*/
                .withClaim("username", "zs35531")/*自定义声明*/
                .withClaim("email", "zs35531@gmail.com")
                .withPayload(phonePayload)
                .sign(algorithm);
    }

    public static void verifyJWT(String jwtToken) throws Exception {
        /*自己写的工具类，从pem中加载公钥*/
        PublicKey publicKey = RSAEncryptDecryptUtil.loadPublicKey("/Users/shuzhan/Documents/princess-kitchen/api-service/src/main/resources/public.pem");
        try {
            Algorithm algorithm = Algorithm.RSA256((RSAPublicKey) publicKey, null);

            /*验证器*/
            JWTVerifier verifier = JWT.require(algorithm)
                    .withIssuer("erick-issuer")/*签发方如果对不上，则验证失败*/
                    .withAudience("lucy-audience")
                    .build();
            /*验证token*/
            DecodedJWT decodedJWT = verifier.verify(jwtToken);
            String username = decodedJWT.getClaim("username").asString();
            String email = decodedJWT.getClaim("email").asString();
            System.out.println(username);
            System.out.println(email);
        } catch (JWTVerificationException e) {
            log.error(e.getClass().getName());
            log.error(e.getMessage());
        }
    }
}
```

## 系统架构

### 签发-Issue

```bash
# 归属
- 专属的认证服务(Auth Service)

# 职责
- 用户登录后，验证用户身份正确后，使用私钥签名
- 管理密钥对(定期轮换)
- 提供公钥端点，供其他服务获取公钥 https
```

### 验证方-Verifier

```bash
# 归属
- API网关(统一验证，减少重复逻辑)
- 各业务服务(独立验证，避免网关成为单点瓶颈)

# 职责
- 使用Auth Service服务发布的公钥验证JWT签名
```

![image-20250326232737356](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20250326232737356.png)

## 密钥管理

### 轮换策略

- 密钥对定期轮换，比如常见的90天

### 私钥泄漏

- 私钥一旦泄漏，任何第三方都可以使用该密钥去伪造token，相当于绕过了去DB检查的步骤，从而可以访问Resource Service的任何的资源

## 会话过期

- 30 min后，当前token已经过期，用户再次操作时，需要再次验证权限

### 重复登录

- 重新定向到login页面，再次获取新的token凭证

```bash
#  安全性
- 每次登录需要验证凭证，敏感操作可结合多因素认证(MFA)
# 中断体验
- 会话过期后，需要重新登录，可能打断工作流程
# 场景
- 高安全要求的系统（如银行、医疗）
```

### Token续签

- 用户无需中断操作

```bash
# 无感知续期
- 用户无需中断操作

# 场景
- 适合长时间使用，用户体验度要求高的场景：后台系统，在线文档、监控仪表盘
```




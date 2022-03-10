---
title: "OpenID Connect 和 OAuth2.0"
date:  2021-12-10 17:53:31 +0800
categories: [授权认证]
tags: [OAuth2,OpenID]
---

## OAuth2.0 获取授权
OAuth is an open authorization protocol
1. Authorization Code Grant（RP在授权过程中不知道这个过程，用户在授权端点登陆，授权码post给后台，后台拿到code后获取access_token和userinfo，用户转入RP主页）
2. Implicit Grant（适用于无后台应用，如js网站，用户在授权端点登陆，RP直接拿到accsess_token，会暴露给资源所有者或其他应用，因为token是jwt,payload 包含了用户profile）
3. Resource Owner Password Credentials Grant(需要用户输入username和password，RP不得保存，仅在1，2授权不可用的情况下)
4. Client Credentials Grant（用于机器对机器，没有用户参与，如后台维护工作）


## OpenID Connect
OpenID Connect将身份验证实现为OAuth 2.0授权过程的扩展
1. Authorisation code flow - 代码流，跳转到授权页面，allow后OP发送code到RP指定的url，RP 后台通过code 再向OP获取id_token和access_token，通过id_token访问UserInfo端点，获取授权码必须走TLS

> 注意: scope中必须包含openid范围值

2. Implicit flow - 隐式流，比如js网站，无后台，授权后直接返回id_token和access_token给RP，易暴露
3. Hybrid flow - 混合流，一些令牌是由授权端点返回的，而其他令牌是从令牌端点返回的


## ID token
1. sub-subject, 用户身份
2. iss-issuing authority，发行机构
3. aud-audience, client ,受众
4. nonce-随机数（避免重放攻击）
5. auth-time-何时认证
6. acr-强度
7. iat-issue at ,发行时间
8. exp-expiration time， 过期时间
9. name & email address
10. 数字签名
11. 加密（JWE）

## 例子
```
  GET /authorize?
    response_type=code
    &scope=openid%20profile%20email
    &client_id=s6BhdRkqt3
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb HTTP/1.1
    Host: server.example.com
```

```
  HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj
```

```
  HTTP/1.1 200 OK
  Content-Type: application/json
  Cache-Control: no-store
  Pragma: no-cache

  {
   "access_token": "SlAV32hkKG",
   "token_type": "Bearer",
   "refresh_token": "8xLOxBtZp8",
   "expires_in": 3600,
   "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.ewogImlzc
     yI6ICJodHRwOi8vc2VydmVyLmV4YW1wbGUuY29tIiwKICJzdWIiOiAiMjQ4Mjg5
     NzYxMDAxIiwKICJhdWQiOiAiczZCaGRSa3F0MyIsCiAibm9uY2UiOiAibi0wUzZ
     fV3pBMk1qIiwKICJleHAiOiAxMzExMjgxOTcwLAogImlhdCI6IDEzMTEyODA5Nz
     AKfQ.ggW8hZ1EuVLuxNuuIJKX_V8a_OMXzR0EHR9R6jgdqrOOF4daGU96Sr_P6q
     Jp6IcmD3HP99Obi1PRs-cwh3LO-p146waJ8IhehcwL7F09JdijmBqkvPeB2T9CJ
     NqeGpe-gccMg4vfKjkM8FcGvnzZUN4_KSP0aAp1tOJ1zZwgjxqGByKHiOtX7Tpd
     QyHE5lcMiKPXfEIQILVq0pc_E2DzL7emopWoaoZTF_m0_N0YzFC6g6EJbOEoRoS
     K5hoDalrcvRYLSrQAZZKflyuVCyixEoV9GfNQC3_osjzw2PAithfubEEBLuVVk4
     XUVrWOLrLl0nx7RkKU8NXNHq-rvKMzqg"
  }
```

Payload
```
{
  "sub"       : "alice",
  "iss"       : "https://openid.c2id.com",
  "aud"       : "client-12345",
  "nonce"     : "n-0S6_WzA2Mj",
  "auth_time" : 1311280969,
  "acr"       : "c2id.loa.hisec",
  "iat"       : 1311280970,
  "exp"       : 1311281970
}
```

## 其他
JWS：JSON Web Signature，Digital signature/HMAC specification（签名）
> Authorisation response 中包含了三个部分，header，payload，signature
>
> signature：可以通过JWS签名，保证数据完整，没有被篡改，返回响应中的header包含了alg（加密方式，如HS256）利用服务端的密钥secret通过哈希256（SHA256）HMACSHA256(base64UrlEncode(header)+ "." + base64UrlEncode(payload),secret（公钥）) 加密取最左128bit，通过jwt网站了解 https://jwt.io/

JWE：JSON Web Encryption，Encryption specification（加密）
> 加密和解密方采用同一个密钥。采用这种模式的算法就叫做对称加密算法。
> 加密和解密方采用不同的密钥。采用这种模式的算法就叫做非对称加密算法。
JWK：JSON Web Key，Public key specification

JWA：JSON Web Algorithms，Algorithms and identifiers specification（算法）

JWT：JSON Web Token

## 参考
[JWT、JWE、JWS 、JWK 到底是什么？该用 JWT 还是 JWS？](https://blog.csdn.net/m0_49051691/article/details/109494815)

[RSA加密、解密、签名、验签的原理及方法](https://www.cnblogs.com/pcheng/p/9629621.html)

[JWT介绍及其安全性分析](https://cloud.tencent.com/developer/article/1539394)

[web环境中如何防止token被窃取进行重放攻击](https://www.zhihu.com/question/308444808)

[OAuth 2.0](https://tools.ietf.org/html/rfc6749)

[OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)

[A Guide To OAuth 2.0 Grants](https://alexbilbie.com/guide-to-oauth-2-grants/)

[What is the OAuth 2.0 Authorization Code Grant Type?](https://developer.okta.com/blog/2018/04/10/oauth-authorization-code-grant-type)

[What Is OpenID Connect and How Does It Work? What You Need to Know](https://www.pingidentity.com/en/resources/client-library/articles/openid-connect.html)

[okta上的讲解，非常经典](https://www.okta.com/openid-connect/)

[有实例，简单易懂](https://connect2id.com/learn/openid-connect)
获取授权码：
1. response_type=code
2. client_id 获取授权码不需要client_secret
3. scope
4. state 防 csrf 攻击

后台服务利用授权码获取token
1. grant_type=authorization_code
2. 带 client_secret

OpenID Connect
OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 protocol.

Clients use OAuth to request access to an API on a user’s behalf, but nothing in the OAuth protocol tells the client user information. OpenID Connect enables a client to access additional information about a user, such as the user's real name, email address, birthdate or other profile information.

The user’s SSO experience is made possible by the delivery of the ID token from the authorization server to the client.

请求中scope=openid



What is the JSON Web Token structure?

* Header
* Payload
* Signature

Therefore, a JWT typically looks like the following.

`xxxxx.yyyyy.zzzzz`

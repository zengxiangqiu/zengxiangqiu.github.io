---
title: "AspnetCore Authorize"
date:  2021-12-28 15:37:11 +0800
categories: [AspnetCore]
tags: [auth]
---

## 1. 重点

jwt 包括三部分，header、payload以及signature，前两部分可以被base64解析，而保证jwt不被篡改的关键在于 **signature**,因为采用了SHA256hash散列算法，当验证服务端（identity server）收到token时，将解析header和payload后利用密钥和算法生成对应的signature,与原token中的作比较，另外jwt中的nonce可以抵御重放攻击。

颁发令牌token之后，客户端所请求的资源范围如与jwt中的不符将不被允许，如客户端或中间人修改jwt playload base64内容，因签名不一致，将被拒绝

参考[这里](https://stackoverflow.com/questions/31309759/what-is-secret-key-for-jwt-based-authentication-and-how-to-generate-it)

## 2. audience

预期的受众

ids 4 中提到

  > When using the scope-only model, no aud (audience) claim will be added to the token, since this concept does not apply. If you need an aud claim, you can enable the EmitStaticAudience setting on the options. This will emit an aud claim in the issuer_name/resources format. If you need more control of the aud claim, use API resources.

EmitStaticAudience=true启用后默认aud=issuer_name/resources

还有[QAStack](https://qastack.cn/programming/28418360/jwt-json-web-token-audience-aud-versus-client-id-whats-the-difference)中的回答：

  > JWT将包含一个aud声明，该声明指定JWT适用于哪些资源服务器。如果aud包含www.myfunwebapp.com，但客户端应用程序尝试在上使用JWT www.supersecretwebapp.com，则访问将被拒绝，因为该资源服务器将看到JWT并不适合它。

例如，当某ApiResourse的scope包含XXX.read，某ApiClient也包含该scope，请求的access token的aud将包含apiresourse的name。

[Identity Server 4 ClientCredentials with POSTMAN](https://www.mrjamiebowman.com/microservices/identity-server-4-clientcredentials-with-postman/) 非常简单易懂的项目，有源码，可以作为入门。


API 资源服务中设置

```csharp
 .AddJwtBearer("Bearer", options =>
           {
               options.Authority = "https://localhost:5001";

               options.TokenValidationParameters = new TokenValidationParameters
               {
                   ValidateAudience = true
               };

               // if you are using API resources, you can specify the name here
               // options.Audience = "https://localhost:5001/resources";
                options.Audience = "StaffApi";
               // IdentityServer emits a typ header by default, recommended extra check
               options.TokenValidationParameters.ValidTypes = new[] { "at+jwt" };
           });
```

表示该API服务的受众是某个授权中心，用户请求的令牌中的aud需与此一致。

## 3. Claims

[Understanding Claims](https://stackoverflow.com/questions/37067938/understanding-claims)提到

> Scopes are identifiers used to specify what access privileges are being requested. Claims are name/value pairs that contain information about a user.

  So an example of a good scope would be "read_only". Whilst an example of a claim would be "email": "john.smith@example.com".

scope代表该token的权限范围，claims 代表user信息。

## 4. Scoped
ids 4 配置 Client Scoped , 客户端（web）通过disconvery 中的 endpoint 发送授权请求，其中request 中 scoped会与ids4中已注册的ApiScoped比较并放入claims，返回granted code 或 access token

客户端 再 利用access token 请求 资源，资源服务（microservice） 通过[Authorize] 和[RequiredScope] 验证 token 中scope 是否符合。

[Secure Applications with OAuth2 and OpenID Connect in ASP.NET Core 5 – Complete Guide](https://procodeguide.com/programming/oauth2-and-openid-connect-in-aspnet-core/) 详尽地介绍了OpenID Connection和OAuth 2.0 ，并结合identity server 4 讲解原理，可读性高，又源码，可以细读学习。

## 5. 笔记
1. **Identity Resources** are some standard open id connect scopes...

    用户身份识别的内容，比如email，profile，website等等

2. **API Resources** are used to define the API that the identity server is protecting

    被保护的资源，比如microservive架构的服务提供，体现在Request中的aud （audient受众）

    ApiResource 包含 one or more ApiScope，参考[Defining Resources](https://identityserver4.readthedocs.io/en/latest/topics/resources.html)

    ```csharp
     new ApiResource("invoice", "Invoice API")
        {
            Scopes = { "invoice.read", "invoice.pay", "manage" }
        },
    ```

3. api服务端

This Authentication configuration will make use of the discovery document on startup to configure the security for this API

api服务利用 ids4 发现文档 知道endpoints，比如验证 token是否有效等

当客户端（网页/手机）请求资源时，必须带上token

```nuget
Install-Package IdentityServer4.AccessTokenValidation
```

```csharp
services.AddAuthentication("Bearer")
.AddIdentityServerAuthentication("Bearer", options =>
{
  options.ApiName = "weatherApi";
  options.Authority = "https://localhost:44343";
});
```

```csharp
app.UseAuthorization();
```
ApiName 对应已注册的API Resources，最后在pipline中调用

4. Client Credentials flow 中不建议根据Scope来限制访问

[Client Credentials](https://www.oauth.com/oauth2-servers/access-tokens/client-credentials/)

> scope (optional)
> Your service can support different scopes for the client credentials grant. In practice, not many services actually support this.

[Authorization based on Scopes and other Claims](https://docs.duendesoftware.com/identityserver/v5/apis/aspnetcore/authorization/)中建议

```csharp
services.AddAuthorization(options =>
{
  options.AddPolicy("StaffRead", policy => policy.RequireClaim(JwtClaimTypes.Scope, "staffApi.read"));
  options.AddPolicy("Staff", policy => policy.RequireClaim(JwtClaimTypes.Scope, "staffApi.read","staffApi.write"));
});
```

```csharp
[Authorize(Policy ="Staff")]
```

5. AddJwtBearer

   AddJwtBearer  从header中提取和验证jwt token

6. client credential grant 客户端授权将无法获取userinfo，返回Forbidden
[Get UserInfo from Access Token - "Forbidden"](https://stackoverflow.com/questions/54145970/get-userinfo-from-access-token-forbidden)

7. client credentials 授权时利用Client中的claim中的role结合Policy限制访问，但不建议，应采用 Policy-Base 比较适合。

claim 针对用户的信息，键值对。

scope 针对token 可以访问的范围

role 针对user identity 的信息

所以下面的方案并不适合客户端授权

```csharp
  new Client
     {
         ClientId = "x",
         ClientName = "x Api",
         AllowedGrantTypes = GrantTypes.ClientCredentials,
         ClientSecrets = new List<Secret> {new Secret("xxx".Sha256())},
         AllowedScopes = new List<string> {
             "staffApi.read"},
         Claims = new List<ClientClaim>{ new ClientClaim(JwtClaimTypes.Role, "Staff") }
     },
```

```csharp
 options.AddPolicy("Staff", policy => policy.RequireClaim("client_role", "Staff"));
```

8. 在不同局域/DNS的部署Identity和其他服务，出现 The remote certificate is invalid according to the validation procedure: RemoteCertificateNameMismatch

参考 [RemoteCertificateNameMismatch](https://stackoverflow.com/questions/68268568/c-sharp-the-remote-certificate-is-invalid-according-to-the-validation-procedure)

> I suspect that remote certificate is issued against some DNS name, but you are connecting to IP address which apparently isn't specified in certificate subject/SAN extension.

9. 返回200，但无内容

在ExceptionMiddleware 中错误被拦截，context.status  = 200

内部报错： The MetadataAddress or Authority must use HTTPS unless disabled for development by setting RequireHttpsMetadata=false.

只用于开发环境

```csharp
.AddJwtBearer("Bearer", options =>
           {
               options.Authority = Configuration.GetSection("Identity")["Authority"];

               //This should be disabled only in development environments
               options.RequireHttpsMetadata = false;
            }
```

## 6. 参考

[IdentityServer4 in ASP.NET Core – Ultimate Beginner’s Guide](https://codewithmukesh.com/blog/identityserver4-in-aspnet-core/)

[Protected web API: Verify scopes and app roles](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-protected-web-api-verification-scope-app-roles?tabs=aspnetcore)

[Role-based authorization in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/roles?view=aspnetcore-6.0)

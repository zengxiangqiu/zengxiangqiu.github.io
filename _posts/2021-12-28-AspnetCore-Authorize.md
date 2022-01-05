---
title: "AspnetCore Authorize"
date:  2021-12-28 15:37:11 +0800
categories: [DevOps]
tags: [auth]
---

## Scoped
ids 4 配置 Client Scoped , 客户端（web）通过disconvery 中的 endpoint 发送授权请求，其中request 中 scoped会与ids4中已注册的ApiScoped比较并放入claims，返回granted code 或 access token

客户端 再 利用access token 请求 资源，资源服务（microservice） 通过[Authorize] 和[RequiredScope] 验证 token 中scope 是否符合。

[Secure Applications with OAuth2 and OpenID Connect in ASP.NET Core 5 – Complete Guide](https://procodeguide.com/programming/oauth2-and-openid-connect-in-aspnet-core/) 详尽地介绍了OpenID Connection和OAuth 2.0 ，并结合identity server 4 讲解原理，可读性高，又源码，可以细读学习。

## 笔记
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



3. This Authentication configuration will make use of the discovery document on startup to configure the security for this API
    ```
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
    aspnetcore webapi 搭建微服务，利用第三方依赖包注册服务，并指定授权服务，ApiName 对应已注册的API Resources，最后在pipline中调用

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

[IdentityServer4 in ASP.NET Core – Ultimate Beginner’s Guide](https://codewithmukesh.com/blog/identityserver4-in-aspnet-core/)

[Protected web API: Verify scopes and app roles](https://docs.microsoft.com/en-us/azure/active-directory/develop/scenario-protected-web-api-verification-scope-app-roles?tabs=aspnetcore)

[Role-based authorization in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/roles?view=aspnetcore-6.0)

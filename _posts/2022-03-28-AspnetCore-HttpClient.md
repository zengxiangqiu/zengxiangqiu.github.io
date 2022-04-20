---
title: "AspnetCore HttpClient"
date:  2022-03-28 11:02:35 +0800
categories: [AspnetCore]
tags: [httpclient]
---

## IHttpClientFactory

using语句实例化HttpClient，会存在套接字耗尽（退出using后套接字没有及时释放，网传维持240s）和无法处理DNS更改问题（因为它的构造函数每次都new HttpMessageHandler）

建议采用DI的IHttpClientFactory，此工厂将池的 HttpMessageHandler 分配给 HttpClient，实现套接字复用，客户端可配置和Polly 的容错策略

## Refit

Refit 将您的 REST API 变成了一个实时接口,下面的内容包含了OAuth的认证

### 安装

```nuget
nuget install IdentityModel.AspnetCore;
nuget install Refit;
```

### 代码

```csharp
public interface IApiOperations
{
    [Get("/staffs/{id}")]
    Task<ApiResponse<Staff>> GetStaff(string id);
}
```

```csharp
services.AddAccessTokenManagement(options =>
{
    options.Client.Clients.Add("api", new ClientCredentialsTokenRequest
    {
        RequestUri = new Uri(new Uri(Configuration.GetSection("Identity")["host"]), new Uri("/identity/connect/token", UriKind.Relative)),
        ClientId = "xxx",
        ClientSecret = "xxx",
    });
});

Func<HttpClientHandler> handlerConfigure = () =>
{
    var handler = new HttpClientHandler();
    if (env.IsDevelopment() || env.IsStaging())
    {
        handler.ServerCertificateCustomValidationCallback = HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;
    }
    return handler;
};

services.AddHttpClient(AccessTokenManagementDefaults.BackChannelHttpClientName)
    .ConfigurePrimaryHttpMessageHandler(handlerConfigure);


services.AddRefitClient<TClient>()
        .ConfigureHttpClient(
                (serviceProvider, httpClient) =>
                {
                    httpClient.BaseAddress = httpClientOptions.BaseAddress;
                    httpClient.Timeout = httpClientOptions.Timeout;
                })
        .ConfigurePrimaryHttpMessageHandler(configureHandler)
        .AddPolicyHandlerFromRegistry(PolicyName.HttpRetry)
        .AddPolicyHandlerFromRegistry(PolicyName.HttpCircuitBreaker)
        .AddClientAccessTokenHandler(configurationSectionName);
```


###  Policies

```csharp
var policyRegistry = services.AddPolicyRegistry();
policyRegistry.Add(
    PolicyName.HttpRetry,
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(
            policyOptions.HttpRetry.Count,
            retryAttempt => TimeSpan.FromSeconds(Math.Pow(policyOptions.HttpRetry.BackoffPower, retryAttempt))));
policyRegistry.Add(
    PolicyName.HttpCircuitBreaker,
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: policyOptions.HttpCircuitBreaker.ExceptionsAllowedBeforeBreaking,
            durationOfBreak: policyOptions.HttpCircuitBreaker.DurationOfBreak));
```

`HandleTransientHttpError` 仅在reponse status code = 408 超时或者 5XX 服务错误时 采用策略

### 测试

可以借助 http://httpstat.us/408?sleep=5000 测试不同status code 的响应



## HttpClient

### 代码

```csharp
public class GitHubService
{
    private readonly HttpClient _httpClient;

    public GitHubService(HttpClient httpClient)
    {
        _httpClient = httpClient;

        _httpClient.BaseAddress = new Uri("https://api.github.com/");
    }

    public async Task<IEnumerable<GitHubBranch>?> GetAspNetCoreDocsBranchesAsync() =>
        await _httpClient.GetFromJsonAsync<IEnumerable<GitHubBranch>>(
            "repos/dotnet/AspNetCore.Docs/branches");
}

//transient
builder.Services.AddHttpClient<GitHubService>();

//解析指定客户端服务
private readonly GitHubService _gitHubService;
```

## 参考

[Bypass SSL Certificate in .NET – Guidelines](https://www.thecodebuzz.com/bypass-ssl-certificate-net-core-windows-linux-ios-net/)

[AuthorizationHeaderValueGetter vs [Header("Authorization")]](https://stackoverflow.com/questions/50319637/refit-and-oauth-authentication-in-c-why-http-once-again/57134930#57134930)

[OAuth2 with refit](https://mindbyte.nl/2021/06/02/simple-oauth2-api-authentication-with-token-caching-and-refetching-in-an-azure-function-using-identitymodel-and-refit.html)

[跨微服务共享 DTO 的方法](https://softwareengineering.stackexchange.com/questions/366235/ways-to-share-dto-across-microservices)

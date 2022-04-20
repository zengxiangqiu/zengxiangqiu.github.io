---
title: "WebApiClient"
date:  2022-04-14 16:20:57 +0800
categories: [nuget]
tags: [httpclient]
---

## WebApiClient

### 基本用法

```csharp

public interface IUserApi
{
    [HttpGet("api/users/{id}")]
    Task<User> GetAsync(string id);

    [HttpPost("api/users")]
    Task<User> PostAsync([JsonContent] User user);
}
```
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHttpApi<IUserApi>();
}
```

```csharp
public class MyService
{
    private readonly IUserApi userApi;
    public MyService(IUserApi userApi)
    {
        this.userApi = userApi;
    }
}
```

### 源码分析

[源码下载](https://github.com/dotnetcore/WebApiClient)

注入，TryAddTransient调用了httiApiProvider.CreateHttpApi，IHttpApiActivator的实现类DefaultHttpApiActivator重写了CreateFactory，

```csharp
services.TryAddTransient(serviceProvider =>
{
  var httiApiProvider = serviceProvider.GetRequiredService<HttpApiProvider<THttpApi>>();
  return httiApiProvider.CreateHttpApi(serviceProvider, name);
});


this.httpApiActivator.CreateInstance(httpApiInterceptor)
  this.factory(apiInterceptor, this.actionInvokers)
  client=httpClientFactory.CreateClient
  HttpApiInterceptor(HttpClientContext context)
    Intercept
      actionInvoker.Invoke(this.context, arguments)
        ApiRequestExecuter.ExecuteAsync(request)
          HandleRequestAsync(request)
            attr.OnRequestAsync(context)//这里利用attribute实现内容格式转换，比如object->json 等

BuildMethods
	 for (var i = 0; i < actionMethods.Length; i++)
    {
      //每个空白的接口方法插入>Intercep方法

      //Intercep方法则call ApiRequestExecuter.ExecuteAsync

      //通过emit将接口类实现为普通类
    iL.Emit(OpCodes.Callvirt, interceptMethod);
    ...
    }

services.AddHttpClient("IUserApi"); //msdn是利用aot代码生成，见上一节的代码，这里利用emit实现

```


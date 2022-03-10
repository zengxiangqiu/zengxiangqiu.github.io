---
title: "AspnetCore Testing"
date:  2022-01-04 14:55:42 +0800
categories: [AspnetCore]
tags: [测试]
---

## 常见的三种测试

1. 单元测试（Unit tests）
2. 集成测试 （Integration tests）
3. 点对点测试 （End-to-end (E2E) tests）

经典实现 a typical approach follows the so-called AAA pattern.

* Arrange. With this action, you prepare all the required data and preconditions.
* Act. This action performs the actual test.
* Assert. This final action checks if the expected result has occurred.

## xunit

* 依赖

  ```csharp
  <ItemGroup>
      <PackageReference Include="MartinCostello.Logging.XUnit" Version="0.2.0" />
      <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="5.0.13" />
      <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="5.0.13" />
      <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.9.4" />
      <PackageReference Include="xunit" Version="2.4.1" />
      <PackageReference Include="xunit.runner.visualstudio" Version="2.4.3">
        <PrivateAssets>all</PrivateAssets>
        <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      </PackageReference>
      <DotNetCliToolReference Include="dotnet-xunit" Version="2.3.1" />
    </ItemGroup>
  ```

* 基本语法 AAA pattern

  ```csharp
      [Fact]
      public void ValidPassword()
      {
        //Arrange
        var passwordValidator = new PasswordValidator();
        const string password = "Th1sIsapassword!";

        //Act
        bool isValid = passwordValidator.IsValid(password);

        //Assert
        Assert.True(isValid, $"The password {password} is not valid");
      }
  ```
* Class Fixtures

  When to use: when you want to create **a single test context** and share it among all the tests in the class, and have it cleaned up after all the tests in the class have finished.

  ...If the test class needs access to the fixture instance, add it as a constructor argument, and it will be provided **automatically**.

  单类中共享单例， 构造函数自动注入

* Collection

  When to use: when you want to create a single test context and share it among tests **in several test classes**, and have it cleaned up after all the tests in the test classes have finished.

  多类中共享单例

  参考 [Shared Context between Tests](https://xunit.net/docs/shared-context#collection-fixture)

  也有人解释为[单例](https://www.cnblogs.com/weihanli/p/dependency-injection-in-xunit.html)


## AspnetCore集成测试

  集成测试中引用`Microsoft.AspNetCore.Mvc.Testing`

  其中mock（模拟授权）的工作原理是 重写以上测试引用包中的 ConfigureWebHost 中的jwt验证，以及post request 前生成伪装的accesstoken。

  具体查看[Using xUnit to Test your C# Code](https://auth0.com/blog/xunit-to-test-csharp-code/)

  WebApplicationFactory<T> 适用于点对点的测试,重写`CreateHostBuilder`和`ConfigureWebHost`

  ```csharp
  public CustomWebApplicationFactory(ITestOutputHelper testOutputHelper)
  {
      _testOutputHelper = testOutputHelper;
  }
  ```

  ```csharp
  protected override IHostBuilder CreateHostBuilder()
  {
      var builder = base.CreateHostBuilder();
      builder.ConfigureLogging(logging =>
      {
          logging.ClearProviders(); // Remove other loggers
          logging.AddXUnit(_testOutputHelper); // Use the ITestOutputHelper instance
      });

      return builder;
  }

  protected override void ConfigureWebHost(IWebHostBuilder builder)
  {
      // Don't run IHostedServices when running as a test
      builder.ConfigureTestServices((services) =>
      {
          services.RemoveAll(typeof(IHostedService));
      });

      builder.UseEnvironment(_environment);

      //Add mock/ test services to the builder here
      builder.ConfigureServices(services =>
      {
          services.AddScoped(sp =>
          {
              // Replace SQLite with in-memory database for tests
              return new DbContextOptionsBuilder<lsj50Context>()
                  .UseInMemoryDatabase("DbForPublicApi")
                  .UseApplicationServiceProvider(sp)
                  .Options;
          });
      });

      // Overwrite registrations from Startup.cs
      builder.ConfigureTestServices(serviceCollection =>
      {
          var authenticationBuilder = serviceCollection.AddAuthentication();
          authenticationBuilder.Services.Configure<AuthenticationOptions>(o =>
          {
              o.SchemeMap.Clear();
              ((IList<AuthenticationSchemeBuilder>)o.Schemes).Clear();
              o.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
          });
      });

      builder.ConfigureTestServices(services =>
      {
          services.PostConfigure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
          {
              options.TokenValidationParameters = new TokenValidationParameters()
              {
                  IssuerSigningKey = FakeJwtManager.SecurityKey,
                  ValidIssuer = FakeJwtManager.Issuer,
                  ValidAudience = FakeJwtManager.Audience
              };
          });
      });
  }
  ```



## 在sqlite或in-memory中测试 ef core

1. in-memory

  引用 `Microsoft.EntityFrameworkCore.InMemory`

  ```csharp
    // The database name allows the scope of the in-memory database
    // to be controlled independently of the context. The in-memory database is shared
    // anywhere the same name is used.
    var options = new DbContextOptionsBuilder<SampleDbContext>()
        .UseInMemoryDatabase(databaseName: "Test1")
        .Options;
  ```

  名字相同的话，共享同一个in-memory database。

2. sqlite

  ```csharp
    var options = new DbContextOptionsBuilder<SampleDbContext>()
      .UseSqlite("DataSource=:memory:")
      .Options;
  ```

  When the connection is opened, a new database is created in memory. This database is destroyed when the connection is closed.

  连接关闭后，databse 销毁

  参考 [Testing EF Core in Memory using SQLite](https://www.meziantou.net/testing-ef-core-in-memory-using-sqlite.htm)

3. xunit 日志 输出

  参考[Converting integration tests to .NET Core 3.0](https://andrewlock.net/converting-integration-tests-to-net-core-3/)

  重点参考 [传入TestOutputHelper](https://stackoverflow.com/questions/59166798/net-core-3-0-issue-with-iclassfixture-unresolved-constructor-arguments-ites)

  在`factory.createClient`前 传入 `factory.TestOutputHelper = ...`

  在测试类中实例化，传入ITestOutputHelper
  ```csharp
  public ListEndpoint(ITestOutputHelper testOutputHelper)
  {
      var  factory = new CustomWebApplicationFactory<Startup>(testOutputHelper);
      httpClient = factory.CreateClient();
      _testOutputHelper = testOutputHelper;
  }
  ```

3. xunit 模拟jwt获取认证授权

  参考 [Using xUnit to Test your C# Code](https://auth0.com/blog/xunit-to-test-csharp-code/#Mocking-External-Dependencies)

  如果服务中添加了服务发现

  ```csharp
  services.AddAuthentication(/*"Bearer"*/options =>
          {
              options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
          })
          .AddJwtBearer("Bearer", options =>
          {
              options.Authority = "https://localhost:5001";

              options.TokenValidationParameters = new TokenValidationParameters
              {
                  ValidateAudience = false
              };

              // if you are using API resources, you can specify the name here
              //options.Audience = "resource1";

              // IdentityServer emits a typ header by default, recommended extra check
              //options.TokenValidationParameters.ValidTypes = new[] { "at+jwt" };
          });
  ```

  需要在测试项目中重写

  For SUTs that still use the Web Host, the test app's builder.ConfigureServices callback is executed before the SUT's Startup.ConfigureServices code. The test app's **builder.ConfigureTestServices** callback is executed **after**.

  ```csharp
  // Overwrite registrations from Startup.cs
  builder.ConfigureTestServices(serviceCollection =>
  {
      var authenticationBuilder = serviceCollection.AddAuthentication();
      authenticationBuilder.Services.Configure<AuthenticationOptions>(o =>
      {
          o.SchemeMap.Clear();
          ((IList<AuthenticationSchemeBuilder>)o.Schemes).Clear();
          o.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
      });
  });
  ```

4. 在测试库中 seed Data

   ```csharp
     if (env.IsDevelopment())
     {
         logger.LogInformation("Seeding Database...");
         using (var scope = app.ApplicationServices.CreateScope())
         {
             var scopedProvider = scope.ServiceProvider;
             try
             {
                 var lsj50Context = scopedProvider.GetRequiredService<lsj50Context>();
                 await lsj50ContextSeed.SeedAsync(lsj50Context, logger);
             }
             catch (Exception ex)
             {
                 logger.LogError(ex, "An error occurred seeding the DB.");
             }
         }
     }
   ```

## 常见问题
1.  Response status code does not indicate success: 415 (Unsupported Media Type).
  ```csharp
  public override async Task<ActionResult<DeleteStaffResponse>> HandleAsync(
    **[FromRoute]**DeleteStaffRequest request
    , CancellationToken cancellationToken = default)

  ```
2. System.InvalidOperationException : Each parameter in constructor 'Void .ctor(System.Guid)' on type 'Lsd.PublicApi.StaffEndpoints.

   序列化需要默认不带参数的构造函数

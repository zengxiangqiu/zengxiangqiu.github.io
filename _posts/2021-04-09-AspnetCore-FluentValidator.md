---
title: "AspnetCore FluentValidator"
date:  2022-01-14 16:29:46 +0800
categories: [AspnetCore]
tags: [FluentValidator]
---

传统上，.NET中的大多数验证都是使用数据注释来完成的
```CSharp
public class SampleClass
{
    [Required]
    public int Id { get; set; }

    [MaxLength(100)]
    public string Name { get; set; }
}
```
这种方法存在一些问题：

- 我们的模型可能会““肿”
- 扩展性有限
- 测试不是最好的体验

为了解决其中的一些问题，我们将使用一个名为FluentValidation的.NET库对我们的类进行验证。

## 1. 安装package
`dotnet add package FluentValidation.AspNetCore`

## 2. 自动注册
```CSharp
public void ConfigureServices(IServiceCollection services)
{

  services.AddControllers()
  .AddFluentValidation(fv => { fv.RegisterValidatorsFromAssemblyContaining<PersonValidator>();
  });
  ...
}

```
## 3. 控制器中使用验证器

  - 定义验证器
  ```CSharp
  public class Person {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
  }

  public class PersonValidator : AbstractValidator<Person> {
    public PersonValidator() {
      RuleFor(x => x.Id).NotNull();
      RuleFor(x => x.Name).Length(0, 10);
      RuleFor(x => x.Email).EmailAddress();
      RuleFor(x => x.Age).InclusiveBetween(18, 60);
    }
  }
  ```

注意,实例生命周期为Transient,而不是单例,避免避免生命周期范围问题


## 4. FluentValidation.AspnetCore

源码 `services.Configure<MvcOptions>` 中 `options.ModelMetadataDetailsProviders.Add(new FluentValidationBindingMetadataProvider());`

将FluentValidationBindingMetadataProvider 添加入 ModelMetadataDetailsProviders

ModelValidationContext -> validator-> Validate()->result.Errors

## 参考

[fluentvalidation文档](https://docs.fluentvalidation.net/en/latest/custom-validators.html)

[fluentvalidationh简单示例](https://code-maze.com/fluentvalidation-in-aspnet/)

[fluentvalidation与asp.net core](https://www.carlrippon.com/fluentvalidation-in-an-asp-net-core-web-api)

[ASP.NET Core + FluentValidation + Swagger(进阶)](https://anexinet.com/blog/asp-net-core-fluentvalidation-swagger/)

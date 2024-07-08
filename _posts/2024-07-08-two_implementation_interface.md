---
title: "استفاده از چند پیاده سازی مختلف از Interface در IOC"
categories:
  - Net
tags:
  - implementation
  - ioc
  - registration
---

گاهی مواقع نیاز دارید تا چند پیاده سازی از یک Interface داشته باشید و با توجه به اینکه در کدام بخش برنامه هستید نوع مختلفی از آن را استفاده کنید.  
بطور مثال یک اینترفیس با نام IQueueReader دارید که یک پیاده سازی از Rabbitmq و دیگری از Kafka استفاده می‌کند. برای اینکه بتوانید از پیاده‌سازی مختلف آن استفاده کنید بصورت زیر می‌توانید عمل کنید.  
ابتدا هر تعداد سرویس دارید را شبیه به زیر رجیستر کنید.  
مورد مهم بعدی رجیستر کردن Func<string, IMyService> است که در داخل آن به ازای هر پیاده‌سازی یک کلید مشخص کرده‌اید.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<MyServiceA>();
    services.AddSingleton<MyServiceB>();

    services.AddSingleton<Func<string, IMyService>>(serviceProvider => key =>
    {
        return key switch
        {
            "A" => serviceProvider.GetService<MyServiceA>(),
            "B" => serviceProvider.GetService<MyServiceB>(),
            _ => throw new KeyNotFoundException($"Service with key '{key}' is not registered.")
        };
    });

    services.AddSingleton<CustomConsumer>();
}
```

```csharp
public interface IMyService
{
    void Execute();
}

public class MyServiceA : IMyService
{
    public void Execute()
    {
        Console.WriteLine("MyServiceA Execute method called.");
    }
}

public class MyServiceB : IMyService
{
    public void Execute()
    {
        Console.WriteLine("MyServiceB Execute method called.");
    }
}
```


برای دریافت پیاده‌سازی به یک کلاس واسط نیاز دارید که با دریافت کلید پیاده‌سازی مرتبط به آن را برگرداند.  

```csharp
public class CustomConsumer
{
    private readonly Func<string, IMyService> _serviceAccessor;

    public MyConsumer(Func<string, IMyService> serviceAccessor)
    {
        _serviceAccessor = serviceAccessor;
    }

    public IMyService GetService(string key)
    {
       return _serviceAccessor(key);
    }
}
```

سپس بصورت زیر می‌توانید از کلاس فوق استفاده کنید و پیاده‌سازی دلخواه خود را دریافت کنید.  

```csharp
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;
    private readonly IMyService _myService;
    private const string ServiceKey = "A";

    public Worker(ILogger<Worker> logger, CustomConsumer customConsumer)
    {
        _logger = logger;
        _myService = customConsumer.GetService(ServiceKey);
    }

    private void Do()
    {
        _myService.Execute();
    }
}
```
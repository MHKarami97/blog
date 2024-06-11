---
title: "Mock کردن IConfiguration در تست‌ها"
categories:
  - Net
tags:
  - moq
  - configuration
  - database
---

در تست‌های IntegrationTest در مواقعی که نیاز دارید ارتباط با دیتابیس را بررسی کنید در زمان‌هایی که نیاز به تغییر ConnectionString وجود دارد توسط کد زیر می‌توانید آن را Mock کنید و مقداری که نیاز دارید را برای تست برگردانید.  

در متود زیر ابتدا IConfigurationSection توسط کتابخانه Mock شده است تا در صورتی که Database خواسته شد مقدار دلخواه ما برگشت داده شود.  
سپس IConfiguration نیز Mock شده است تا وقتی خود آبجکت ConnectionStrings خواسته شد مقدار کانفیگ شده در خط قبل برگشت داده شود.  
سپس نسخه ماک شده به کلاس دلخواه پاس داده شده است.  

```csharp
{
  "ConnectionStrings": {
    "Database": "Server=.;Database=MyDb;Persist Security Info=True;User ID=guest;Password=guest;MultipleActiveResultSets=True;TrustServerCertificate=True;Max Pool Size=500;Application Name=MySystem"
  }
}
```

```c#
using Moq;

private void ConfigDataUpdater()
{
    var mockConfSection = new Mock<IConfigurationSection>();
    mockConfSection.SetupGet(m => m[It.Is<string>(s => s == "Database")]).Returns("MyConnectionString");

    var mockConfiguration = new Mock<IConfiguration>();
    mockConfiguration.Setup(a => a.GetSection(It.Is<string>(s => s == "ConnectionStrings"))).Returns(mockConfSection.Object);

    _myRepository = new MyRepository(new MyDbContext(mockConfiguration.Object));
}
```

پیاده سازی MyDbContext در کد بالا نیز بصورت زیر است:  

```csharp

public class MyDbContext: IDbConnector<MyDbContext>
{
    private readonly string _connectionString;

    public MyDbContext(IConfiguration configuration)
    {
        _connectionString = configuration.GetConnectionString("Database");
    }

    public IDbConnection CreateConnection()
    {
        return new SqlConnection(_connectionString);
    }
}
```


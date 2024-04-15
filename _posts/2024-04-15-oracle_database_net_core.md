---
title: "استفاده از دیتابیس Oracle در .Net Core"
categories:
  - Net
tags:
  - net
  - oracle
  - entity_framework_core
  - database
---

یکی از خوبی‌های استفاده از EntityFrameworkCore در برنامه این است که بدون نیاز به تغییر خاصی در کد و با کمترین کار می‌توانید دیتابیس سیستم خود را تغییر دهید و از یک دیتابیس جدید استفاده کنید.  
استفاده از oracle در .net core نیز به سادگی با نصب پکیچ امکان‌پذیر است که در البته چند نکته خاص دارد که در این مطلب آنها را بیان می‌کنیم.  
ابتدا nuget پکیچ زیر را نصب بفرمایید:  
[nuget](https://www.nuget.org/packages/Oracle.EntityFrameworkCore/)  

سپس یک کلاس با نام دلخواه بسازید و از DbContext ارث‌بری کنید. نکته مهم در این بخش این است که فیلد `TimeStamp` در اوراکل با فیلد DateTime در .net تفاوت دارد و نیاز هست هر کدام را بصورت جداگانه شبیه به کد زیر معرفی بکنید.  

```csharp
using Microsoft.EntityFrameworkCore;

namespace DataAccess;

public class OracleDbContext : DbContext
{
    public OracleDbContext(DbContextOptions<OracleDbContext> options) : base(options)
    {
        
    }
        
    public DbSet<MainLog> MainLogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<MainLog>().Property(p => p.REQUEST_TIME).HasColumnType("TIMESTAMP");
        modelBuilder.Entity<MainLog>().Property(p => p.RECEIVE_TIME).HasColumnType("TIMESTAMP");

        base.OnModelCreating(modelBuilder);
    }
}
```
نمونه Entity:  

```csharp
[Table("LOG_INFO", Schema = "MAIN")]
public sealed class MainLog : PreparedLog
{
    public DateTime REQUEST_TIME { get; set; }
}
```
سپس کافی است در program برنامه خود بصورت زیر دیتابیس را معرفی بکنید:  

```csharp
services.AddDbContextFactory<OracleDbContext>(options => { options.UseOracle(configuration.GetConnectionString("OracleDatabase")); });
```

نکته مهم تفاوت ساختار connectionString است که در اوراکل متفاوت است. نمونه آن در زیر آمده است که می‌توانید از آن استفاده کنید.  

```csharp
"ConnectionStrings": {
"OracleDatabase": "Data Source=152.24.64.51:1521/MyDbName;PERSIST SECURITY INFO=True;User Id=MyUser;Password=MyPass;Min Pool Size=50;Connection Lifetime=120;Connection Timeout=60;Incr Pool Size=5;Decr Pool Size=2;",
}
```
در بخش Repository نیز کافی است شبیه به کد زیر که بین دیتابیس ‌های مختلف مشترک است با دیتابیس کار بکنید و بطور مثال رکورد خود را در دیتابیس اضافه کنید.  

```csharp
public class LogSenderRepository : ILogSenderRepository
{
    private readonly IDbContextFactory<OracleDbContext> _dbContextFactory;
    private readonly ILogger<LogSenderRepository> _logger;

    public LogSenderRepository(IDbContextFactory<OracleDbContext> dbContextFactory)
    {
        _dbContextFactory = dbContextFactory;
    }

    public async Task SendLogAsync(IReadOnlyCollection<MainLog> input)
    {
        using (Tracing.ActivitySource.StartActivity())
        {
            await using var context = await _dbContextFactory.CreateDbContextAsync();
            context.ChangeTracker.AutoDetectChangesEnabled = false;

            await context.MainLogs.AddRangeAsync(input);

            await context.SaveChangesAsync();
        }
    }
}
```
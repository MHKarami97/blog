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
"OracleDatabase": "Data Source=172.0.0.1:1521/DbName;PERSIST SECURITY INFO=True;User Id=MyUser;Password=MyPass;Pooling=true;Min Pool Size=1;Max Pool Size=50;Incr Pool Size=5;Connection Lifetime=180",
}
```
`کانکشن بالا بصورت بهینه ایجاد شده است و تغییر مقادیر آن ممکن است باعث بروز خطا در اتصال به دیتابیس شود`  

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

نکته مهم: `استفاده از کتابخانه بالا در بخش SaveChangeAsync باعث 100 درصد شدن cpu در بعضی از مواقع می‌شود و پیشنهاد می‌شود از روش زیر استفاده شود`  

خطا:  
```csharp
Oracle error ORA-12571 encountered
 ---> OracleInternal.Network.NetworkException (0x80004005): Oracle error ORA-12571 encountered
 ---> System.Net.Sockets.SocketException (110): Connection timed out
```

در روش جایگزین از این کتابخانه استفاده می‌شود:  

```csharp
<PackageReference Include="Oracle.ManagedDataAccess.Core" Version="3.21.140" />
```

و سپس بصورت مستقیم کوئری نوشته می‌شود. همچنین برای تبدیل یک کلاس به کوئری Insert نیز از روش زیر و جلوگیری از بروز خطا در آن نیز یک کلاس بصورت خودکار به کوئری تبدیل شده است.  
در انتها اگر خطا عدم اتصال به دیتابیس نیز دریافت شود 3 بار تلاش مجدد نیز در کد قرار داده شده است.  

```csharp
private async Task SaveUsingOracleSingleCopyAsync(string destTableName, PreparedLog item, int retry = 3)
{
    await using var connection = new OracleConnection(_connectionString);

    try
    {
        await using var command = connection.CreateCommand();

        await connection.OpenAsync();

        var columnNames = string.Join(',', typeof(PreparedLog).GetProperties().Select(p => $"{p.Name}"));

        var parameterPlaceholders = string.Join(',', typeof(PreparedLog).GetProperties().Select(p => $":{p.Name}"));

        command.CommandText = $"INSERT INTO {BourseSchema}.{destTableName} ({columnNames}) VALUES ({parameterPlaceholders})";

        foreach (var property in typeof(PreparedLog).GetProperties())
        {
            command.Parameters.Add(property.Name, property.GetValue(item));
        }

        await command.ExecuteNonQueryAsync();
    }
    catch (OracleException e)
    {
        _logger.LogWarning(e, "retry on send to bourse");

        if (e.Number is 12570 or 03135 or 12571)
        {
            if (retry == 0)
                throw new AppException("ex on SaveUsingOracleSingleCopy after 3 time retry", e);

            await SaveUsingOracleSingleCopyAsync(destTableName, item, retry - 1);
        }

        throw;
    }
    finally
    {
        await connection.CloseAsync();
    }
}
```

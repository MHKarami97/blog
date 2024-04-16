---
title: "مدیریت خطاها در EntityFramework"
categories:
  - Net
tags:
  - net
  - exception
  - entity_framework_core
  - database
---

بصورت پیش‌فرض خطاهای دیتابیسی از نوع `DbUpdateException` بازگردانده می‌شوند. اگر نیاز دارید که جزئیات خطا را متوجه شوید تا بتوانید در رابطه با آن تصمیم بگیرید می‌توانید از `EntityFramework.Exceptions` استفاده کنید.  
بدین منظور ابتدا پکیج زیر را نصب کنید:  

[nuget](https://www.nuget.org/packages/EntityFrameworkCore.Exceptions.Common)  

اکنون کافی است در `OnConfiguring` متود زیر را اضافه کنید:  

```csharp
public class MyContext : DbContext
{
    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Product>().HasIndex(u => u.Name).IsUnique();
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseExceptionProcessor();
    }
}
```

و یا بصورت زیر از آن استفاده کنید:  

```csharp
builder.Services.AddDbContextFactory<DemoContext>(options => options.UseSqlServer(config.GetConnectionString("MyConnection")).UseExceptionProcessor());
```

اکنون می‌توانید خطاهای زیر را در سیستم دریافت کنید:  

- ReferenceConstraintException 
- NumericOverflowException
- MaxLengthExceededException
- CannotInsertNullException
- UniqueConstraintException

[github](https://github.com/Giorgi/EntityFramework.Exceptions)  


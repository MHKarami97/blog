---
title: "لاگ کردن کوئری در EntryFrameWork"
categories:
  - Net
tags:
  - net
  - sql
  - entryFrameWork
---

یکی از ویژگی‌هایی که `Entity Framework Core` دارد، قابلیت لاگ کردن کوئری نهایی است که به دیتابیس ارسال می‌شود. توسط این امکان می‌توانید حالت های مختلف را امتحان کنید تا به بهینه ترین حالت برای کوئری زدن به دیتابیس برسید.  
فعال‌سازی این قابلیت در `EF Core 5.0` نیز بسیار راحت‌تر شده است که در این مطلب به آن می‌پردازیم.  

برای فعال‌سازی قابلیت لاگ کافی است متود `OnConfiguring` را جایی که Context خود را می‌سازید. بازنویسی کنید:  

```c#
public class BloggingContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=Blogging;Trusted_Connection=True");
    }
}
```

و خط زیر را به آن اضافه کنید تا کوئری های شما در کنسول نشان داده شوند:  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(Console.WriteLine);
```

اگر می‌خواهید کوئری ها فقط در صفحه `Debug window` نشان داده شوند، کافی است بجای خط بالا، از کد زیر استفاده کنید:  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(message => Debug.WriteLine(message));
```

اگر می‌خواهید لاگ‌ها را در محل دیگر مانند File بنویسید، کافی است آن را در متود `LogTo` فراخوانی کنید تا لاگ‌های شما در آن محل نوشته شود:  

```c#
private readonly StreamWriter _logStream = new StreamWriter("mylog.txt", append: true);

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(_logStream.WriteLine);

public override void Dispose()
{
    base.Dispose();
    _logStream.Dispose();
}

public override async ValueTask DisposeAsync()
{
    await base.DisposeAsync();
    await _logStream.DisposeAsync();
}
```

بصورت پیش‌فرض، لاگ‌های نوشته شده شامل اطلاعات ارسالی به دیتابیس نیست و فقط کوئری لاگ می‌شود. اگر نیاز دارید تا مقادیر ارسالی به دیتابیس نیز در لاگ نهایی باشند، لازم است تا `EnableSensitiveDataLogging` به کد خود اضافه کنید.  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .LogTo(Console.WriteLine)
        .EnableSensitiveDataLogging();
```

اگر کد شما در بلاک `Try/Catch` باشد و نوع آن نیز `Read` باشد، کوئری نهایی آن به دلیل Performance لاگ نمی‌شود. اگر نیاز دارید تا این موارد نیز در لاگ نوشته شوند کافی است `EnableDetailedErrors` را به کد خود اضافه کنید.  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .LogTo(Console.WriteLine)
        .EnableDetailedErrors();
```

نوع پیش‌فرض لاگ ساخته شده `Debug` است و تمام مواردی که نوع آنها Debug و بالاتر باشد، لاگ خواهند شد.  
اگر می‌خواهید حداقل مرحله برای لاگ را تغییر دهید می‌توانید بصورت زیر عمل کنید:  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder.LogTo(Console.WriteLine, LogLevel.Information);
```

بر روی مواردی که لاگ می‌شوند نیز می‌توانید فیلترهای مختلف قرار بدهید. بطور مثال در کد زیر فقط زمانی که Context در حالت Init یا Dispose است، لاگ زده می‌شود.  
این عمل توسط `EventId` مدیریت می‌شود که لیست بیشتر این موارد را می‌توانید در لینک زیر مشاهده کنید:  

[core event id](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.diagnostics.coreeventid?view=efcore-5.0)  

```c#
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    => optionsBuilder
        .LogTo(Console.WriteLine, new[] { CoreEventId.ContextDisposed, CoreEventId.ContextInitialized });
```

[توضیحات بیشتر](https://docs.microsoft.com/en-us/ef/core/logging-events-diagnostics/simple-logging)
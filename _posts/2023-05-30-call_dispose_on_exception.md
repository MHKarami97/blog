---
title: "فراخوانی Dispose در صورت بروز Exception"
categories:
  - Net
tags:
  - exception
  - idisposable
  - net
---

در صورتی که در کد خود کلاسی دارید که از `IDisposable` ارث‌بری کرده باشد :  

```csharp
namespace DisposeChecker;

public sealed class TraceLogger : IDisposable
{
    private string _activity = "none";

    public TraceLogger()
    {
        _activity = "ctor";
        Console.WriteLine(_activity);
    }

    public void Dispose()
    {
        _activity = "dispose";
        Console.WriteLine(_activity);
    }
}
```

و بصورت `using` از آن در کد خود استفاده کرده باشید:  

```csharp
using DisposeChecker;

Console.WriteLine("Hello, World!");

using (new TraceLogger())
{
    Console.WriteLine("in using");

    try
    {
        Console.WriteLine("in try");

        throw new Exception("1");
    }
    catch (Exception e)
    {
        Console.WriteLine("in catch");
    }

    try
    {
        Console.WriteLine("in try 2");
        
        throw new Exception("2");
    }
    catch (Exception e)
    {
        Console.WriteLine("in catch2");

        throw;
    }
}

Console.WriteLine("end");
```

در صورتی که در کد خود به خطا بخورید و آن را به لایه‌های بالایی هم ارسال کرده باشید باز هم متود `Dispose` فراخوانی می‌شود  و نیازی به استفاده از `finally` در کد برای فراخوانی آن نیست.  
بطور مثال در کد بالا در هر صورت dispose در کنسول نوشته می‌شود.  
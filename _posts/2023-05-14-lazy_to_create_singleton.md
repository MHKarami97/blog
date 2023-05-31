---
title: "استفاده از Lazy برای ایجاد کردن کلاس Singleton"
categories:
  - Net
tags:
  - net
  - singleton
  - lazy
---

قبلا در مطلبی دیزاین پترن Singleton معرفی شده بود:  

[Singleton](https://blog.mhkarami97.ir/designpattern/singleton_design_pattern/)  

یکی از روش‌های بهتر پیاده سازی این نوع کلاس استفاده از Lazy است که استفاده از Lock را غیر ضروری می‌کند.  
استفاده از این روش خوبی‌های زیر را دارد:  

- ساده شدن کد و افزایش خوانایی آن
- ازبین بردن هزینه استفاده از Lock
- ازبین بردن سربار Init شدن کلاس در زمان‌هایی که به آن نیاز نیست

```csharp
public sealed class Singleton
{
    private Singleton()
    {
        // Do some initialization here
    }

    public void DoSomething()
    {
        // Do something here
    }
}

private static Lazy<Singleton> lazy = new Lazy<Singleton>(() => new Singleton());

public static Singleton Instance
{
    get
    {
        return lazy.Value;
    }
}
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        var singleton = Singleton.Instance;

        singleton.DoSomething();

        // Print the hash code of the instance to verify that it is the same for all threads
        Console.WriteLine("Singleton instance {0}", singleton.GetHashCode());
    }
}
```

[lazy](https://learn.microsoft.com/en-us/dotnet/api/system.lazy-1?view=net-7.0)  
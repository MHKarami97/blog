---
title: "استفاده از متد async در foreach"
categories:
  - Net
tags:
  - net
  - async
  - foreach
---

برای انجام عملیات بر روی تمام آیتم های یک لیست، روش های مختلفی وجود دارد که تفاوت هایی با هم دارند.  
یکی از مهم‌ترین موارد در زمان‌هایی است که شما می‌خواهید یک عملیات async را بر روی آیتم‌های یک لیستم انجام بدهید.  
بطور مثال کد زیر را در نظر بگیرید که شبیه ساز یک عملیات httpClient است:  

```c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class Example1
{
    public static async Task Main()
    {
        var data = new List<MyClass>
        {
            new()
            {
                Id = 1,
                Title = "ali"
            },
            new()
            {
                Id = 2,
                Title = "mohammad"
            }
        };

        try
        {
            data.ForEach(a => SendData(a, 1, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }

        try
        {
            data.ForEach(async a => await SendData(a, 2, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }

        foreach (var item in data)
        {
            try
            {
                await SendData(item, 3, 5);
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
        }

        try
        {
            data.ForEach(a => SendData(a, 4, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }
    }

    private static async Task SendData(MyClass input, int witchInput, int delay)
    {
        Console.WriteLine($"SendData start, Id = {input.Id}, Witch = {witchInput}");

        await Task.Delay(delay);

        Console.WriteLine($"SendData finish, Id = {input.Id}, Witch = {witchInput}");

        throw new Exception($"ex throw Id = {input.Id}, Witch = {witchInput}");
    }
}

public class MyClass
{
    public int Id { get; set; }
    public string Title { get; set; }
}
```

خروجی کد بالا بصورت زیر است که البته به این دلیل که دو خط data.ForEach منتظر پایان عملیات نمی‌مانند، خروجی ممکن است متفاوت باشد.  
از موارد مهم در کد زیر، دریافت نشدن exception های حالت 1 است.  
به دلیل wait نشدن در حالت 1 و 2 برنامه به کار خود ادامه می‌دهد و منتظر پاسخ و یا خطای آن نمی‌ماند.  

حالت شماره 2 نیز مانند حالت شماره 1 است و در این حالت خروجی بصورت `async void` است. زیرا خروجی .foreach از نوع `Action<T>` است.  

[Action](https://docs.microsoft.com/en-us/dotnet/api/System.Action-1?view=net-6.0&viewFallbackFrom=netcore-6.0)  

[List.ForEach](https://docs.microsoft.com/en-us/dotnet/api/System.Collections.Generic.List-1.ForEach?view=net-6.0&viewFallbackFrom=netcore-6.0)  

خطای این مورد نیز در catch حالت 3 دریافت شده است، نه catch خود حالت 2.  برای تست این حالت می‌توانید کد را بصورت تکه کد بعدی تغییر دهید و یا مقدار delay حالت 2 را افزایش دهید تا بیشتر از مجموع حالت 3 طول بکشد.  


``` c#
SendData start, Id = 1, Witch = 1
SendData start, Id = 2, Witch = 1
SendData start, Id = 1, Witch = 2
SendData start, Id = 2, Witch = 2
SendData start, Id = 1, Witch = 3
SendData finish, Id = 1, Witch = 2
SendData finish, Id = 2, Witch = 2
SendData finish, Id = 1, Witch = 3
SendData finish, Id = 1, Witch = 1
SendData finish, Id = 2, Witch = 1
Unhandled exception. Unhandled exception. System.Exception: ex throw Id = 1, Witch = 3
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.Main() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 59
SendData start, Id = 2, Witch = 3
System.Exception: ex throw Id = 1, Witch = 2
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.<>c.<<Main>b__0_1>d.MoveNext() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 48
--- End of stack trace from previous location ---
   at System.Threading.Tasks.Task.<>c.<ThrowAsync>b__127_1(Object state)
   at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
   at System.Threading.ThreadPoolWorkQueue.Dispatch()
   at System.Threading.PortableThreadPool.WorkerThread.WorkerThreadStart()
   at System.Threading.Thread.StartCallback()
System.Exception: ex throw Id = 2, Witch = 2
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.<>c.<<Main>b__0_1>d.MoveNext() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 48
--- End of stack trace from previous location ---
   at System.Threading.Tasks.Task.<>c.<ThrowAsync>b__127_1(Object state)
   at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
   at System.Threading.ThreadPoolWorkQueue.Dispatch()
   at System.Threading.PortableThreadPool.WorkerThread.WorkerThreadStart()
   at System.Threading.Thread.StartCallback()
SendData finish, Id = 2, Witch = 3
System.Exception: ex throw Id = 2, Witch = 3
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.Main() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 59
SendData start, Id = 1, Witch = 4
SendData start, Id = 2, Witch = 4

Process finished with exit code 0.
```

اما اگر ترتیب اجرا کدهای زیر را تغییر دهیم و روش درست فراخوانی foreach را به بالا منتقل کنیم:  

``` c#
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class Example1
{
    public static async Task Main()
    {
        var data = new List<MyClass>
        {
            new()
            {
                Id = 1,
                Title = "ali"
            },
            new()
            {
                Id = 2,
                Title = "mohammad"
            }
        };

        try
        {
            data.ForEach(a => SendData(a, 1, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }

        foreach (var item in data)
        {
            try
            {
                await SendData(item, 3, 5);
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }
        }
        
        try
        {
            data.ForEach(async a => await SendData(a, 2, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }

        try
        {
            data.ForEach(a => SendData(a, 4, 5));
        }
        catch (Exception e)
        {
            Console.WriteLine(e);
        }
    }

    private static async Task SendData(MyClass input, int witchInput, int delay)
    {
        Console.WriteLine($"SendData start, Id = {input.Id}, Witch = {witchInput}");

        await Task.Delay(delay);

        Console.WriteLine($"SendData finish, Id = {input.Id}, Witch = {witchInput}");

        throw new Exception($"ex throw Id = {input.Id}, Witch = {witchInput}");
    }
}

public class MyClass
{
    public int Id { get; set; }
    public string Title { get; set; }
}
```

با خروجی زیر مواجه می‌شویم که هیچ کدام از exception های آیتم های 1 و 2 دریافت نشده‌اند:  

``` c#
SendData start, Id = 1, Witch = 1
SendData start, Id = 2, Witch = 1
SendData start, Id = 1, Witch = 3
SendData finish, Id = 2, Witch = 1
SendData finish, Id = 1, Witch = 3
System.Exception: ex throw Id = 1, Witch = 3
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.Main() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 50
SendData start, Id = 2, Witch = 3
SendData finish, Id = 1, Witch = 1
SendData finish, Id = 2, Witch = 3
System.Exception: ex throw Id = 2, Witch = 3
   at Example1.SendData(MyClass input, Int32 witchInput, Int32 delay) in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 85
   at Example1.Main() in D:\Local\ConsoleApp2\ConsoleApp2\Program1.cs:line 50
SendData start, Id = 1, Witch = 2
SendData start, Id = 2, Witch = 2
SendData start, Id = 1, Witch = 4
SendData start, Id = 2, Witch = 4

Process finished with exit code 0.
```

پس روش استفاده بصورت زیر نیز درست نیست و روش فراخوانی درست همان حلقه foreach معمولی است.  

``` c#
data.ForEach(async a => await SendData(a, 2, 5));
```

اطلاعات بیشتر:  
[avoid async void](https://docs.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming#avoid-async-void)  

[SynchronizationContext](https://docs.microsoft.com/en-us/archive/msdn-magazine/2011/february/msdn-magazine-parallel-computing-it-s-all-about-the-synchronizationcontext)  
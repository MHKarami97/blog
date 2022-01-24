---
title: "مدیریت خطا با Exception و Operation Result"
date: 2022-01-24T09:00:00-00:00
categories:
  - Net
tags:
  - net
  - exception
  - operationResult
  - error
---

نکته مهم : `زمانی که نمی‌دانید با exception اتفاق افتاده چه کاری می‌توانید بکنید، آن را catch نکنید`


دو روش برخورد با یک حالت:  

```c#
if (conn.State != ConnectionState.Closed)
{
    conn.Close();
}
```

```c#
try
{
    conn.Close();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.GetType().FullName);
    Console.WriteLine(ex.Message);
}
```

از حالت اول زمانی بهتر است استفاده کنید که حالت گفته شده زیاد اتفاق می‌افتد، در این سناریو حالت گفته شده زیاد اتفاق می‌افتد، پس با این کار حجم کدها را کمتر کرده‌ایم و سربار اضافی catch را هم نداریم.  
حالت دوم نیز برای زمان‌هایی که این حالت کم اتفاق می‌افتد مناسبتر است.  


اگر متود مورد استفاده شما از `IDisposable` ارث بری کرده باشد نیز روش استفاده بصورت زیر است:  

```c#
using(var item = new MethodWithDispose()) {
  var result = item.DoWork();
}
```

```c#
var item = new MethodTest()
try
{
    var result = item.DoWork();
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.ToString());
}
finally {
  item.Dispose();
}
```

حالت استفاده از using نیز، بخش finally را بصورت خودکار انجام می‌دهد و اگر داخل آن به خطا بخورید نیز، متود dispose آن فراخوانی می‌شود.  
همچنین فرض کنید اتفاق افتادن خطا را می‌خواهید در متودی که از IDisposable ارث بری کرده است بدانید. برای این کار می‌توانید از کد زیر استفاده کنید:  

```c#
using(new MyMethod())
{
	if(new Random().Next(1,10)%2 == 0)
          throw new Exception();
}

public class MyMethod : IDisposable
{
    public void Dispose()
    {
        if (Marshal.GetExceptionCode() == 0)
            Console.WriteLine("Completed Successfully!");
        else
            Console.WriteLine("Exception");
    }
}
```






```c#

```

```c#

```

### Exception در برابر برگرداندن خطا
Exception این اطمینان را می‌دهد که فراخواننده متود به کار خود ادامه نمی‌دهد.  
در حالیکه اگر خطا برگردانده شود، امکان چک نکردن وضعیت توسط فراخواننده متود وجود دارد.  




اطلاعات بیشتر:  

[best-practices-for-exceptions](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)  

[exceptions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)  

[system.exception](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=net-6.0)  

[how-to-create-user-defined-exceptions](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions)  

[microsoft]()  
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
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
متن بالا به این معنی است که اگر در متودی که بطور مثال برای نوشتن بر روی فایل استفاده می‌شود به خطا پیدا نشدن فایل می‌خورید، آن را قرار دهید و لازم نیست تمام خطاها را catch کنید:  

```c#
try
{
    file.WriteLine(msg)
}
catch (FileNotFoundException ex)
{
    Console.WriteLine(ex.ToString());
    
   // create file or throw
}
```

```c#
try
{
    file.WriteLine(msg)
}
catch (FileNotFoundException ex)
{
    Console.WriteLine(ex.ToString());
    
   // create file or throw
}
catch (Exception ex) // not good
{
    Console.WriteLine(ex.ToString());
}
```


### روش برخورد با خطا

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

[using](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/using-objects)  


### Exception در برابر برگرداندن خطا
Exception این اطمینان را می‌دهد که فراخواننده متود به کار خود ادامه نمی‌دهد.  
در حالیکه اگر خطا برگردانده شود، امکان چک نکردن وضعیت توسط فراخواننده متود وجود دارد.  


### Nullable<T>
یکی از راه‌ها برگرداندن null در صورت بخطا خوردن متود مورد نظر است تا در لایه‌های دیگر مدیریت شود.  


### Exception های از پیش تعریف شده
از exception های تعریف شده توسط خودتان در صورتی استفاده کنید که موارد پیش‌فرض مانند `ArgumentException` جوابگو نیازهای شما نباشد.  


```c#
public class MyFileNotFoundException : Exception
{
  MyFileNotFoundException(int customValue) base:(message) {

  }

  MyFileNotFoundException(string message, int customValue) base:(message) {

  }
}
```

[how-to-create-user-defined-exceptions](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions)  

[choosing-standard-exceptions](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=net-6.0#choosing-standard-exceptions)  


### از وجود داشتن اطلاعات exception اطمینان حاصل کنید
اگر یک exception دلخواه تعریف می‌کنید، اطمینان حاصل کنید که متودی که این خطا را دریافت می‌کند نیز اطلاعات مورد نظر را داشته باشد.  
بطور مثال exception شما در assembly 1 تعریف شده و خطا به یک متود دیگر در assembly 2 پاس داده می‌شود. اگر این اسمبلی به خطا گفته شده دسترسی نداشته باشد شما خطا `FileNotFoundException` را دریافت می‌کنید.  


### ترجمه متن خطا
خطایی که می‌خواهید به سیستم خارجی و یا کاربر بدهید باید از قابلیت Localized برای ترجمه متن خطا استفاده کنید.  

[localized-exception-messages](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-localized-exception-messages)  


### استفاده از throw
اگر خطا اتفاق افتاده را می‌خواهید به لایه بالاتر بدهید، حواستان باشد آن new نکنید تا اطلاعات `Stack Trace` پاک نشود.  

```c#
try
{
    var result = item.DoWork();
}
catch (Exception e)
{
    throw;
}
```

```c#
try
{
    var result = item.DoWork();
}
catch (Exception e)
{
    throw e;
}
```

کد اول اطلاعات stack trace را نیز به لایه بالاتر می‌دهد ولی کد دوم فقط خطا لایه فعلی را پاس می‌دهد.  

### حواستان به State باشد
فرض کنید در عملیات انتقال پول بین دو کاربر، مرحله کسر از حساب شخص اول به درستی انجام شده و یا مرحله اضافه کردن به حساب دوم به خطا خورده است.  

```c#
public void TransferFunds(Account from, Account to, decimal amount)
{
    from.Withdrawal(amount);

    to.Deposit(amount);
}
```

در کد بالا اگر بخش اول به خطا بخورد قسمت دوم نباید اجرا شود.  
یکی راه‌های برگرداندن عملیات انجام شده بطور مثال افزایش دوباره موجودی کاربر اول در صورت بخطا خوردن بصورت زیر است:  

```c#
public void TransferFunds(Account from, Account to, decimal amount)
{
    string withdrawalTrxID = from.Withdrawal(amount);

    try
    {
        to.Deposit(amount);
    }
    catch
    {
        from.RollbackTransaction(withdrawalTrxID);

        throw;
    }
}
```

یکی از کاربردهای exception تعریف شده توسط کاربر در همین بخش است که بطور مثال مبلغ و کاربران را می‌توان در خطا مشخص کرد:  

```c#
public void TransferFunds(Account from, Account to, decimal amount)
{
    string withdrawalTrxID = from.Withdrawal(amount);

    try
    {
        to.Deposit(amount);
    }
    catch (Exception ex){
    from.RollbackTransaction(withdrawalTrxID);

    throw new TransferFundsException("Withdrawal failed.", innerException: ex)
    {
        From = from,
        To = to,
        Amount = amount
    };
}
```


### حواستان به Performance باشد
استفاده از try/catch بار اضافه‌تری را بر روی سیستم می‌گذارد.  
بطور مثال اگر در یک حلقه با تعداد زیاد بلاک try/catch قرار دهید، با حالتی که بلاک وجود ندارد تفاوت زیادی را مشاهده می‌کنید.  

اطلاعات بیشتر:  

[best-practices-for-exceptions](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions)  

[exceptions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/)  

[system.exception](https://docs.microsoft.com/en-us/dotnet/api/system.exception?view=net-6.0)  




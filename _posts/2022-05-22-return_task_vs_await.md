---
title: "تفاوت برگشت Task , Async-Await"
categories:
  - Net
tags:
  - net
  - thread
  - await
  - async
---

برای فراخوانی یک متود از نوع Task دو روش زیر را می‌توان استفاده کرد که در این مطلب به توضیح تفاوت آنها می‌پردازیم.  
تفاوت اصلی این دو متود در `انتشار خطا` یا `exception propagation` است.   
در حالت دوم که await دارد، خطا در داخل Task ذخیره می‌شود و تا زمانی که به یکی از موارد `await task , task.Wait() , task.Result , task.GetAwaiter().GetResult()` نخورد، برگشت داده  نمی‌شود.  
اما حالت اول درست زمان قبل از بقیه موارد خطا را می‌دهد. در این رابطه می‌توانید مثال آخر را مشاهده کنید.  

تفاوت دیگر `سربار` استفاده از await است.  
در صورت استفاده از await متود مورد نظر توسط کامپایلر به `state machine` تبدیل می‌شود.  
اگر در متود شما بعد از await کد دیگری نبود، می‌توانید همان Task را برگردانید و از این سربار جلوگیری کنید.  


```c#
private Task TestAsync() 
{
    return Task.Delay(1000);
}

private async Task TestAsync() 
{
    await Task.Delay(1000);
}
```

---

در صورتی که متود مورد نظر را داخل `using` یا `Scope` استفاده کرده باشید، نباید از حالت بدون await استفاده کنید.  

> BUT never do inside a using, or other scoped construct where the object required by the inner call requires context to successfully complete.

> In the following example, the first variation, without the await, returns a database call that escapes from the context of the database connection at the end of the method, so when the call is made, the database connection has been closed. You need the await in the second example to postpone the disposal of the database connection until the GetUserAsync call returns.

```c#
class UserRepository
{
    public UserRepository(IDbConnectionFactory dbConnectionFactory)
    {
        _dbConnectionFactory = dbConnectionFactory;
    }

    public Task<UserDetailsResult> GetUserDetails(string userId)
    {
        using (var dbConnection = _dbConnectionFactory.CreateConnection())
        {
            return GetUserAsync(userId, dbConnection);
        }
    }

    public async Task<UserDetailsResult> GetUserDetailsAsync(string userId)
    {
        using (var dbConnection = _dbConnectionFactory.CreateConnection())
        {
            return await GetUserAsync(userId, dbConnection);
        }
    }
}
```

---

در مثال زیر هم با توجه به اینکه کدام مورد را فراخوانی کنید، `Exception` رخ داده در زمان متفاوت نشان داده می‌شود.  

```c#
private static void DoTestAsync(Func<int, Task> whatTest, int n)
{
    Task task = null;
    try
    {
        // start the task
        task = whatTest(n);

        // do some other stuff, 
        // while the task is pending
        Console.Write("Press enter to continue");
        Console.ReadLine();

        task.Wait();
    }
    catch (Exception ex)
    {
        Console.Write("Error: " + ex.Message);
    }
}

private static async Task OneTestAsync(int n)
{
    await Task.Delay(n);
}

private static Task AnotherTestAsync(int n)
{
    return Task.Delay(n);
}
```

```s
DoTestAsync(OneTestAsync, -2)

Press enter to continue
Error: One or more errors occurred.await Task.Delay
```

```s
DoTestAsync(AnotherTestAsync, -2)

Error: The value needs to be either -1 (signifying an infinite timeout), 0 or a positive integer.
Parameter name: millisecondsDelayError: 1st
```

---

تفاوت دیگر در زمان استفاده در موارد وابسته به UI است. بطور مثال کد زیر در `WPF` که مورد گفته شده باعث `Dead Lock` می‌شود.  

```c#
public void Form_Load(object sender, EventArgs e)
{
    TestAsync().Wait(); // dead-lock here
    TestAsync2().Wait(); // not dead-lock
}

private static async Task TestAsync()
{
    await Task.Delay(1000);
}

private Task TestAsync2() 
{
    return Task.Delay(1000);
}
```


[sharplab](https://sharplab.io/#v2:EYLgZgpghgLgrgJwgZwLQFEAeAHJzkCWA9gHYBqUCBUwANigD4ACADAARMCMA3ALABQrDpwCsffgKYBmDgCY2AYTYBvAW3UcZTABwcAbGwCCyAJ4kAxoYDuUAjAAUASjYBeAHwcAnEdMWFUWnoIJ3ENNjUNaX0fM3MAJQh4BBInVw9jWP9AiGDHUI0I9VwCADdYCGiMvwCg1Pd9ADoFIgBbbHoYCAATJj1xAF8gA)  
[async-return-types](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/async-return-types)  
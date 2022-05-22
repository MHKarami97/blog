---
title: "تفاوت برگشت Task , Async-Await"
date: 2022-05-22T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - thread
  - await
  - async
---

برای فراخوانی یک متود از نوع Task دو روش زیر را می‌توان استفاده کرد که در این مطلب به توضیح تفاوت آنها می‌پردازیم.  


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
Press enter to continue
Error: One or more errors occurred.await Task.Delay
```

```s
Error: The value needs to be either -1 (signifying an infinite timeout), 0 or a positive integer.
Parameter name: millisecondsDelayError: 1st
```

---

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
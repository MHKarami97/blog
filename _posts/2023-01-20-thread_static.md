---
title: "استفاده از ThreadStatic برای شبیه سازی Scoped Dependency در C#"
categories:
  - Net
tags:
  - static
  - scoped
  - thread_static
  - raabitmq
---

یکی از انواع Dependency LifeTime که در .net Core وجود دارد `Scoped` است که در طول یک Request Web معتبر است.  
فرض کنید می‌خواهید این طول عمر را خودتان شبیه سازی کنید یا شبیه به موردی که ما در پروژه نیاز به آن داشتیم، می‌خواهید به ازای هر Thread فقط یک Instance از کلاس مورد نظر داشته باشید.  
کد زیر را در نظر بگیرید:  

```csharp
using System;
using System.Collections.Generic;

namespace External.Writer
{
    public class BlockCommandWriter : RabbitmqSyncWriter
    {
        [ThreadStatic]
        private static BlockCommandWriter _instance;
        
        private static readonly object LockObj = new object();

        private BlockCommandWriter(List<string> hostNames,
            string username,
            string password,
            string exchange,
            string routingKey,
            bool shouldSend,
            bool automaticRecoveryEnabled)
            : base(hostNames, username, password, exchange, routingKey, shouldSend, automaticRecoveryEnabled, true)
        {
        }

        public static BlockCommandWriter GetInstance()
        {
            if (_instance == null)
            {
                lock (LockObj)
                {
                    if (_instance == null)
                    {
                        var hostNames = new List<string>();
                        var hosts = Configs.Hostname;

                        if (hosts.Contains(","))
                        {
                            hostNames.AddRange(hosts.Split(','));
                        }
                        else
                        {
                            hostNames.Add(hosts);
                        }

                        _instance = new DecreaseBlockCommandWriter(hostNames,
                            Configs.Username,
                            Configs.Password,
                            Configs.Exchange,
                            Configs.BlockRoutingKey,
                            Configs.BlockShouldSend,
                            Configs.AutomaticRecoveryEnabled);
                    }
                }
            }

            return _instance;
        }

        public void SendToQueue(string message)
        {
            base.Send(message);
        }
    }
}
```

### ThreadStatic

اگر در کد بالا از `[ThreadStatic]` استفاده نکنیم، در تمام برنامه یک Instance از آن وجود دارد. در بعضی مواقع مانند Publish بر روی RabbitMQ استفاده از کد بالا ممکن است باعث مشکل شود.  
اگر برنامه شما چند Thread مختلف داشته باشد که تمام آنها از یک Connection در RabbitMQ استفاده کنند، باعث بروز خطا می‌شود و در داکیومنت خود ربیت هم گفته شده است تا یک کانکشن را بین تردهای مختلف استفاده نکنید.  
یکی از راه‌ها قرار دادن صف داخلی در کلاس Base بالا است تا تمام تردها بر روی آن بنویسند و ربیت از روی آن پیام‌ها را برداشته و منتشر کند اما این روش ممکن است باعث ازبین رفتن دیتا در صورت ریست ایجنت شود.  
راه دیگر استفاده از روش بالا است که باعث می‌شود متغیر Static که همان کلاس ما است به ازای هر ترد مقدار منحصر به فرد داشته باشد که مشکل گفته شده را حل می‌کند.  

دقت کنید که aticattribute گفته شده را بر روی آبجکتی که Lock توسط آن انجام می‌شود قرار ندهید. در غیر این صورت با خطا Null بودن آن مواجه می‌شوید.  

[system threadstaticattribute](https://learn.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?view=net-7.0)  

[rabbitmq connections](https://www.rabbitmq.com/connections.html)  

در استفاده از این آیتم به این نکته هم دقت کنید که Initializer فقط برای Thread اول انجام می‌شود و برای دیگر Thread ها مقدار Null قرار داده می‌شود که خودتان باید آن را پر کنید.  
بطور مثال کد زیر را در نظر بگیرید:  

```csharp
[ThreadStatic]
private static int _item = 40;
```

در کد بالا فقط Thread اول مقدار 40 را می‌بیند و Thread های دیگر مقدار پیش‌فرض 0 را برای آن می‌بینند.  

### ThreadLocal

یکی دیگر از روش‌های استفاده از متغیر به ازای هر Thread استفاده از ThreadLocal است.  
این کلاس از Generic پشتیبانی می‌کند و علاوه بر مقادیر Static برای دیگر متغیرها هم کاربرد دارد و همچنین مشکل گفته شده در بالا را ندارد و به ازای هر Thread عملیات Initializer را انجام می‌‍دهد.  

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

private static ThreadLocal<Singleton> threadLocal = new ThreadLocal<Singleton>(() => new Singleton());

public static Singleton Instance
{
    get
    {
        return threadLocal.Value;
    }
}
```

استفاده از کد بالا:  

```csharp

// Example usage
class Program
{
    static void Main(string[] args)
    {
        var tasks = new Task[10];

        for (int i = 0; i < 10; i++)
        {
            tasks[i] = Task.Run(() =>
            {
                var singleton = Singleton.Instance;

                singleton.DoSomething();

                // Print the hash code of the instance to verify that it is different for each thread
                Console.WriteLine("Thread {0} has singleton instance {1}", Thread.CurrentThread.ManagedThreadId, singleton.GetHashCode());
            });
        }

        Task.WaitAll(tasks);
    }
}
```

[system.threading.threadlocal](https://learn.microsoft.com/en-us/dotnet/api/system.threading.threadlocal-1)  
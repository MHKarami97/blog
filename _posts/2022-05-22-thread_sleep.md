---
title: "بررسی Thread Sleep در .Net"
categories:
  - Net
tags:
  - net
  - thread
  - sleep
  - async
---

برای جلوگیری از استفاده تمام منابع cpu در زبان‌های برنامه‌نویسی می‌توان از تابع Thread.Sleep استفاده کرد که منابع را از ترد جاری می‌گیرد.  
این تابع ورودی یک عدد مثبت می‌گیرد که البته مقدار پاس داده شده به آن معنی خاصی دارد.  

مقدار `0` باعث می‌شود که cpu به یک ترد دیگر که به منبع نیاز دارد داده شود و اگر ترد دیگری نبود در دست همان ترد می‌ماند.  

> The number of milliseconds for which the thread is suspended. If the value of the millisecondsTimeout argument is zero, the thread relinquishes the remainder of its time slice to any thread of equal priority that is ready to run. If there are no other threads of equal priority that are ready to run, execution of the current thread is not suspended.

نکته دیگر مقادیر بسیار کم است. با توجه به اینکه دستور در هر `Clock` CPU اجرا می‌شود. اگر مقدار آن کمتر از هر کلاک باشد هم در واقع زمان Sleep همان اندازه یک کلاک می‌شود. بطور مثال اگر کلاک هر 15 میلی‌ثانیه اجرا شود و شما مقدار 1 را وارد کنید، مقدار Sleep در واقع همان 15 است.  
همچنین مقادیر صحیح عدد در نظر گرفته می‌شوند و مقادیر کمتر صرف نظر می‌شوند.  

> The system clock ticks at a specific rate called the clock resolution. The actual timeout might not be exactly the specified timeout, because the specified timeout will be adjusted to coincide with clock ticks. For more information on clock resolution and the waiting time, see the Sleep function from the Windows system APIs.


```c#
Thread.Sleep(0);

Thread.Sleep(1);

Thread.Sleep(15);

Thread.Sleep(1000);
```

[system.threading.thread.sleep](https://docs.microsoft.com/en-us/dotnet/api/system.threading.thread.sleep)  
[nf-synchapi-sleep](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep)  
[gigahertz](https://medium.com/swlh/what-does-gigahertz-ghz-actually-mean-c72151da6a1d)  
[ghz-mean-computer-processor](https://smallbusiness.chron.com/ghz-mean-computer-processor-66857.html)  
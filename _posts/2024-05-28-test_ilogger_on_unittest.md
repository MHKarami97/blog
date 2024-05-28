---
title: "بررسی فراخوانی شدن ILogger در UnitTest"
categories:
  - Net
tags:
  - moq
  - test
  - log
---

 در زمان نوشتن تست‌ها بعضی مواقع امکان بررسی اینکه متود خطا برمی‌گرداند وجود ندارد و فقط در داخل خود متود لاگ زده می‌شود. بطور مثال در سناریوهایی که پیام‌ها از یک صف خوانده می‌شوند و در صورت خطا خوردن به پیام دیگر می‌رویم.  
 در این موارد برای بررسی اینکه برنامه به خطا خورده است و لاگ زده شده است می‌توانید از این روش استفاده کنید.  
 ابتدا کتابخانه‌های moq , xunit را نصب کنید و سپس شبیه به کد زیر ILogger را کانفیگ کنید:  

```c#
using Moq;

private readonly Mock<ILogger<Worker>> _logger;

public WorkerTest()
{
    _logger = new Mock<ILogger<Worker>>();

    ConfigLogger();
}

private void ConfigLogger()
{
    _logger
        .Setup(x =>
            x.Log(
                It.IsAny<LogLevel>(),
                It.IsAny<EventId>(),
                It.IsAny<It.IsAnyType>(),
                It.IsAny<Exception>(),
                ((Func<It.IsAnyType, Exception, string>)It.IsAny<object>())!
            )
        )
        .Callback(new InvocationAction(invocation =>
        {
            var logLevel = (LogLevel)invocation.Arguments[0];
            var eventId = (EventId)invocation.Arguments[1];
            var state = invocation.Arguments[2];
            var exception = (Exception)invocation.Arguments[3];
            var formatter = invocation.Arguments[4];

            var invokeMethod = formatter.GetType().GetMethod("Invoke");
            var logMessage = (string)invokeMethod?.Invoke(formatter, new[] { state, exception });

            Trace.WriteLine($"{logLevel} - {logMessage}");
        }));
}
```
اکنون کافی است در کد خود مشابه به زیر به خطا خوردن و یا نخوردن را بررسی کنید.  
در این کد فقط خطا با لول Critical و حداقل یکبار اتفاق افتاد با کد Times.AtLeast(1) بررسی شده است که می‌توانید این موارد را تغییر دهید.  

```csharp
[Fact]
public void GetNewItem_BeforeTime_LogError()
{
    //Arrange
    ...

    // Act
    ...

    //Assert
    _logger.Verify(
        x => x.Log(
            It.Is<LogLevel>(l => l == LogLevel.Critical),
            It.IsAny<EventId>(),
            It.Is<It.IsAnyType>((v, t) => true),
            It.IsAny<Exception>(),
            It.Is<Func<It.IsAnyType, Exception, string>>((v, t) => true)!),
        Times.AtLeast(1));
}
```
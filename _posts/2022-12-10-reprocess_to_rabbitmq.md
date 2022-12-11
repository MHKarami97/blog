---
title: "اضافه کردن قابلیت پردازش دوباره پیام به Rabbitmq"
date: 2022-12-10T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - rabbit
  - reprocess
---

در یکی از مطالب قبلی وبلاگ چگونگی ارسال پیام‌ها به DeadLetter آموزش داده شد:  

[dead_letter_on_rabbitmq](https://blog.mhkarami97.ir/net/dead_letter_on_rabbitmq/)

فرض کنید قبل از ارسال پیام‌ها به DeadLetter می‌خواهید پیام‌ها را حداقل به یک تعداد مشخص پردازش کنید. برای انجام این کار می‌توانید از کد زیر استفاده کنید:  

```csharp
_consumer.Received += (model, result) =>
{
    try
    {
        var message = Encoding.UTF8.GetString(result.Body.ToArray());

        // Do some work on message
        var isSuccessful = Received.Invoke(message);

        if (isSuccessful)
        {
            counter = 0;

            _channel.BasicAck(result.DeliveryTag, false);

            return;
        }

        if (counter > config.RequeueMessageRetryCount)
        {
            counter = 0;

            // Requeue = false is important
            _channel.BasicReject(result.DeliveryTag, false);

            return;
        }
    }
    catch (Exception ex)
    {
        LogError(ex);
    }

    counter++;

    // Prevent Very fast failed message process
    Thread.Sleep(WaitInNackOnMilliSecond);

     // Requeue = true is important to reprocess
    _channel.BasicNack(result.DeliveryTag, false, true);
};

_channel.BasicConsume(config.Queue, false, _consumer);
```

نکته‌ای که در این کد وجود دارد این است که اگر شما چند instance داشته باشید، تعداد پردازش‌ها بیشتر می‌شود. زیرا پیام‌ها بصورت RoundRobin بین سرورها پخش می‌شود که هرکدام هر طبق کد بالا آنها را 3 بار پردازش می‌کنند.  
بطور مثال اگر 4 سرور داشته باشید، 4*3=12 بار پیام پردازش می‌شود و سپس بر روی deadLetter قرار داده می‌شود.  
برای حل این مشکل می‌توانید از کد زیر استفاده کنید:  

```csharp
_consumer.Received += (model, result) =>
{
    try
    {
        var message = Encoding.UTF8.GetString(result.Body.ToArray());

        while (counter < config.RequeueMessageRetryCount)
        {
            var isSuccess = false;

            try
            {
                isSuccess = Received.Invoke(message);
            }
            catch (Exception)
            {
                // ignored
            }

            if (isSuccess)
            {
                counter = 0;

                _channel.BasicAck(result.DeliveryTag, false);

                return;
            }

            counter++;

            Thread.Sleep(WaitInReprocessOnMilliSecond);
        }

        counter = 0;

        _channel.BasicReject(result.DeliveryTag, false);
    }
    catch (Exception ex)
    {
        ExceptionOccured(ex);
    }
};

_channel.BasicConsume(config.Queue, false, _consumer);
```
---
title: "استفاده از Dead Letter Exchange در RabbitMQ"
date: 2022-11-07T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - rabbitmq
  - dead_letter
---

 یکی از قابلیت‌های کاربردی RabbitMQ امکان ارسال پیام‌های به خطا خورده به یک صف دیگر برای پردازش یا داشتن لاگ از پیام‌های پردازش نشده است.  
بطور مثال فرض کنید پیام‌های انتقال موجودی کاربر را بر روی صف قرار می‌دهید و همچنین قابلیت پردازش دوباره پیام‌ها را هم فعال کرده‌اید تا به تعداد دلخواه پیام‌ها دوباره پردازش شوند. اما بعد از رسیدن به تعداد مشخص چه؟  
اگر پیام را دور بیاندازید دیگر امکان بررسی آن را ندارید و همچنین اگر بگذارید در صف بماند باز هم پردازش می‌شود و به خطا می‌خورد. همچنین جلو پردازش دیگر پیام‌های صف را هم می‌گیرد.  
در این موقع می‌توانید از این قابلیت استفاده کنید.  

برای این کار در صفی که می‌خواهید باید دو مقدار `x-dead-letter-exchange` و `x-dead-letter-routing-key` را بدهید.  
با دادن این مقادیر اگر پیغام `basic.Reject` شود و یا `Basic.Nack` داده شود و همچنین مقدار `Requeue` در آن برابر `False` باشد پیغام مورد نظر به `Exchange` گفته شده ارسال می‌شود.  
مقدار `Routing Key` هم اگر Exchange مورد نظر را بصورت `Direct` تعریف کرده باشید مهم است. اگر نوع آن `FanOut` باشد نیاز به این مقدار ندارید.  

```csharp
_channel.ExchangeDeclare(
                config.Exchange,
                config.ExchangeType,
                config.Durable,
                config.ExchangeAutoDelete);

            _channel.QueueDeclare(
                config.Queue,
                config.Durable,
                false,
                false,
                new Dictionary<string, object>
                {
                    { "x-dead-letter-exchange", config.DeadLetter.Exchange },
                    { "x-dead-letter-routing-key", config.DeadLetter.RoutingKey }
                });

            _channel.QueueBind(
                config.Queue,
                config.Exchange,
                config.RouteKey);
```

دقت کنید که Exchange گفته شده را هم خودتان باید در کد ایجاد کنید و همچنین یک صف هم بر روی آن درست کنید.  

```csharp
_channel.ExchangeDeclare(
                exchange: config.DeadLetter.Exchange,
                type: "direct",
                durable: true,
                autoDelete: false);

            _channel.QueueDeclare(
                queue: config.DeadLetter.Queue,
                durable: true,
                exclusive: false,
                autoDelete: false);
            
            _channel.QueueBind(
                config.DeadLetter.Queue,
                config.DeadLetter.Exchange,
                config.DeadLetter.RoutingKey);
```

برای اضافه کردن قابلیت پردازش چندباره یک پیام هم می‌توانید از کد زیر استفاده کنید:  

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

[dlx](https://www.rabbitmq.com/dlx.html)  
---
title: "استفاده از BasicQos در Rabbitmq"
categories:
  - Rabbitmq
tags:
  - basicqos
  - consumer
  - queue
---

فرض کنید که در سیستم خود از RabbitMQ برای ارتباط بین سیستم‌ها استفاده می‌کنید و در قسمتی چند Consumer دارید که پیام‌ها را از روی یک صف می‌خواند.  
اگر تعدادی پیام بر روی Queue موجود باشد و شما ایجنت خود را ریست بدهید، بعد از بالا آمدن اولین Consumer که سریعتر به صف وصل شود و Consumer های بعدی با تاخیر وصل شوند، تمام پیام‌ها بصورت UnAck در Consumer اولی است و بقیه Consumer ها کاری نمی‌کنند.  
و یا در حالت دیگر یکی از Consumer ها به دلیل مشکل پیام‌ها را بردارد و به دلیل به خطا خوردن آنها را دوباره پردازش کند و یا به صف DealLetter بفرستد، در این حالت هم تعداد پیام‌های زیادی در این Consumer بصورت UnAck موجود است.  
برای جلوگیری از این مشکل می‌توان از متود زیر استفاده کرد و تعداد پیام‌های UnAck را محدود کرد. در کد زیر این مقدار به 100 محدود شده است.  

```csharp
Channel.BasicQos(0, 100, false);
```

[ConsumerPrefetch](https://www.rabbitmq.com/consumer-prefetch.html)  
---
title: "اعتبار سنجی مقادیر Enum"
categories:
  - Net
tags:
  - enum
  - default_value
  - validation
---

یکی از موارد مهم که در استفاده از کافکا باید به آن دقت کرد، کانفیگ اولیه اتصال به آن است که جلوی بسیاری از مشکلات آینده را می‌گیرد. در این بخش این کانفیگ‌ها را با هم بررسی کرده‌ایم.  

 > BootstrapServers : لیست سرورهای کافکا
 > AutoOffsetReset : مقدار مناسب برای آن Earliest است. با این مقدار در صورتی که ایجنت شما ریست شود از ابتدا مواردی که دریافت نکرده است را می‌خواند.  
 > GroupId : این مورد در صورتی که شما بطور مثال برای افزایش سرعت از ConsumerGroup استفاده کرده باشید مهم است. این مورد در تمام آنها باید یکسان باشد تا یک گروه حساب شوند.  
 > ClientId : قرار دادن مقدار ثابت برای این مورد هم برای شناسایی راحتتر در پنل کافکا و هم برای اینکه بعد از ریست به عنوان یک ایجنت جدید شناخته نشود و همان Offest های قبلی به آن داده نشود مهم است.  
 > EnableAutoCommit : با false کردن این مورد وظیفه commit کردن بر عهده خودتان است. در غیر این صورت در صورتی که پیام دریافت شود به عنوان consume شده در نظر گرفته می‌شود.  
 > EnablePartitionEof : با true کردن این مورد می‌توانید متوجه شوید که یک پیام آخرین offset موجود بر روی صف است یا نه

```csharp
private void ConfigKafka(KafkaConfig config)
{
    var consumerConfig = new ConsumerConfig
    {
        BootstrapServers = GetBootstrapServers(config.BootstrapServers),
        AutoOffsetReset = AutoOffsetReset.Earliest,
        GroupId = config.GroupId,
        ClientId = config.ClientId,
        SaslUsername = config.Username,
        SaslPassword = config.Password,
        SecurityProtocol = (SecurityProtocol?)config.SecurityProtocol,
        SaslMechanism = (SaslMechanism?)config.SaslMechanism,
        SessionTimeoutMs = 6000,
        EnableAutoCommit = false,
        EnablePartitionEof = true,
    };

    _consumer = new ConsumerBuilder<Ignore, string>(consumerConfig).Build();
}
```

برای بخش Produce نیز کانفیگ‌های زیر کاربردی است.  

 > EnableIdempotence : با true کردن این مورد اطمینان حاصل می‌شود که پیام به کافکا رسیده باشد.  
 > Acks : با مقدار All اطمینان حاصل می‌شود که پیام به تمام کلاسترهای Broker رسیده باشد و بعد تایید می‌شود.  


```csharp
private void ConfigKafkaPublisher(KafkaConfig config)
{
    var producerConfig = new ProducerConfig
    {
        BootstrapServers = GetBootstrapServers(config.BootstrapServers),
        SaslUsername = config.ProducerUsername,
        SaslPassword = config.ProducerPassword,
        ClientId = config.ClientId,
        SecurityProtocol = (SecurityProtocol?)config.SecurityProtocol,
        SaslMechanism = (SaslMechanism?)config.SaslMechanism,
        EnableIdempotence = true,
        Acks = Acks.All,
    };

    _producer = new ProducerBuilder<Null, string>(producerConfig).Build();
}
```

متود GetBootstrapServers نیز برای جدا کردن سرورهای کافکا است. بطور مثال فرض کنید 2 سرور را برای کافکا بصورت Cluster ایجاد کرده‌اید. توسط این

```csharp
private string GetBootstrapServers(List<string> l)
{
    return l.Count == 1 ? l.First() : string.Join(",", l);
}
```

نمونه کانفیگ:  

```csharp
"Kafka": {
    "BootstrapServers": ["server-alpha-1:9043", "server-alpha-2:9043"],
    "TopicName": "topic",
    "ProducerTopic": "dead-letter",
    "GroupId": "group",
    "ClientId": "client",
    "Username": "kafka",
    "Password": "pass",
    "ProducerUsername": "kafka",
    "ProducerPassword": "pass",
    "SecurityProtocol": "SaslPlaintext",
    "SaslMechanism": "Plain"
}
```

نمونه خواندن پیام به همراه DeadLetter :  

```csharp
private void Do(CancellationToken stoppingToken)
{
    IsLastMessage = false;

    _consumer.Subscribe(_kafkaConfig.TopicName);

    while ((!stoppingToken.IsCancellationRequested && _shouldWork) || !IsLastMessage)
    {
        try
        {
            var consumeResult = _consumer.Consume();

            if (consumeResult != null)
            {
                if (consumeResult.Message != null)
                {
                    using (Tracing.ActivitySource.StartActivity())
                    {
                        try
                        {
                            Activity.Current?.SetTag("Offset", consumeResult.Offset.Value);

                            OnMessageReceived?.Invoke(this, new MessageReceivedEventArgs(consumeResult.Message.Value));

                            _consumer.Commit();
                        }
                        catch (TimeoutException ex)
                        {
                            _logger.LogCritical(ex, $"Time out Invoke Kafka message: {ex.Message}");
                        }
                        catch (InvalidOperationException ex)
                        {
                            _logger.LogCritical(ex, $"Time out Invoke Kafka message: {ex.Message}");
                        }
                        catch (TaskCanceledException)
                        {
                            _logger.LogInformation("Task Cancellation");
                        }
                        catch (OperationCanceledException)
                        {
                            _logger.LogInformation("Task Cancellation");
                        }
                        catch (Exception ex)
                        {
                            _logger.LogCritical(ex, $"Error Invoke Kafka message: {ex.Message}");

                            _producer.Produce(_kafkaConfig.ProducerTopic, new Message<Null, string>
                            {
                                Value = consumeResult.Message.Value
                            });

                            _consumer.Commit();
                        }
                    }
                }

                IsLastMessage = consumeResult.IsPartitionEOF;

                if (IsLastMessage)
                {
                    _logger.LogInformation($"LastMessage, Offset: {consumeResult.Offset.Value}");
                }
            }
            else
            {
                Thread.Sleep(100);
            }
        }
        catch (OperationCanceledException)
        {
            _logger.LogInformation("Task Cancellation");

            Thread.Sleep(100);
        }
        catch (Exception ex)
        {
            _logger.LogCritical(ex, $"Error processing Kafka message: {ex.Message}");
        }
    }
}
```

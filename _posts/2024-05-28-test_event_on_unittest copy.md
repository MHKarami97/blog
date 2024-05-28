---
title: "تست کردن Event در UnitTest"
categories:
  - Net
tags:
  - event
  - test
  - log
---

فرض کنید برای خواندن پیام‌ها از صف Kafka از کدی مشابه زیر استفاده کرده‌اید که پیام‌ها بعد از دریافت توسط delegate و Event به کلاس دیگری داده می‌شود.  

```c#
public interface IDataReader
{
    public event MessageReceivedEventHandler OnMessageReceived;
    Task StartAsync(CancellationToken stoppingToken);
}
```

```csharp
public class MessageReceivedEventArgs : EventArgs
{
    public string Message { get; }

    public MessageReceivedEventArgs(string message)
    {
        Message = message;
    }
}

public delegate Task MessageReceivedEventHandler(object sender, MessageReceivedEventArgs e);
```

```csharp
public class KafkaReader : IDataReader
{
    private readonly KafkaConfig _kafkaConfig;
    private readonly ILogger<KafkaReader> _logger;
    private IConsumer<Ignore, string> _consumer;
    public event MessageReceivedEventHandler OnMessageReceived;

    public KafkaReader(ILogger<KafkaReader> logger, IOptionsMonitor<KafkaConfig> kafkaConfig)
    {
        _logger = logger;
        _kafkaConfig = kafkaConfig.CurrentValue;

        ConfigKafka(_kafkaConfig);
    }

    public async Task StartAsync(CancellationToken stoppingToken)
    {
        _consumer.Subscribe(_kafkaConfig.TopicName);

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                var consumeResult = _consumer.Consume(stoppingToken);

                await Task.Run(() => OnMessageReceived?.Invoke(this, new MessageReceivedEventArgs(consumeResult.Message.Value)), stoppingToken);
            }
            catch (Exception ex)
            {
                _logger.LogCritical(ex, $"Error processing Kafka message: {ex.Message}");
            }
        }

        Stop();
    }

    private void Stop()
    {
        _consumer.Close();
    }

    private void ConfigKafka(KafkaConfig config)
    {
        var consumerConfig = new ConsumerConfig
        {
            BootstrapServers = config.BootstrapServers,
            AutoOffsetReset = AutoOffsetReset.Earliest
        };

        _consumer = new ConsumerBuilder<Ignore, string>(consumerConfig).Build();
    }
}
```
و شبیه به کدی مشابه زیر از آن استفاده کرده‌اید.  

```csharp
public class Worker
{
    private readonly ILogger<Worker> _logger;
    private readonly IDataReader _dataReader;

    public Worker(ILogger<Worker> logger, IDataReader dataReader)
    {
        _logger = logger;
        _dataReader = dataReader;
    }

    public async Task Execute()
    {
        try
        {
            _logger.LogInformation($"Worker start at {_dateTimeProvider.Now}");
    
            var cancellationToken = GetNewCancellationToken(_appSettings.StopQueueReaderAfter);
    
            _dataReader.OnMessageReceived += HandleMessageAsync;
    
            await _dataReader.StartAsync(cancellationToken);
    
            _dataReader.OnMessageReceived -= HandleMessageAsync;
        }
        catch (TaskCanceledException)
        {
            _dataReader.OnMessageReceived -= HandleMessageAsync;
    
            _logger.LogInformation("Task Cancellation");
        }
        catch (Exception e)
        {
            _logger.LogCritical(e, e.Message);
        }
    
        _logger.LogInformation($"Worker finished at {_dateTimeProvider.Now}");
    }
    
    private static CancellationToken GetNewCancellationToken(TimeSpan cancelAfter)
    {
        var cancellationTokenSource = new CancellationTokenSource();
        cancellationTokenSource.CancelAfter(cancelAfter);
    
        return cancellationTokenSource.Token;
    }

}
```
برای Mock کردن interface برای تست کردن کلاس فراخوانی کننده آن می‌توانید مشابه زیر عمل کنید.  
ابتدا یک کلاس در پروژه UnitTest خود مشابه زیر بسازید:  

```csharp
public class KafkaReaderFaker : IDataReader
{
    private readonly QueueModelDataBuilder _queueModelDataBuilder;

    public KafkaReaderFaker(QueueModelDataBuilder queueModelDataBuilder)
    {
        _queueModelDataBuilder = queueModelDataBuilder;
    }

    public event MessageReceivedEventHandler? OnMessageReceived;

    public async Task StartAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var data = _queueModelDataBuilder.Build();

            var dataSerialize = JsonSerializer.Serialize(data.First());

            OnMessageReceived?.Invoke(this, new MessageReceivedEventArgs(dataSerialize));

            await Task.Delay(TimeSpan.FromMilliseconds(1), stoppingToken);
        }
    }
}
```
این کد کلاسی که پیام‌ها را از صف می‌خواند را شبیه‌سازی می‌کند و هر 1 ثانیه یک پیام جدید را Invoke می‌کند.  

برای استفاده از آن در تست کافی است مشابه زیر عمل کنید:  

```csharp
[Collection("NullCollection")]
public class WorkerTest
{
    private readonly NUllFixture _databaseContainer;
    private readonly Mock<ILogger<Worker>> _logger;

    public WorkerTest(NUllFixture databaseContainer)
    {
        _databaseContainer = databaseContainer;
    }

    [Fact]
    public void GetNewItemFromQueue_BeforeTime_ShouldNotThrowError()
    {
        //Arrange
        var fakeReader = new KafkaReaderFaker(new QueueModelDataBuilder().WithZeroQuantity());

        var worker = new Worker(_logger.Object, fakeReader);

        // Act
        worker.Execute(_jobExecutionContext.Object).Wait();

        //Assert
        ...
    }
}
```
برای ساخت دیتا برای تست نیز از کتابخانه Bogus استفاده شده است.  

```csharp
public class QueueModelDataBuilder
{
    private Faker<QueueModel> Faker { get; set; }

    public QueueModelDataBuilder()
    {
        var now = DateTime.Now;
        var startTime = new DateTime(now.Year, now.Month, now.Day, 9, 0, 0);

        Faker = new Faker<QueueModel>()
            .RuleFor(c => c.Quantity, f => f.Random.Int(1, 99))
            .RuleFor(c => c.ModifyDate, f => f.Date.Between(startTime.AddHours(-1), startTime));
    }

    public IReadOnlyCollection<QueueModel> Build(int count = 1)
    {
        return Faker.Generate(count);
    }

    public QueueModelDataBuilder WithZeroQuantity()
    {
        Faker = Faker.RuleFor(d => d.Quantity, f => 0L);
        return this;
    }
}
```
با این روش OnMessageReceived را بصورت Fake استفاده کرده‌اید و می‌توانید کلاس اصلی خود را تست کنید.  
---
title: "استفاده از Consistent Hash Exchange در RabbitMQ"
categories:
  - Net
tags:
  - rabbitmq
  - exchange
  - consistent_hash
---


```csharp
using Utf8Json;

var data = JsonSerializer.ToJsonString(message);
base.SendByRoutingKey(data, $"{_routingKeyPrefix}.{message.CustomerCode}");
```

```csharp
protected void SendByRoutingKey(string message, string routingKey)
{
    if (_shouldSend)
    {
        _properties.Timestamp = new AmqpTimestamp(DateTimeOffset.UtcNow.ToUnixTimeSeconds());

        _channel.BasicPublish(_exchange,
            routingKey,
            _properties,
            Encoding.UTF8.GetBytes(message));

        if (_isPublisherConfirm && !_channel.WaitForConfirms(TimeSpan.FromSeconds(TimoutForReceivingAckInSeconds)))
        {
            throw new TimeoutException($"Publisher confirms timed out for the Message: {message}");
        }
    }
}
```

```csharp
public static void Make()
{
    var channel = Connection.CreateModel();

    //MyQueue Config
    channel.ExchangeDeclare(
        exchange: Configs.MyExchange,
        type: "x-consistent-hash",
        durable: bool.Parse(Configs.IsDurableExchange),
        autoDelete: bool.Parse(Configs.IsAutoDeleteExchange)
    );

    //UnhandledMyQueue Config
    channel.ExchangeDeclare(
        exchange: Configs.UnhandledMyExchange,
        type: "fanout",
        durable: bool.Parse(Configs.IsDurableExchange),
        autoDelete: bool.Parse(Configs.IsAutoDeleteExchange)
    );

    channel.QueueDeclare(
        Configs.UnhandledMyQueue,
        bool.Parse(Configs.IsDurableQueue),
        bool.Parse(Configs.IsExclusiveQueue),
        bool.Parse(Configs.IsAutoDeleteQueue),
        new Dictionary<string, object>
        {
            { "x-dead-letter-exchange", Configs.DeadLetterExchange },
        });

    channel.QueueBind(
        Configs.UnhandledMyQueue,
        Configs.UnhandledMyExchange,
        string.Empty);

    //DeadLetterQueue Config
    channel.ExchangeDeclare(
        exchange: Configs.DeadLetterExchange,
        type: "fanout",
        durable: bool.Parse(Configs.IsDurableExchange),
        autoDelete: bool.Parse(Configs.IsAutoDeleteExchange));

    channel.QueueDeclare(
        Configs.DeadLetterQueue,
        bool.Parse(Configs.IsDurableQueue),
        bool.Parse(Configs.IsExclusiveQueue),
        bool.Parse(Configs.IsAutoDeleteQueue),
        new Dictionary<string, object>
        {
            { "x-dead-letter-exchange", Configs.UnhandledMyExchange },
            { "x-message-ttl", 180_000 }
        });

    channel.QueueBind(
        Configs.DeadLetterQueue,
        Configs.DeadLetterExchange,
        string.Empty);

    channel.Dispose();
}
```

```csharp
public static void AddNewQueue(string queueName)
{
    var channel = Connection.CreateModel();

    channel.QueueDeclare(
            queueName,
            bool.Parse(Configs.IsDurableQueue),
            bool.Parse(Configs.IsExclusiveQueue),
            bool.Parse(Configs.IsAutoDeleteQueue),
            new Dictionary<string, object>
            {
                { "x-dead-letter-exchange", Configs.UnhandledMyExchange },
                { "x-message-ttl", Configs.MyQueueTTL}
            });

    // 1 for routing key is because we use consistent-hash exchange
    channel.QueueBind(
        queueName,
        Configs.MyExchange,
        "1");

    channel.Dispose();
}
```

```csharp
var queueName = $"{Configs.MyQueuePrefix}.{Environment.MachineName}.{GetServiceName()}";
// MyQueue.S2-ST-WM.MyAgentRunner

AddNewQueue(queueName);

private string GetServiceName()
{
    var processId = Process.GetCurrentProcess().Id;
    var query = "SELECT * FROM Win32_Service WHERE ProcessId = " + processId;
    var searcher = new System.Management.ManagementObjectSearcher(query);

    foreach (var queryObj in searcher.Get())
    {
        return queryObj["Name"].ToString();
    }

    return processId.ToString();
}
```


```csharp
// Parent
namespace RabbitmqDetector
{
    public abstract class RabbitMqMultiThreadDetector : IQueueDetector
    {
        private readonly IModel _channel;
        private readonly object _lockObject = new object();
        private readonly RabbitMqDetectorConfig _config;
        private readonly EventingBasicConsumer _consumer;
        private readonly IConnection _connection;
        private readonly List<QueueProcessor> _processors;
        private int _nextProcessor = 0;

        protected RabbitMqMultiThreadDetector(RabbitMqDetectorConfig config, int threadCount)
        {
            _config = config;
            if (_config == null)
                throw new ApplicationException("RabbitMQ configuration is missing.");

            var connectionFactory = new ConnectionFactory
            {
                UserName = _config.Username,
                Password = _config.Password,
                AutomaticRecoveryEnabled = _config.AutomaticRecoveryEnabled,
                RequestedConnectionTimeout = TimeSpan.FromMilliseconds(2000)
            };

            var endpoints = ConvertHostNames(_config.HostNames)
                .Select(item => new AmqpTcpEndpoint(item.Name, item.Port))
                .ToList();

            _connection = connectionFactory.CreateConnection(endpoints);
            _channel = _connection.CreateModel();

            _channel.BasicQos(0, _config.PrefetchCount, false);
            _consumer = new EventingBasicConsumer(_channel);
            _consumer.Received += HandleReceivedInternal;

            _processors = new List<QueueProcessor>(threadCount);
            for (var i = 0; i < threadCount; i++)
            {
                var processor = new QueueProcessor(HandleReceived);
                _processors.Add(processor);
            }
        }

        protected void Ack(BasicDeliverEventArgs result)
        {
            lock (_lockObject)
            {
                _channel.BasicAck(result.DeliveryTag, false);
            }
        }

        protected void Nack(BasicDeliverEventArgs result)
        {
            lock (_lockObject)
            {
                _channel.BasicNack(result.DeliveryTag, false, false);
            }
        }

        public void Start()
        {
            try
            {
                _channel.BasicConsume(_config.Queue, false, _consumer);
            }
            catch (Exception exception)
            {
                _logger.Critical("Error on starting RabbitMqMessageConsumer", exception);
            }
        }

        protected abstract void HandleReceived(BasicDeliverEventArgs result);

        private void HandleReceivedInternal(object model, BasicDeliverEventArgs result)
        {
            // _nextProcessor go to _processors.Count and then be zero (example 64 % 64 = 0)
            _nextProcessor = (_nextProcessor + 1) % _processors.Count;
            _processors[_nextProcessor].ProcessMessage(result);
        }

        protected virtual void HandleException(Exception exp)
        {
            ConsoleWriter.WriteIfUserInteractive(exp);
            ConsoleWriter.WriteIfUserInteractive(exp.StackTrace);
            RepoContext.Instance.Log4Repo.Critical(exp);
        }

        public virtual void Dispose()
        {
            if (_channel?.IsOpen == true && _consumer != null)
            {
                _channel.BasicCancel(_consumer.ConsumerTags[0]);
            }

            _channel?.Dispose();
            _connection?.Dispose();
        }

        public virtual void Stop()
        {
            Dispose();
            foreach (var processor in _processors)
            {
                processor.Stop();
            }
        }

        private static List<RabbitEndpoint> ConvertHostNames(string hostNames)
        {
            var hosts = hostNames.Split(',');
            return hosts.Select(h => new RabbitEndpoint(h)).ToList();
        }
    }
}
```

```csharp
//Child
protected override void HandleReceived(BasicDeliverEventArgs result)
{
    var counter = 0;
    try
    {
        var message = Encoding.UTF8.GetString(result.Body.ToArray());

        if (string.IsNullOrEmpty(message))
            throw new ArgumentNullException("Message", "Message cannot be null or empty.");

        while (counter < RetryCount)
        {
            var isSuccess = false;

            var myDto = JsonSerializer.Deserialize<MyDto>(message);
            isSuccess = ProcessMessage(myDto);

            if (isSuccess)
            {
                counter = 0;
                Ack(result);
                return;
            }

            counter++;
            if (counter != RetryCount)
            {
                Thread.Sleep(WaitInReprocessOnMilliSecond);
            }
        }

        _logger.Critical("Error in Detector", $"Message could not be processed after 3 times and sent to UnhandledMyQueue. data: {message}");

        counter = 0;
        Nack(result);
    }
    catch (Exception ex)
    {
        Nack(result);
        HandleException(ex);
    }
}
```

```csharp
//UnhandledDetector
namespace UnhandledDetector
{
    public class UnhandledDetector : RabbitMqDetector
    {
        private readonly Facade _facade;
        private const int RetryCount = 3;

        public UnhandledDetector(RabbitMqDetectorConfig config) : base(config)
        {
            _facade = new Facade();
        }

        protected override void HandleReceived(object model, BasicDeliverEventArgs result)
        {
            var xDeathCount = 0;
            try
            {
                ReviseTopology(result);

                xDeathCount = GetXDeathCount(result);
                var message = Encoding.UTF8.GetString(result.Body.ToArray());

                if (string.IsNullOrEmpty(message))
                    throw new ArgumentNullException("Message", "Message cannot be null or empty.");

                var myDto = JsonSerializer.Deserialize<MyDto>(message);

                if (xDeathCount < RetryCount)
                {
                    var isSuccess = ProcessMessage(myDto);

                    if (isSuccess)
                    {
                        Channel.BasicAck(result.DeliveryTag, false);
                        return;
                    }

                    if ((xDeathCount + 1) != RetryCount)
                    {
                        Channel.BasicNack(result.DeliveryTag, false, false);
                        return;
                    }

                    Channel.BasicAck(result.DeliveryTag, false);

                    _logger.Critical("Error in UnhandledDetector", $"Message could not be closed after 3 times try and was discarded. Message: {message}");
                }
                else
                {
                    //طبیعتا کد نباید وارد این بلاک بشود ولی برای اطمینان، در اینجا هم یک اَک ارسال میکنیم تا مطمئن شویم که پیام پاک میشود.
                    Channel.BasicAck(result.DeliveryTag, false);
                    _logger.Error("Message discarded because xDeathCount >= RetryCount.");
                }
            }
            catch (FieldAccessException e)
            {
                Channel.BasicAck(result.DeliveryTag, false);
                _logger.Error("Message discarded because the header cannot be processed properly.", e);
            }
            catch (Exception e)
            {
                HandleException(e);
                if (xDeathCount < RetryCount)
                {
                    Channel.BasicNack(result.DeliveryTag, false, false);
                }
                else
                {
                    Channel.BasicAck(result.DeliveryTag, false);
                    _logger.Error("Message Discarded.", e);
                }
            }
        }

        private void ReviseTopology(BasicDeliverEventArgs result) 
        {
            var firstDeathQueue = result.BasicProperties.Headers.FirstOrDefault(c => c.Key == "x-first-death-queue");

            if (firstDeathQueue.Value is byte[] queueBytes)
            {
                var queue = Encoding.UTF8.GetString(queueBytes);
                try
                {
                    using (var channel = Connection.CreateModel())
                    {
                        var consumerCount = channel.ConsumerCount(queue);

                        if (consumerCount <= 0)
                        {
                            channel.QueueUnbind(queue, Configs.MyExchange, "1", null);
                        }
                        channel.QueueDelete(queue, true, true);
                    }
                }
                catch (Exception e)
                {
                    _logger.Critical(e);
                }
            }
        }

        private int GetXDeathCount(BasicDeliverEventArgs result)
        {
            var value = 0;
            try
            {
                if (result.BasicProperties.Headers.ContainsKey("x-death"))
                {
                    _ = result.BasicProperties.Headers.TryGetValue("x-death", out var xDeathPropertyValue);

                    var a = (List<object>)xDeathPropertyValue;
                    var b = (Dictionary<string, object>)a[0];
                    var queue = Encoding.UTF8.GetString((byte[])b["queue"]);
                    var count = 0;

                    if (queue == Configs.DeadLetterQueue)
                    {
                        count = (int)b["count"];
                    }
                    else
                    {
                        count = 0;
                    }

                    value = Convert.ToInt32(count);
                }

                return value;
            }
            catch (Exception e)
            {
                throw new FieldAccessException("ex on GetXDeathCount", e);
            }
        }

        private bool ProcessMessage(MyDto myDto)
        {
            try
            {
                _facade.CloseMessage(myDto);
                return true;
            }
            catch (Exception e)
            {
                _logger.Critical("Exception in Facade Close", e);
                return false;
            }
        }
    }
}
```


[rabbitmq_consistent_hash_exchange](https://github.com/rabbitmq/rabbitmq-server/tree/main/deps/rabbitmq_consistent_hash_exchange)  
---
title: "استفاده از OpenTelemetry در Agentها"
date: 2022-11-07T00:00:00-00:00
categories:
  - Net
tags:
  - opentelemetry
  - metric
  - monitor
---

از کتابخانه‌هایی که برای Tracing, Monitoring وجود دارد می‌توان به `OpenTelemetry` و `AppMetrics` اشاره کرد. کتابخانه دوم طبق تجربه کاربرد بیشتری دارد و کار کردن با آن راحتتر است مخصوصا اگر از .Net Core استفاده می‌کنید. اما اگر از .Net FreamWork استفاده می‌کنید ممکن است در زمان استفاده از این کتابخانه به مشکل بخورید مخصوصا اگر می‌خواهید متریک‌ها را در یک ایجنت انجام بدهید.  
کتابخانه OpenTelemetry این قابلیت را دارد تا در ایجنت و همچنین .Net Freamwork استفاده شود. 
بدین منظور یک کلاس بصورت زیر تعریف می‌کنیم:  

```csharp
using OpenTelemetry;
using OpenTelemetry.Metrics;
using OpenTelemetry.Resources;
using System.Diagnostics.Metrics;

namespace My
{
    public class Monitoring
    {
        public static readonly Meter Meter = new Meter("name.me");

        public static void InitOpenTelemetry(string serviceName)
        {
            Sdk.CreateMeterProviderBuilder()
                    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(serviceName))
                    .AddMeter("name.me")
                    .AddPrometheusExporter(options =>
                    {
                        options.HttpListenerPrefixes = new[] { "http://site-me.com:4123" };
                    })
                    .Build();
        }
    }
}
```

در کد بالا آدرس مورد نظر همان آدرس نهایی است که مقادیر در آن نشان داده می‌شوند. از `Promethus` هم به عنوان Exporter استفاده شده است.  

اکنون برای استفاده از آن در یک ایجنت کافی است کد زیر را به اول آن اضافه کنیم و نام دلخواه بطور مثال ایجنت را به آن پاس بدهیم:  

```csharp
Monitoring.InitOpenTelemetry("agent-name");
```

سپس برای اضافه کردن یک متریک بطور مثال `Counter` می‌توان بصورت زیر عمل کرد:  

```csharp
private readonly Counter<long> _acceptedMessageCounter;

protected QMessageDetector(string queueAddress)
{
    _acceptedMessageCounter = Monitoring.Meter.CreateCounter<long>("AcceptedMessage");
}

private void queue_ReceiveCompleted(string msg)
{
    _acceptedMessageCounter.Add(1, new KeyValuePair<string, object>("queue", msg.Length));
}
```

در نهایت پس از اجرا ایجنت در آدرس زیر مقادیر مورد نظر در دسترس هستند:  

>> http://site-me.com:4123\metrics

توسط کد زیر هم می‌توانید عملیات `Trace` را انجام بدهید:  

```csharp
using OpenTelemetry;
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using System;
using System.Diagnostics;

namespace My
{
    public sealed class TraceLogger : IDisposable
    {
        private Activity _activity;
        public static readonly ActivitySource ActivitySource = new ActivitySource("my.tracer");

        public TraceLogger(string actionName)
        {
            _activity = ActivitySource.StartActivity(actionName);
        }

        public void Dispose()
        {
            _activity?.Dispose();
        }

        public static void InitOpenTelemetry(string serviceName, Func<TracerProviderBuilder, TracerProviderBuilder> custom = null)
        {
            var sdk = Sdk.CreateTracerProviderBuilder()
                    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(serviceName))
                    .AddSource("my.tracer")
                    .AddAspNetInstrumentation();

            if (custom != null)
            {
                sdk = custom(sdk);
            }

            sdk.AddJaegerExporter(options =>
            {
                options.AgentHost = "172.20.19.68";
                options.AgentPort = "31392";
            }).Build();
        }
    }
}
```

نمونه استفاده:  

```csharp
TraceLogger.InitOpenTelemetry("my-agent");



internal void Dispatch(string message)
{
    using (new TraceLogger("Msg_Dispatch"))
    {
        AddToTransactionalQueue(queue, message);
    }
}
```

[opentelemetry](https://opentelemetry.io/docs/)  
---
title: "استفاده از APM Elastic در .Net"
categories:
  - Net
tags:
  - apm_elastic
  - tracing
  - opentelementry
---

Application Performance Monitoring یا به اختصار APM وظیفه نظارت بر کد شما را دارد که توسط آن می‌توانید هر بخش از کد خود را مانیتور کنید و بطور مثال خطاها یا کندی سیستم خود را تشخیص دهید.  

برای استفاده کافی است کد زیر را در برنامه خود قرار دهید. در این کد از opentelementry استفاده شده است و از پروتوکل otlp برای export استفاده شده است. نسخه 7 به بعد از این پروتکل پشتیبانی می‌کند و می‌توانید از آن استفاده کنید.  

```csharp
    public sealed class TraceLogger : IDisposable
    {
        private Activity _activity;
        public static readonly ActivitySource ActivitySource = new ActivitySource("My.Activity");

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
            try
            {
                var sdk = Sdk.CreateTracerProviderBuilder()
                    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(serviceName))
                    .AddSource("my.source")
                    .AddAspNetInstrumentation();

                if (custom != null)
                {
                    sdk = custom(sdk);
                }

                sdk.AddOtlpExporter(options =>
                {
                    options.Endpoint = new Uri(Configs.OtlpEndpoint); //http://localhost:8200
                    options.Headers = "Authorization= ApiKey " + Configs.OtlpAuthorizationHeader; // LthOr1hva0JLTXJzeEpvqG4nUkk6WUdhLW5NdlzTNlMzbWt3SWUwNzVNQQ==
                }).Build();
            }
            catch (Exception e)
            {
                ConsoleWriter.WriteIfUserInteractive(e);
            }
        }
    }
```

برای trace کردن هم کافی است کلاس خود را در یکی using استفاده کنید و اگر خواستید توسط متود SetTag مقادیر دلخواه را به آن اضافه کنید.  

```csharp
public void Handle(NotificationMessageDto input)
{
    using (new TraceLogger(nameof(Handle)))
    {
        Activity.Current?.SetTag("isConverted", isConverted);

        // ...
    }
}
```

برای استفاده نیاز است که در ابتدای سرویس یا ایجنت خود متود زیر را فراخوانی کنید تا تنظیمات اولیه انجام شود:  

```csharp
static class Program
{
    static void Main()
    {
        TraceLogger.InitOpenTelemetry("my-engine");
        // TraceLogger.InitOpenTelemetry("my-services", o => o.AddWcfInstrumentation());

        var servicesToRun = new ServiceBase[] { drsvc };
        ServiceBase.Run(servicesToRun);
    }
}
```

در پنل Kibana می‌توانید Trace های خود را مشاهده کنید:  

![mhkarami97](/assets/img/apm01-min.jpg)  

اگر به داخل هر کدام از Service های خود بروید می‌توانید جزئیات هر کدام از آنها را مشاهده کنید.  
برای دیدن trace ها به تب Transactions بروید و زمان مشاهده را با توجه به نیاز خود تغییر دهید.  

![mhkarami97](/assets/img/apm02-min.jpg)  

اگر به پایین این صفحه بروید می‌توانید تمام Trace ها را مشاهده کنید.  
بصورت پیش فرض آخرین trace انجام گرفته در این بخش نشان داده می‌شود که اگر trace قبلی را می‌خواستید می‌توانید به صفحه بعد که در عکس زیر با عدد 51 مشخص شده بروید.  

![mhkarami97](/assets/img/apm03-min.jpg)  

اگر بر روی هر Span هم کلیک کنید می‌توانید جزئیات آن و همچنین Tag هایی که خودتان اضافه کرده اید را مشاهده کنید.  

![mhkarami97](/assets/img/apm04-min.jpg)  


[opentelemetry otlp](https://opentelemetry.io/docs/specs/otlp/)  
[otlp exporter](https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/src/OpenTelemetry.Exporter.OpenTelemetryProtocol/README.md)  
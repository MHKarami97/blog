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
using OpenTelemetry;
using OpenTelemetry.Resources;
using OpenTelemetry.Trace;
using System;
using System.Collections.Generic;
using System.Diagnostics;

namespace Utility
{
    public sealed class TraceLogger : IDisposable
    {
        private const string Source = "My";
        private readonly Activity _activity;
        private static readonly ActivitySource ActivitySource = new ActivitySource(Source);

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
                var attributes = new List<KeyValuePair<string, object>>
                {
                    new KeyValuePair<string, object>("deployment.environment", Source),
                    new KeyValuePair<string, object>("host.name", Environment.MachineName)
                };

                var sdk = Sdk.CreateTracerProviderBuilder()
                    .SetErrorStatusOnException()
                    .SetResourceBuilder(ResourceBuilder.CreateDefault()
                        .AddService(serviceName, serviceName)
                        .AddAttributes(attributes)
                        .AddEnvironmentVariableDetector())
                    .AddSource(Source)
                    .AddAspNetInstrumentation(a =>
                    {
                        a.RecordException = true;
                    })
                    .AddSqlClientInstrumentation(a =>
                    {
                        a.EnableConnectionLevelAttributes = true;
                        a.SetDbStatement = true;
                    })
                    .AddHttpClientInstrumentation(a =>
                    {
                        a.RecordException = true;
                        a.SetHttpFlavor = true;
                    });

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
}
```

```csharp
  <package id="OpenTelemetry.Exporter.OpenTelemetryProtocol" version="1.2.0-rc5" targetFramework="net461" />
  <package id="OpenTelemetry.Extensions.Hosting" version="1.0.0-rc9.2" targetFramework="net461" />
  <package id="OpenTelemetry.Instrumentation.Http" version="1.0.0-rc9" targetFramework="net461" />
  <package id="OpenTelemetry.Instrumentation.SqlClient" version="1.0.0-rc9" targetFramework="net461" />
  <package id="OpenTelemetry.Exporter.Prometheus" version="1.2.0-rc5" targetFramework="net461" />
  <package id="OpenTelemetry.Instrumentation.AspNet" version="1.0.0-rc9" targetFramework="net461" />
  <package id="OpenTelemetry.Instrumentation.AspNet.TelemetryHttpModule" version="1.0.0-rc9" targetFramework="net461" />
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

برای ثبت خطا هم می‌توانید شبیه به کد زیر در Exeption خود عمل کنید:  

```csharp
public void Critical(Exception ex)
{
    SetSeverity(nameof(Critical));
    SetException(ex);
    SerilogLogger.Fatal(ex, ex.Message);
}

private void SetSeverity(string severity)
{
    Activity.Current?.SetTag(nameof(severity), severity);
}

private void SetException(Exception exception)
{
    SetStatusCode();
    Activity.Current?.RecordException(exception);
}

private void SetStatusCode()
{
    Activity.Current?.SetStatus(ActivityStatusCode.Error, "ERROR");
}

```

برای اضافه کردن Tracing به سیستم هم می‌توانید از کد زیر استفاده کنید و نمودارهای آن را در kibana مشاهده کنید:  

```csharp
using System;
using OpenTelemetry;
using OpenTelemetry.Metrics;
using OpenTelemetry.Resources;
using System.Diagnostics.Metrics;

namespace Utility
{
    public class Monitoring
    {
        private const string Source = "My";
        public static readonly Meter Meter = new Meter(Source);

        public static void InitOpenTelemetry(string serviceName)
        {
            try
            {
                Sdk.CreateMeterProviderBuilder()
                    .SetResourceBuilder(ResourceBuilder.CreateDefault().AddService(serviceName, serviceName))
                    .AddMeter(Source)
                    .AddHttpClientInstrumentation()
                    .AddOtlpExporter(options =>
                    {
                        options.Endpoint = new Uri(Configs.OtlpEndpoint);
                        options.Headers = "Authorization= ApiKey " + Configs.OtlpAuthorizationHeader;
                    }).Build();
            }
            catch (Exception e)
            {
                ConsoleWriter.WriteIfUserInteractive(e);
            }
        }
    }
}
```

```csharp
private readonly Counter<long> _successMessageCounter = Monitoring.Meter.CreateCounter<long>("successMessage");
private readonly Counter<long> _failedMessageCounter = Monitoring.Meter.CreateCounter<long>("failedMessage");


public async Task Run(ChangeDto dto)
{
    using (new TraceLogger(nameof(ChangeWorker)))
    {
        try
        {
            // do work

            _successMessageCounter.Add(1);
        }
        catch (Exception ex)
        {
            _logger.Critical(ex);
            _failedMessageCounter.Add(1);
            throw;
        }
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


برای اضافه کردن tracing هم کافی است از سمت چپ بالا آیکون منو و سپس Dashboard را انتخاب کنید و سپس Create Dashboard را بزنید.  

![mhkarami97](/assets/img/apm05-min.jpg)  

در صفحه باز شده با توجه به tracing اضافه شده یکی از آیتم‌ها بطور مثال counter-rate را انتخاب و سپس در آیتم کناری آن نام trace خود را انتخاب کنید.  

![mhkarami97](/assets/img/apm06-min.jpg)  

[opentelemetry otlp](https://opentelemetry.io/docs/specs/otlp/)  
[exporter](https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/src/OpenTelemetry.Exporter.OpenTelemetryProtocol/README.md)  
[instrumentation](https://github.com/open-telemetry/opentelemetry-dotnet/blob/main/src/OpenTelemetry.Instrumentation.SqlClient/README.md)  
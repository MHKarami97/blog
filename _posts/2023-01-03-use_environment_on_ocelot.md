---
title: "استفاده از Environment های مختلف در Ocelot"
date: 2023-01-03T00:00:00-00:00
categories:
  - Net
tags:
  - ocelot
  - environment
  - api_gateway
---

یکی از پروژه‌های خوب که برای پیاده سازی `Api GateWay` وجود دارد، پروژه `Ocelot` است که تقریبا تمام امکانات مورد نیاز شما را فراهم می‌کند.  

[Ocelot](https://github.com/ThreeMammals/Ocelot)  

یکی از امکاناتی که احتمالا به آن نیاز پیدا خواهید کرد، امکان داشتن کانفیگ‌های مختلف برای محیط‌های مختلف است. در داکیومنت خود این کتابخانه توضیح مناسبی برای پیاده سازی این امکان داده نشده است که در ادامه روش پیاده سازی آن آمده است.  

[configuration](https://ocelot.readthedocs.io/en/latest/features/configuration.html)  

ابتدا لازم است شبیه کد زیر محل کانفیگ‌های مورد نظر را به کتابخانه معرفی کنید:  

```csharp
public static void ConfigureOcelot(this ConfigurationManager configuration, WebApplicationBuilder builder)
{
    if (builder is null)
    {
        throw new ArgumentException("not valid builder", nameof(builder));
    }

    _ = configuration.AddOcelot($"Routes/{builder.Environment.EnvironmentName}", builder.Environment);

    _ = builder.Configuration
        .SetBasePath(builder.Environment.ContentRootPath)
        .AddJsonFile("ocelot.json", optional: false, reloadOnChange: true);
}
```

سپس کافی است در پوشه `Routes` به تعداد محیط‌هایی که نیاز دارید فایل کانفیگ درست کنید:  

- Routes
    - Staging
        - ocelot.sys.Staging.json
        - ocelot.otherSys.Staging.json
    - Prod
        - ocelot.sys.Prod.json
        - ocelot.otherSys.Prod.json

نمونه کد روش گفته شده هم در آدرس زیر در دسترس است. با استفاده از این روش به راحتی به ازای سیستم‌های مختلف و محیط‌ها می‌توانید فایل کانفیگ داشته باشید.

[github.com/MHKarami97/ApiGatewayOcelot](https://github.com/MHKarami97/ApiGatewayOcelot/tree/main/ApiGateway/Routes)
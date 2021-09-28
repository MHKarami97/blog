---
title: "استفاده از چند Hostname در RabbitMQ در حالت Cluster"
date: 2021-09-28T13:34:00-00:00
categories:
  - rabbitmq
tags:
  - rabbitmq
  - cluster
  - hostname
  - net
---

یکی از کتابخانه هایی که برای انتقال پیام بین سیستم های مختلف وجود دارد، Rabbitmq هست.  
یکی از قابلیت های مفید این Message Broker قابلیت Clustering هست که در زمان هایی که یکی از سرورهای شما دچار مشکل می‌شود، می‌تواند این بصورت خودکار پیام ها را به یک سرور دیگر ارسال کند.  

برای اینکه در سمت فرستنده پیام، چند Host را برای ارسال پیام معرفی کنید، می‌توانید بصورت زیر عمل کنید:  

```c#
var factory = new ConnectionFactory
{
    //HostName = hostNames.First(),
    UserName = username,
    Password = password,
    Port = port,
    AutomaticRecoveryEnabled = automaticRecoveryEnabled
};

var endpoints = hostNames
    .Select(item => new AmqpTcpEndpoint(new Uri(item)))
    .ToList();

var connection = factory.CreateConnection(endpoints);

_channel = connection.CreateModel();

_properties = _channel.CreateBasicProperties();
```

تفاوت کد بالا با حالتی که فقط یک هاست دارید، در خط کامنت شده و endpoints می‌باشد.  
در حالتی که فقط یک هاست دارید می‌توانید آن را در قسمت کامنت شده معرفی کنید، اما در زمان‌هایی که تعداد آنها بیشتر است باید آنها را در CreateConnection معرفی کنید.  
این کلاس دارای Constructor های مختلفی است که یکی از آنها یک List<string> دریافت می‌کند و یکی دیگر List<AmqpTcpEndpoint>.  
تفاوت آنها در عمل زمان های است که آدرس های شما دارای پورت نیز می‌باشند. اگر آدرس های شما دارای پورت نیز باشند، نمی‌توانید آنها را بصورت یک لیست از String پاس بدهید و حتما باید آنها را بصورت حالت دوم ایجاد کنید.  

بطور مثال اگر هاست های شما بصورت زیر باشد:  

```c#
private static List<string> Hosts = new()
{
    "192.168.99.100:30000",
    "192.168.99.100:30002",
    "192.168.99.100:30004"
};
```

در صورتی که همین لیست را به کلاس گفته شده پاس بدهید با خطا مواجه می‌شوید و حتما باید آن را بصورت `AmqpTcpEndpoint` دربیاورید.  

اطلاعات بیشتر:  

[rabbitmq](https://www.rabbitmq.com/dotnet-api-guide.html#endpoints-list)  

قوانین آدرس ها در ربیت:  
[url](https://www.rabbitmq.com/uri-spec.html)  
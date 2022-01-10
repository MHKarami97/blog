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

البته اگر از آدرس‌ها بصورت `uri` استفاده می‌کنید، حتما http را قبل از آدرس ها قرار دهید.  

```c#
private static List<string> Hosts = new()
{
    "http://192.168.99.100:30000",
    "https://192.168.99.100:30002"
};
```

اگر ورودی ها را بصورت زیر قرار بدهید، در مرحله تبدیل به AmqpTcpEndpoint با مشکل مواجه می‌شوید و مقدار خالی به عنوان آدرس که به معنی `localhost` است قرار داده می‌شود:  

```c#
private static List<string> Hosts = new()
{
    "s-mysystem-t1:30000",
    "s-mysystem-t2:30002"
};
```

اگر ورودی‌ها بصورت بالا باشد خروجی endpoints بصورت زیر می‌شود که اشتباه است:  

```c#
amqp://:30000
amqp://:30002
```

راه دیگر استفاده از کد زیر است که هردو حالت ip و نام سیستم را پشتیبانی می‌کند:  

```c#
var hostNames = new List<string>();
var hosts = "my-r1:5672,my-r2:5672,192.168.1.1:5066";

if (hosts.Contains(","))
{
    hostNames.AddRange(hosts.Split(','));
}
else
{
    hostNames.Add(hosts);
}

var endpoints = new List<AmqpTcpEndpoint>();

foreach (var hostName in hostNames)
{
    var temp = hostName.Split(':');

    var url = temp[0];
    var port = Convert.ToInt32(temp[1]);

    endpoints.Add(new AmqpTcpEndpoint(url, port));
}

var connection = factory.CreateConnection(endpoints);
```

همچنین برای بهبود عملکرد می‌توانید در کانفیگ مقدار `AutomaticRecoveryEnabled` را برابر `True` قرار بدهید.  

اطلاعات بیشتر:  

[rabbitmq](https://www.rabbitmq.com/dotnet-api-guide.html#endpoints-list)  

قوانین آدرس ها در ربیت:  
[url](https://www.rabbitmq.com/uri-spec.html)  
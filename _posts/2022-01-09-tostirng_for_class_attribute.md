---
title: "فراخوانی ToString برای تمام Attribute های یک کلاس"
date: 2022-04-22T11:07:00-00:00
categories:
  - Net
tags:
  - net
  - toString
  - properties
---

فرض کنید که شما یک API از نوع GET دارید و می‌خواهید آن را در قسمتی از کد خود فراخوانی کنید. اما مقادیری که می‌خواهید به آن پاس بدهید در اتریبوت‌های یک کلاس ذخیره شده‌اند.  
با استفاده از قابلیت override متود ToString می‌توانید تمام Attribute های یک کلاس را همراه با نام و مقدار آنها بدست بیاورید.  
کد کلی این کار بصورت زیر است:  


```c#
namespace Panel.Api
{
    public class OrderRequests
    {
        private const string Separator = "&";

        public int? Take { get; set; }
        public int? Skip { get; set; }
        public string? SortColumn { get; set; }
        public bool AscendingSort { get; set; }
        public RequestState State { get; set; }

        public override string ToString()
        {
            var data = GetType().GetProperties()
                .Select(info => (info.Name, Value: info.GetValue(this, null) ?? ""))
                .Aggregate(
                    new StringBuilder(),
                    (sb, pair) =>
                        sb.Append(
                            $"{pair.Name}={(pair.Value is Enum ? pair.Value.GetHashCode() : pair.Value)}{Separator}"),
                    sb => sb.ToString());

            if (data.EndsWith('&'))
            {
                data = data.Remove(data.Length - 1);
            }

            return data;
        }
    }
}
```

ابتدا توسط GetProperties اطلاعات پروپرتی‌های کلاس جاری را بدست می‌آوریم و سپس نام و مقادیر آنها را انتخاب می‌کنیم، البته اگر مقدار null باشد آن را با `""` جایگزین می‌کنیم.  
سپس توسط Aggregate و StringBuilder نتیجه دلخواه خود که در این حالت `نام متغیر` و `مقدار متغیر` است را طبق ساختار دلخواه `Name=Value&` به یکدیگر می‌چسبانیم تا خروجی نهایی آماده شود.  
در نهایت نیز آخرین `&` را از نتیجه پاک می‌کنیم تا نتیجه دلخواه ما آماده شود.  

همچنین در صورتی که متغیر ما از نوع `Enum` باشد، بجای نام آن از مقدار آن که توسط `GetHashCode` بدست می‌آید استفاده کرده‌ایم.  


اطلاعات بیشتر:  

[aggregate](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate?view=net-6.0)  

[gettype](https://docs.microsoft.com/en-us/dotnet/api/system.object.gettype?view=net-6.0)  

[getproperties](https://docs.microsoft.com/en-us/dotnet/api/system.type.getproperties?view=net-6.0)  
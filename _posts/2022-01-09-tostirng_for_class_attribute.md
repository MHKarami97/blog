---
title: "فراخوانی ToString برای تمام Attribute های یک کلاس"
date: 2022-01-22T11:07:00-00:00
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

نمونه کاملتر کد بالا با پشتیبانی از List و همچنین قرار ندادن آیتم ها با مقدار null :    

```c#
private const string Separator = "&";
private const string Equality = "=";

public override string ToString()
{
    var data = GetType().GetProperties()
        .Select(info => (info.Name, Value: info.GetValue(this, null)))
        .Where(a => a.Value != null)
        .Aggregate(
            new StringBuilder(),
            (sb, pair) =>
                sb.Append(
                    GetData(pair.Name, pair.Value)),
            sb => sb.ToString());

    if (data.EndsWith('&'))
    {
        data = data.Remove(data.Length - 1);
    }

    return data;
}

private string GetData(string name, object value)
{
    if (value is IList temp)
    {
        var resultValue = "";

        foreach (var item in temp)
        {
            if (item is Enum temp2)
            {
                resultValue += name + Equality + temp2.GetHashCode() + Separator;
            }
            else
            {
                resultValue = name + Equality + item + Separator;
            }
        }

        return resultValue;
    }

    return value switch
    {
        Enum _ => name + Equality + value.GetHashCode() + Separator,
        _ => name + Equality + value + Separator
    };
}
```

اگر از کتابخانه `FlUrl` برای فراخوانی API استفاده می‌کنید نیز متود `SetQueryParams` در آن وجود دارد که کار بالا را انجام می‌دهد.  

مثال استفاده از کتابخانه:  

```c#
protected Task<IActionResult> Get(string url, object? body = null)
{
    var request = ConfigureHeader(url).SetQueryParams(body).GetAsync();
    return Execute(request);
}

private IFlurlRequest ConfigureHeader(string url)
{
    return url.WithHeader("UserId", GetUserId().ToString());
}

private async Task<IActionResult> Execute(Task<IFlurlResponse> task)
{
    try
    {
        var result = await task;
        var response = await result.ResponseMessage.Content.ReadAsStringAsync();
        return StatusCode(result.StatusCode, response);
    }
    catch (FlurlHttpException e)
    {
        var response = await e.ReadAsStringAsync();

        return StatusCode(e.Call.Response.StatusCode, response);
    }
}
```

اطلاعات بیشتر:  

[aggregate](https://docs.microsoft.com/en-us/dotnet/api/system.linq.enumerable.aggregate?view=net-6.0)  

[gettype](https://docs.microsoft.com/en-us/dotnet/api/system.object.gettype?view=net-6.0)  

[getproperties](https://docs.microsoft.com/en-us/dotnet/api/system.type.getproperties?view=net-6.0)  

[flurl](https://flurl.dev/docs/fluent-url/)  


[flurl code](https://github.com/tmenier/Flurl/blob/a67bdffcd7cdebe4631b486e1abc2e741fadaa50/src/Flurl/Url.cs#L368)  
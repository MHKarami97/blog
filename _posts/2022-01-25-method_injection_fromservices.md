---
title: "تزریق وابستگی با FromServices"
date: 2022-01-25T09:00:00-00:00
categories:
  - Net
tags:
  - net
  - fromServices
  - inject
  - constructors
---

روش‌های مختلفی برای تزریق وابستگی‌ها وجود دارد که در لینک زیر می‌توانید درباره آنها مطالعه کنید:  

[dependency-injection](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)  

قسمتی که ما در این مطلب به آن می‌پردازیم، تفاوت تزریق constructor و method در controller ها است.  
روش کلی تزریق وابستگی بصورت زیر است:  

```c#
public class HomeController : Controller
{
    private readonly IProductService _productService;

    public HomeController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public ProductModel Get()
    {
        return _productService.GetItems();
    }

}
```

بجای کد بالا می‌توانید بصورت زیر نیز عمل کنید و وابستگی را مستقیما در متود خود بیاورید.  

کد زیر در مواقعی کاربرد دارد که شما از یک سرویس فقط در یک متود کنتلر استفاده می‌کنید، در این مواقع می‌توانید از روش زیر استفاده کنید.  

و یا در زمان‌هایی که سرویس استفاده شده سربار زیادی برای ساخت دارد، پس شما آن را فقط در سرویسی که به آن نیاز است تزریق می‌کنید و به عبارتی از `` استفاده می‌کنید تا سربار ساخت کلاس مورد نظر را نداشته باشید.  

```c#
[HttpGet]
public ProductModel Get([FromServices] IProductService service)
{
    return service.GetItems();
}
```

البته کد بالا مشکلاتی نیز دارد:  

  - ممکن است `[FromServices]` را فراموش کنید و در موقع اجرا کد متوجه آن شوید.
  - اگر از حالت بالا برای کم کردن حجم constructor استفاده کرده‌اید، در واقع مشکل را حل نکرده‌اید و فقط آن را تغییر داده اید. اگر حجم سرویس‌های استفاده شده زیاد است شما اصل `SRP` را نقض کرده‌اید و باید کنتلر خود را بشکنید، نه اینکه از روش بالا استفاده کنید.
  - اگر برای performance از روش بالا استفاده کرده‌اید نیز مشکل جدی‌تر در سرویس تزریق شده است که constructor آن زمان زیادی می‌برد و بهتر است آن را بهبود ببخشید. بصورت کلی ساخت یک کلاس نباید تایم زیادی ببرد.


اطلاعات بیشتر:  

[action-injection-with-fromservices](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/dependency-injection?view=aspnetcore-6.0#action-injection-with-fromservices)  

[fromservicesattribute](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.fromservicesattribute?view=aspnetcore-6.0)  
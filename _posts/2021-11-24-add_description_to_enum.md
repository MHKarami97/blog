---
title: "اضافه کردن Attribute توضیحات به Enum"
date: 2021-11-24T09:44:00-00:00
categories:
  - Net
tags:
  - net
  - enum
  - description
  - attribute
---

گاهی مواقع برای مواردی که بسیار کم تغییر می‌کنند و همچنین تعداد آنها نیز کم است، بجای استفاده از دیتابیس از Enum ها می‌توان استفاده کرد. بطور مثال برای جنسیت افراد می‌توان یک enum درست کرد.  
برای اینکه به آیتم های اضافه شده یک توضیح مانند (مرد/زن) نیز اضافه کنید تا در خروجی ها به کاربر نشان بدهید، می‌توانید از `Custom Attributes` ها استفاده کنید و برای خواندن آن از کد نیز استفاده کنید:  

```c#
public static string GetDescription<T>(this T source)
{
    var fieldInfo = source?.GetType().GetField(source.ToString() ?? string.Empty);

    var attributes = (DescriptionAttribute[])fieldInfo?.GetCustomAttributes(typeof(DescriptionAttribute), false)!;

    return attributes?.Length > 0 ? attributes[0].Description : source?.ToString();
}
```

برای اضافه کردن توضیحات به یک آیتم نیز کافی است بصورت زیر عمل کنیم:  
برای شناسایی قسمت اضافه شده نیز کافی است `System.ComponentModel` را اضافه کنیم.  

```c#
public enum MyEnum
{
    [Description("بیشترین تاثیر در شاخص")]
    MaxIndexAffect = 1,

    [Description("بیشترین حجم معامله")]
    MaxVolumeAmount = 2,

    [Description("بیشترین درصد افزایش")]
    MaxIncreasePercent = 3,

    [Description("بیشترین درصد کاهش")]
    MaxDecreasePercent = 4
}
```

روش فراخوانی نیز بصورت زیر است که ابتدا تمام آیتم های یک Enum را در متغیر data ریخته ایم و سپس آن را در یک Dto دلخواه وارد کرده ایم و توسط متد GetDescription توضیحات آن Enum را دریافت کرده ایم.  

```c#
var data = ((MyEnum[])Enum.GetValues(typeof(MyEnum)))
    .OrderBy(x => x.GetHashCode())
    .ToList();

var result = data.Select(i => new MyDto
{
    Id = i.GetHashCode(),
    Title = i.GetDescription()
}).ToList();
```
---
title: "ویرایش PSD توسط .Net"
date: 2023-01-03T00:00:00-00:00
categories:
  - Net
tags:
  - psd
  - png
  - aspose
---

برای یکی از پروژه‌های شخصی نیاز به ویرایش فایل `PSD` توسط کد بود تا نیاز به ویرایش دستی آن فایل و خروجی گرفتن از آن نباشد.  
بدین منظور با کتابخانه `Aspose.PSD` آشنا شدم که البته نسخه اصلی آن بیشتر موارد را فقط در نسخه پولی پشتیبانی می‌کرد.  
با کمی جستجو نسخه رایگان آن را پیدا کردم و با استفاده از داکیومنت خود کتابخانه کد زیر را پیاده سازی کردم. این کد فایل PSD را می‌خواند و سپس با توجه به کانفیگی که به آن در فایل Data.json داده شده است موارد مورد نظر را ویرایش می‌کند و در نهایت خروجی عکس آن را ذخیره می‌کند.  

[aspose.com](https://docs.aspose.com/psd/net/developer-guide/)  
[example](https://github.com/aspose-psd/Aspose.PSD-for-.NET)  
[nuget](https://www.nuget.org/packages/Aspose.PSD)  

```csharp
using System.Text.Json;
using Aspose.PSD;
using Aspose.PSD.FileFormats.Psd;
using Aspose.PSD.FileFormats.Psd.Layers;
using Aspose.PSD.ImageOptions;
using PsdEditor;

try
{
    Console.WriteLine("Start");

    new License().SetLicense("File/" + "License" + ".txt");

    await using var openStream = File.OpenRead("File/" + "data" + ".json");
    var data = await JsonSerializer.DeserializeAsync<Data>(openStream);

    if (data is null) throw new AggregateException("not valid data");

    var items = data.ToDictionary();

    using var psdImage = (PsdImage)Image.Load("File/" + "File" + ".psd");

    foreach (var layer in psdImage.Layers)
    {
        if (layer is TextLayer textLayer)
        {
            var isExist = items.TryGetValue(layer.DisplayName, out var item);

            if (isExist)
            {
                textLayer.UpdateText(item);
            }
        }
    }

    Directory.CreateDirectory("Result");
    psdImage.Save("Result/" + "FileResult" + ".png", new PngOptions());
    psdImage.Save("Result/" + "FileResult" + ".jpg", new JpegOptions());

    Console.WriteLine("Finish");
}
catch (Exception e)
{
    Console.WriteLine(e);
}
```

[github.com/MHKarami97/PsdEditor](https://github.com/MHKarami97/PsdEditor/blob/main/PsdEditor/Program.cs)
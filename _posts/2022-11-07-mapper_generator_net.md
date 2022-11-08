---
title: "Map کردن راحت Dto ها در زبان‌های برنامه‌نویسی"
date: 2022-11-07T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - map
  - regex
---

گاهی مواقع نیاز دارید تا در زبان‌های برنامه نویسی بطور مثال سی‌شارپ یک مدل را به مدل دیگری تبدیل کنید. یکی از پلاگین‌هایی که این کار را راحت می‌کند `AutoMapper` است. فرض کنید بنا به دلایل مختلف امکان استفاده از این پلاگین را ندارید و می‌خواهید همان بصورت دستی این کار را انجام بدهید.  
روش ساده نوشتن دستی این موارد بصورت زیر است:  

```csharp
public class CustomModel {
    public string AddressLine1 { get; set; }

    public DateTime? Dob { get; set; }

    public string EmailAddress { get; set; }

    public string FirstName { get; set; }

    public string LastName { get; set; }

    public string PostCode { get; set; }

    public string PrimaryPhoneNo { get; set; }

    public string Title { get; set; }
}
```

```csharp
private CustomDto Convert(CustomModel modelInput){
    return new CustomModel{
        AddressLine1 = modelInput.AddressLine1,
        ...
        ...
    };
}
```

اگر تعداد مدل‌ها زیاد شود و یا مدل شما پروپرتی‌های زیادی داشته باشد کار بالا زمان‌بر می‌شود و همچنین امکان اشتباه زیاد می‌شود.  
را‌حل جازگزین استفاده از افزونه `MappingGenerator` در `Visual Studio` است.  

[mappinggenerator](https://mappinggenerator.net/)  
[MappingGenerator code](https://github.com/cezarypiatek/MappingGenerator)  

اگر امکان استفاده از این افزونه را هم نداشتید راه دیگر استفاده از `Regex` برای مپ کردن است.  
برای این کار کافی است فقط پروپرتی‌های مدل خود را در سایت زیر یا نرم‌افزارهایی که قابلیت Replace با Regex دارد قرار دهید.  
سپس این خط را در قسمت Search وارد کنید:  

[regex101](https://regex101.com)  

```r
public [A-Za-z\?]* ([A-Za-z0-9]*) .*
```

سپس در قسمت Replace خط زیر را وارد کنید:  

```r
$1 = modelInput.$1,
```

با این کار کدی شبیه به زیر ساخته می‌شود:  

```csharp       
    AddressLine1 = modelInput.AddressLine6,

    Dob = modelInput.Dob,

    EmailAddress = modelInput.EmailAddress,

    FirstName = modelInput.FirstName,

    LastName = modelInput.LastName,

    PostCode = modelInput.PostCode,

    PrimaryPhoneNo = modelInput.PrimaryPhoneNo,

    Title = modelInput.Title,
```

[regex101 replace](https://regex101.com/r/dZ1vT6/73)  
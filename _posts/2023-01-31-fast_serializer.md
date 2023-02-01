---
title: "سریع‌ترین Json Serializer در برنامه‌نویسی"
date: 2023-01-31T00:00:00-00:00
categories:
  - Net
tags:
  - csharp
  - message_pack
  - utf8json
  - serializer
---

یکی از سریعترین کتاب‌خانه ها برای Serializer Json کتابخانه‌ای به اسم `Utf8Json` است که البته ورژن جدیدتر آن `MessagePack` است که جایگزین قبلی شده است.  
طبق تست‌های واقعی با ریت پیام بالا در محیط عملیاتی کتابخانه newtonsoft نسبت به کتابخانه معرفی شده بسیار کند است که در پیام با تعداد بالا بسیار تاثیرگذار است.  
بطور مثال ایجنتی که پیام‌ها را از صف برمی‌دارد و آنها را از حالت json به object تبدیل می‌کند توسط کتابخانه newtonsoft تقریبا 2 میلی‌ثانیه طول می‌کشد در حالیکه با کتابخانه MessagePack زمان آن به 1 میکرو‌ثانیه کاهش پیدا می‌کند.  

```csharp
var bytes = MessagePackSerializer.Serialize(mc);
var mc2 = MessagePackSerializer.Deserialize<MyClass>(bytes);

var json = MessagePackSerializer.ConvertToJson(bytes);
Console.WriteLine(json);
```

[Utf8Json](https://github.com/neuecc/Utf8Json)  

[MessagePack](https://github.com/neuecc/MessagePack-CSharp)  
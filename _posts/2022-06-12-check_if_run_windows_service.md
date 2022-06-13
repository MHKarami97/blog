---
title: "بررسی اجرا شدن برنامه در حالت سرویس در .net core"
date: 2022-06-12T00:00:00-00:00
categories:
  - Net
tags:
  - net
  - isWindowsService
  - userInteractive
  - net_core
---

فرض کنید برای دیباگ یا در دسترس بودن امکانات بیشتر یک Agent که با .Net Core نوشته شده است در حالت پابلیش شده می‌خواهید قسمتی به کد اضافه کنید. بطور مثال اگر برنامه بصورت دستی و فایل exe اجرا شد از کاربر Connection String دیتابیس را بگیرد و اگر بصورت Service ویندوز اجرا شد خودکار از فایل کانفیگ آن را بخواند.  
در این موارد می‌توانید از کد زیر استفاده کنید:  

```c#
Environment.UserInteractive
```

البته کد بالا در .Net Core 2.1 , 3.1 همیشه مقدار True را بازگشت می‌دهد و در ورژن 5.0 درست شده است.  

[WindowsServiceHelpers 5.0](https://github.com/dotnet/extensions/blob/release/5.0/src/Hosting/WindowsServices/src/WindowsServiceHelpers.cs)  

[WindowsServiceHelpers 3.1](https://github.com/dotnet/extensions/blob/release/3.1/src/Hosting/WindowsServices/src/WindowsServiceHelpers.cs)  

در این مواقع می‌توانید از متود زیر استفاده کنید که در ورژن‌های گفته شده خروجی درست می‌دهد.  

```c#
WindowsServiceHelpers.IsWindowsService()
```

[Environment 3.1](https://github.com/dotnet/corefx/blob/release/3.1/src/Common/src/CoreLib/System/Environment.cs#L129)  

[Environment 5.0](https://github.com/dotnet/runtime/blob/release/5.0/src/libraries/System.Private.CoreLib/src/System/Environment.Windows.cs#L126)  

البته این مورد فقط در ویندوز کار می‌کنند و در بقیه موارد مقدار False می‌دهد.  
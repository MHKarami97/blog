---
title: "دریافت لیست نرم افزارهای نصب شده بر روی ویندوز"
categories:
  - Windows
tags:
  - windows
  - trick
  - app_list
---

در صورتی که نیاز داشتید تا لیست تمام نرم افزارهای نصب شده بر روی سیستم عامل ویندوز خود را بدست بیاورید، می‌توانید از تکه کد زیر استفاده کنید.  

کافی است `PowerShell` را بصورت Admin اجرا کنید و دستور زیر تا خروجی مورد نظر نشان داده شود.  
برای نمایش تمام ستون‌ها بهتر است PowerShell را بصورت FullScreen کنید تا تمام اطلاعات نمایش داده شود.  
برای کپی هم کافی است تمام موارد را Drag کنید و سپس با Ctrl+C آن را کپی کنید.  

```s
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, Publisher, InstallDate | Format-Table –AutoSize
```

در صورتی که در زمان اجرا کد بالا با خطا مواجه شدید کافی است دستور زیر را هم اجرا کنید.  

```s
Set-ExecutionPolicy Unrestricted
```

با `CMD` هم می‌توانید عملیات بالا را انجام دهید البته با توجه به تست‌های انجام شده خروجی مورد نظر تمام نرم افزارها را نشان نمی‌دهد.  
ابتدا CMD را بصورت Admin اجرا کنید و سپس دستور زیر را وارد کنید:  

```s
WMIC
```

سپس دستور زیر را وارد کنید تا پس از مدتی لیست نرم افزارها در فایل گفته شده در دستور ذخیره شود.  

```s
/output:C:\InstallList.txt product get name,version
```
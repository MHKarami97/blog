---
title: "توقف یا Kill کردن یک سرویس در ویندوز"
date: 2022-03-12T12:00:00-00:00
categories:
  - Windows
tags:
  - windows
  - kill
  - service
---

یکی از مشکلاتی که در زمان کار با Service ها در ویندوز ممکن است با آن روبرو شوید، Stop نشدن سرویس مورد نظر است.  
یکی از راه‌های سریع برای توقف سرویس مورد نظر استفاده از دستور زیر است که سرویس مورد نظر را kill می‌کند.  
البته از دستور زیر نباید در تمام مواقع استفاده کنید و باید مشکل اصلی Stop نشدن سرویس را پیدا کنید.  
برای پیدا کردن PID سرویس مورد نظر هم کافی است از سربرگ Services در Task Manager استفاده کنید.  

```powershell
taskkill /PID 17812 /F
```

لینک‌های مفید:  

[taskkill](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/taskkill)  